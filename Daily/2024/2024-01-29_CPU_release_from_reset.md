# 2024-01-29 (CPU release from reset)

문제 현상

- Remapper SFR 을 세팅하였으나, T32 attach 해서 0x0 을 보면 원하는 주소로 보이지 않음 (주소 옵션이 MX 로 보임, A로 볼 때와 axi 로 볼 때 결과도 다름)

- CPU 를 동작시키기 위한 레지스터 설정 후에도 동작 시작하지 않음



문제 해결

- 세부 블락 내에 CPU 동작시키기 위한 추가 SFR이 있었음
- 해당 레지스터로 동작시키면 0x200 에 가있음

Exception Vector Table 중 0x200 은 아래 자료 참고

- Remapper 의 문제는 (1) T32 script 에 값을 쓰는 코드가 있었음 (2) System MMU 가 enable 되어있었음

- 해당 코드를 원하는 대로 수정하고, System MMU 는 disable 한 후에 CPU 를 동작시켜서 해결 완료



[참고 자료 1 SMMU]

https://jang574.tistory.com/50

![img](https://t1.daumcdn.net/cfile/tistory/200AC7494F4501370C)

> SMMU의 이점으로
>
> 1) 연속된 물리적인 메모리 공간을 필요로 하는
> 주요 Memory Accessing Device (GPU, Video engine 등)들이
>
> 메모리 단편화의 문제로 물리적으로 연속된 메모리 공간을 reserved 하여 사용하던 기존 방식에서
> 물리적으로 단편화된 메모리를 영역을 SMMU를 통해 하나의 연속된 메모리 공간으로 Accessing을 할 수 있다는 것.
>
> 2) Virtualization 입장에서는 SMMU를 통해 Stage2 Address Translation을 처리하므로, performance 향상이 가능하다는 것



[참고자료 2 Exception Vector Table]

https://developer.arm.com/documentation/100933/0100/AArch64-exception-vector-table

| **Address**      | **Exception type** | **Description**        |
| ---------------- | ------------------ | ---------------------- |
| VBAR_ELn + 0x000 | Synchronous        | Current EL with SP0    |
| + 0x080          | IRQ/vIRQ           |                        |
| + 0x100          | FIQ/vFIQ           |                        |
| + 0x180          | SError/vSError     |                        |
| + 0x200          | Synchronous        | Current EL with SPx    |
| + 0x280          | IRQ/vIRQ           |                        |
| + 0x300          | FIQ/vFIQ           |                        |
| + 0x380          | SError/vSError     |                        |
| + 0x400          | Synchronous        | Lower EL using AArch64 |
| + 0x480          | IRQ/vIRQ           |                        |
| + 0x500          | FIQ/vFIQ           |                        |
| + 0x580          | SError/vSError     |                        |
| + 0x600          | Synchronous        | Lower EL using AArch32 |
| + 0x680          | IRQ/vIRQ           |                        |
| + 0x700          | FIQ/vFIQ           |                        |
| + 0x780          | SError/vSError     |                        |



0x200에 간 이유는 코드가 제대로 들어있지 않아서 assembly instruction 으로 0 을 실행해서, exception 이 발생했기 때문이다. 또한 VBAR 레지스터 설정을 안 했기 때문에 0x0 위치에서부터 0x200 offset 을 갔다.

0x0 이 아닌 0x200 으로 간 이유는 SPsel 레지스터를 설정해두지 않았기 때문에 각 Exception Level 에 맞는 Stack Pointer 를 사용해서이다.

https://whatdocumentary.tistory.com/88

