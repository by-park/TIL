# 2022-11-14 (spin lock irqsave)

### spin_lock, spin_lock_irq, spin_lock_irqsave란 무엇인가

http://egloos.zum.com/nimhaplz/v/5301468

- mutex lock 은 퍼포먼스 낭비가 있기 때문에 spin lock 이 등장하였다.
- spin lock 은 dead lock 이 발생할 수 있다 (오늘 경험하였다!! 동시에 2개 인터럽트가 발생했는데, 인터럽트 핸들러 안에 spin_lock, spin_unlock 을 잡았더니 첫번째 인터럽트가 다 처리되지 않았을 때 같은 코어에 두 번째 인터럽트가 발생하면 spin_lock 에 걸려버렸다. softirq 로 같은 코어에 그 인터럽트가 발생하도록 한 것이기 때문에 무조건 dead lock 이 생길 수 밖에 없었다.)
- spin_lock_irq 함수를 사용하면 인터럽트를 disable 시키기 때문에 dead lock 문제가 해결된다. 그러나 unlock 할 때 irq 를 enable 시켜야하는게 맞는지 알 수 없다. (disable 이 2번 들어왔을 수도 있다. 그러면 enable 을 2번 해줘야 진짜 enable 될 것)
- spin_lock_irqsave 함수를 사용하면 irq 가 현재 enable 상태인지 disable 상태인지를 저장한다. 따라서 위의 문제가 해결된다. 그러나 메모리에 저장하는 동작이 추가되어있으므로 퍼포먼스 오버헤드가 있다.



### irq 관련 이슈 디버깅

1. softirq 가 2번 발생

gic_interrupt 안에 do_IRQ() 전에 내가 원하는 인터럽트가 발생하면 함수를 실행하도록 softirq 를 짰다.

`if ( irq == 특정 번호) cpu_raise_softirq(0, SOFTIRQ 번호 매크로);`

그러나 해당 softirq 에 등록해둔 함수가 2번 연속 호출되면서 문제가 발생하였다. softirq 윗부분을 실행하는 동안, 인터럽트가 들어와서 softirq 에 또 등록시킨 것. 그래서 동기화할 수 있도록 `cpu_raise_softirq` 앞 뒤에 보호 코드를 넣었다. 그래야 softirq 자체가 2번 발생하지 않기 때문이었다. (softirq 내부에 보호 코드를 넣으면 소용이 없었다.) 동기화 용도로 spin lock 은 같은 core 에서 softirq 를 처리하느라 dead lock 이 발생하였고, spin lock irqsave 는 내부에서 interrupt 가 disable 되면 안 되는 동작이 있어서 에러가 발생하였다. 그래서 변수로 보호하였다. 테스트는 실제 문제 현상을 만들 방법이 없어서 softirq 에 등록됨 함수가 실행 되는 중간에 mailbox register를 이용해서 인터럽트를 강제로 발생시켜보았다.

xen/arch/arm/gic.c

```c
void gic_interrupt(struct cpu_user_regs *regs, int is_fiq)
{
    unsigned int irq;

    do  {
        /* Reading IRQ will ACK it */
        irq = gic_hw_ops->read_irq();

        if ( likely(irq >= 16 && irq < 1020) )
        {
            isb();
            do_IRQ(regs, irq, is_fiq);
        }
        else if ( is_lpi(irq) )
        {
            isb();
            gic_hw_ops->do_LPI(irq);
        }
        else if ( unlikely(irq < 16) )
        {
            do_sgi(regs, irq);
        }
        else
        {
            local_irq_disable();
            break;
        }
    } while (1);
}
```

softirq 는 idle_loop 에 들어가면 처리된다.

xen/arch/arm/domain.c

```c
void idle_loop(void)
{
    unsigned int cpu = smp_processor_id();

    for ( ; ; )
    {
        if ( cpu_is_offline(cpu) )
            stop_cpu();

        /* Are we here for running vcpu context tasklets, or for idling? */
        if ( unlikely(tasklet_work_to_do(cpu)) )
        {
            do_tasklet();
            /* Livepatch work is always kicked off via a tasklet. */
            check_for_livepatch_work();
        }
        /*
         * Test softirqs twice --- first to see if should even try scrubbing
         * and then, after it is done, whether softirqs became pending
         * while we were scrubbing.
         */
        else if ( !softirq_pending(cpu) && !scrub_free_pages() &&
                  !softirq_pending(cpu) )
            do_idle();

        do_softirq();
    }
}
```



2. 처리를 원하지 않는 irq 막기

매우 W/A 용이긴 하지만 irq 가 등록된 domain 에서 해당 irq 를 처리하지 못하도록 막고 싶으면 이렇게 할 수 있다.

xen/arch/arm/irq.c

```c
    if ( test_bit(_IRQ_GUEST, &desc->status) )
    {
        struct irq_guest *info = irq_get_guest_info(desc);

        perfc_incr(guest_irqs);
        desc->handler->end(desc);

        // 여기에 특정 irq 만 막을 수 있다.
        if (IRQ == 특정 번호) {
            gic_set_active_state(desc, false);
        } else { // 여기가 원래 mainline 실행 내용
            set_bit(_IRQ_INPROGRESS, &desc->status);

            /*
             * The irq cannot be a PPI, we only support delivery of SPIs to
             * guests.
             */
            vgic_inject_irq(info->d, NULL, info->virq, true);           
        }
        goto out_no_end;
    }

    if ( test_bit(_IRQ_DISABLED, &desc->status) )
        goto out;

    set_bit(_IRQ_INPROGRESS, &desc->status);

    action = desc->action;

    spin_unlock_irq(&desc->lock);

```

