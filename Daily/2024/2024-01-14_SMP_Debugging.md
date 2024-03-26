# 2024-01-14 (SMP Debugging)

## 문제 상황

Linux Kernel

SMP 를 위해 Secondary CPU up 하던 중 문제가 발생하였다.

cpu 0 번에서 cpu 1 번 cpu on 을 요청한 후에, 완료되지 않고 거기에 멈춰있는 것

```bash
[0.001884][T1] EFI services will not be available.
[0.001964][T1] smp: Bringing up secondary CPUs ...
```



cpu 0 번 상태 (T32 로 보면 WFI 에 멈춰있다.)

```bash
cpu_do_idle: dsb sy
wfi
____ret____
nop
arch_cpu_idle: stp x29,x30,[sp,#-0x10]! ;x29,x30,[sp,#-16]!
mov x0,#0xE0 ;x0,#224
mov x29,sp
```

stack 을 보면 다음과 같다.

```bash
-000|cpu_do_idle(asm)
-001|arch_cpu_idle(asm)
-002|default_idle_call(asm)
-003|do_idle(asm)
-004|cpu_startup_entry(asm)
-005|rest_init(asm)
-006|arch_call_rest_init(asm)
-007|start_kernel(asm)
-008|__primary_switched(asm)
--|end of frame
```



cpu 1 번 상태

```bash
dsb ishst
wfe
____wfi_____
b 0xFFFFFFC00801F238
cpu_emulate_effective_ctr: mrs x0,#0x3,#0x3,c0,c0,#0x1 ;x0,CTR_EL0
tbnz x0,#0x1C,0xFFFFFFC00801F25C; x0,#28,0xFFFFFFC00801F25C
```

stack 을 보면 다음과 같다

```bash
-000|verify_local_cpu_caps(asm)
-001|check_local_cpu_capabilities(asm)
-002|secondary_start_kernel(asm)
-003|_secondary_switched(asm)
--|end of frame
```



주석처리하며 디버깅해보니 CPU1 의  `verify_local_cpu_caps` 을 주석처리하고 넘어가게 하니 아래와 같은 로그가 보였다.

```bash
[0.004429][T1]CPU features: detected: Speculative Store Bypassing Safe (SSBS)
[0.004562][T1]------------[ cut here ]------------
[0.004640][T1]CPU: CPUs started in inconsistent modes
[0.004642][T1]WARNING: CPU: 0 PID: 1 at arch/arm64/kernel/smp.c:434 smp_cpus_done+0x68/0x8c
[0.004875][T1] Modules linked in:
```

CPU 0 번은 EL1 으로 시작하게 했는데, CPU 1 번은 EL2 로 시작하게 했기 때문이다. 만일 2개가 둘다 EL2 로 시작한다면 아래와 같은 로그를 볼 수 있다.

```bash
[0.384116][1.742484][0: swapper/0: 1]CPU: All CPU(s) started at EL2
```



## 해결책 1

CPU 1 번을 EL1 으로 시작하게 하면 된다.

eret 전에 SPSR 을 EL1 모드로 설정한다.



## 분석

5.x 버전과 6.x 버전의 차이가 있다.

예전 버전에서는 EL2 로 kernel 을 시작하면 (primary_entry 든 secondary_entry든) 결국 EL1 으로 보내줬는데

이제는 KVM 이 default 로 enable 되면서 EL2 로 시작하면 kvm mode 에 따라 EL2 로 그대로 진행되기도 한다.



CPU 0 번이 secondary cpu 를 켜는 요청을 보낼 때 secondary_entry 의 물리적인 위치를 전달한다.

// arch/arm64/kernel/psci.c

```c
static int cpu_psci_cpu_boot(unsigned int cpu)
{
        phys_addr_t pa_secondary_entry = __pa_symbol(secondary_entry);
        int err = psci_ops.cpu_on(cpu_logical_map(cpu), pa_secondary_entry);
        if (err)
                pr_err("failed to boot CPU%d (%d)\n", cpu, err);

        return err;
}
```



