# 2023-05-03 (EL2 Exception)

EL1 에서 PSTATE 의 I bit 를 disable 해도 EL2 의 interrupt 를 받는 것이 가능한지?

ARMv8 reference manual 에 따르면, target exception level 이 current exception level 이면 PSTATE.I 로 interrupt 를 masking 할 수 있다.

그러나, HCR_EL2.E2H 나 HCR_EL2.TGE 가 0 일 때, 인터럽트의 target exception level 이 EL2 라면 EL1 상태에서 PSTATE.I bit 를 설정해도 interrupt 를 마스킹할 수 없다.

https://stackoverflow.com/questions/69249099/pi4-hypervisor-handle-irq

https://developer.arm.com/documentation/ddi0595/2021-06/AArch64-Registers/HCR-EL2--Hypervisor-Configuration-Register



EL2 로 exception 이 발생하면 (인터럽트 포함) 무조건 EL1 의 stack 을 저장한다.

xen/arch/arm/arm64/entry.S

// 진입할 때

```c
        .macro  entry, hyp, compat, save_x0_x1=1
        sub     sp, sp, #(UREGS_SPSR_el1 - UREGS_LR) /* CPSR, PC, SP, LR */

        .if \hyp == 0         /* Guest mode */
        clobber_gp_top_halves compat=\compat, save_x0_x1=\save_x0_x1
        .endif

        push    x28, x29
        push    x26, x27
        push    x24, x25
        push    x22, x23
        push    x20, x21
        push    x18, x19
        push    x16, x17
        push    x14, x15
        push    x12, x13
        push    x10, x11
        push    x8, x9
        push    x6, x7
        push    x4, x5
        push    x2, x3

```

// 되돌아갈 때

```c
return_from_trap:
        msr     daifset, #IFLAGS___I_ /* Mask interrupts */

        ldr     x21, [sp, #UREGS_PC]            /* load ELR */
        ldr     x22, [sp, #UREGS_CPSR]          /* load SPSR */

        pop     x0, x1
        pop     x2, x3
        pop     x4, x5
        pop     x6, x7
        pop     x8, x9

        msr     elr_el2, x21                    /* set up the return data */
        msr     spsr_el2, x22

        pop     x10, x11
        pop     x12, x13
        pop     x14, x15
        pop     x16, x17
        pop     x18, x19
        pop     x20, x21
        pop     x22, x23
        pop     x24, x25
        pop     x26, x27
        pop     x28, x29

        ldr     lr, [sp], #(UREGS_SPSR_el1 - UREGS_LR) /* CPSR, PC, SP, LR */

        eret
        sb
```

EL2 의 exception vector 위치는 이렇게 설정된다.

xen/arch/arm/traps.c

```c
void init_traps(void)
{
    /*
     * Setup Hyp vector base. Note they might get updated with the
     * branch predictor hardening.
     */
    WRITE_SYSREG((vaddr_t)hyp_traps_vector, VBAR_EL2);
```

그러면 exception 발생시 hyp_traps_vector 위치로 뛰게 된다.

xen/arch/arm/arm64/entry.S

```c
ENTRY(hyp_traps_vector)
        ventry  hyp_sync_invalid            /* Synchronous EL2t */
        ventry  hyp_irq_invalid             /* IRQ EL2t */
        ventry  hyp_fiq_invalid             /* FIQ EL2t */
        ventry  hyp_error_invalid           /* Error EL2t */

        ventry  hyp_sync                    /* Synchronous EL2h */
        ventry  hyp_irq                     /* IRQ EL2h */
        ventry  hyp_fiq_invalid             /* FIQ EL2h */
        ventry  hyp_error                   /* Error EL2h */

        ventry  guest_sync                  /* Synchronous 64-bit EL0/EL1 */
        ventry  guest_irq                   /* IRQ 64-bit EL0/EL1 */
        ventry  guest_fiq_invalid           /* FIQ 64-bit EL0/EL1 */
        ventry  guest_error                 /* Error 64-bit EL0/EL1 */

        ventry  guest_sync_compat           /* Synchronous 32-bit EL0/EL1 */
        ventry  guest_irq_compat            /* IRQ 32-bit EL0/EL1 */
        ventry  guest_fiq_invalid_compat    /* FIQ 32-bit EL0/EL1 */
        ventry  guest_error_compat          /* Error 32-bit EL0/EL1 */
```

인터럽트가 발생했다면 hyp_irq 로 이동한다.

```c
hyp_irq:
        entry   hyp=1

        /* Inherit D, A, F interrupts and keep I masked */
        mrs     x0, SPSR_el2
        mov     x1, #(PSR_DBG_MASK | PSR_ABT_MASK | PSR_FIQ_MASK)
        and     x0, x0, x1
        orr     x0, x0, #PSR_IRQ_MASK
        msr     daif, x0

        mov     x0, sp
        bl      do_trap_irq
        exit    hyp=1
```

여기서 entry 와 exit 은 매크로로 동작한다. (위에 스택에 EL1 의 레지스터 정보를 저장하고 꺼내는 매크로 코드가 있다.)

do_trap_irq 로 이동하면,

xen/arch/arm/traps.c

```c
void do_trap_irq(struct cpu_user_regs *regs)
{
    gic_interrupt(regs, 0);
}
```

gic_interrupt 를 호출한다.

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

do_IRQ 에서는 등록된 handler 를 호출한다.

xen/arch/arm/irq.c

```c
void do_IRQ(struct cpu_user_regs *regs, unsigned int irq, int is_fiq)
{
    struct irq_desc *desc = irq_to_desc(irq);
    struct irqaction *action;

    perfc_incr(irqs);

    ASSERT(irq >= 16); /* SGIs do not come down this path */

    if ( irq < 32 )
        perfc_incr(ppis);
    else
        perfc_incr(spis);

    /* TODO: this_cpu(irq_count)++; */

    irq_enter();

    spin_lock(&desc->lock);
    desc->handler->ack(desc);

#ifndef NDEBUG
    if ( !desc->action )
    {
        printk("Unknown %s %#3.3x\n",
               is_fiq ? "FIQ" : "IRQ", irq);
        goto out;
    }
#endif

    if ( test_bit(_IRQ_GUEST, &desc->status) )
    {
        struct irq_guest *info = irq_get_guest_info(desc);

        perfc_incr(guest_irqs);
        desc->handler->end(desc);

        set_bit(_IRQ_INPROGRESS, &desc->status);

        /*
         * The irq cannot be a PPI, we only support delivery of SPIs to
         * guests.
         */
        vgic_inject_irq(info->d, NULL, info->virq, true);
        goto out_no_end;
    }

    if ( test_bit(_IRQ_DISABLED, &desc->status) )
        goto out;

    set_bit(_IRQ_INPROGRESS, &desc->status);

    action = desc->action;

    spin_unlock_irq(&desc->lock);
    do
    {
        action->handler(irq, action->dev_id, regs);
        action = action->next;
    } while ( action );

    spin_lock_irq(&desc->lock);

    clear_bit(_IRQ_INPROGRESS, &desc->status);

out:
    desc->handler->end(desc);
out_no_end:
    spin_unlock(&desc->lock);
    irq_exit();
}
```

