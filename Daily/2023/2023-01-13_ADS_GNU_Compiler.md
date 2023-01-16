# 2023-01-13 (ADS_GNU_Compiler)

ADS 와 GNU 비교 (임베디드 레시피)

>  ARM사에서 내 놓은 ADS나 RVCT라는 컴파일러와 GNU진영에서 내놓은 ARM compiler 사이에 Assembly Syntax Grammar는 거의 1:1로 똑같습니다. Assembly 자체는 같다고 보시면 되고요, Directive가 소문자, 그리고 앞에 .으로 시작한다는 것만 다릅니다.

http://recipes.egloos.com/v/5025867



GNU Linker Scripter

https://community.silabs.com/s/article/understand-the-gnu-linker-script-of-cortex-m4?language=en_US

```text
MEMORY
{
	FLASH (rx) : ORIGIN = 0x0, LENGTH = 0x200000 /* 2048k */
	RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 0x80000 /* 512k */
}
```

ADS 의 Scatter Loading Mechanism

http://bhduan.blogspot.com/2006/09/scatter-loading-mechanism-in-arm.html

![img](http://photos1.blogger.com/blogger2/7489/787201684971151/400/scatter4.1.jpg)

ARM ADS(ARM Developer Suite) 1.2

https://developer.arm.com/documentation/dui0064/d/introduction/about-the-arm-developer-suite/components-of-ads

scatter loading

> 프로그램을 만들어서 컴파일하면 CODE, TEXT, BSS, RO,RW,ZI 등 여러가지 타입으로 메모리에 배치되지만 크게 3가지로 나눌 수 있습니다.
>
> RO(Read Only) : 읽기만 가능한 값
> RW(Read & Write) : 읽고 쓰기가 가능한 값
> ZI(Zero Initialize) : 0으로 초기화 되는값

http://egloos.zum.com/slgi97/v/10850585