그러면 CPU 1번은 secondary_entry 에서 시작해서 다음과 같이 진행한다.

// arch/arm64/kernel/head.S

```assembly
SYM_FUNC_START(secondary_entry)
        bl      init_kernel_el                  // w0=cpu_boot_mode
        b       secondary_startup
SYM_FUNC_END(secondary_entry)
```



`init_kernel_el` 에서는 현재 EL 을 확인하고 그에 맞는 설정을 시작한다. 그리고 eret 을 하기 때문에 EL2 로 진입하더라도 여기서 EL1 으로 변경된다.

```assembly
SYM_FUNC_START(init_kernel_el)
        mrs     x0, CurrentEL
        cmp     x0, #CurrentEL_EL2
        b.eq    init_el2

SYM_INNER_LABEL(init_el1, SYM_L_LOCAL)
        mov_q   x0, INIT_SCTLR_EL1_MMU_OFF
        msr     sctlr_el1, x0
        isb
        mov_q   x0, INIT_PSTATE_EL1
        msr     spsr_el1, x0
        msr     elr_el1, lr
        mov     w0, #BOOT_CPU_MODE_EL1
        eret

SYM_INNER_LABEL(init_el2, SYM_L_LOCAL)
        mov_q   x0, HCR_HOST_NVHE_FLAGS
        msr     hcr_el2, x0
        isb

        init_el2_state

        /* Hypervisor stub */
        adr_l   x0, __hyp_stub_vectors
        msr     vbar_el2, x0
        isb

        mov_q   x1, INIT_SCTLR_EL1_MMU_OFF

        /*
         * Fruity CPUs seem to have HCR_EL2.E2H set to RES1,
         * making it impossible to start in nVHE mode. Is that
         * compliant with the architecture? Absolutely not!
         */
        mrs     x0, hcr_el2
        and     x0, x0, #HCR_E2H
        cbz     x0, 1f

        /* Set a sane SCTLR_EL1, the VHE way */
        msr_s   SYS_SCTLR_EL12, x1
        mov     x2, #BOOT_CPU_FLAG_E2H
        b       2f

1:
        msr     sctlr_el1, x1
        mov     x2, xzr
2:
        msr     elr_el2, lr
        mov     w0, #BOOT_CPU_MODE_EL2
        orr     x0, x0, x2
        eret
SYM_FUNC_END(init_kernel_el)
```

그 후에 `secondary_startup` 을 진행한다.

```assembly
SYM_FUNC_START_LOCAL(secondary_startup)
        /*
         * Common entry point for secondary CPUs.
         */
        mov     x20, x0                         // preserve boot mode
        bl      finalise_el2
        bl      __cpu_secondary_check52bitva
#if VA_BITS > 48
        ldr_l   x0, vabits_actual
#endif
        bl      __cpu_setup                     // initialise processor
        adrp    x1, swapper_pg_dir
        adrp    x2, idmap_pg_dir
        bl      __enable_mmu
        ldr     x8, =__secondary_switched
        br      x8
SYM_FUNC_END(secondary_startup)
```

여기서 `finalise_el2` 를 호출하면 아래와 같은 코드가 실행되고 hvc 이기 때문에 hypervisor call 이 호출되면서 다시 EL2 가 된다. (위의 코드를 보면 hvc 때 호출될 수 있도록 __hyp_stub_vectors 를 설정하는 부분이 보인다.)

// arch/arm64/kernel/hyp-stub.S

```assembly
SYM_FUNC_START(finalise_el2)
        // Need to have booted at EL2
        cmp     w0, #BOOT_CPU_MODE_EL2
        b.ne    1f

        // and still be at EL1
        mrs     x0, CurrentEL
        cmp     x0, #CurrentEL_EL1
        b.ne    1f

        mov     x0, #HVC_FINALISE_EL2
        hvc     #0
1:
        ret
SYM_FUNC_END(finalise_el2)
```

`__hyp_stub_vectors`->`elx_sync`->`__finalise_el2` 로 아래를 진행하다가, 만일 id_aa64mmfr1 의 VH 비트가 1이 아니라면 error 로 빠지게 된다.