![img](https://blog.kakaocdn.net/dn/cVvQ3F/btqWKkaKraj/tl5d9GqihDf1PqkCcMbfxk/img.png)

> **SPsel 레지스터의 SP 비트에** **1을 설정하면 익셉션 레벨에 따른 SP_ELn을 사용**하고, **0을 설정하면 익셉션 레벨과 상관없이 SP_EL0를 사용**한다.
>
> 이 설정에 따라 익셉션 발생 시 이동하는 벡터(Vector) 위치도 달라진다.
>
> **t는 스레드(Thread)의 약어**이며, **SP0의 스택을 사용한다는 뜻**이다.
>
> **h는 핸들러(Handler)의 약어**이며, **SPn의 스택을 사용한다는 뜻**이다.
>
> 리눅스 커널은 SPsel을 설정하지 않고 필요에 따라 해당 EL의 레벨을 직접 지정하는 방식을 구현하고 있다.



사용하지 않아도 칸을 채워두고자 할 때 코드에서 이런 식

```assembly
	.align 11
vectors:
	.align 7
bad_sync:
	b	.	/* Current EL Synchronous Thread */
	.align 7
bad_irq:
	b	.	/* Current EL IRQ Thread */
	.align 7
bad_fiq:
	b	.	/* Current EL FIQ Thread */
	.align 7
bad_serror:
	b	.	/* Current EL SError Thread */
	.align 7
do_sync:
	b	.	/* Current EL Synchronous Handler */
	.align 7
// skip
```



https://grasslab.github.io/osdi/en/labs/lab3.html

```assembly
// Simple vector table
.align 11 // vector table should be aligned to 0x800
.global exception_table
exception_table:
  b exception_handler // branch to a handler function.
  .align 7 // entry size is 0x80, .align will pad 0
  b exception_handler
  .align 7
  b exception_handler
  .align 7
  b exception_handler
  .align 7

```



https://hackmd.io/@LJP/BJ6GgXjQq

https://wenboshen.org/posts/2016-04-10-aarch64-exception

```assembly
    .align    11
SYM_CODE_START(vectors)
    kernel_ventry    1, t, 64, sync       // Synchronous EL1t
    kernel_ventry    1, t, 64, irq        // IRQ EL1t
    kernel_ventry    1, t, 64, fiq        // FIQ EL1h
    kernel_ventry    1, t, 64, error      // Error EL1t

    kernel_ventry    1, h, 64, sync       // Synchronous EL1h
    kernel_ventry    1, h, 64, irq        // IRQ EL1h
    kernel_ventry    1, h, 64, fiq        // FIQ EL1h
    kernel_ventry    1, h, 64, error      // Error EL1h
```



https://community.arm.com/support-forums/f/architectures-and-processors-forum/49774/exceptions-s-and-cache-s-in-uboot

> Wanted to understand why align 11 and align 7 are used in exceptions.s. Also the processor reference manual has exception vectors of 12 but there are only 8 present in exceptions.s why?
>
> => Because u-boot runs in Aarch64 mode, so no Aarch32 vectors are needed.



Exception Vector Table 관련 그림 자료 (PPT)

https://tc.gts3.org/cs3210/2020/spring/l/lec15/lec15.html



Q: 각 Exception Level 마다 stack pointer 를 사용하면 될 것 같은데, 공유가 필요한 경우가 왜 있을까?

https://developer.arm.com/documentation/ka005621/latest/

> SPSel 기능은 OS 커널과 같은 권한 수준(EL1, EL2 또는 EL3)에서 실행되는 소프트웨어가 사용자 애플리케이션의 스택을 사용하여 커널 스택 오버플로를 방지할 수 있도록 설계되었습니다. 그러나 이러한 사용법은 Linux 커널에 없습니다.

![img](https://documentation-service.arm.com/static/64d35de538511951cb7a56ff?token=)

> SP_EL0 레지스터에 "현재"를 저장하면 커널이 언제든지 쉽고 빠르게 "현재" 값에 액세스할 수 있습니다.
>
> 사용자 공간에서 실행될 때 SP_EL0은 사용자 공간 스택의 SP로 사용됩니다. SP_EL0을 커널 공간에서 어떻게 "현재"로 사용할 수 있습니까? 사용자 공간에서 커널 공간으로 전환할 때 예외 처리기에서 kernel_entry가 실행됩니다. 다음 코드는 Linux 커널이 이를 구현하는 방법을 보여줍니다.



https://stackoverflow.com/questions/71168020/when-and-why-we-choose-sp-el0-as-stack-pointer

https://stackoverflow.com/questions/65059491/why-save-init-task-struct-address-to-sp-el0-in-arm64-boot-code-primary-switche

> EL0 모드에서는 sp_el0만 스택 포인터로 사용할 수 있지만, EL1에서는 PSTATE 레지스터의 lsb(최하위 비트)를 사용하여 sp_el1과 sp_el0 중 어느 것이 스택 포인터인지 선택할 수 있습니다. 현재 Linux에서 이 값은 1입니다. 따라서 EL1 모드(커널 모드)에서는 sp_el1이 sp로 사용됩니다. sp_el0은 빠른 컨텍스트 전환을 위해 현재 작업의 task_struct에 대한 포인터로 사용됩니다. 



https://community.nxp.com/t5/Kinetis-Microcontrollers/What-is-CONTROL-SPSEL-value/m-p/357258

> OS 환경에서 ARM은 스레드 모드에서 실행되는 스레드가 프로세스 스택을 사용하고 커널 및 예외 처리기가 기본 스택을 사용하도록 권장합니다.



※ ARM Developer 페이지에 레지스터 이름이 ACTLR_EL3, CLIDR_EL1 이런식으로 표현되는데, 그 특정 EL을 위한 레지스터라는 의미가 아니라 그 EL 이상이면 접근이 가능하다로 이해해야한다.

