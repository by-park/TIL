# 2021-11-04 (cpu idle time)

### cpu idle 확인

```shell
$ perf top
```



irq handler 내부에 udelay 함수를 썼는데, delay 함수는 찾아보면 cpu 를 점유하는 busy waiting 방식이다. 그러나 irq는 저 명령어에서 detect 하는 대상이 아니라, cpu idle 비율에 반영되지 않았다. (context 가 다르기 때문인 것 같다. 인터럽트 루틴을 제외한 커널 동작만 반영하는 듯 하다.)

즉, `perf top` 에는 irq 핸들링은 detect 되지 않음

참고로 `perf irq report` 명령어가 있는 거로 봐서 irq 를 확인하는 명령어도 따로 있는 것 같다.



```shell
$ htop
```

`htop` 명령어로 치면 irq handler 에서 걸리는 시간까지 확인할 수 있다.



그런데 `htop`을 쳐도 실제 hypervisor에서 EL2 level 에서 cpu idle time 을 측정한 (EL1 wfi 진입할 때 EL2 호출된 함수 내에서 계산된다) 결과와 다르게 나왔는데, timer irq handler 가 kernel scheduler 가 스케줄링 하는 것보다 빨리 동작하는 것 같은데, 그게 영향을 줄 수도 있을 것 같다.

