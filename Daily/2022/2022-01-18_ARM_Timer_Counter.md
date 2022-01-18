# 2022-01-18 (ARM Timer Counter)

ARM v8-A 에서 간단히 시간 측정을 할 수 있는 방법

CNTPCT_ELn 레지스터를 확인하면 된다. Timer의 count 값을 가지고 있는 ARM System register이다.

https://developer.arm.com/documentation/den0024/a/ARMv8-Registers/System-registers

https://developer.arm.com/documentation/ddi0595/2021-09/External-Registers/CNTPCT--Counter-timer-Physical-Count



xen mainline 코드를 참고해보면 mrs 명령어를 통해 아래와 같이 접근할 수 있다.

```c
xen/include/asm-arm/cpregs.h:#define CNTPCT          p15,0,c14       /* Time counter value */
xen/include/asm-arm/cpregs.h:#define CNTPCT_EL0              CNTPCT
xen/include/asm-arm/time.h:        return READ_SYSREG64(CNTPCT_EL0);
```

```c
// xen/include/asm-arm/arm64/sysregs.h
#define READ_SYSREG64(name) ({                          \
    uint64_t _r;                                        \
    asm volatile("mrs  %0, "__stringify(name) : "=r" (_r));         \
    _r; })
```

그러면 `mrs CNTPCT_EL0` 로 값을 읽어온 후에, 일정 시간 후에 다시 `mrs CNTPCT_EL0` 로 읽어온 값과 차이를 구하면 흘러간 시간을 계산할 수 있다. 인가되는 Hz 정보를 이용하면 정확한 시간을 알 수 있을 것이다.