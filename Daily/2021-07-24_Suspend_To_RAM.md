# 2021-07-24 (Suspend To RAM)

S2R을 하면 xen이나 kernel에서 cpu_resume 함수를 호출하게 된다.



cpu_resume 함수의 원형인 것 같은 자료를 찾았다.

**[RFC,05/14] arm64: kernel: cpu_{suspend/resume} implementation**

> 
> Kernel subsystems like CPU idle and suspend to RAM require a generic
> mechanism to suspend a processor, save its context and put it into
> a quiescent state. The cpu_{suspend}/{resume} implementation provides
> such a framework through a kernel interface allowing to save/restore
> registers, flush the context to DRAM and suspend/resume to/from
> low-power states where processor context may be lost.
> 
> Different SoCs might require different operations to be carried out
> before a power down request is committed. For this reason the kernel
> allows the caller of cpu_suspend to provide a function pointer (fn in
> the cpu_suspend prototype below),
> 
> int cpu_suspend(unsigned long arg, int (*fn)(unsigned long));
> 

https://patchwork.kernel.org/project/linux-arm-kernel/patch/1377689766-17642-6-git-send-email-lorenzo.pieralisi@arm.com/#6078671

이 패치 전에는 어떻게 suspend와 resume이 동작했던 걸까?



cpu_resume 함수에 hypervisor 환경 고민하는 패치도 있다.

https://lists.cs.columbia.edu/pipermail/kvmarm/2014-March/008326.html



### kernel suspend code flow

텍스트로 flow가 설명된 문서

> **Platform-dependent Resume Code Flow**
>
> 1. Platform-specific system wakeup.
>
> The platform is woken up by a signal from one of the designated system wakeup devices (which need not be an in-band hardware interrupt) and control is passed back to the kernel (the working configuration of the platform may need to be restored by the platform firmware before the kernel gets control again).
>
> 2. Resuming core system components.
>
> The suspend-time configuration of the core system components is restored and the timekeeping is resumed.
>
> 3. Re-enabling non-boot CPUs.
>
> The CPUs disabled in step 4 of the preceding suspend transition are taken back online and their suspend-time configuration is restored.
>
> 4. Resuming devices and restoring the working-state configuration of IRQs.
>
> This step is the same as step 2 of the suspend-to-idle suspend transition described above.
>
> 5. Thawing tasks.
>
> This step is the same as step 3 of the suspend-to-idle suspend transition described above.
>
> 6. Invoking system-wide resume notifiers.
>
> This step is the same as step 4 of the suspend-to-idle suspend transition described above.

https://www.kernel.org/doc/html/latest//admin-guide/pm/suspend-flows.html



### CPU suspend와 resume path

Linaro 발표자료인데, suspend와 resume path를 좀 더 상세하게 적어뒀다.

https://www.slideshare.net/linaroorg/idling-ar-msinabusyworld



### S2R in power management

Power Management 방법 중 하나로 S2R을 설명하는데, 상세히 process와 샘플 코드까지 넣어서 잘 설명된 문서인 것 같다.

![ １](https://www.programmersought.com/images/605/1cfc7520013ed2a16ab75b7bc568456d.png)

![ ](https://www.programmersought.com/images/760/3f9198f8b08bb2825a913eb072e52948.png)

https://www.programmersought.com/article/44725798934/



### S2R Command

`# echo standby > /sys/power/state`

`# echo mem > /sys/power/state`

https://developer.toradex.com/knowledge-base/suspend-resume-linux



### 참고 - CPU Hotplug out

suspend, resume과 단순 hotplug in/out 은 다를 수 있지만, resume 때 cpu 살리는 거, 아니면 booting때 SMP로 cpu 살리는 거 hotplug 처럼 cpuhp_state 랑 관련 함수로 관리하는 듯하다.

https://www.kernel.org/doc/html/latest/core-api/cpu_hotplug.html