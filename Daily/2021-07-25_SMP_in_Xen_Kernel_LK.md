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



### SMP in Virtualization

XEN에서 SMP 살릴 때는 cpu_up 함수를 이용해 실제로 CPU power를 켜주는데, kernel에서 HVC로 요청할 때는 vpsci 함수를 통해서 CPU power는 켜지 않지만 PC 값은 세팅해준다.

xen/arch/arm/vpsci.c

```c
/*
 * PSCI 0.2 or later calls. It will return false if the function ID is
 * not handled.
 */
bool do_vpsci_0_2_call(struct cpu_user_regs *regs, uint32_t fid)
{
    /*
     * /!\ VPSCI_NR_FUNCS (in asm-arm/vpsci.h) should be updated when
     * adding/removing a function. SCCC_SMCCC_*_REVISION should be
     * updated once per release.
     */
    switch ( fid )
    {
    case PSCI_0_2_FN32_PSCI_VERSION:
        perfc_incr(vpsci_version);
        PSCI_SET_RESULT(regs, do_psci_0_2_version());
        return true;

    case PSCI_0_2_FN32_CPU_OFF:
        perfc_incr(vpsci_cpu_off);
        PSCI_SET_RESULT(regs, do_psci_0_2_cpu_off());
        return true;
//생략
    case PSCI_0_2_FN32_CPU_ON:
    case PSCI_0_2_FN64_CPU_ON:
    {
        register_t vcpuid = PSCI_ARG(regs, 1);
        register_t epoint = PSCI_ARG(regs, 2);
        register_t cid = PSCI_ARG(regs, 3);

        perfc_incr(vpsci_cpu_on);
        PSCI_SET_RESULT(regs, do_psci_0_2_cpu_on(vcpuid, epoint, cid));
        return true;
    }

```