```assembly
.Lskip_sme:

        // nVHE? No way! Give me the real thing!
        // Sanity check: MMU *must* be off
        mrs     x1, sctlr_el2
        tbnz    x1, #0, 1f

        // Needs to be VHE capable, obviously
        check_override id_aa64mmfr1 ID_AA64MMFR1_EL1_VH_SHIFT 2f 1f

1:      mov_q   x0, HVC_STUB_ERR
        eret
2:
        // Engage the VHE magic!
        mov_q   x0, HCR_HOST_VHE_FLAGS
        msr     hcr_el2, x0
        isb
```

그게 아니라면 spsr 을 EL2 로 머물도록 설정한 후, 최종적으로 `enter_vhe` 내에서 eret 을 하고 돌아오게 된다.

```assembly
        // Hack the exception return to stay at EL2
        mrs     x0, spsr_el1
        and     x0, x0, #~PSR_MODE_MASK
        mov     x1, #PSR_MODE_EL2h
        orr     x0, x0, x1
        msr     spsr_el1, x0

        b       enter_vhe
SYM_CODE_END(__finalise_el2)
```

다시 돌아와서 (위의 error 경로든 아니든 코드 진행은 같다. EL2 로 동작하는지, EL1 으로 동작하는지의 차이만 생긴다.) `__secondary_switched` 를 호출한다.

// arch/arm64/kernel/head.S

```assembly
SYM_FUNC_START_LOCAL(secondary_startup)
        /*
         * Common entry point for secondary CPUs.
         */
        mov     x20, x0                         // preserve boot mode
        bl      finalise_el2
        bl      __cpu_secondary_check52bitva
#if VA_BITS > 48
        ldr_l   x0, vabits_actual
#endif
        bl      __cpu_setup                     // initialise processor
        adrp    x1, swapper_pg_dir
        adrp    x2, idmap_pg_dir
        bl      __enable_mmu
        ldr     x8, =__secondary_switched
        br      x8
SYM_FUNC_END(secondary_startup)
```

그러면 `secondary_start_kernel` 을 호출하고,

```assembly
SYM_FUNC_START_LOCAL(__secondary_switched)
        mov     x0, x20
        bl      set_cpu_boot_mode_flag
        str_l   xzr, __early_cpu_boot_status, x3
        adr_l   x5, vectors
        msr     vbar_el1, x5
        isb

        adr_l   x0, secondary_data
        ldr     x2, [x0, #CPU_BOOT_TASK]
        cbz     x2, __secondary_too_slow

        init_cpu_task x2, x1, x3

#ifdef CONFIG_ARM64_PTR_AUTH
        ptrauth_keys_init_cpu x2, x3, x4, x5
#endif

        bl      secondary_start_kernel
        ASM_BUG()
SYM_FUNC_END(__secondary_switched)
```

// arch/arm64/kernel/smp.c

cpu1 번 동작에 필요한 코드를 쭉 진행한다.

```c
/*
 * This is the secondary CPU boot entry.  We're using this CPUs
 * idle thread stack, but a set of temporary page tables.
 */
asmlinkage notrace void secondary_start_kernel(void)
{
        u64 mpidr = read_cpuid_mpidr() & MPIDR_HWID_BITMASK;
        struct mm_struct *mm = &init_mm;
        const struct cpu_operations *ops;
        unsigned int cpu = smp_processor_id();

        /*
         * All kernel threads share the same mm context; grab a
         * reference and switch to it.
         */
        mmgrab(mm);
        current->active_mm = mm;

        /*
         * TTBR0 is only used for the identity mapping at this stage. Make it
         * point to zero page to avoid speculatively fetching new entries.
         */
        cpu_uninstall_idmap();

        if (system_uses_irq_prio_masking())
                init_gic_priority_masking();

        rcu_cpu_starting(cpu);
        trace_hardirqs_off();
```



따라서 처음부터 EL1 으로 시작한다면, `finalise_el2` 를 위해 hvc 를 호출할 일이 없다.



