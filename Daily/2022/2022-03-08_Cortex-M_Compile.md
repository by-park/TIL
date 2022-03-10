# 2022-03-08 (Cortex-M Compile)

Arm Cortex M Compile

### Scatter file

https://usermanual.wiki/Document/compileruserguide100748061000en.1373020596/html

```c
//scatter.scat
LR1 0x0000 0x20000
{ 
  ER_RO 0x0 
  { 
    init.o (INIT, +FIRST) 
    *(+RO)
  }
  
  ER_RW 0x400000 
  {
     *(+RW)
  }
 
  ER_ZI 0x405000 
  { 
     *(+ZI) 
  }
}
```

https://www.keil.com/support/man/docs/armlink/armlink_pge1362075656353.htm

![Components of a scatter file](https://www.keil.com/support/man/docs/armlink/armlink_pge1462449627174.png)

위의 scatter.scat 사용

https://www.keil.com/support/man/docs/armclang_intro/armclang_intro_pge1362066010024.htm

```shell
// build command
armlink --scatter=scatter.scat init.o main.o
```

참고) scatter file syntax

https://developer.arm.com/documentation/dui0474/m/scatter-file-syntax

https://stackoverflow.com/questions/68033760/understanding-this-c-syntax-for-arm-cortex-m-interrupt-vector-table-definition

scatter loading 설명

http://recipes.egloos.com/5013253

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=pjsin865&logNo=120060750272

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=pjsin865&logNo=120060751669



### Section

scatter file 에 들어가는 영역은 이렇게 지정해줄 수 있음 (c 파일에서 아래와 같이 \__attribute__ 하고 선언하면 됨 )

https://dhpark1212.tistory.com/entry/ARM-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC-%EA%B4%80%EB%A0%A8

```c
#define SECT90(name) __attribute__ ((section (#name),used))
```



### Vector Table

VTOR 레지스터가 제공되지 않는 경우 (Cortex-M0) 아래와 같이 코드를 작성했다고 한다.

https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=actionprog&logNo=220284327230&parentCategoryNo=2&categoryNo=&viewDate=&isShowPopularPosts=true&from=search

> Readme.txt 를 보면, Vector Table을 SRAM에 복사해서, 사용하라고 되어있다.
>
> A solution will be to relocate by software the vector table to the internal SRAM.
>
> 메인 함수의 최상단에 위치해야 한다.
>
> This operation should be done at the intialization phase of the application.

```c
#define _ApplicationAddress (uint32_t)0x08003000

__IO uint32_t pulVectorTable[48] __attribute__((section(".RAMVectorTable")));

int main(void)
{
    uint32_t ulTemp = 0;
    for (ulTemp = 0; ulTemp < 48; ulTemp++)
        pulVectorTable[ulTemp] = *(__IO uint32_t*)(_AppliationAddress + (ulTemp << 2));
    __SYSCFG_CLK_ENABLE();
    __HAL_REMAPMEMORY_SRAM();
}
```

relocate 방식 다른 코드

https://community.infineon.com/t5/XMC/XMC4800-Vector-table-remap-relocation-from-flash-to-RAM/td-p/307521

```c
// system_XMC4800.c
__attribute__((section(".ram_vectors")))
uint8_t SYSTEM_ramVectors[512];					/** Place holder for vector table in RAM. Will be placed in correct segment by linker,
													filled by reset vector and activated by SYSTEM_relocateVectorsToRam() */

/**
 * Relocate vectors to RAM.
 *
 * Must be called after assembler start-up script performed the copy operation from flash.
 */
void
SYSTEM_relocateVectorsToRam(void) {

	/* relocate vector table to RAM */
	__disable_irq();
	SCB->VTOR = (uint32_t) &SYSTEM_ramVectors;
	__DSB();
	__enable_irq();
}
```

참고) bootloader 가 vector table copy

https://techblog.paalijarvi.fi/2021/09/05/firmware-interrupt-vector-table-relocation-by-bootloader-considered-harmful/

