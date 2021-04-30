# 2021-04-30 (Linux Power Management)

### Linux의 전반적인 Power management 정리

아래 자료들을 찾아보면 크게 두 가지로 정리되는 것 같다.

(1) Sleep State 로 만들어서 Power Management를 하는 방식이 있고,

(2) Active 상태일 때 DVFS 기법을 통해 Power Management를 하는 방식이 있다.

DVFS를 검색하면 대부분 cpufreq driver에 대한 설명이 나오는데 때때로 devfreq에 대한 설명도 찾을 수 있었다.



### Power management

The Linux kernel uses several power management strategies:

- **Dynamic voltage frequency scaling (DVFS)** adjusts the CPU frequency and the different voltages depending on the system's load. The combination of a CPU frequency and a set of voltages for that frequency is called an operating point.
- **Suspend to memory** allows for the system to sleep waiting for an event. On suspend, all system devices, including CPU and memory, enter low power mode. On resume, the system will continue from the same state it was in before it suspended.
- **Power off**, which brings the system to a halt until an event wakes the system. On power off the system power remains enabled and the system is placed on its lowest consumption mode. On wake up, the bootloader starts up again and the system is initialized.

[출처] 

https://www.digi.com/resources/documentation/digidocs/90001546/reference/bsp/v4-1_cc6/r_power_management.htm?TocPath=Digi%20Embedded%20Yocto%7CSystem%20development%7CLinux%20Board%20Support%20Package%7CDevices%20and%20interfaces%7C_____14



### Power Management

- [Power Management Strategies](https://www.kernel.org/doc/html/v4.14/admin-guide/pm/strategies.html)
- System-Wide Power Management
  - [System Sleep States](https://www.kernel.org/doc/html/v4.14/admin-guide/pm/sleep-states.html)
- Working-State Power Management
  - [CPU Performance Scaling](https://www.kernel.org/doc/html/v4.14/admin-guide/pm/cpufreq.html)
  - [`intel_pstate` CPU Performance Scaling Driver](https://www.kernel.org/doc/html/v4.14/admin-guide/pm/intel_pstate.html)

[출처] https://www.kernel.org/doc/html/v4.14/admin-guide/pm/index.html



### ARM Power management for Linux

- cpufreq (DVFS)
- cpuidle (hotplug)
- Energy Aware Scheduling (EAS)
- System Suspend to RAM

[출처] 

https://community.arm.com/developer/tools-software/oss-platforms/w/docs/527/--------power-management--------



cf)

Linux 의 CONFIG 자세한 설명 사이트

https://cateee.net/lkddb/web-lkddb/PM_DEVFREQ.html