## 해결책 2

id_aa64mmfr1 의 VH 비트가 0 이면 error 로 빠지면서 EL1 으로 돌아오므로, device tree 의 argument 를 다음과 같이 설정해도 해결되긴 한다.

```c
bootargs = "console=ttySAC0,115200n8 loglevel=3 printk.devkmsg=on log_buf_len=512K bootconfig arm64.nopauth kvm-arm.mode=protected"
```



## 분석

위의 코드 중 특히 `kvm-arm.mode=protected` 를 보면 결국 `id_aa64mmfr1.vh` 를 0 으로 설정하는 것을 알 수 있다.

// arch/arm64/kernel/idreg-override.c

```c
static const struct {
        char    alias[FTR_ALIAS_NAME_LEN];
        char    feature[FTR_ALIAS_OPTION_LEN];
} aliases[] __initconst = {
        { "kvm-arm.mode=nvhe",          "id_aa64mmfr1.vh=0" },
        { "kvm-arm.mode=protected",     "id_aa64mmfr1.vh=0" },
        { "arm64.nosve",                "id_aa64pfr0.sve=0 id_aa64pfr1.sme=0" },
        { "arm64.nosme",                "id_aa64pfr1.sme=0" },
        { "arm64.nobti",                "id_aa64pfr1.bt=0" },
        { "arm64.nopauth",
          "id_aa64isar1.gpi=0 id_aa64isar1.gpa=0 "
          "id_aa64isar1.api=0 id_aa64isar1.apa=0 "
          "id_aa64isar2.gpa3=0 id_aa64isar2.apa3=0"        },
        { "arm64.nomte",                "id_aa64pfr1.mte=0" },
        { "nokaslr",                    "kaslr.disabled=1" },
};
```

실제 레지스터 값이 아닌, linux kernel 에서 설정한 값을 변수로 따로 관리한다.

만일 `check_override id_aa64mmfr1 ID_AA64MMFR1_EL1_VH_SHIFT 2f 1f` 를 호출했다면 `id_aa64mmfr1_override` 가 호출된다.

// arch/arm64/kernel/hyp-stub.S

```assembly
.macro __check_override idreg, fld, width, pass, fail
        ubfx    x1, x1, #\fld, #\width
        cbz     x1, \fail

        adr_l   x1, \idreg\()_override
        ldr     x2, [x1, FTR_OVR_VAL_OFFSET]
        ldr     x1, [x1, FTR_OVR_MASK_OFFSET]
        ubfx    x2, x2, #\fld, #\width
        ubfx    x1, x1, #\fld, #\width
        cmp     x1, xzr
        and     x2, x2, x1
        csinv   x2, x2, xzr, ne
        cbnz    x2, \pass
        b       \fail
.endm
```

이 값은 실제 레지스터 값이 아닌 따로 변수로 관리하고 있는 값이다.

// arch/arm64/kernel/cpufeature.c

```c
struct arm64_ftr_override __ro_after_init id_aa64mmfr1_override;


static const struct __ftr_reg_entry {
        u32                     sys_id;
        struct arm64_ftr_reg    *reg;
} arm64_ftr_regs[] = {

        /* Op1 = 0, CRn = 0, CRm = 7 */
        ARM64_FTR_REG(SYS_ID_AA64MMFR0_EL1, ftr_id_aa64mmfr0),
        ARM64_FTR_REG_OVERRIDE(SYS_ID_AA64MMFR1_EL1, ftr_id_aa64mmfr1,
                               &id_aa64mmfr1_override),
        ARM64_FTR_REG(SYS_ID_AA64MMFR2_EL1, ftr_id_aa64mmfr2),
```



부팅 때 아래와 같은 로그를 뜨게 한다.

```bash
[0.000000][T0] CPU features: SYS_ID_AA64ISAR2_EL1[11:8]: already set to 0
[0.000000][T0] CPU features: SYS_ID_AA64MMFR1_EL1[11:8]: forced to 0
[0.000000][T0] CPU features: detected: GIC system register CPU interface
```



