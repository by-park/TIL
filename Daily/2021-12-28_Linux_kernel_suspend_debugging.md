# 2021-12-28 (Linux kernel suspend debugging)

Linux kernel 에서 아래 명령어로 suspend to ram 을 실행하려고 했는데, suspend to idle 이 실행되었다.

```shell
$ echo mem > /sys/power/state
```

`make menuconfig` 를 해봤는데 CONFIG_PM_SLEEP_DEBUG 등 SLEEP 관련 config들은 다 y 로 켜져있는 것을 확인하였다. 디버깅 메세지도 더 출력되게 하려고 아래 명령어로 출력 레벨 조정을 해주었으나, 변함이 없었다.

```shell
$ echo 8 > /proc/sys/kernel/printk
```

https://www.kernel.bz/boardPost/118679/17



mem_sleep_default 가 dt 에 선언되어있지 않아서 default 값은 변경되지 않았다.

// kernel/power/suspend.c

```c
static int __init mem_sleep_default_setup(char *str)
{
        suspend_state_t state;

        for (state = PM_SUSPEND_TO_IDLE; state <= PM_SUSPEND_MEM; state++)
                if (mem_sleep_labels[state] &&
                    !strcmp(str, mem_sleep_labels[state])) {
                        mem_sleep_default = state;
                        break;
                }

        return 1;
}
__setup("mem_sleep_default=", mem_sleep_default_setup);
```



https://www.kernel.org/doc/Documentation/power/states.txt

`/sys/power/mem_sleep` 에는 deep 으로 라벨이 들어가있어야하고, `/sys/power/state` 에 mem 을 치면 s2r에 진입한다.



알고 보니 suspend psci 의 return 값이 error 였기 때문이었다. xen hypervisor 에서 suspend psci 를 추가해줬더니 (mainline 에 없는 듯하다) `/sys/power/mem_sleep` 에 deep 이 추가되었다.

// drivers/firmware/psci/psci.c

```c
static void __init psci_init_system_suspend(void)
{
        int ret;

        if (!IS_ENABLED(CONFIG_SUSPEND))
                return;

        ret = psci_features(PSCI_FN_NATIVE(1_0, SYSTEM_SUSPEND));

        if (ret != PSCI_RET_NOT_SUPPORTED)
                suspend_set_ops(&psci_suspend_ops);
}

```



위의 설정 후, freeze task 부분에서 더 이상 console 이 나오지 않아서 비슷한 증상이 있는지 검색하다가 아래의 질문 답변을 보게 되었다. (결과적으로 kernel console 이 늦게 serial 에 print 되는데, 그 전에 s2r path 에서 error 발생해서 뻗어버린 것이었다. kernel log 를 저장해 둔 램 공간을 보니까 어디까지 진행되었는지 알 수 있었다.) 

https://askubuntu.com/questions/475749/ubuntu-stuck-suspending-console-while-going-to-sleep

본 질문 답변에서 `no_console_suspend` 를 설정해보라는 말이 있어서 확인해봤는데, dt 에 이미 1로 설정해두었다. 그러면 suspend 때도 sleep 이 print 된다.



dt 말고 소스코드로 설정할 수도 있다. 아래에 console_suspend_enabled를 설정할 수 있다.

// kernel/printk/printk.c

```c
bool console_suspend_enabled = true;
EXPORT_SYMBOL(console_suspend_enabled);

static int __init console_suspend_disable(char *str)
{
        console_suspend_enabled = false;
        return 1;
}
__setup("no_console_suspend", console_suspend_disable);
```



suspend to ram 동작 확인할 때 사용하기 좋은 명령어가 또 있다.

```shell
$ echo 1 > /sys/power/pm_print_times
```

https://javamana.com/2021/06/20210620135914555r.html



Linux kernel sleep wake up debugging 에 도움이 되는 정리 글

https://javamana.com/2021/06/20210620135914555r.html



INTEL 의 suspend 디버깅 참고용 게시글!!

https://01.org/blogs/rzhang/2015/best-practice-debug-linux-suspend/hibernate-issues







### SWAP 코드

프로그래머가 몰랐던 멀티코어 CPU 이야기

http://www.yes24.com/Product/Goods/3858484

![상세 이미지 1](http://image.yes24.com/momo/TopCate91/MidCate05/9048197.jpg)

temp 를 사용하지 않고 XOR 연산을 사용하는 swap 방법이 있다.

이에 대한 추가적인 설명은 여기에서 볼 수 있다.

https://minjang.github.io/2016/11/28/xor-temp-swap/



### OS 프로젝트 정리

다양한 오픈소스 등 Operating System 프로젝트들을 한 번에 볼 수 있다.

https://github.com/jubalh/awesome-os