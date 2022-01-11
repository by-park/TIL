# 2022-01-11 (Xen scheduler)

- xen 의 tasklet: open_softirq(), cpu_raise_softirq()

```c
// xen/include/xen/softirq.h
enum {
    TIMER_SOFTIRQ = 0,
    RCU_SOFTIRQ,
    SCHED_SLAVE_SOFTIRQ,
    SCHEDULE_SOFTIRQ,
    NEW_TLBFLUSH_CLOCK_PERIOD_SOFTIRQ,
    TASKLET_SOFTIRQ,
    NR_COMMON_SOFTIRQS
};

// xen/common/sched/core.c
void scheduler_enable(void)
{
    open_softirq(SCHEDULE_SOFTIRQ, schedule);
    open_softirq(SCHED_SLAVE_SOFTIRQ, sched_slave);
    scheduler_active = true;
}

// xen/common/sched/credit.c
cpu_raise_softirq(cpu, SCHEDULE_SOFTIRQ);
```

softirq 가 동작하는 내부를 보면, SGI 를 보내도록 되어있다.

```c
// xen/common/softirq.c
void cpu_raise_softirq(unsigned int cpu, unsigned int nr)
{
    unsigned int this_cpu = smp_processor_id();

    if ( test_and_set_bit(nr, &softirq_pending(cpu))
         || (cpu == this_cpu)
         || arch_skip_send_event_check(cpu) )
        return;

    if ( !per_cpu(batching, this_cpu) || in_irq() )
        smp_send_event_check_cpu(cpu);
    else
        __cpumask_set_cpu(cpu, &per_cpu(batch_mask, this_cpu));
}

// xen/include/xen/smp.h
#define smp_send_event_check_cpu(cpu) \
    smp_send_event_check_mask(cpumask_of(cpu))

// xen/arch/arm/smp.c
void smp_send_event_check_mask(const cpumask_t *mask)
{
    send_SGI_mask(mask, GIC_SGI_EVENT_CHECK);
}

// xen/arch/arm/gic.c
void send_SGI_mask(const cpumask_t *cpumask, enum gic_sgi sgi)
{
    gic_hw_ops->send_SGI(sgi, SGI_TARGET_LIST, cpumask);
}
```



- schedule 함수의 동작

> 이전 스케줄링했던 vcpu와 다음 스케줄링해야할 vcpu가 같다면 continue_running() 함수를 호출하고 같지 않으면 이전 vcpu 와 다음 vcpu 의 상태를 변경한다. vcpu 의 상태는 다음과 같다.
>
> (1) RUNSTATE_running
>
> (2) RUNSTATE_runnable
>
> (3) RUNSTATE_blocked
>
> (4) RUNSTATE_offline

https://jhk-research.tistory.com/19

ppt에 vcpu 별 Start time, End time 도식화 되어 있음

https://slideplayer.com/slide/5265439



- xen scheduler의 종류

`sched=` 으로 boot parameter 값으로 스케줄러를 결정한다.

(1) Credit: General Purpose

(2) Credit2: General Purpose. Optimized for low latency, scalability, high VM density

(3) RTDS: Soft & Firm Real-time Embedded, mobile & automotive Graphics & Gaming in the Cloud

(4) ARINC 653: Hard Real-time Avionics, Drones, Medical

https://wiki.xenproject.org/wiki/Xen_Project_Schedulers



Q. xen/common/sched/null.c 에 있는 스케줄러는 무엇인지?

```c
// xen/common/sched/null.c
static const struct scheduler sched_null_def = {
    .name           = "null Scheduler",
    .opt_name       = "null",
    .sched_id       = XEN_SCHEDULER_NULL,
    .sched_data     = NULL,

```

