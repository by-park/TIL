# 2023-12-09 Compile error (Cast size)

## cast from pointer to integer of different size

https://jchern.tistory.com/25

> 32비트 머신에서는 warning 없이 잘 컴파일 되던 것이 64비트 머신으로 갔더니
>
> cast from pointer to integer of difference size
>
> 이런 메세지가 뜬다..
>
> unsigned short int* 형을 unsigned int 형으로 바꾸는 부분에서 발생한 warning인데
>
> sizeof로 확인해보니
>
> unsigned short int* 가 64비트에서는 8byte 자료형이고, unsigned int는 4byte 자료형이라서 발생하는 문제였다.
>
> 그래서 unsigned int 를 unsigned long int 로 수정했더니 warning이 사라졌다.

https://blog.naver.com/yababies/220761796716

> 그동안 아무문제 없이 컴파일 잘되던 소스에서 "cast from pointer to integer of different size"와 같인 waring이 떴다.
>
> 왜?
> 서버를 옮긴것도 아닌데 갑자기 왜 그럴까?
>
> 64 bit 머신에서 -m32 옵션을 통해 32 bit로 컴파일 하고 있고 한참 됐는데 왜 갑자기 이런 메시지가...
> -> 32 bit에서 64 bit로 변경한 것이면 포인터의 크가가 달라서 pointer 쓰는 모든 부분에서 날 것이다.
>   32bit pointer 크기 : 4bytes
>   64bit pointer 크기 : 8bytes
>   예를 들어 int * 였으면 long int * 등으로 변경하면 별 문제 없이 해결된다.
>
> 이번 경우는 생각해 보니 얼마전에 gcc 컴파일러를 upgrade 했었다.
> gcc 4.1.2 버전에서 4.4.7 버전으로 테스트 할 게 있어서 upgrade를 했던게 생각이 났다.
> 아마 정책이 더 강화됐나 보다.
> 분명히 어딘가 size를 잘못해서 쓴 부분이 있었을 것이다.
>
> 문제가 되는 코드는 아래 부분이었다.
> struct에서 특정 필드의 위치를 가져와 주는 macro에서 발생했었다.
>
> \#define  POS(type, field)  ((short) &((type *) 0)->field)
>
> 32bit 머신에서 pointer의 크기는 4bytes이고 그렇기에 (short) & .... 이부분 pointer를 short type으로 casting하는 부분에서 warning이 발생했던 것이다.
>
> 근데 왜?
> 32bit pointer 값을 32bit 크기가 아닌 명시적으로 16bit 크기인 short으로 casting하고 싶은데 왜 warning이 뜨는 건지는 모르겠다.
> 그냥 정책이 강화됐나 보다 하고 (int) & .... 로 고쳤고 모든 warning이 사라졌다.



## cast to pointer from integer of different size

https://stackoverflow.com/questions/19527965/cast-to-pointer-from-integer-of-different-size-pthread-code



내 문제는 cast to pointer 였는데, 검색하면 cast from pointer 가 훨씬 많이 나온다.

결국 해결책은 pointer 로 변환하기 전에 값을 unsigned long long 으로 변환해서 64 bit 자료형이 되도록 맞춰주는 것이었다.

그 후에 (volatile unsigned int *) 등 원하는 형태로 변환해서 주소를 접근하면 된다.

이렇게 해야했던 이유는 주소값이 0x00000000 같은 형태였고, 여기에서 offset 으로 정해진 주소들을 접근해야했기 때문에 미리 * 로 변환하고 + offset 을 하는 형태를 할 수는 없었다. (그러면 배열 접근처럼 되어버리기 때문에)