결론적으로, VH bit 를 0 이 되도록 device tree 에 kvm mode 를 protected 혹은 nvhe 모드로 설정하면 `finalise_el2`에서 `HVC_STUB_ERR`가 되면서 EL1 으로 동작하기는 하지만, 정확한 해결책은 해결책 1번과 같이 EL1 으로 시작하도록 하는 것으로 보인다.

kernel 5 점 초반대 버전에서 KVM 을 고려하지 않는 path 를 비교해보면 좋을 것 같다.



참고)

VH 비트는 Virtualization Host Extension 이 지원되면 무조건 1로 세팅되어있는 비트다. (수정 불가능)

https://developer.arm.com/documentation/100442/0100/register-descriptions/aarch64-system-registers/id-aa64mmfr1-el1--aarch64-memory-model-feature-register-1--el1

이걸 kernel 에서는 vhe 로 부르고 있는 것으로 보인다.



ARM KVM 설명한 글

https://selfish-developer.com/entry/KVM-ARM

[v3,16/21] arm64: Make kvm-arm.mode={nvhe, protected} an alias of id_aa64mmfr1.vh=0

https://patchwork.kernel.org/project/linux-arm-kernel/patch/20210111132811.2455113-17-maz@kernel.org/

arm virtualization host extensions

https://developer.arm.com/documentation/102142/0100/Virtualization-host-extensions



참고)

device tree 에 cpu map 작성하는 방법

3.14 버전 정도 (device tree 도입된 버전) 부터 NR_CPUS config 대신 dt 에 선언한 것을 사용한다. `parse_dt_topology` 함수에서 parsing 하는 것으로 보인다. CONFIG_NR_CPUS 나 nr_cpus, maxcpus 등을 검색해보면 사용 예가 없다.

// arch/arm64/boot/dts/amd/amd-seattle-cpus.dtsi

```c
        cpus {
                #address-cells = <0x1>;
                #size-cells = <0x0>;

                cpu-map {
                        cluster0 {
                                core0 {
                                        cpu = <&CPU0>;
                                };
                                core1 {
                                        cpu = <&CPU1>;
                                };
                        };
                };

                CPU0: cpu@0 {
                        device_type = "cpu";
                        compatible = "arm,cortex-a57";
                        reg = <0x0>;
                        enable-method = "psci";
                };

                CPU1: cpu@1 {
                        device_type = "cpu";
                        compatible = "arm,cortex-a57";
                        reg = <0x1>;
                        enable-method = "psci";
                };
```

interrupt-controller 에 문제가 있어서 동작하지 않고 wfi 에 멈춘건가 싶기도 했는데 device tree 에 잘 선언되어있었다. 그리고 uart console 이 잘 동작했으므로 인터럽트의 문제는 아닌 것으로 결론 내렸다.

// arch/arm64/boot/dts/amd/amd-seattle-soc.dtsi

```c
        gic0: interrupt-controller@e1101000 {
                compatible = "arm,gic-400", "arm,cortex-a15-gic";
                interrupt-controller;
                #interrupt-cells = <3>;
                reg = <0x0 0xe1110000 0 0x1000>, /* GICD 주소 */
                      <0x0 0xe112f000 0 0x2000>; /* GICR 주소 */
                interrupts = <1 9 0xf04>;
        };
```



참고)

cpu on path 정리

```bash
[0.001964][T1] smp: Bringing up secondary CPUs ...
```

1. `smp_init`

// kernel/smp.c

```c
/* Called by boot processor to activate the rest. */
void __init smp_init(void)
{
        int num_nodes, num_cpus;

        idle_threads_init();
        cpuhp_threads_init();

        pr_info("Bringing up secondary CPUs ...\n");

        bringup_nonboot_cpus(setup_max_cpus);

        num_nodes = num_online_nodes();
        num_cpus  = num_online_cpus();
        pr_info("Brought up %d node%s, %d CPU%s\n",
                num_nodes, (num_nodes > 1 ? "s" : ""),
                num_cpus,  (num_cpus  > 1 ? "s" : ""));

        /* Any cleanup work */
        smp_cpus_done(setup_max_cpus);
}
```

