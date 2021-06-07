# 2021-06-07 (Little Endian)

```c
#include <stdio.h>

int main()
{
    unsigned int args[2] = {0, 0};
    unsigned long long time = 0x12345678654321;
    *(unsigned long long *)args = time;
    printf("0x%x 0x%x\n", args[0], args[1]);
    return 0;
}
```

위의 결과를 출력하면 `0x78654321 0x123456` 이 나온다.

32 bit 2개를 64 bit에 넣으면 순서가 반대로 들어간다.

![Little-Endian](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Little-Endian.svg/200px-Little-Endian.svg.png)

보통 레지스터를 그릴 때는 0 bit 자리를 오른쪽에 그리지만, 실제로 메모리에 들어갈 때는 0 bit 를 왼쪽으로 생각하면 된다.

0x0 0x4 0x8 0xc 이런 식으로 메모리가 있으면, 안에 값은 뒤집어서 들어가게 된다. 0x1234 로 넣으면

0x34 0x12 이런 식으로 들어가게 된다. 그래서 0x12345678654321 을 넣으면 맨 뒤에 0x21 이 맨 앞으로 오게 된다.