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

\+ INTEL 의 linux 관련 document 링크

https://01.org/linuxgraphics/documentation/code-documentation



### Suspend To RAM path

suspend 명령어 입력

```shell
$ echo mem > /sys/power/state
```

해당 sysfs 처리하는 함수로 이동

// kernel/power/main.c

```c
static ssize_t state_store(struct kobject *kobj, struct kobj_attribute *attr,
                           const char *buf, size_t n)
{
        suspend_state_t state;
        int error;

        error = pm_autosleep_lock();
        if (error)
                return error;

        if (pm_autosleep_state() > PM_SUSPEND_ON) {
                error = -EBUSY;
                goto out;
        }

        state = decode_state(buf, n);
        if (state < PM_SUSPEND_MAX) {
                if (state == PM_SUSPEND_MEM)
                        state = mem_sleep_current;

                error = pm_suspend(state);
        } else if (state == PM_SUSPEND_MAX) {
                error = hibernate();
        } else {
                error = -EINVAL;
        }

 out:
        pm_autosleep_unlock();
        return error ? error : n;
}
```

위의 함수 중 pm_suspend() 로 진입

※ 인자로 들어가는 mem_sleep_current 값은 psci driver에  suspend_ops 가 등록되어있으면 deep 으로 들어가고, 아니면 idle 이 된다.

// kernel/power/suspend.c

```c
int pm_suspend(suspend_state_t state)
{
        int error;

        if (state <= PM_SUSPEND_ON || state >= PM_SUSPEND_MAX)
                return -EINVAL;

        pr_info("suspend entry (%s)\n", mem_sleep_labels[state]);
        error = enter_state(state);
        if (error) {
                suspend_stats.fail++;
                dpm_save_failed_errno(error);
        } else {
                suspend_stats.success++;
        }
        pr_info("suspend exit\n");
        return error;
}
```

여기서 아래 로그가 찍힘

```shell
PM: suspend entry (deep)
```

그리고 enter_state() 함수로 진입

// kernel/power/suspend.c

```c
static int enter_state(suspend_state_t state)
{
        int error;

        trace_suspend_resume(TPS("suspend_enter"), state, true);
        if (state == PM_SUSPEND_TO_IDLE) {
#ifdef CONFIG_PM_DEBUG
                if (pm_test_level != TEST_NONE && pm_test_level <= TEST_CPUS) {
                        pr_warn("Unsupported test mode for suspend to idle, please choose none/freezer/devices/platform.\n");
                        return -EAGAIN;
                }
#endif
        } else if (!valid_state(state)) {
                return -EINVAL;
        }
        if (!mutex_trylock(&system_transition_mutex))
                return -EBUSY;

        if (state == PM_SUSPEND_TO_IDLE)
                s2idle_begin();

        if (sync_on_suspend_enabled) {
                trace_suspend_resume(TPS("sync_filesystems"), 0, true);
                ksys_sync_helper();
                trace_suspend_resume(TPS("sync_filesystems"), 0, false);
        }

        pm_pr_dbg("Preparing system for sleep (%s)\n", mem_sleep_labels[state]);
        pm_suspend_clear_flags();
        error = suspend_prepare(state);
        if (error)
                goto Unlock;

        if (suspend_test(TEST_FREEZER))
                goto Finish;

        trace_suspend_resume(TPS("suspend_enter"), state, false);
        pm_pr_dbg("Suspending system (%s)\n", mem_sleep_labels[state]);
        pm_restrict_gfp_mask();
        error = suspend_devices_and_enter(state);
        pm_restore_gfp_mask();

 Finish:
        events_check_enabled = false;
        pm_pr_dbg("Finishing wakeup.\n");
        suspend_finish();
 Unlock:
        mutex_unlock(&system_transition_mutex);
        return error;
}
```

sync_on_suspend_enabled 는 config 를 확인하는 건데, 이건 default 가 꺼져있기 때문에, 위의 코드에서 ksys_sync_helper() 함수를 진입한다. 

(참고: CONFIG_SUSPEND_SKIP_SYNC)

```c
kernel/power/main.c:bool sync_on_suspend_enabled = !IS_ENABLED(CONFIG_SUSPEND_SKIP_SYNC);
```

ksys sync 작업 후에는 아래와 같은 메시지가 출력된다.

