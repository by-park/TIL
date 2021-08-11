# 2021-08-11 (printf formatting)

### print formatting 정리

%d, %c, %p, %x, %X, %o, %s, %u, %ld, %lld, %lu, %llu, %f, %lf, %e, %E

https://reakwon.tistory.com/169



### 실수형 빈 공간 0으로 채우기

소수점을 표기하고자 할 때 `printf("[%6.f.%6.f] %s\n", characteristic, mantissa, __func__);` 이렇게 썼는데 그러면 뒤에 자리가 0인 경우에 이상하게 출력된다.

[123.   111] 이런 식이다. 이걸 [123.000111] 이렇게 바꾸고 싶었다.



빈 자리에 0을 넣으려면 숫자 앞에 0을 쓰면 된다.

`printf("%.08f\n", 1.23);` 이런식으로 쓰면 된다. 그래서 위의 코드를 다음과 같이 변경하였다.

`printf("[%6.f.%06.f] %s\n", characteristic, mantissa, __func__);`