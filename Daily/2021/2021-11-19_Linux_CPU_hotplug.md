# 2021-11-19 (Linux CPU hotplug)

### Linux CPU Hotplug out 방법

`echo 0 > /sys/devices/system/cpu/cpu1/online` 같은 명령어로 cpu 를 hotplug out 하게 되면, cpu_is_offline을 세팅해준다. 그러면 나중에 그 cpu가 할 일을 끝내고 idle (wfi) 상태로 가게 되면, `arch_cpu_idle_dead();` 함수 통해서 CPU를 꺼버리는 psci 를 요청하게 된다.



kernel/sched/idle.c

```c
/*
 * Generic idle loop implementation
 *
 * Called with polling cleared.
 */
static void do_idle(void)
{
        int cpu = smp_processor_id();
        /*
         * If the arch has a polling bit, we maintain an invariant:
         *
         * Our polling bit is clear if we're not scheduled (i.e. if rq->curr !=
         * rq->idle). This means that, if rq->idle has the polling bit set,
         * then setting need_resched is guaranteed to cause the CPU to
         * reschedule.
         */

        __current_set_polling();
        tick_nohz_idle_enter();

        while (!need_resched()) {
                rmb();

                local_irq_disable();

                if (cpu_is_offline(cpu)) {
                        tick_nohz_idle_stop_tick();
                        cpuhp_report_idle_dead();
                        arch_cpu_idle_dead();
                }

                arch_cpu_idle_enter();

                /*
                 * In poll mode we reenable interrupts and spin. Also if we
                 * detected in the wakeup from idle path that the tick
                 * broadcast device expired for us, we don't want to go deep
                 * idle as we know that the IPI is going to arrive right away.
                 */
                if (cpu_idle_force_poll || tick_check_broadcast_expired()) {
                        tick_nohz_idle_restart_tick();
                        cpu_idle_poll();
                } else {
                        cpuidle_idle_call();
                }
                arch_cpu_idle_exit();
        }

```



arch/arm64/kernel/process.c

```c
#ifdef CONFIG_HOTPLUG_CPU
void arch_cpu_idle_dead(void)
{
       cpu_die();
}
#endif
```

arch/arm64/kernel/psci.c

```c
static void cpu_psci_cpu_die(unsigned int cpu)
{
        /*
         * There are no known implementations of PSCI actually using the
         * power state field, pass a sensible default for now.
         */
        u32 state = PSCI_POWER_STATE_TYPE_POWER_DOWN <<
                    PSCI_0_2_POWER_STATE_TYPE_SHIFT;

        psci_ops.cpu_off(state);
}
```



### Xen CPU Hotplug out 방법

(Xen 도 Linux 기반)

XEN 도 Linux 와 같은 방식으로 코드가 구성되어있다. Linux 에서 한 요청을 Xen 에서 어떻게 처리하도록 연동하였는지는 확인이 필요하다.

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

xen/arch/arm/smpboot.c

```c
void stop_cpu(void)
{
    local_irq_disable();
    cpu_is_dead = true;
    /* Make sure the write happens before we sleep forever */
    dsb(sy);
    isb();
    call_psci_cpu_off();

    while ( 1 )
        wfi();
}
```

xen/arch/arm/vpsci.c

```c
static int32_t do_psci_cpu_off(uint32_t power_state)
{
    struct vcpu *v = current;
    if ( !test_and_set_bit(_VPF_down, &v->pause_flags) )
        vcpu_sleep_nosync(v);
    return PSCI_SUCCESS;
}

bool do_vpsci_0_2_call(struct cpu_user_regs *regs, uint32_t fid)
{
    /*
     * /!\ VPSCI_NR_FUNCS (in asm-arm/vpsci.h) should be updated when
     * adding/removing a function. SCCC_SMCCC_*_REVISION should be
     * updated once per release.
     */
    switch ( fid )
    {
    case PSCI_0_2_FN32_PSCI_VERSION:
        perfc_incr(vpsci_version);
        PSCI_SET_RESULT(regs, do_psci_0_2_version());
        return true;

    case PSCI_0_2_FN32_CPU_OFF:
        perfc_incr(vpsci_cpu_off);
        PSCI_SET_RESULT(regs, do_psci_0_2_cpu_off());
        return true;
```



### 참고

elf 파일 안에 코드별 size 확인하는 방법

```shell
$ readelf -S
$ fromelf -cdegrstyz *.axf --output=disassembled.lst
```