2. `bringup_nonboot_cpus`

// kernel/cpu.c

```c
void bringup_nonboot_cpus(unsigned int setup_max_cpus)
{
        unsigned int cpu;

        for_each_present_cpu(cpu) {
                if (num_online_cpus() >= setup_max_cpus)
                        break;
                if (!cpu_online(cpu))
                        cpu_up(cpu, CPUHP_ONLINE);
        }
}
```

3. `cpu_up`

```c
static int cpu_up(unsigned int cpu, enum cpuhp_state target)
{
        int err = 0;

        if (!cpu_possible(cpu)) {
                pr_err("can't online cpu %d because it is not configured as may-hotadd at boot time\n",
                       cpu);
#if defined(CONFIG_IA64)
                pr_err("please check additional_cpus= boot parameter\n");
#endif
                return -EINVAL;
        }

        err = try_online_node(cpu_to_node(cpu));
        if (err)
                return err;

        cpu_maps_update_begin();

        if (cpu_hotplug_disabled) {
                err = -EBUSY;
                goto out;
        }
        if (!cpu_smt_allowed(cpu)) {
                err = -EPERM;
                goto out;
        }

        err = _cpu_up(cpu, 0, target);
out:
        cpu_maps_update_done();
        return err;
}
```

4. `_cpu_up`

```c
/* Requires cpu_add_remove_lock to be held */
static int _cpu_up(unsigned int cpu, int tasks_frozen, enum cpuhp_state target)
{
//
            /*
         * Try to reach the target state. We max out on the BP at
         * CPUHP_BRINGUP_CPU. After that the AP hotplug thread is
         * responsible for bringing it up to the target state.
         */
        target = min((int)target, CPUHP_BRINGUP_CPU);
        ret = cpuhp_up_callbacks(cpu, st, target);
```

5. `cpuhp_up_callbacks`

```c
static int cpuhp_up_callbacks(unsigned int cpu, struct cpuhp_cpu_state *st,
                              enum cpuhp_state target)
{
        enum cpuhp_state prev_state = st->state;
        int ret = 0;

        ret = cpuhp_invoke_callback_range(true, cpu, st, target);
        if (ret) {
                pr_debug("CPU UP failed (%d) CPU %u state %s (%d)\n",
                         ret, cpu, cpuhp_get_step(st->state)->name,
                         st->state);

                cpuhp_reset_state(cpu, st, prev_state);
                if (can_rollback_cpu(st))
                        WARN_ON(cpuhp_invoke_callback_range(false, cpu, st,
                                                            prev_state));
        }
        return ret;
}
```

6. `cpuhp_invoke_callback_range`

```c
static int cpuhp_invoke_callback_range(bool bringup,
                                       unsigned int cpu,
                                       struct cpuhp_cpu_state *st,
                                       enum cpuhp_state target)
{
        enum cpuhp_state state;
        int err = 0;

        while (cpuhp_next_state(bringup, &state, st, target)) {
                err = cpuhp_invoke_callback(cpu, state, bringup, NULL, NULL);
                if (err)
                        break;
        }

        return err;
}
```

7. `cpuhp_invoke_callback`

```c
static int cpuhp_invoke_callback(unsigned int cpu, enum cpuhp_state state,
                                 bool bringup, struct hlist_node *node,
                                 struct hlist_node **lastp)
{
        struct cpuhp_cpu_state *st = per_cpu_ptr(&cpuhp_state, cpu);
        struct cpuhp_step *step = cpuhp_get_step(state);
        int (*cbm)(unsigned int cpu, struct hlist_node *node);
        int (*cb)(unsigned int cpu);
        int ret, cnt;
//
        /* State transition. Invoke on all instances */
        cnt = 0;
        hlist_for_each(node, &step->list) {
                if (lastp && node == *lastp)
                        break;

                trace_cpuhp_multi_enter(cpu, st->target, state, cbm, node);
                ret = cbm(cpu, node);
                trace_cpuhp_exit(cpu, st->state, state, ret);
                if (ret) {
                        if (!lastp)
                                goto err;

                        *lastp = node;
                        return ret;
                }
                cnt++;
        }
        if (lastp)
                *lastp = NULL;
        return 0;

```

