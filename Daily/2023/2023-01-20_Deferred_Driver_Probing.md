# 2023-01-20 (Deferred Driver Probing)

Linux kernel driver probe 과정 중 문제가 생겼을 때 순서를 뒤로 미룰 수 있는 방법

drivers/thermal/ti-soc-thermal/ti-thermal-common.c (linux 5.9.11)

```c
        data->policy = cpufreq_cpu_get(0);
        if (!data->policy) {
                pr_debug("%s: CPUFreq policy not found\n", __func__);
                return -EPROBE_DEFER;
        }
```

include/linux/errno.h (linux 5.9.11)

```c
/*
 * These should never be seen by user programs.  To return one of ERESTART*
 * codes, signal_pending() MUST be set.  Note that ptrace can observe these
 * at syscall exit tracing, but they will never be left for the debugged user
 * process to see.
 */
#define ERESTARTSYS     512
#define ERESTARTNOINTR  513
#define ERESTARTNOHAND  514     /* restart if no handler.. */
#define ENOIOCTLCMD     515     /* No ioctl command */
#define ERESTART_RESTARTBLOCK 516 /* restart by calling sys_restart_syscall */
#define EPROBE_DEFER    517     /* Driver requests probe retry */
#define EOPENSTALE      518     /* open found a stale dentry */
#define ENOPARAM        519     /* Parameter not supported */
```

[참고] https://lwn.net/Articles/450460/