# 2022-01-17 (Disable optimization)

### inline 금지

```c
int fcn(void) __attribute__((noinline));
...
int fcn(void( { return ~~~;} 
```

https://mantdu.tistory.com/876



### 메모리 장벽

linux 에서는 barrier() 를 통해 메모리를 순차적으로 접근하도록 보장하는 방법이 있다. CPU가 반드시 소스 코드 상에 나온 순서대로 instruction 을 실행하도록 지시한다.

```c
/* Optimization barrier */
/* The "volatile" is due to gcc bugs */
#define barrier() __asm__ __volatile__("": : :"memory")
```



http://egloos.zum.com/studyfoss/v/5128961

http://cloudrain21.com/linux-kernel-memory-barrier-implementation

https://blog.dasomoli.org/222/



```c
// arch/arm/include/asm/barrier.h

#define isb(option) __asm__ __volatile__ ("isb " #option : : : "memory")
#define dsb(option) __asm__ __volatile__ ("dsb " #option : : : "memory")
#define dmb(option) __asm__ __volatile__ ("dmb " #option : : : "memory")
```

> - DMB, DSB, ISB의 세 명령
> - ARMv6까지는 CP15 레지스터를 이용하여 명령을 수행하지만 ARMv7부터는 전용 명령을 사용한다.

https://outoftheblackbox.site/Assembly-Barrier-instruction/

https://tot0rokr.github.io/arm64/barrier/pipeline-barrier-isb-dmb-dsb-ordering/

sy 가 default 옵션이라고 한다. 위의 매크로를 사용하지 않고 간단하게 쓴다면 `asm volatile ("dmb sy");`, `asm volatile("dsb sy");`, `asm volatile("isb sy")` 와 같은 식으로 작성할 수 있을 것이다.



cf) c에서 assem keyword 사용 방법

https://stackoverflow.com/questions/52550016/difference-between-asm-and-asm-in-kernel-code



cf) kernel 코드 내에 barrier 가 필요한 것인지 관련 질문

http://www.iamroot.org/xe/index.php?mid=Knowledge&document_srl=213162



### volatile

특정 변수를 최적화하지 않는 방법

```c
void foo(char *buf, int size)
{
     int i;
     volatile char *p = (volatile char *)0x8C0F;

     for (i = 0 ; i < size; i++)
     {
         buf[i] = *p;
         ...
     }
}
```

> volatile로 선언된 변수는 외부적인 요인으로 그 값이 언제든지 바뀔 수 있음을 뜻한다. 따라서 컴파일러는 volatile 선언된 변수에 대해서는 최적화를 수행하지 않는다. volatile 변수를 참조할 경우 레지스터에 로드된 값을 사용하지 않고 매번 메모리를 참조한다

https://skyul.tistory.com/337



### Optimize O0

특정 함수를 최적화하지 않는 방법

```c
void __attribute__((optimize("O0"))) func(void) { }
```

코드 범위에 최적화 끄는 방법

```c
#pragma GCC push_options
#pragma GCC optimize ("O0")
//Write your code
#pragma GCC pop_options
```



https://sonseungha.tistory.com/522

http://egloos.zum.com/rousalome/v/10016801