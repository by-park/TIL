# 2021-10-08 (SW Optimization)

### 프로그램 최적화

1. 비효율적인 함수 호출

```c
void lower(char *s) {
    for (size_t i = 0; i < strlen(s); i++) {
        if (s[i] >= 'A' && s[i] <= 'Z') {
            s[i] -= ('A' - 'a');
}
```

strlen 함수가 for 문이 반복될 때마다 매번 호출된다. 컴파일러가 함수까지 최적화해주지는 않는다.

2. 비효율적인 메모리 참조

```c
void sum_rows1(double *a, double *b, long n) {
    long i, j;
    for (i = 0; i < n; i++) {
        b[i] = 0;
        for (j = 0; j < n; j++)
            b[i] += a[i*n + j];
    }
}

// 위 코드의 어셈블리
.L3:
    vmovsd  (%rax), %xmm0           ; FP load
    vaddsd  (%rdi), %xmm0, %xmm0    ; FP add
    vmovsd  %xmm0, (%rax)           ; FP store
    addq    $8, %rdi
    cmpq    %rdx, %rdi
    jne     .L3
```

루프마다 메모리 액세스를 3번씩한다.

아래와 같이 메모리 영역이 겹치는 경우가 있을 수 있어서 매번 로드, 저장을 다시 하도록 만들었다고 한다. (memory aliasing) 해결방법은 지역변수를 사용하는 것이다.

```c
double B[3] = A + 3;
```

3. Loop unrolling

전체 연산을 반으로 나눠서 루프 내부에서 연산을 2번 하도록 한다.

> 이 최적화의 기본 아이디어는 루프를 실행하는데 드는 오버헤드를 줄이는데 있다. 루프는 loop condition을 검사하고 jump하는 오버헤드가 있기 때문에 루프 수를 줄이면 branching을 줄이고 cache hit ratio를 높여주는 효과가 있다.

```c
for (i = 0; i < 100; i++)
  g ();

// 아래와 같이 변경

for (i = 0; i < 100; i += 2)
{
  g ();
  g ();
}
```

4. 재결합 변환(Reassociation Transform)

병렬성 향상을 위하여 코드를 변경한다. 첫번째 곱 연산이 끝나기 전까지 두번째 곱 연산을 할 수 없다가 괄호 위치를 바꿔주고 나서 첫번째 곱 연산을 하는 동안 두 번째 곱연산을 할 수 있게 되었다.

```c
for (i = 0; i < limit; i += 2) {
    acc = (acc OP data[i]) OP data[i+1];
}

// 아래와 같이 변경

for (i = 0; i < limit; i += 2) {
    acc = acc OP (data[i] OP data[i+1]);
}
```



5. 병렬 누산기(Separate Accumulators)

지역변수를 명시적으로 써서 순차적 의존성을 깬다.

```c
void combine6(vec_ptr v, data_t *dest) {
    long i;
    long length = vec_length(v);
    long limit = length - 1;
    data_t *data = get_vec_start(v);
    data_t acc0 = IDENT;
    data_t acc1 = IDENT;

    for (i = 0; i < limit; i += 2) {
        acc0 = acc0 OP data[i];
        acc1 = acc1 OP data[i+1];
    }

    for (; i < length; i++) {
        acc0 = acc0 OP data[i];
    }
    *dest = acc0 OP acc1;
}
```





### 출처

전체1: https://koyo.kr/post/csapp-optimizing-program-1/

전체2: https://koyo.kr/post/csapp-optimizing-program-2/

Loop unrolling: https://skyul.tistory.com/36