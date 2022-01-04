# 2021-04-16 (Operator precedence)

연산자 우선순위와 관련해서 생각한 것과 다른 결과가 나오는 코드를 알게 되었다.

```c++
#include <stdio.h>

int main(void)
{
	int x = 0;
	int y = 0;
	
	if (x == x++) {
		printf("Yes\n");
	} else{
		printf("No\n");
	}
	
	if (y == ++y) {
		printf("Success\n");
	}else{
		printf("Fail\n");
	}
	
	return 0;
}
```

결과는 증감 연산자의 우선순위가 높아서 Yes, Fail 이 나와야할 것 같지만 No, Success가 나온다.