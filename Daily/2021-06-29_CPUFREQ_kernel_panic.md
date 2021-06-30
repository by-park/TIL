# 2021-06-29 (CPUFREQ kernel panic)

CPUFREQ driver에서 kernel panic이 발생하여, 이와 관련된 내용들을 탐색해보았다.



### BUG ON 매크로 제거

irqs_disabled 확인 부분이 치명적인 failure가 아니라 BUG_ON 매크로를 제거해야한다는 의견이 있었다.

그러나 실제로 BUG인 경우와 아닌 경우 (false positive) 를 구별할 수 없기 때문에 해당 패치는 받아들여지지 않았다.

```c
> diff --git a/drivers/cpufreq/cpufreq.c b/drivers/cpufreq/cpufreq.c
> index a09a29c..a5aa2fa 100644
> --- a/drivers/cpufreq/cpufreq.c
> +++ b/drivers/cpufreq/cpufreq.c
> @@ -281,7 +281,10 @@ static inline void adjust_jiffies(unsigned long val, struct cpufreq_freqs *ci)
>  static void __cpufreq_notify_transition(struct cpufreq_policy *policy,
>                 struct cpufreq_freqs *freqs, unsigned int state)
>  {
> -       BUG_ON(irqs_disabled());
> +       if (irqs_disabled()) {
> +               WARN(1, "IRQs disabled!\n");
> +               return;
> +       }
```

https://lore.kernel.org/patchwork/patch/527415/



### S2R 후 CPU 1 fail

suspend 후 resume 하였을 때 cpu 0만 남아있고, cpu1 이 보이지 않는 경우

ACPI 문제였고, 커널 버전 업데이트 (코드 수정) 후 해결되었다고 한다.

> ```
> Steps to Reproduce:
> 1. Suspend to RAM
> 2. Resume
> 3. Check Kpowersave
> 4. Run cpufreq-info and cat /proc/cpuinfo (results described in detail below)
> 5. Check /sys/device/system/cpu/cpu1/ ---> "cpufreq" directory is no longer present
> ```

https://bugzilla.redhat.com/show_bug.cgi?id=415521



### governor change시 kernel panic

conservative 에서 ondemand로 governor change를 할 때 kernel panic이 발생한 적이 있는데, linux를 ubuntu로 변경하고나서 발생 안해서 닫힌 것 같다.

> There are two CPU entries in sysfs:
> /sys/devices/system/cpu/cpu0/cpufreq/
> /sys/devices/system/cpu/cpu1/cpufreq/
>
> I usually change governor for both CPUs at the same time (but only for cpu0 in this case).
>
> ---- Kernel panic output: ----
> [17249509.052000] Unable to handle kernel NULL pointer dereference at virtual address 0000001c
> [17249509.052000] printing eip:
> [17249509.052000] f8c2f8be
> [17249509.052000] *pde = 00000000
> [17249509.052000] Oops: 0000 [#1]
> [17249509.052000] PREEMPT SMP

https://bugs.launchpad.net/ubuntu/+source/linux-source-2.6.15/+bug/61449



### max cpu frequency 설정 오류

max cpu frequency 가 설정이 안 되는 이슈가 있었다. BIOS 수정으로 fix되었다고 한다.

```shell
cat /sys/devices/system/cpu/cpu1/cpufreq/cpuinfo_max_freq
1667000

cat /sys/devices/system/cpu/cpu1/cpufreq/scaling_max_freq
1333000

echo "1667000" > /sys/devices/system/cpu/cpu1/cpufreq/scaling_max_freq

cat /sys/devices/system/cpu/cpu1/cpufreq/scaling_max_freq
1333000 <---this should be 1.6Ghz!!
```

https://bugs.launchpad.net/ubuntu/+source/linux-source-2.6.15/+bug/55689



### BUG_ON 매크로 & interrupt 관련 ppt

https://www.cs.purdue.edu/homes/lintan/publications/aComment-icse11-slides-no-animation.pdf



### BUG_ON(irqs_disabled()) 관련 직접적 질문들

내부에서 mutex_lock을 사용하기 때문에 irq가 disable되면 안 된다는 것 같다.

제목: BUG_ON(irqs_disabled()) in cpufreq.c

https://lists.linaro.org/pipermail/linaro-kernel/2013-August/005515.html

https://www.spinics.net/lists/cpufreq/msg06575.html

https://www.spinics.net/lists/cpufreq/msg06597.html

https://www.spinics.net/lists/cpufreq/msg06598.html

제목: [PATCH v2 2/2\] ide: Remove BUG_ON(in_interrupt() || irqs_disabled()) from ide_unregister()

https://www.mail-archive.com/linux-kernel@vger.kernel.org/msg2388004.html

제목: [2/2] ide: Remove BUG_ON(in_interrupt() || irqs_disabled()) from ide_unregister()

https://patchwork.ozlabs.org/project/linux-ide/patch/20201113161021.2217361-3-bigeasy@linutronix.de/



[2021.06.30] 이슈 정리

interrupt handler에서 pm qos update request를 하도록 하였다. 기존 cpufreq mainline code는 pm qos notifier가 없었는데, pm qos notifier를 직접 만들어서 달아두었다. 그 notifier에서 cpufreq target 값 변경 함수가 호출되게 하였는데, 거기서 BUG_ON(irqs_disabled()) 에 걸렸다. 여기서 걸리는 게 당연한게, 인터럽트 컨텍스트이기 때문에 DAIF 레지스터 (혹은 PSTATE의 DAIF 비트 필드) 의 I 가 masked 된 상태여서 그렇다. 그러므로 해결책으로는 pm qos update request를 하는게 아니라 work queue 같은 거로 추후에 호출되도록 해야할 것 같다.