# 2021-07-25 (SMP in Xen, Kernel, LK)

부팅시 SMP 를 지원하기 위해 CPU on 되는 path를 정리해보았다.

### XEN

*xen 4.15*

CPU 0

```c
start_xen
ㄴset_processor_id(0)
ㄴsmp_clear_cpu_maps // cpu_possible_map과 cpu_online_map에 0번 세팅
ㄴsetup_mm
ㄴsmp_init_cpus // dt parsing해서 cpu_possible_map에 추가
ㄴgic_init
ㄴsmp_prepare_cpus
ㄴfor_each_present_cpu(i)
    ㄴcpu_up(i) // ★ 여기서 CPU 1, 2, 3을 깨운다
ㄴiommu_setup
ㄴsetup_virt_paging
// 이후 dom0 와 domU 생성
```

CPU 1, 2, 3

```c
init_secondary
ㄴcpu_init
ㄴenable_mmu
ㄴsecondary_switched
    ㄴstart_secondary
    	ㄴset_processor_id(cpu_id)
    	ㄴmmu_init_secondary(cpu)
    	ㄴgic_init_secondary(cpu)
    	ㄴcpumask_set_cpu(cpu) // cpu_online_map에 cpu 번호 추가
```

cpu_up(i) 함수를 통해 CPU 1, 2, 3을 깨우면 위와 같은 path를 타는데, cpu_up(i) 함수 내부는 이렇게 동작한다.

```c
cpu_up(i)
ㄴ__cpu_up(cpu)
    ㄴarch_cpu_up(cpu)
    	ㄴcall_psci_cpu_on(cpu) // psci를 보낼 때 entry point로 init_secondary를 보낸다.
```



### Kernel

*kernel 5.14*

CPU 0

```c
start_kernel
ㄴsmp_setup_processor_id
ㄴboot_cpu_init // cpu_online_mask, active_mask, present_mask, possible_mask에 모두 setting
ㄴsmp_prepare_boot_cpu
ㄴmm_init
    ㄴarch_call_rest_init
   		ㄴrest_init
    		ㄴkernel_init
    			ㄴkernel_init_freeable
    				ㄴsmp_prepare_cpus(max_cpus)
    				ㄴdo_pre_smp_initcalls
    				ㄴsmp_init
    					ㄴbringup_nonboot_cpus(max_cpus)
    						ㄴfor_each_present_cpu(cpu)
    							ㄴcpu_up(cpu) // ★ 여기서 CPU 1, 2, 3을 깨운다
```

CPU 1, 2, 3

```c
secondary_entry
ㄴsecondary_startup
    ㄴ__cpu_setup
    ㄴ__enable_mmu
    ㄴ__secondary_switched
    	ㄴsecondary_start_kernel
    		ㄴcpuinfo_store_cpu
    		ㄴnotify_cpu_starting(cpu) // enable GIC and Timers
    		ㄴset_cpu_online(cpu, true)
```

cpu_up(cpu) 함수를 통해 CPU 1, 2, 3을 깨우면 위와 같은 path를 타는데, cpu_up(cpu) 함수 내부는 이렇게 동작한다.

```c
cpu_up(cpu)
ㄴdo_cpu_up(cpu, CPUHP_ONLINE)
	ㄴ_cpu_up(cpu, 0, CPUHP_ONLINE)
		ㄴbringup_cpu
    		ㄴ__cpu_up(cpu, idle)
    			ㄴboot_secondary(cpu, idle)
    				ㄴcpu_psci_up_boot(cpu) // psci를 보낼 때 entry point로 secondary_entry를 보낸다.
```



### LK

*LK 2021.06*

CPU 0

```c
_start
ㄴlk_main // thread init
    ㄴbootstrap2
    	ㄴarch_init
    		ㄴlk_init_secondary_cpus(SMP_MAX_CPUS-1)
```

CPU 1, 2, 3

```c
_start
ㄴmmu_enable_secondary
ㄴsecondary_boot
    ㄴarm64_secondary_entry
    	ㄴlk_secondary_cpu_entry // thread resume
    		ㄴsecondary_cpu_bootstrap2
```

LK는 xen과 kernel과 달리 SMP-aware kernel이라 LK 를 실행시킬 때 각 CPU들에 모두 다 시작 point를 주고 켜야하는 듯하다.



- path 확인, 설명 확인
- psci 보내는 함수 내부 확인
- SMP 사용시 Virtual Memory 반드시 필요한지?
- XEN에서 Kernel로 이어질 때 CPU가 이미 켜져있는데, 이걸 어떻게 처리한 것인지? 만일 LK 부터 SMP가 켜진 상태로 이어진다면 XEN에서 CPU가 이미 켜져있는 걸 어떻게 받아들여야할지?