8. `cpuhp_hp_states` 의 CPUHP_OFFLINE 부터 CPUHP_ONLINE 까지 각 state 의 callback function 을 호출하고, CPUHP_BRINGUP_CPU 는 그 중간에 위치한다.

```c
/* Boot processor state steps */
static struct cpuhp_step cpuhp_hp_states[] = {
        [CPUHP_OFFLINE] = {
                .name                   = "offline",
                .startup.single         = NULL,
                .teardown.single        = NULL,
        },
#ifdef CONFIG_SMP
//
        /* Kicks the plugged cpu into life */
        [CPUHP_BRINGUP_CPU] = {
                .name                   = "cpu:bringup",
                .startup.single         = bringup_cpu,
                .teardown.single        = finish_cpu,
                .cant_stop              = true,
        },
//
        /* CPU is fully up and running. */
        [CPUHP_ONLINE] = {
                .name                   = "online",
                .startup.single         = NULL,
                .teardown.single        = NULL,
        },
};
```

9. `bringup_cpu`

```c
static int bringup_cpu(unsigned int cpu)
{
        struct task_struct *idle = idle_thread_get(cpu);
        int ret;

        /*
         * Reset stale stack state from the last time this CPU was online.
         */
        scs_task_reset(idle);
        kasan_unpoison_task_stack(idle);

        /*
         * Some architectures have to walk the irq descriptors to
         * setup the vector space for the cpu which comes online.
         * Prevent irq alloc/free across the bringup.
         */
        irq_lock_sparse();

        /* Arch-specific enabling code. */
        ret = __cpu_up(cpu, idle);
        irq_unlock_sparse();
        if (ret)
                return ret;
        return bringup_wait_for_ap(cpu);
}
```

10. `__cpu_up`

// arch/arm64/kernel/smp.c

```c
int __cpu_up(unsigned int cpu, struct task_struct *idle)
{
        int ret;
        long status;

        /*
         * We need to tell the secondary core where to find its stack and the
         * page tables.
         */
        secondary_data.task = idle;
        update_cpu_boot_status(CPU_MMU_OFF);

        /* Now bring the CPU into our world */
        ret = boot_secondary(cpu, idle);
        if (ret) {
                pr_err("CPU%u: failed to boot: %d\n", cpu, ret);
                return ret;
        }
```

11. `boot_secondary`

```c
static int boot_secondary(unsigned int cpu, struct task_struct *idle)
{
        const struct cpu_operations *ops = get_cpu_ops(cpu);

        if (ops->cpu_boot)
                return ops->cpu_boot(cpu);

        return -EOPNOTSUPP;
}
```

12. `cpu_psci_cpu_boot` = `ops->cpu_boot`

```c
static int cpu_psci_cpu_boot(unsigned int cpu)
{
        phys_addr_t pa_secondary_entry = __pa_symbol(secondary_entry);
        int err = psci_ops.cpu_on(cpu_logical_map(cpu), pa_secondary_entry);
        if (err)
                pr_err("failed to boot CPU%d (%d)\n", cpu, err);

        return err;
}
```

13. `psci_0_1_cpu_on`

// drivers/firmware/psci/psci.c

```c
static int psci_0_1_cpu_on(unsigned long cpuid, unsigned long entry_point)
{
        return __psci_cpu_on(psci_0_1_function_ids.cpu_on, cpuid, entry_point);
}

static int __psci_cpu_on(u32 fn, unsigned long cpuid, unsigned long entry_point)
{
        int err;

        err = invoke_psci_fn(fn, cpuid, entry_point, 0);
        return psci_to_linux_errno(err);
}
```



참고)

CPU0 번이 start_kernel 부터 진행되는 과정

https://community.st.com/t5/stm32-mpus-embedded-software/linux-5-4-31-stuck-on-boot-in-start-kernel-arch-cpu-idle/td-p/316707

