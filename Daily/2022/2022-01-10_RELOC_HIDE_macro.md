# 2022-01-10 (RELOC_HIDE macro)

Xen hypervisor 코드를 보다가 RELOC_HIDE 매크로를 보았는데, Linux kernel 에서도 동일한 이름으로 사용되고 있었다. pointer 를 casting 해서 접근하고자 할 때, 원래 데이터 크기보다 더 넘어가면 컴파일러에서 허용하지 않는 경우가 있었다고 한다. 그래서 컴파일러가 모르게 포인터 최적화를 못하도록 방해하는 용도이다.



http://egloos.zum.com/studyfoss/v/5374731

```c
include/linux/compiler-gcc.h:
#define RELOC_HIDE(ptr, off)                    \
  ({ unsigned long __ptr;                       \
    __asm__ ("" : "=r"(__ptr) : "0"(ptr));      \
    (typeof(ptr)) (__ptr + (off)); })
```



> 하지만 ptr이 가리키는 원래의 객체의 크기를 넘어가는 영역에 접근하기 위해서
> (type cast하여) 강제로 off 값을 더하게 되면 컴파일러가 잘못된 접근을 감지하고
> 컴파일 시에 의도하지 않은 코드를 생성하게 될 가능성이 있다.
>
> C 표준에서는 포인터를 통해 지정된 크기를 넘어서 접근하는 경우는 undefined behavior로 명시하고 있다.
> 실제로 PPC64 아키텍처에서 gcc 4.1 이전의 버전을 사용하는 경우 이러한 문제가 발생했다고 하는데
> 이 경우 ptr 값을 unsigned long 타입으로 cast하더라도 원래의 포인터 정보가 남아있어서
> (copy propagation에 의해?) 동일한 문제가 발생한 것 같다.