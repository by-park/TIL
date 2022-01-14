# 2022-01-14 (Linux S2R step debug)

Linux S2R 디버깅 방법 중,

CONFIG_PM_DEBUG 를 enable 로 하고 (default y)

`echo devices > /sys/power/pm_test` 를 하면, TEST_DEVICES 라고 표시된 곳 까지 실행된다.

위의 명령어를 입력한 후에, `echo mem > /sys/power/state` 를 입력하였다.

(+ 단계별로 나오는 devices 들의 이름을 확인하기 위해 맨 처음에 `echo 1 > /sys/power/pm_print_times` 도 입력해주었다.)

이걸 수정해서, 더 다양한 단계별로 테스트 가능한 패치를 만들어보았다.

![img](https://01.org/sites/default/files/users/u32403/linuxpmframework.png)

[그림 출처] https://01.org/blogs/qwang59/2018/how-achieve-s0ix-states-linux



kernel version: linux-5.9.11

```c
diff --git a/kernel/power/main.c b/kernel/power/main.c
index 0aefd6f..aa73532 100644
--- a/kernel/power/main.c
+++ b/kernel/power/main.c
@@ -232,7 +232,11 @@ static const char * const pm_tests[__TEST_AFTER_LAST] = {
        [TEST_CORE] = "core",
        [TEST_CPUS] = "processors",
        [TEST_PLATFORM] = "platform",
+       [TEST_PLATFORM_LATE] = "platform_late",
+       [TEST_PLATFORM_NOIRQ] = "platform_noirq",
        [TEST_DEVICES] = "devices",
+       [TEST_DEVICES_LATE] = "devices_late",
+       [TEST_DEVICES_NOIRQ] = "devices_noirq",
        [TEST_FREEZER] = "freezer",
 };

diff --git a/kernel/power/power.h b/kernel/power/power.h
index 24f12d5..f75e1d7 100644
--- a/kernel/power/power.h
+++ b/kernel/power/power.h
@@ -230,7 +230,11 @@ enum {
        TEST_CORE,
        TEST_CPUS,
        TEST_PLATFORM,
+       TEST_PLATFORM_LATE,
+       TEST_PLATFORM_NOIRQ,
        TEST_DEVICES,
+       TEST_DEVICES_LATE,
+       TEST_DEVICES_NOIRQ,
        TEST_FREEZER,
        /* keep last */
        __TEST_AFTER_LAST
diff --git a/kernel/power/suspend.c b/kernel/power/suspend.c
index 32391ac..f6b30f7 100644
--- a/kernel/power/suspend.c
+++ b/kernel/power/suspend.c
@@ -399,19 +399,33 @@ static int suspend_enter(suspend_state_t state, bool *wakeup)
                pr_err("late suspend of devices failed\n");
                goto Platform_finish;
        }
+
+       if (suspend_test(TEST_DEVICES_LATE))
+               goto Devices_early_resume;
+
        error = platform_suspend_prepare_late(state);
        if (error)
                goto Devices_early_resume;

+       if (suspend_test(TEST_PLATFORM_LATE))
+               goto Platform_early_resume;
+
        error = dpm_suspend_noirq(PMSG_SUSPEND);
        if (error) {
                pr_err("noirq suspend of devices failed\n");
                goto Platform_early_resume;
        }
+
+       if (suspend_test(TEST_DEVICES_NOIRQ))
+               goto Devices_wake;
+
        error = platform_suspend_prepare_noirq(state);
        if (error)
                goto Platform_wake;

+       if (suspend_test(TEST_PLATFORM_NOIRQ))
+               goto Platform_wake;
+
        if (suspend_test(TEST_PLATFORM))
                goto Platform_wake;

@@ -454,6 +468,8 @@ static int suspend_enter(suspend_state_t state, bool *wakeup)

  Platform_wake:
        platform_resume_noirq(state);
+
+ Devices_wake:
        dpm_resume_noirq(PMSG_RESUME);

  Platform_early_resume:
(END)
```