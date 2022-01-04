# 2021-08-24 (Make my own printk)

printf 처럼 포맷팅을 이용하여 사용자가 char 배열에 저장하게 하려고 함수를 만들었다.

```c
int my_printk(const char *fmt, ...)
{
    va_list args;
    char buf[30] = {0,};
    int ret;
    
    va_start(args, fmt);
    ret = vsnprintf(buf, sizeof(buf), fmt, args);
    va_end(args);
    
    // buf를 실제 메모리에 저장하는 코드
    
    return ret;
}
```

이렇게 함수를 만들면 `my_printk("I want to log anything! %d\n", 1234);` 과 같이 포맷팅을 이용할 수 있다.

`va_start()` 및 `vsnprintf` 등을 합쳐서 이미 `snprintf` 가 있길래 이걸 이용하려고 하였지만, 함수의 인자로 넘어온 fmt를 그대로 넘긴다고 해서 동작하지는 않았다. args로 들어온 값이 잘못된 값을 가리키고 있었다. 아마 fmt 를 다시 함수 인자로 넘겨줄 때 그것만 스택에 새로 넣기 때문에 포맷팅용 인자는 같이 넘어가지 못한 것 같다.