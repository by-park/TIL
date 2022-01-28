# 2022-01-25 (LK Thread scheduling)

### LK 코드

https://github.com/littlekernel/lk



## Overview

### Multi Level Queue & Round Robin
- 우선 순위 별로 queue를 관리하는 선점형 방식인 MLQ 기법을 사용

- 높은 우선 순위 queue의 thread 들이 모두 작업이 끝나야 다음 우선 순위 queue의 thread 작업이 실행됨

- 같은 우선 순위 queue 안의 thread 들은 RR 방식으로 관리 됨

![img](https://media.geeksforgeeks.org/wp-content/uploads/Multilevel-Feedback-Queue-Scheduling-300x269.png)

![Round Robin scheduling](https://www.myassignmenthelp.net/images/round-robin-scheduling.gif)



## Thread initialization

### Thread struct

// kernel/include/kernel/thread.h

```c
typedef struct thread {
    int magic;
    struct list_node thread_list_node;

    /* active bits */
    struct list_node queue_node;
    int priority;
    enum thread_state state;
    int remaining_quantum;
    unsigned int flags;
#if WITH_SMP
    int curr_cpu;
    int pinned_cpu; /* only run on pinned_cpu if >= 0 */
#endif
#if WITH_KERNEL_VM
    vmm_aspace_t *aspace;
#endif
// ...
} thread_t;
```

// kernel/include/kernel/thread.h

```c
/* thread priority */
#define NUM_PRIORITIES 32
#define LOWEST_PRIORITY 0
#define HIGHEST_PRIORITY (NUM_PRIORITIES - 1)
#define DPC_PRIORITY (NUM_PRIORITIES - 2)
#define IDLE_PRIORITY LOWEST_PRIORITY
#define LOW_PRIORITY (NUM_PRIORITIES / 4)
#define DEFAULT_PRIORITY (NUM_PRIORITIES / 2)
#define HIGH_PRIORITY ((NUM_PRIORITIES / 4) * 3)
```

// kernel/include/kernel/thread.h

```c
enum thread_state {
    THREAD_SUSPENDED = 0,
    THREAD_READY,
    THREAD_RUNNING,
    THREAD_BLOCKED,
    THREAD_SLEEPING,
    THREAD_DEATH,
};
```

// kernel/thread.c

```c
/* the run queue */
static struct list_node run_queue[NUM_PRIORITIES];
static uint32_t run_queue_bitmap;
```

// kernel/thread.c

```c
/* global thread list */
static struct list_node thread_list;
```



- 우선 순위는 0~31

- Thread를 관리하는 list는 run_queue와 thread_list를 가짐

- 현재 실행되지 않는 thread를 run_queue에 넣어 뒀다가 이후 꺼내서 실행시키는 구조



### Timer

// kernel/timer.c

```c
struct timer_state {
    struct list_node timer_queue;
} __CPU_ALIGN;

static struct timer_state timers[SMP_MAX_CPUS];
```

// kernel/thread.c

```c
#if PLATFORM_HAS_DYNAMIC_TIMER
/* preemption timer */
static timer_t preempt_timer[SMP_MAX_CPUS];
#endif
```

// kernel/include/kernel/timer.h

```c
typedef struct timer {
    int magic;
    struct list_node node;

    lk_time_t scheduled_time;
    lk_time_t periodic_time;

    timer_callback callback;
    void *arg;
} timer_t;
```



- CPU 마다 하나의 HW timer와 timer 구조체를 담는 Double Linked List를 가짐

- 일정 주기 마다 HW timer의 handle가 실행되고, 현재 시간과 비교하여 처리 되어야할 timer 구조체를 처리



### idle thread 생성

// kernel/thread.c => idle_thread(0)

```c
void thread_init_early(void) {
    int i;

    DEBUG_ASSERT(arch_curr_cpu_num() == 0);

    /* initialize the run queues */
    for (i=0; i < NUM_PRIORITIES; i++)
        list_initialize(&run_queue[i]);

    /* initialize the thread list */
    list_initialize(&thread_list);

    /* create a thread to cover the current running state */
    thread_t *t = idle_thread(0);
    init_thread_struct(t, "bootstrap");
```

// kernel/thread.c

```c
/* the idle thread(s) (statically allocated) */
#if WITH_SMP
static thread_t _idle_threads[SMP_MAX_CPUS];
#define idle_thread(cpu) (&_idle_threads[cpu])
#else
static thread_t _idle_thread;
#define idle_thread(cpu) (&_idle_thread)
#endif
```

// kernel/thread.c

```c
void init_thread_struct(thread_t *t, const char *name) {
    memset(t, 0, sizeof(thread_t));
    t->magic = THREAD_MAGIC;
    thread_set_pinned_cpu(t, -1);
    strlcpy(t->name, name, sizeof(t->name));
}
```

// kernel/thread.c => t->priority = HIGHEST_PRIORITY

```c
void thread_init_early(void) {
//...
    /* create a thread to cover the current running state */
    thread_t *t = idle_thread(0);
    init_thread_struct(t, "bootstrap");

    /* half construct this thread, since we're already running */
    t->priority = HIGHEST_PRIORITY;
    t->state = THREAD_RUNNING;
    t->flags = THREAD_FLAG_DETACHED;
    thread_set_curr_cpu(t, 0);
    thread_set_pinned_cpu(t, 0);
    wait_queue_init(&t->retcode_wait_queue);
    list_add_head(&thread_list, &t->thread_list_node);
    set_current_thread(t);
}
```

// kernel/include/kernel/thread.h

```c
static inline void set_current_thread(thread_t *t) {
    arch_set_current_thread(t);
}
```

// arch/arm64/include/arch/arch_ops.h

```c
static inline void arch_set_current_thread(struct thread *t) {
    ARM64_WRITE_SYSREG(tpidr_el1, (uint64_t)t);
}
```



- thread_init_early() 함수에서 idle thread 구조체 생성

- 이후 main thread는 실제 부팅을 담당하는 bootstrap2 thread를 생성하고 idle thread로 switching 됨 



### bootstrap2 thread 생성

// top/main.c

```c
static int bootstrap2(void *arg) {
    dprintf(SPEW, "top of bootstrap2()\n");

    lk_primary_cpu_init_level(LK_INIT_LEVEL_THREADING, LK_INIT_LEVEL_ARCH - 1);
    arch_init();

    // initialize the rest of the platform
    dprintf(SPEW, "initializing platform\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_ARCH, LK_INIT_LEVEL_PLATFORM - 1);
    platform_init();

    // initialize the target
    dprintf(SPEW, "initializing target\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_PLATFORM, LK_INIT_LEVEL_TARGET - 1);
    target_init();

    dprintf(SPEW, "initializing apps\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_TARGET, LK_INIT_LEVEL_APPS - 1);
    apps_init();

    lk_primary_cpu_init_level(LK_INIT_LEVEL_APPS, LK_INIT_LEVEL_LAST);

    return 0;
}
```

// top/main.c => thread_become_idle 호출

```c
/* called from arch code */
void lk_main(ulong arg0, ulong arg1, ulong arg2, ulong arg3) {
    // save the boot args
    lk_boot_args[0] = arg0;
    lk_boot_args[1] = arg1;
    lk_boot_args[2] = arg2;
    lk_boot_args[3] = arg3;

    // get us into some sort of thread context
    thread_init_early();

    // early arch stuff
    lk_primary_cpu_init_level(LK_INIT_LEVEL_EARLIEST, LK_INIT_LEVEL_ARCH_EARLY - 1);
    arch_early_init();

    // do any super early platform initialization
    lk_primary_cpu_init_level(LK_INIT_LEVEL_ARCH_EARLY, LK_INIT_LEVEL_PLATFORM_EARLY - 1);
    platform_early_init();

    // do any super early target initialization
    lk_primary_cpu_init_level(LK_INIT_LEVEL_PLATFORM_EARLY, LK_INIT_LEVEL_TARGET_EARLY - 1);
    target_early_init();

#if WITH_SMP
    dprintf(INFO, "\nwelcome to lk/MP\n\n");
#else
    dprintf(INFO, "\nwelcome to lk\n\n");
#endif
    dprintf(INFO, "boot args 0x%lx 0x%lx 0x%lx 0x%lx\n",
            lk_boot_args[0], lk_boot_args[1], lk_boot_args[2], lk_boot_args[3]);
    // bring up the kernel heap
    lk_primary_cpu_init_level(LK_INIT_LEVEL_TARGET_EARLY, LK_INIT_LEVEL_HEAP - 1);
    dprintf(SPEW, "initializing heap\n");
    heap_init();

    // deal with any static constructors
    dprintf(SPEW, "calling constructors\n");
    call_constructors();

    // initialize the kernel
    lk_primary_cpu_init_level(LK_INIT_LEVEL_HEAP, LK_INIT_LEVEL_KERNEL - 1);
    kernel_init();

    lk_primary_cpu_init_level(LK_INIT_LEVEL_KERNEL, LK_INIT_LEVEL_THREADING - 1);

    // create a thread to complete system initialization
    dprintf(SPEW, "creating bootstrap completion thread\n");
    thread_t *t = thread_create("bootstrap2", &bootstrap2, NULL, DEFAULT_PRIORITY, DEFAULT_STACK_SIZE);
    thread_set_pinned_cpu(t, 0);
    thread_detach(t);
    thread_resume(t);

    // become the idle thread and enable interrupts to start the scheduler
    thread_become_idle();
}
```



- 실제 LK 부팅을 담당하는 thread (bootstrap2)

- thread_resume() 이후 scheduling이 되어야하지만, 아직 HW timer가 설정되지 않아 실행 되지 않음

- thread_become_idle() 에서 HW timer와 interrupt 설정 이후 실행됨



### thread_yield() 및 idle thread switching

// kernel/thread.c => thread_yield 와 idle_thread_routine

```c
void thread_become_idle(void) {
    DEBUG_ASSERT(arch_ints_disabled());

    thread_t *t = get_current_thread();

#if WITH_SMP
    char name[16];
    snprintf(name, sizeof(name), "idle %d", arch_curr_cpu_num());
    thread_set_name(name);
#else
    thread_set_name("idle");
#endif

    /* mark ourself as idle */
    t->priority = IDLE_PRIORITY;
    t->flags |= THREAD_FLAG_IDLE;
    thread_set_pinned_cpu(t, arch_curr_cpu_num());

    mp_set_curr_cpu_active(true);
    mp_set_cpu_idle(arch_curr_cpu_num());

    /* enable interrupts and start the scheduler */
    arch_enable_ints();
    thread_yield();

    idle_thread_routine();
}
```

// kernel/thread.c => thread_resched()

```c
void thread_yield(void) {
    thread_t *current_thread = get_current_thread();

    DEBUG_ASSERT(current_thread->magic == THREAD_MAGIC);
    DEBUG_ASSERT(current_thread->state == THREAD_RUNNING);

    THREAD_LOCK(state);

    THREAD_STATS_INC(yields);

    /* we are yielding the cpu, so stick ourselves into the tail of the run queue and reschedule */
    current_thread->state = THREAD_READY;
    current_thread->remaining_quantum = 0;
    if (likely(!thread_is_idle(current_thread))) { /* idle thread doesn't go in the run queue */
        insert_in_run_queue_tail(current_thread);
    }
    thread_resched();

    THREAD_UNLOCK(state);
}
```



- thread_yield()를 통해 thread가 reschedule 되는 과정에서 최초로 timer 실행

- 이후 idle_thread_routine()을 통해 idle thread로 변경



### thread_resched()에서 timer 설정

// kernel/thread.c => thread_is_real_time_or_idle / timer_set_periodic

```c
void thread_resched(void) {
    thread_t *oldthread;
    thread_t *newthread;

    thread_t *current_thread = get_current_thread();
    uint cpu = arch_curr_cpu_num();
// ...
#if PLATFORM_HAS_DYNAMIC_TIMER
    if (thread_is_real_time_or_idle(newthread)) {
        if (!thread_is_real_time_or_idle(oldthread)) {
            /* if we're switching from a non real time to a real time, cancel
             * the preemption timer. */
#if DEBUG_THREAD_CONTEXT_SWITCH
            dprintf(ALWAYS, "arch_context_switch: stop preempt, cpu %d, old %p (%s), new %p (%s)\n",
                    cpu, oldthread, oldthread->name, newthread, newthread->name);
#endif
            timer_cancel(&preempt_timer[cpu]);
        }
    } else if (thread_is_real_time_or_idle(oldthread)) {
        /* if we're switching from a real time (or idle thread) to a regular one,
         * set up a periodic timer to run our preemption tick. */
#if DEBUG_THREAD_CONTEXT_SWITCH
        dprintf(ALWAYS, "arch_context_switch: start preempt, cpu %d, old %p (%s), new %p (%s)\n",
                cpu, oldthread, oldthread->name, newthread, newthread->name);
#endif
        timer_set_periodic(&preempt_timer[cpu], 10, thread_timer_tick, NULL);
    }
#endif
// ...
}
```

// kernel/timer.c

```c
void timer_set_periodic(timer_t *timer, lk_time_t period, timer_callback callback, void *arg) {
    if (period == 0)
        period = 1;
    timer_set(timer, period, period, callback, arg);
}
```

// kernel/timer.c

```c
static void timer_set(timer_t *timer, lk_time_t delay, lk_time_t period, timer_callback callback, void *arg) {
    lk_time_t now;

    LTRACEF("timer %p, delay %u, period %u, callback %p, arg %p\n", timer, delay, period, callback, arg);

    DEBUG_ASSERT(timer->magic == TIMER_MAGIC);

    if (list_in_list(&timer->node)) {
        panic("timer %p already in list\n", timer);
    }

    now = current_time();
    timer->scheduled_time = now + delay;
    timer->periodic_time = period;
    timer->callback = callback;
    timer->arg = arg;

    LTRACEF("scheduled time %u\n", timer->scheduled_time);

    spin_lock_saved_state_t state;
    spin_lock_irqsave(&timer_lock, state);

    uint cpu = arch_curr_cpu_num();
    insert_timer_in_queue(cpu, timer);

#if PLATFORM_HAS_DYNAMIC_TIMER
    if (list_peek_head_type(&timers[cpu].timer_queue, timer_t, node) == timer) {
        /* we just modified the head of the timer queue */
        LTRACEF("setting new timer for %u msecs\n", delay);
        platform_set_oneshot_timer(timer_tick, NULL, delay);
    }
#endif

    spin_unlock_irqrestore(&timer_lock, state);
}
```



## Thread scheduling

### HW Timer interrupt handler

// kernel/timer.c

```c
static enum handler_return timer_tick(void *arg, lk_time_t now) {
    timer_t *timer;
    enum handler_return ret = INT_NO_RESCHEDULE;
// ...
    for (;;) {
        /* see if there's an event to process */
        timer = list_peek_head_type(&timers[cpu].timer_queue, timer_t, node);
        if (likely(timer == 0))
            break;
        LTRACEF("next item on timer queue %p at %u now %u (%p, arg %p)\n", timer, timer->scheduled_time, now, timer->callback, timer->arg);
        if (likely(TIME_LT(now, timer->scheduled_time)))
            break;
// ...
        bool periodic = timer->periodic_time > 0;

        LTRACEF("timer %p firing callback %p, arg %p\n", timer, timer->callback, timer->arg);
        KEVLOG_TIMER_CALL(timer->callback, timer->arg);
        if (timer->callback(timer, now, timer->arg) == INT_RESCHEDULE)
            ret = INT_RESCHEDULE;
// ...
        /* if it was a periodic timer and it hasn't been requeued
         * by the callback put it back in the list
         */
        if (periodic && !list_in_list(&timer->node) && timer->periodic_time > 0) {
            LTRACEF("periodic timer, period %u\n", timer->periodic_time);
            timer->scheduled_time = now + timer->periodic_time;
            insert_timer_in_queue(cpu, timer);
        }
    }
// ...
}
```



- timer_tick 함수 안에 list_peek_head_type(); 이 List의 head에 있는 timer를 가져옴
if (timer->callback(timer, now, timer->arg) == INT_RESCHEDULE)
ret = INT_RESCHEDULE 에서는 Timer의 callback 함수 실행
if (periodic && !list_in_list(&timer->node) && timer->periodic_time > 0) 에서는 주기적으로 실행되어야하는 timer인 경우 다시 list에 삽입



### Timer callback function

// kernel/thread.c

```c
enum handler_return thread_timer_tick(struct timer *t, lk_time_t now, void *arg) {
    thread_t *current_thread = get_current_thread();

    if (thread_is_real_time_or_idle(current_thread))
        return INT_NO_RESCHEDULE;

    current_thread->remaining_quantum--;
    if (current_thread->remaining_quantum <= 0) {
        return INT_RESCHEDULE;
    } else {
        return INT_NO_RESCHEDULE;
    }
}
```

// lib/libc/include/sys/types.h

```c
enum handler_return {
    INT_NO_RESCHEDULE = 0,
    INT_RESCHEDULE,
};
```

// arch/arm64/exceptions.S => thread_preempt

```assembly
.macro irq_exception
    regsave_short
    msr daifclr, #1 /* reenable fiqs once elr and spsr have been saved */
    mov x0, sp
    bl  platform_irq
    cbz x0, .Lirq_exception_no_preempt\@
    bl  thread_preempt
.Lirq_exception_no_preempt\@:
    msr daifset, #1 /* disable fiqs to protect elr and spsr restore */
    b   arm64_exc_shared_restore_short
.endm
```

// kernel/thread.c => thread_resched()

```c
void thread_preempt(void) {
    thread_t *current_thread = get_current_thread();

    DEBUG_ASSERT(current_thread->magic == THREAD_MAGIC);
    DEBUG_ASSERT(current_thread->state == THREAD_RUNNING);

#if THREAD_STATS
    if (!thread_is_idle(current_thread))
        THREAD_STATS_INC(preempts); /* only track when a meaningful preempt happens */
#endif

    KEVLOG_THREAD_PREEMPT(current_thread);

    THREAD_LOCK(state);

    /* we are being preempted, so we get to go back into the front of the run queue if we have quantum left */
    current_thread->state = THREAD_READY;
    if (likely(!thread_is_idle(current_thread))) { /* idle thread doesn't go in the run queue */
        if (current_thread->remaining_quantum > 0)
            insert_in_run_queue_head(current_thread);
        else
            insert_in_run_queue_tail(current_thread); /* if we're out of quantum, go to the tail of the queue */
    }
    thread_resched();

    THREAD_UNLOCK(state);
}
```



### thread_resched()

// kernel/thread.c => get_top_thread(cpu) 를 통해 run_queue 에서 thread를 불러옴

```c
void thread_resched(void) {
    thread_t *oldthread;
    thread_t *newthread;

    thread_t *current_thread = get_current_thread();
    uint cpu = arch_curr_cpu_num();
// ...
    newthread = get_top_thread(cpu);
// ...
}
```

// kernel/thread.c => 가장 높은 우선순위부터 적용(run_queue_bitmap)

```c
static thread_t *get_top_thread(int cpu) {
    thread_t *newthread;
    uint32_t local_run_queue_bitmap = run_queue_bitmap;

    while (local_run_queue_bitmap) {
        /* find the first (remaining) queue with a thread in it */
        uint next_queue = sizeof(run_queue_bitmap) * 8 - 1 - __builtin_clz(local_run_queue_bitmap);

        list_for_every_entry(&run_queue[next_queue], newthread, thread_t, queue_node) {
#if WITH_SMP
            if (newthread->pinned_cpu < 0 || newthread->pinned_cpu == cpu)
#endif
            {
                list_delete(&newthread->queue_node);

                if (list_is_empty(&run_queue[next_queue]))
                    run_queue_bitmap &= ~(1<<next_queue);

                return newthread;
            }
        }

        local_run_queue_bitmap &= ~(1<<next_queue);
    }
    /* no threads to run, select the idle thread for this cpu */
    return idle_thread(cpu);
}
```

// kernel/thread.c => set_current_thread와 arch_context_switch

```c
void thread_resched(void) {
    thread_t *oldthread;
    thread_t *newthread;

    thread_t *current_thread = get_current_thread();
    uint cpu = arch_curr_cpu_num();
// ...
    /* do the switch */
    set_current_thread(newthread);
// ...
    /* do the low level context switch */
    arch_context_switch(oldthread, newthread);
// ...
}
```

// kernel/include/kernel/thread.h

```c
static inline void set_current_thread(thread_t *t) {
    arch_set_current_thread(t);
}
```

// arch/arm64/include/arch/arch_ops.h

```c
static inline void arch_set_current_thread(struct thread *t) {
    ARM64_WRITE_SYSREG(tpidr_el1, (uint64_t)t);
}
```

// arch/arm64/thread.c

```c
void arch_context_switch(thread_t *oldthread, thread_t *newthread) {
    LTRACEF("old %p (%s), new %p (%s)\n", oldthread, oldthread->name, newthread, newthread->name);
    arm64_fpu_pre_context_switch(oldthread);
#if WITH_SMP
    DSB; /* broadcast tlb operations in case the thread moves to another cpu */
#endif
    arm64_context_switch(&oldthread->arch.sp, newthread->arch.sp);
}
```



- 새로운 thread를 동작 시키기 위한 context switch 진행