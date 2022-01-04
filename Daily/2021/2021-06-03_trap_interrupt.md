# 2021-06-03 (trap / interrupt)

### interrupt vs. trap

> 먼저 인터럽트부터 한번 되짚어 보고 가겠습니다. 인터럽트는 하드웨어가 시스템의 실행 흐름을 바꾸기 위해 의도적으로 발생시켜 CPU에게 보내는 것입니다. 그렇다면 트랩은 무엇일까요? 트랩은 소프트웨어가 시스템 호출을 하거나 실행중 오류가 생겼을때 자발적으로 발생합니다.
>
>  둘 사이의 명확한 차이가 보이시나요? 트랩은 소프트웨어로부터, 인터럽트는 하드웨어로부터 발생하는 신호입니다. 그런데 시스템 호출이 뭔지 아시나요? 아신다면 복습을, 모르신다면 알아보도록 하죠, 시스템 호출(System Call)은 커널에 접근하기 위해서 사용합니다. 제가 과거에 커널은 운영체제의 가장 중요한 부분 중 하나라 말씀드렸죠? 커널만이 할 수 있는 일이 있는데, 커널의 서비스를 이용하기 위해 요청하는 것이 '시스템 콜'입니다. 오류의 예로는 뭐가 있을까요? 대표적으로 0을 나눌 때 발생하는 것이 그 예입니다. 간단하죠? 
>
> 출처: https://haun25ne.tistory.com/41 [Your Dream]



1. Asynchronous Interrupt (비동기식 인터럽트)

우리가 흔히 인터럽트라고 부르는 것. 하드웨어 인터럽트라고도 한다. 어떤 기준에 맞춰서 이벤트가 발생하는 것이 아니라 정해진 기준 없이 그때 그때 이벤트가 발생하는 것. ex) I/O interrupt, keyboard event, timer tick 등



2. Synchronous Interrupt (동기식 인터럽트)

흔히 exception 혹은 software interrupt라고 부르는 것들. 이벤트가 언제든지 발생하는 것이 아니라 기준이나 시간에 맞추어 정해진 일을 시키는 것. 그래서 동기식 인터럽트는 명령어 실행 후 그 결과 (instruction fault)로 발생하는 경우가 많다. 예를 들어 CPU가 0으로 나누려고 했거나 page fault 가 발생한 경우 등이 있다. 소프트웨어 인터럽트가 exception으로 여겨지기도 한다. 프로그램을 작성하는 프로그래머의 요청에 의해 발생되는데 control unit이 trap으로 처리하고 주로 시스템 콜이나 디버깅에 사용된다고 한다. trap은 실행 중인 프로그램 내에 테스트를 위해 특별한 조건을 걸어 놓은 것을 말한다.



- Interrupt : 비동기식 이벤트. 다른 하드웨어 장치가 CPU와 상관없이 발생시킴. keyboard event, I/O interrupt, timer ticks
- Exception: 동기식 이벤트, 내부적으로 CPU control unit이 명령어의 실행 결과로 자주 발생시킴. 0으로 나누기, page fault
- Trap: 특별한 조건을 걸어놓고 조건에 부합하는 상황이 걸리게 하는 역할. 각 상황에 맞는 handler 또는 서비스 루틴으로 매핑해줌

https://donghoson.tistory.com/1



### trap example code

lk 코드에서 trap 코드

잘못된 cpuid 가 들어오면 while loop로 빠지게 한 듯하다.

https://github.com/littlekernel/lk/blob/master/arch/arm64/start.S

```c
#if WITH_SMP
.Lsecondary_boot:
    and     tmp, cpuid, #0xff
    cmp     tmp, #(1 << SMP_CPU_CLUSTER_SHIFT)
    bge     .Lunsupported_cpu_trap
    bic     cpuid, cpuid, #0xff
    orr     cpuid, tmp, cpuid, LSR #(8 - SMP_CPU_CLUSTER_SHIFT)

    cmp     cpuid, #SMP_MAX_CPUS
    bge     .Lunsupported_cpu_trap

    /* Set up the stack */
    ldr     tmp, =__stack_end
    mov     tmp2, #ARCH_DEFAULT_STACK_SIZE
    mul     tmp2, tmp2, cpuid
    sub     sp, tmp, tmp2

    mov     x0, cpuid
    bl      arm64_secondary_entry

.Lunsupported_cpu_trap:
    wfe
    b       .Lunsupported_cpu_trap
#endif
```



### upstream

upstream kernel & mainline kernel

upstream은 어느 branch든 최신 코드. mainline은 리누스님이 머지하여 관리하는 branch. android common kernel에는 linux mainline 코드가 반영된다. 주체는 android 쪽.