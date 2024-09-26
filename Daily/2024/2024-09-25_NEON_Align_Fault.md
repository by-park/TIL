# 2024-09-25 (NEON Align Fault)

WITH_KERNEL_VM 을 0 으로 설정해서 가상 메모리를 끄고, MMU 의 table 은 1:1로 매핑해 두었다.

그리고 부팅하면 아래와 같은 에러 메세지로 죽는다.

```text
...
creating bootstrap completion thread
data fault: Write access from PC 0x560315c4, FAR 0x5604f1e8, iss 0x61 (DFSC 0x21)
Alignment fault
ESR 0x96000061: ec 0x25, il 0x2000000, iss 0x61
iframe 0x5604ae60:
x0 0x5604f1e8 x1 x2 x3
...
x28 x29 0x5604af70 lr 0x56031760 usp 0x0
elr 0x560315c4
spsr 0x385
stack trace:
0x560315c4
0x560359bc
0x56036090
0x56031f84
panic (caller 0x56031538): die
HALT: spinning forever, reason "software panic"
```



에러 메세지를 해석하면 다음과 같다.

https://developer.arm.com/documentation/ddi0595/2020-12/AArch64-Registers/ESR-EL1--Exception-Syndrome-Register--EL1-

ESR 의 [31:26] EC (Exception Class ) 비트를 보면 Data Abort 이다.

IC [25] (Instruction Length) 는 1로 체크되어서 32 bit instruction 관련하여 발생한 문제다.

ISS[24:0] (Instruction Specific Syndrome) 은 EC (Exception Class) 에 따라 다르게 인코딩됨

ISS encoding for an exception from a Data Abort 참고

WnR (Write not Read) 비트는 1이므로 write 과정에서 발생했다.

DSFC (Data Fault Status Code) 를 보면 alignment fault 이다.



FAR 은 Fault address register 로 문제 생긴 접근 주소를 알려준다.

https://developer.arm.com/documentation/ddi0229/c/configuration/registers/fault-address-register



죽은 위치는 (PC) arm64_fpu_pre_context_switch 였다.

그래서 ARCH_COMPILEFLAGS_NOFLOAT := -mgeneral-regs-only 주석처리해뒀던 것을 다시 설정해줬다.

floating point 연산을 하지 않게 하면 해당 함수에서 죽지 않을 것이라고 생각하였는데, 아니었다. 그런 로직은 없었다.

https://github.com/littlekernel/lk/blob/master/arch/arm64/rules.mk

CPACR 레지스터를 읽어서 하드웨어가 지원하면 무조건 쓰도록 하고 있어서 해당 함수를 주석처리해야만 정상적으로 부팅했다.

https://github.com/littlekernel/lk/blob/d9362e4dd505a409776066b3d63bf82ea8f2a894/arch/arm64/include/arch/arm64.h#L62

```c
static inline void arm64_fpu_pre_context_switch(struct thread *thread) {
    uint32_t cpacr = ARM64_READ_SYSREG(cpacr_el1);
    if ((cpacr >> 20) & 3) {
        arm64_fpu_save_state(thread);
        cpacr &= ~(3 << 20);
        ARM64_WRITE_SYSREG(cpacr_el1, cpacr);
    }
}
```

https://developer.arm.com/documentation/ddi0595/2020-12/AArch64-Registers/CPACR-EL1--Architectural-Feature-Access-Control-Register



그리고 문제 생긴 지점은 `stp q0, q1, [x0]` 였다.

https://github.com/littlekernel/lk/blob/d9362e4dd505a409776066b3d63bf82ea8f2a894/arch/arm64/fpu.c#L17

```c
void arm64_fpu_save_state(struct thread *t) {
    struct fpstate *fpstate = &t->arch.fpstate;
    __asm__ volatile("stp     q0, q1, [%2, #(0 * 32)]\n"
```

이 아래로는 다 [x0, #32] [x0, #64] 이렇게 32 바이트 단위로 offset 을 추가해서 접근하는 코드이다.

thread_become_idle -> idle_thread_routine -> thread_resched -> arm64_fpu_save_state



문제는 NEON register 인 q0 등이 128 비트 (16 바이트) 단위이기 때문에 stp 를 이용한 memory access 시 16 바이트 align 을 요구하는 것으로 보인다. 아래 2가지 사항 추가로 해결되는 것을 보았다. (둘 중 하나만 적용하면 안 된다.)

NEON register 크기: https://developer.arm.com/documentation/dht0002/a/Introducing-NEON/NEON-architecture-overview/NEON-instructions

동일한 문제 (라즈베리 파이3): https://forums.raspberrypi.com/viewtopic.php?t=254677

1. 16 byte align 을 맞추도록 구조체들에 attribute 를 설정한다.

https://github.com/littlekernel/lk/blob/d9362e4dd505a409776066b3d63bf82ea8f2a894/arch/arm64/include/arch/arch_thread.h#L12

```c
struct fpstate {
    uint64_t    regs[64] __attribute__ ((aligned (16)));
    uint32_t    fpcr;
    uint32_t    fpsr;
    uint        current_cpu;
};

struct arch_thread {
    vaddr_t sp __attribute__ ((aligned (16)));
    struct fpstate fpstate __attribute__ ((aligned (16)));
};
```

https://github.com/littlekernel/lk/blob/d9362e4dd505a409776066b3d63bf82ea8f2a894/kernel/include/kernel/thread.h#L103

```c
typedef struct thread {
    int magic __attribute__ ((aligned (16)));
    struct list_node thread_list_node __attribute__ ((aligned (16)));

    /* architecture stuff */
    struct arch_thread arch __attribute__ ((aligned (16)));
//
    #if THREAD_STATS
    struct thread_specific_stats stats;
#endif
} thread_t __attribute__ ((aligned (16)));
```

2. arm64_fpu_save_state 에서 접근하는 기본 시작 주소인 thread 를 만들 때도 16 byte align 이 되게 생성한다. 

https://github.com/littlekernel/lk/blob/master/kernel/thread.c

```c
thread_t *thread_create_etc(thread_t *t, const char *name, thread_start_routine entry, void *arg, int priority, void *stack, size_t stack_size) {
    unsigned int flags = 0;

    if (!t) {
        //t = malloc(sizeof(thread_t));
        t = memalign(16, sizeof(thread_t));
        if (!t)
            return NULL;
```



1. 만 적용하면 arm64_fpu_load_state 에서 alignment fault 가 발생한다.

참고로, strict-align 은 unaligned 된 접근은 속도가 느리기 때문에 빠른 속도를 위해 align 을 강제하는 옵션으로 보인다.

https://gcc.gnu.org/onlinedocs/gcc-9.2.0/gcc/AArch64-Function-Attributes.html

그리고 본 테스트는 WITH_VM_KERNEL 을 끄고 테스트를 진행했다. 옵션을 켜서 physical memory 를 virtual memory 로 매핑하여 동작시키면 alignment 로 인한 문제는 발생하지 않는다.

