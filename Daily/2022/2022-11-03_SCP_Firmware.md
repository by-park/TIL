# 2022-11-03 (SCP Firmware)

### SCP Firmware 발표 영상 & 자료

Linaro 공식 영상: https://resources.linaro.org/en/resource/PZgSfTGcN3qrz86cbitkMn

유튜브 영상: https://www.youtube.com/watch?v=hR5YwfoyVy4

발표자료 pdf: https://static.linaro.org/connect/san19/presentations/san19-117.pdf



### SCMI 공식 문서

https://developer.arm.com/documentation/den0056/latest

https://developer.arm.com/documentation/102886/001/?lang=en



### Cortex M3 Architecture

http://www.jkelec.co.kr/img/lecture/cortex_arch/cortex_arch_2.html

https://class.ece.uw.edu/474/peckol/doc/StellarisDocumentation/IntroToCortex-M3.pdf



### Cortex M3 에서 main 호출 방법

Reset Handler 에서 main 까지

http://recipes.egloos.com/5044366

(1) \_\_main 을 reset handler 앞에 두거나, (2) reset vector 에 \_\_main 을 불러준다.

SCP 코드를 보면, reset 함수 안에서 \_\_main 을 호출하는 걸 볼 수 있다.

arch/arm/arm-m/src/arch_handlers.c

https://github.com/ARM-software/SCP-firmware/blob/master/arch/arm/arm-m/src/arch_handlers.c

```c
noreturn void arch_exception_reset(void)
{
    /*
     * When entering the firmware, before the framework is entered the following
     * things happen:
     *  1. The toolchain-specific C runtime is initialized
     *     For Arm Compiler:
     *       1. Zero-initialized data is zeroed
     *       2. Initialized data is decompressed and copied
     *     For GCC/Newlib:
     *       1. Zero-initialized data is zeroed
     *       2. Initialized data is copied by software_init_hook()
     * 2. The main() function is called by the C runtime
     */

#ifdef __ARMCC_VERSION
    extern noreturn void __main(void);

    __main();
#else
    extern noreturn void _start(void);

    _start();
#endif
}
```

