# 2024-03-26 (GCC Error Debugging)

**에러 로그**

파일명:라인:undefined reference to \'함수이름1\'

파일명:라인:(.text.함수이름2+0x38)relocation truncated to fit: R_AARCH64_CALL26 against undefined symbol \'함수이름1\' 

https://stackoverflow.com/questions/10486116/what-does-this-gcc-error-relocation-truncated-to-fit-mean



**에러 해결 방법**

아래와 같은 코드에서 문제가 발생하였다.

```c
inline void set_bit(unsigned int *register_address, unsigned int outfield)
{
    *((volatile unsigned int *)((unsigned int)register_address + 0xC000)) = outfield;
}
```

register_address 를 포인터로 변환하려면 값이 64 bit인 unsigned long long 이어야해서 에러를 일으킨 것인가 의심했는데 아니었다. set_bit 를 호출하는 함수도 inline 이었는데, inline 을 중첩해서 사용했더니 이런 문제가 발생하였다. inline 하나를 제거하고 해결되었다.



참고)

포인터를 변환하지 않고 0xC000 을 더했다가 값이 지나치게 커져버리는 오류가 있었다. 형변환을 세심하게 봐야한다.