```shell
Filesystems sync: 0.004 seconds
```

그러면 그 다음은 suspend_prepare() 함수가 실행된다.

// kernel/power/suspend.c

```c
static int suspend_prepare(suspend_state_t state)
{
        int error;

        if (!sleep_state_supported(state))
                return -EPERM;

        pm_prepare_console();

        error = pm_notifier_call_chain_robust(PM_SUSPEND_PREPARE, PM_POST_SUSPEND);
        if (error)
                goto Restore;

        trace_suspend_resume(TPS("freeze_processes"), 0, true);
        error = suspend_freeze_processes();
        trace_suspend_resume(TPS("freeze_processes"), 0, false);
        if (!error)
                return 0;

        suspend_stats.failed_freeze++;
        dpm_save_failed_step(SUSPEND_FREEZE);
        pm_notifier_call_chain(PM_POST_SUSPEND);
 Restore:
        pm_restore_console();
        return error;
}
```

이 안에서는 suspend_freeze_processes() 를 수행하면서 진행 중이던 프로세스들을 멈춘다. 그리고 아래 메시지가 출력된다.

```shell
Freezing user space processes ... (elapsed 0.006 seconds) done.
OOM killer disabled.
Freezing remaining freezable tasks ... (elapsed 0.012 seconds) done.
```

그 후에 suspend_devices_and_enter() 로 진입한다.

이 함수 안에서 linux kernel suspend path 에 대해 보면 나오는 아래 device suspend & resume 함수를 볼 수 있다.