![img](https://techblog.paalijarvi.fi/wp-content/uploads/2021/09/bootloader_vtable_harmful_01_original_situation-1024x609.jpg)



Cortex-M 3 이나 4 는 VTOR 레지스터 사용

https://developer.arm.com/documentation/101273/0101/Cortex-M55-Processor-level-components-and-system-registers---Reference-Material/System-Control-and-Implementation-Control-Block/Vector-Table-Offset-Register

https://developer.arm.com/documentation/dui0552/a/cortex-m3-peripherals/system-control-block/vector-table-offset-register

`SCB->VTOR` 이며 주소는 `0xE000ED08`이다.

![img](https://documentation-service.arm.com/static/61953ad3f45f0b1fbf3a8cd1?token=)

Non secure 용 VTOR 주소는 `0xE000ED08` 이고, Secure 용 VTOR 주소는 `0xE002ED08` 이다.

https://www.keil.com/support/man/docs/uv4/uv4_cp_armv8_nvic.htm

ARM manual 을 보면 VTOR reset value 는 unknown 이다.

https://developer.arm.com/documentation/100235/0004/the-cortex-m33-peripherals/system-control-block/system-control-block-registers-summary?lang=en

VTOR 의 reset  value는 0 이라는 답변

https://stackoverflow.com/questions/43577121/vtor-usage-in-arm-m3

VTOR 의 기본값은 칩 설계자에 의해 정의된다.

https://www.elec4.co.kr/article/articleView.asp?idx=17179

https://discourse-production.oss-cn-shanghai.aliyuncs.com/original/3X/a/2/a2c6f2922ad7fd61a9dd333a0931608d124a90d9.pdf



Vector Table 에 선언하는 법. 맨 처음은 Stack Pointer 이고, 그 다음 주소가 main 을 실행시키기 위한 시작 PC (Reset Handler)이다.

https://arm-software.github.io/CMSIS_5/Core/html/startup_c_pg.html

```c
/*----------------------------------------------------------------------------
  Exception / Interrupt Vector table
 *----------------------------------------------------------------------------*/
extern const pFunc __VECTOR_TABLE[240];
       const pFunc __VECTOR_TABLE[240] __VECTOR_TABLE_ATTRIBUTE = {
  (pFunc)(&__INITIAL_SP),                   /*     Initial Stack Pointer */
  Reset_Handler,                            /*     Reset Handler */
  NMI_Handler,                              /* -14 NMI Handler */
  HardFault_Handler,                        /* -13 Hard Fault Handler */
  MemManage_Handler,                        /* -12 MPU Fault Handler */
  BusFault_Handler,                         /* -11 Bus Fault Handler */
  UsageFault_Handler,                       /* -10 Usage Fault Handler */
  0,                                        /*     Reserved */
  0,                                        /*     Reserved */
  0,                                        /*     Reserved */
  0,                                        /*     Reserved */
  SVC_Handler,                              /*  -5 SVC Handler */
  DebugMon_Handler,                         /*  -4 Debug Monitor Handler */
  0,                                        /*     Reserved */
  PendSV_Handler,                           /*  -2 PendSV Handler */
  SysTick_Handler,                          /*  -1 SysTick Handler */
  /* Interrupts */
  WAKEUP0_IRQHandler,                       /*   0 Wakeup PIO0.0 */
  WAKEUP1_IRQHandler,                       /*   1 Wakeup PIO0.1 */
  WAKEUP2_IRQHandler,                       /*   2 Wakeup PIO0.2 */
  // :
  // :
  EINT1_IRQHandler,                         /*  30 PIO INT1 */
  EINT2_IRQHandler,                         /*  31 PIO INT2 */
  // :
  // :
};
```

제일 첫번째에는 Stack Pointer 위치가 들어있는데, reset 시에 프로세서는 이 위치 (0x00000000) 에 있는 값을 MSP (Main Stack Pointer) 레지스터에 넣는다.

https://stackoverflow.com/questions/60452383/behaviour-of-vtor-register-on-stm32h7-at-reset

ARM의 Cortex-M 프로세서를 쓰면 Reset 동작 시에 0x0 위치에 있는 것은 자동으로 SP 에 들어가고, 0x4 위치에 있는 것은 PC 로 옮겨줘서 어셈 없이 c 파일로만 코드를 짤 수 있다.

Vector table

https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/exception-model/vector-table

![Figure 2.2. Vector table](https://documentation-service.arm.com/static/5ea823e69931941038df1b02?token=)

https://www.elec4.co.kr/article/articleView.asp?idx=17179

![img](http://elec4.co.kr/photo/2017/170504Design-ARM1.jpg)

Cortex-M 의 인터럽트 컨트롤러 NVIC

![img](https://www.elec4.co.kr/photo/2017/170504Design-ARM2.jpg)