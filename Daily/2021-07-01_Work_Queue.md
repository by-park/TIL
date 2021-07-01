# 2021-07-01 (Work Queue)

리눅스에서 인터럽트 초반부 / 후반부 처리를 나누는데 사용되는 워크큐 개념이 있다.

인터럽트는 최대한 빨리 처리해야하므로, 인터럽트를 빨리 처리해서 인터럽트 컨텍스트를 최대한 빨리 벗어나고자 워크큐에 등록해놓고 끝내는 방식이다. 커널에서 워크큐에 등록된 워크들을 처리하기 때문에 처리되는 시점이 조금 늦을 수는 있다.

```c
struct work_struct my_work;
```

멤버 변수로 넣든, 위와 같이 work_struct 구조체가 필요하다.

또한 work queue 로 등록할 함수가 있어야한다.

```c
static void my_work_func(struct work_struct *work)
{}
```

그리고 driver의 probe 함수에서 INIT을 시켜야한다.

```c
int driver_probe(platform_device... etc)
{
    INIT_WORK(&my_work, my_work_func);
}
```

그리고 인터럽트 처리시 불리는 핸들러에서 work 실행을 요청하면 된다.

```c
int interrupt_handler(etc...)
{
    schedule_work(&my_work);
    return IRQ_HANDLED;
}
```

