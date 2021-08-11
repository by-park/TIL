# 2021-08-10 (GCOV & Optimization)

GCOV로 kernel 코드 커버리지를 돌려보면, 실행된 함수인데, 실행되지 않은 것처럼 잡히는 경우가 있다. O2로 최적화가 된 경우, 빌드시에 간단한 코드는 생략되거나, 함수가 inline화 되면서 해당 라인이 제대로 인식되지 않게 되는 것이다. O0 로 빌드해서 이런 최적화를 풀어서 확인해볼 수 있다.

(Discussion 참고 https://gcc.gnu.org/onlinedocs/gcc/Gcov-and-Optimization.html)



### GCC Optimization Options

아래는 GCC Optimization Options 종류이다.

https://wiki.kldp.org/wiki.php/GccOptimizationOptions

거의 동일하게, GCC 최적화 옵션 종류에 대한 설명이다.

https://blog.daum.net/jjiyong/16779883

자세한 최적화 옵션 설명이 있다. (공식 document)

https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html



### Optimization Disable

Optimization 없애는 방법

1. 함수에 다음 적용

```c
void __attribute__((optimize("O0"))) func(void) {
}
```

2. 코드 범위 내에 다음 적용

```c
#pragma GCC push_options 
#pragma GCC optimize ("O0") 
//Write your code 
#pragma GCC pop_options
```

https://sonseungha.tistory.com/522



### Q&A about GCOV coverage problem

관련 Q&A 모음

Incorrect coverage in C++ header files #164

https://github.com/r-lib/covr/issues/164



### CTC++

최적화 관련 커버 안 되는 문제 해결을 위해 gcov 대신 CTC++을 쓸 수도 있다고 한다.

https://www.verifysoft.com/en_ctcpp_vs_gcov.html