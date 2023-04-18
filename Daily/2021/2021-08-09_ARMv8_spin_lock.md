# 2021-08-09 (ARMv8 spin lock)

LK 코드에서 MMU를 켜지 않고, SMP config만 켜서 cpu 1, 2, 3을 켰다. 그러나 mp_reschedule을 하면서 IPI를 보내기 전에는 arm64_secondary_entry 에 있는 spin_lock에서 나오지 못했다. 아래에서 조사한 내용을 정리해보면 MMU 및 Cache를 켜지 않으면 spin_unlock 했을 때 자동으로 spin_lock 안의 wfe (=wait for event) 상태를 깨우지 못한다. (자동으로 event를 보내주는 Global Monitor가 동작하지 않아서) printf 에 spin_lock 을 추가해놨더니 printf에서도 걸렸다. 그래서 야매로 arch_spin_unlock 함수 안에 ret 직전에 sev 를 써봤는데 정상적으로 spin_lock 들을 다 빠져나오는 것을 확인하였다.

```c
// arch/arm64/arch.c
#if WITH_SMP
void arm64_secondary_entry(ulong asm_cpu_num) {
    uint cpu = arch_curr_cpu_num();
    if (cpu != asm_cpu_num)
        return;

    arm64_cpu_early_init();

    spin_lock(&arm_boot_cpu_lock); //=> secondary core들이 여기서 걸렸다
    spin_unlock(&arm_boot_cpu_lock);

    /* run early secondary cpu init routines up to the threading level */
    lk_init_level(LK_INIT_FLAG_SECONDARY_CPUS, LK_INIT_LEVEL_EARLIEST, LK_INIT_LEVEL_THREADING - 1);

    arch_mp_init_percpu();
```



(2023.04.18 업데이트)

spinlock 코드 내부에서 필요한 ldrex 와 같은 exclusive 가 필요한 연산 (혹은 atomic 하다고도 표현했다 [출처](https://github.com/rust-embedded/rust-raspberrypi-OS-tutorials/discussions/132)) 을 사용하기 위해서는 MMU 가 필요하다. 또한 이전에 조사한 global monitor 가 메모리 간 접근에 대해서 동기화를 해주기 위해서는 MMU 가 켜져야한다. 무조건 event 를 보내주는 것이 아니라 메모리 간 동기화 관련한 업데이트가 발생할 때 event 를 보내주는 역할을 하는 것으로 보인다.

참고

- spinlock 을 사용하기 위한 전제 조건은 MMU enable (질문 답변): https://github.com/rust-embedded/rust-raspberrypi-OS-tutorials/discussions/132
- local monitor & global monitor 설명 (개인 블로그): http://cloudrain21.com/local-exclusive-monitor-global-exclusive-monitor
- Exclusive monitor 설명 (ARM 공식문서): https://developer.arm.com/documentation/dht0008/a/arm-synchronization-primitives/exclusive-accesses/exclusive-monitors
- MMU 내부에 Exclusive instruction 관련 동작 설명 (ARM 공식문서): https://developer.arm.com/documentation/100442/0100/functional-description/memory-management-unit/mmu-memory-accesses/hardware-management-of-the-access-flag-and-dirty-state



### Spin lock 코드

LK 코드에 보면 spin lock 코드가 다음과 같이 구현되어있다.

```assembly
// arch/arm64/spinlock.S
FUNCTION(arch_spin_trylock)
        mov     x2, x0
        mov     x1, #1
        ldaxr   x0, [x2]
        cbnz    x0, 1f
        stxr    w0, x1, [x2]
1:
        ret

FUNCTION(arch_spin_lock)
        mov     x1, #1
        sevl
1:
        wfe
        ldaxr   x2, [x0]
        cbnz    x2, 1b
        stxr    w2, x1, [x0]
        cbnz    w2, 1b
        ret

FUNCTION(arch_spin_unlock)
        stlr    xzr, [x0]
        ret

```

spin_unlcok 코드를 보면 sev (=send event) 코드가 없는데, ARMv8 부터는 sev를 하지 않아도 global monitor를 통해서 자동으로 event가 간다고 한다. (이 내용에 대해서 확인하기 위해서는 ARMv8 Reference Manual을 보라고 되어있다.)

> Which leaves the question of why in ARMv8-A you don't always need the SEV in unlock/release function. This is down to a change introduced in ARMv8-A where clearing the Global Monitor automatically generates an event (see D1.17.1 "Wait for Event mechanism and Send event" in the ARMv8-A Architecture Reference Manual).

https://community.arm.com/developer/ip-products/processors/f/cortex-a-forum/4273/how-to-understand-armv8-sevl-instruction-in-spin-lock

load-acquire와 store-release 어셈블리를 쓰면 global monitor가 event를 보내준다는 답변

> When locking, the line
>
> ```
> "   ldaxrh  %w2, %4\n"
> ```
>
> after the `wfe` performs an exclusive load-acquire of the lock. As stated in the previous comment, this will mark the address of the lock with the global monitor.
>
> The unlock code performs a store-release on the same address
>
> ```
> "   stlrh   %w1, %0\n"
> ```
>
> This will generate the event. That is the reason why they use a load-acquire for the lock in the locking function, as opposed to regular load, and why you don't need a SEV when unlocking.

https://stackoverflow.com/questions/32276313/how-is-a-spin-lock-woken-up-in-linux-arm64



### Global Monitor

Exclusive Monitor에 대한 설명

Gloval Monitor는 여러 코어를 연결하는 메모리 (L2 memory) 간에 한 개 이상 존재하고 공유한다는 설명이 있다.

https://tot0rokr.github.io/arm64/synchronization/exclusive-monitor/

![local exclusive monitor, global exclusive monitor](http://cloudrain21.com/wordpress/wp-content/uploads/2014/08/local-exclusive-monitor-global-exclusive-monitor-300x267.png)

그림 및 local & global exclusive monitor 설명: http://cloudrain21.com/local-exclusive-monitor-global-exclusive-monitor

그러므로 global monitor는 mmu 및 cache가 enable 되어야 동작할 것이다. 이와 관련된 질문 답변을 찾았다.

> Hi,
> Unless I am missing something, there is a no spin lock acquisition with caches disabled in the secondary CPU warm boot code path when HW_ASSISTED_COHERENCY is 0.
>
> If you referring to the lock acquisition here: https://github.com/ARM-software/arm-trusted-firmware/blob/master/lib/psci/psci_common.c#L767, this is a `bakery` [1] lock which is designed to work with varying cache states.
>
> The only spin_lock acquired is done with caches enabled (https://github.com/ARM-software/arm-trusted-firmware/blob/master/lib/psci/psci_on.c#L178). The caches are enabled in `psci_do_pwrup_cache_maintenance()`.
>
> Which one of the 2 locks mentioned above are you having trouble with?
>
> [1] http://lamport.azurewebsites.net/pubs/bakery.pdf

https://github.com/ARM-software/tf-issues/issues/540



### spin lock 동작을 그림으로 표현한 블로그

https://www.fatalerrors.org/a/arm64-linux5.x-spin-lock.html





+ LK의 owner 인 트래비스 지셀브리지트 (Travis Geiselbrecht) 님의 이름을 읽는 법을 찾아보았다.