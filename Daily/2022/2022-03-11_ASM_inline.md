# 2022-03-11 (ASM inline)

C 언어의 ASM inline 문법

https://wiki.kldp.org/KoreanDoc/html/EmbeddedKernel-KLDP/app3.basic.html

```c++
__asm__ __volatile__ (asms : output : input : clobber);
```

> - __asm__
>
>   다음에 나오는 것이 인라인 어셈블리 임을 나타낸다. ANSI엔 __asm__ 으로만 정으되어 있으므로 asm 과 같은 키워드는 사용하지 않는 것이 바람직하다.
>
> - __volatile__
>
>   이 키워드를 사용하면 컴파일러는 프로그래머가 입력한 그래도 남겨두게 된다. 즉 최적화 나 위치를 옮기는 등의 일을 하지 않는다. 예를 들어 output 변수중 하나가 인라인 어셈블리엔 명시되어 있지만 다른 곳에서 사용되지 않는다고 판단되면 컴파일러는 이 변수를 알아서 잘 없애주기도 한다. 이런 경우 이런 것을 고려해 프로그램을 짰다면 상관 없겠지만 만에 하나 컴파일러가 자동으로 해준 일 때문에 버그가 발생할 수도 있다. 그러므로 __volatile__ 키워드를 사용해 주는 것이 좋다.
>
> - asms
>
>   따옴표로 둘러싸인 어셈블리 코드. 코드 내에서는 %x과 같은 형태로 input, output 파라미터를 사용할 수 있으며 컴파일 하면 파라미터가 치환된 대로 어셈블리 코드로 나타난다.
>
> - output
>
>   변수들을 적어 주고 각각은 쉼표고 구분된다. 결과 값을 출력하는 변수를 적는다.
>
> - input
>
>   output과 같은 방식으로 사용하고 인라인 어셈블리 코드에 넘겨주는 파라미터를 적는다.
>
> - clobber
>
>   output, input에 명시되어 있진 않지만 asms를 실행해서 값이 변하는 것을 적어 준다. 각 변수는 쉼표로 구분되고 각각을 따옴표로 감싸준다.

이런 식으로 없는 값은 생략도 가능하다

```c
__asm__ __volatile__ (asms : output : : clobber);
```

인자 넣는 예시

%0 이 첫번째 인자, %1 이 두번째 인자라고 한다.

https://gcc.gnu.org/onlinedocs/gcc-5.3.0/gcc/Extended-Asm.html

% 사용법 관련 질문 답변

https://stackoverflow.com/questions/3589157/what-is-r-and-double-percent-in-gcc-inline-assembly-language

```c
     void DoCheck(uint32_t dwSomeValue)
     {
        uint32_t dwRes;
     
        // Assumes dwSomeValue is not zero.
        asm ("bsfl %1,%0"
          : "=r" (dwRes)
          : "r" (dwSomeValue)
          : "cc");
     
        assert(dwRes > 3);
     }
```

내가 만들어본 예시는 아래와 같다. 

(SystemInit 함수 예시: https://chowdera.com/2021/08/20210817183039959u.html)

(MSP 세팅 예시: https://contiki-ng.readthedocs.io/en/latest/_api/cmsis__armcc__V6_8h_source.html)

```c
#define STACK_BASE 0xD00

void SystemInit(void) {
    __asm("mov r0, %0"::"r"(STACK_BASE));
	__asm("msr MSP, r0");
	__main();
}
```

MSP (Main Stack Pointer) 에 강제로 stack 주소를 쓰는 것이다. (Cortex-M 은 원래 0x0 이 SP 값으로 들어가지만, secure 에서 non-secure 로 이동하는 경우 stack 주소를 세팅할 방법을 몰라서 강제로 설정해주었다.) 괄호 안에 매크로처럼 숫자나 변수 이름을 그대로 넣어도 된다. linux kernel 에 예시가 많아서 참고해서 사용하였다.



※ secure 에서 non secure 로 가기 전에 MSP_NS 와 VTOR_NS 를 설정해주는 예시

https://mcuoneclipse.com/2019/04/27/trustzone-with-armv8-m-and-the-nxp-lpc55s69-evk/

```c
#define NON_SECURE_START          0x00010000
 
/* typedef for non-secure callback functions */
typedef void (*funcptr_ns) (void) __attribute__((cmse_nonsecure_call));
 
int main(void)
{
    funcptr_ns ResetHandler_ns;
 
    /* Init board hardware. */
    /* attach main clock divide to FLEXCOMM0 (debug console) */
    CLOCK_AttachClk(BOARD_DEBUG_UART_CLK_ATTACH);
 
    BOARD_InitPins();
    BOARD_BootClockFROHF96M();
    BOARD_InitDebugConsole();
 
    PRINTF("Hello from secure world!\r\n");
  
    /* Set non-secure main stack (MSP_NS) */
    __TZ_set_MSP_NS(*((uint32_t *)(NON_SECURE_START)));
  
    /* Set non-secure vector table */
    SCB_NS-&amp;amp;gt;VTOR = NON_SECURE_START;
     
    /* Get non-secure reset handler */
    ResetHandler_ns = (funcptr_ns)(*((uint32_t *)((NON_SECURE_START) + 4U)));
      
    /* Call non-secure application */
    PRINTF("Entering normal world.\r\n");
    /* Jump to normal world */
    ResetHandler_ns();
    while (1)
    {
        /* This point should never be reached */
    }
}
```