![S0ix via Linux system PM framework](https://01.org/sites/default/files/resize/users/u32403/linuxpmframework-680x400.png)

https://01.org/blogs/qwang59/2018/how-achieve-s0ix-states-linux

![image](https://www.programmerall.com/images/592/6c/6cd3fa7b404a840a376d3124253133b8.png)

https://www.programmerall.com/article/7173138815/

// kernel/power/suspend.c

```c
int suspend_devices_and_enter(suspend_state_t state)
{
        int error;
        bool wakeup = false;

        if (!sleep_state_supported(state))
                return -ENOSYS;

        pm_suspend_target_state = state;

        if (state == PM_SUSPEND_TO_IDLE)
                pm_set_suspend_no_platform();

        error = platform_suspend_begin(state);
        if (error)
                goto Close;

        suspend_console();
        suspend_test_start();
        error = dpm_suspend_start(PMSG_SUSPEND);
        if (error) {
                pr_err("Some devices failed to suspend, or early wake event detected\n");
                goto Recover_platform;
        }
        suspend_test_finish("suspend devices");
        if (suspend_test(TEST_DEVICES))
                goto Recover_platform;

        do {
                error = suspend_enter(state, &wakeup);
        } while (!error && !wakeup && platform_suspend_again(state));

 Resume_devices:
        suspend_test_start();
        dpm_resume_end(PMSG_RESUME);
        suspend_test_finish("resume devices");
        trace_suspend_resume(TPS("resume_console"), state, true);
        resume_console();
        trace_suspend_resume(TPS("resume_console"), state, false);

 Close:
        platform_resume_end(state);
        pm_suspend_target_state = PM_SUSPEND_ON;
        return error;

 Recover_platform:
        platform_recover(state);
        goto Resume_devices;
}
```

suspend_enter() 함수를 진입하면 다음과 같이 실행된다.

platform_suspend_prepare

dpm_suspend_late

platform_suspend_prepare_late

dpm_suspend_noirq

platform_suspend_prepare_noirq



// kernel/power/suspend.c

```c
static int suspend_enter(suspend_state_t state, bool *wakeup)
{
        int error;

        error = platform_suspend_prepare(state);
        if (error)
                goto Platform_finish;

        error = dpm_suspend_late(PMSG_SUSPEND);
        if (error) {
                pr_err("late suspend of devices failed\n");
                goto Platform_finish;
        }
        error = platform_suspend_prepare_late(state);
        if (error)
                goto Devices_early_resume;

        error = dpm_suspend_noirq(PMSG_SUSPEND);
        if (error) {
                pr_err("noirq suspend of devices failed\n");
                goto Platform_early_resume;
        }
        error = platform_suspend_prepare_noirq(state);
        if (error)
                goto Platform_wake;

        if (suspend_test(TEST_PLATFORM))
                goto Platform_wake;

        if (state == PM_SUSPEND_TO_IDLE) {
                s2idle_loop();
                goto Platform_wake;
        }

        error = suspend_disable_secondary_cpus();
        if (error || suspend_test(TEST_CPUS))
                goto Enable_cpus;

        arch_suspend_disable_irqs();
        BUG_ON(!irqs_disabled());

        system_state = SYSTEM_SUSPEND;

        error = syscore_suspend();
        if (!error) {
                *wakeup = pm_wakeup_pending();
                if (!(suspend_test(TEST_CORE) || *wakeup)) {
                        trace_suspend_resume(TPS("machine_suspend"),
                                state, true);
                        error = suspend_ops->enter(state);
                        trace_suspend_resume(TPS("machine_suspend"),
                                state, false);
                } else if (*wakeup) {
                        error = -EBUSY;
                }
                syscore_resume();
        }

        system_state = SYSTEM_RUNNING;

        arch_suspend_enable_irqs();
        BUG_ON(irqs_disabled());

 Enable_cpus:
        suspend_enable_secondary_cpus();

 Platform_wake:
        platform_resume_noirq(state);
        dpm_resume_noirq(PMSG_RESUME);

 Platform_early_resume:
        platform_resume_early(state);

 Devices_early_resume:
        dpm_resume_early(PMSG_RESUME);

 Platform_finish:
        platform_resume_finish(state);
        return error;
}
```

그리고 위에서 suspend_disable_secondary_cpus() 함수를 호출하면 CPU들이 꺼지면서 아래와 같이 출력된다.

```shell
Disabling non-boot CPUs ...
CPU1: shutdown
psci: CPU1 killed (polled 0 ms)
CPU2: shutdown
// 등등 CPU 개수별로
```

그리고 suspend 에 진입했다가 바로 켜지게 되면 아래로 이어서 진행이 된다.

```shell
Enabling non-boot CPUs ...
Detected VIPT I-cache on CPU1
GICv3: CPU1: found redistributor 1 region 0:0x00000000388a0000
CPU1: Booted secondary processor 0x0000000001 [0x410fd034]
CPU1 is up
// CPU 들 켜지고
// driver 들 resume callback 함수 출력하고
OOM killer enabled.
Restarting tasks ... done.
PM: suspend exit
```

(전체 shell 출력 결과 참고)

https://community.nxp.com/t5/i-MX-Processors/i-MX8MM-Detect-Wakeup-Cause/m-p/1200351



Linux Kernel S2R testing 사용 방법 공식 문서

https://www.kernel.org/doc/html/latest/power/drivers-testing.html

https://www.kernel.org/doc/html/latest/power/basic-pm-debugging.html

```shell
echo freezer > /sys/power/pm_test
echo devices > /sys/power/pm_test
echo platform > /sys/power/pm_test
echo processors > /sys/power/pm_test
echo core > /sys/power/pm_test
```

위와 같이 레벨을 지정해주면, 아래와 같이 suspend_test() 함수에서 멈추게 된다.

// kernel/power/suspend.c

```c
static int suspend_enter(suspend_state_t state, bool *wakeup)
{
        int error;
// 생략
        error = platform_suspend_prepare_noirq(state);
        if (error)
                goto Platform_wake;

        if (suspend_test(TEST_PLATFORM))
                goto Platform_wake;

        if (state == PM_SUSPEND_TO_IDLE) {
                s2idle_loop();
                goto Platform_wake;
        }

        error = suspend_disable_secondary_cpus();
        if (error || suspend_test(TEST_CPUS))
                goto Enable_cpus;
```

단, 이 함수를 사용하기 위해서는 CONFIG_PM_DEBUG 를 y로 변경하고 커널을 빌드해야한다.

// kernel/power/suspend.c

```c
static int suspend_test(int level)
{
#ifdef CONFIG_PM_DEBUG
        if (pm_test_level == level) {
                pr_info("suspend debug: Waiting for %d second(s).\n",
                                pm_test_delay);
                mdelay(pm_test_delay * 1000);
                return 1;
        }
#endif /* !CONFIG_PM_DEBUG */
        return 0;
}
```



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