```c
#0 cpu_v7_do_idle () at arch/arm/mm/proc-v7.S:78
#1 0xc010a1a4 in arch_cpu_idle () at arch/arm/kernel/process.c:71
#2 0xc0158064 in cpuidle_idle_call () at kernel/sched/idle.c:154
#3 do_idle () at kernel/sched/idle.c:263
#4 0xc01583d8 in cpu_startup_entry (state=CPUHP_ONLINE) at kernel/sched/idle.c:355
#5 0xc0acb5b8 in rest_init () at init/main.c:451
#6 0xc0f00a40 in arch_call_rest_init () at init/main.c:573
#7 0xc0f00f10 in start_kernel () at init/main.c:785
#8 0x00000000 in ?? ()
```



참고)

kernel 부팅을 EL2, EL1 중에 선택하는 방법 질문

CONFIG_ARM64_VHE 옵션을 언급하였는데, 해당 옵션은 존재하지 않았음

https://unix.stackexchange.com/questions/685769/in-arm64-linux-when-linux-is-to-be-run-in-el2-bootloader-in-el3-do-i-have-to



참고)

ARM Trusted Firmware (EL3) 에서 운영체제를 실행시킬 때 레벨을 설정하는 방법

// lib/el3_runtime/aarch64/context_mgmt.c

```c
void cm_setup_context(cpu_context_t *ctx, const entry_point_info_t *ep)
{
//
        scr_el3 = read_scr();
        scr_el3 &= ~(SCR_NS_BIT | SCR_RW_BIT | SCR_FIQ_BIT | SCR_IRQ_BIT |
                        SCR_ST_BIT | SCR_HCE_BIT);
        /*
         * SCR_NS: Set the security state of the next EL.
         */
        if (security_state != SECURE)
                scr_el3 |= SCR_NS_BIT;
        /*
         * SCR_EL3.RW: Set the execution state, AArch32 or AArch64, for next
         *  Exception level as specified by SPSR.
         */
        if (GET_RW(ep->spsr) == MODE_RW_64)
                scr_el3 |= SCR_RW_BIT;
//
         /*
         * Populate EL3 state so that we've the right context
         * before doing ERET
         */
        state = get_el3state_ctx(ctx);
        write_ctx_reg(state, CTX_SCR_EL3, scr_el3);
        write_ctx_reg(state, CTX_ELR_EL3, ep->pc);
        write_ctx_reg(state, CTX_SPSR_EL3, ep->spsr);
```



SPSR 설정하는 부분 찾기

// bl31/bl31_main.c

```c
        /* Program EL3 registers to enable entry into the next EL */
        next_image_info = bl31_plat_get_next_image_ep_info(image_type);
        assert(next_image_info != NULL);
        assert(image_type == GET_SECURITY_STATE(next_image_info->h.attr));

        INFO("BL31: Preparing for EL3 exit to %s world\n",
                (image_type == SECURE) ? "secure" : "normal");
        print_entry_point_info(next_image_info);
        cm_init_my_context(next_image_info);
        cm_prepare_el3_exit(image_type);
```

SPSR 레지스터에 MODE_EL1 을 설정하고 eret 을 하면 EL1 으로 동작하게 됨. ELR 에 동작했을 때 점프할 주소도 적어두어야한다.

// plat/arm/common/arm_common.c

```c
#ifdef __aarch64__
uint32_t arm_get_spsr_for_bl33_entry(void)
{
        unsigned int mode;
        uint32_t spsr;

        /* Figure out what mode we enter the non-secure world in */
        mode = (el_implemented(2) != EL_IMPL_NONE) ? MODE_EL2 : MODE_EL1;

        /*
         * TODO: Consider the possibility of specifying the SPSR in
         * the FIP ToC and allowing the platform to have a say as
         * well.
         */
        spsr = SPSR_64((uint64_t)mode, MODE_SP_ELX, DISABLE_ALL_EXCEPTIONS);
        return spsr;
}
```







참고)

hint #0x22 뜻 조사하기





