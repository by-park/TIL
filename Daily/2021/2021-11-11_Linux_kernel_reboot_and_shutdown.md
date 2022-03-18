# 2021-11-11 (Linux kernel reboot and shutdown)

### Linux command

Linux kernel 에서 `reboot` 을 치면 재부팅이 되고, `poweroff`를 치면 shutdown이 된다. PC나 모바일에서 재시작 및 전원 종료 버튼을 누른 것과 같다.

### Linux Kernel (EL1)

kernel 에서 reboot 및 shutdown path는 다음과 같다.

// kernel/reboot.c

```c
void kernel_restart(char *cmd)
{
        kernel_restart_prepare(cmd);
        migrate_to_reboot_cpu();
        syscore_shutdown();
        if (!cmd)
                pr_emerg("Restarting system\n");
        else
                pr_emerg("Restarting system with command '%s'\n", cmd);
        kmsg_dump(KMSG_DUMP_SHUTDOWN);
        machine_restart(cmd);
}
EXPORT_SYMBOL_GPL(kernel_restart);

void kernel_power_off(void)
{
        kernel_shutdown_prepare(SYSTEM_POWER_OFF);
        if (pm_power_off_prepare)
                pm_power_off_prepare();
        migrate_to_reboot_cpu();
        syscore_shutdown();
        pr_emerg("Power down\n");
        kmsg_dump(KMSG_DUMP_SHUTDOWN);
        machine_power_off();
}
EXPORT_SYMBOL_GPL(kernel_power_off);
```

kernel_shutdown_prepare 에 보면 reboot_notifier 와 device shutdown 이 불린다.

// kernel/reboot.c

```c
static void kernel_shutdown_prepare(enum system_states state)
{
        blocking_notifier_call_chain(&reboot_notifier_list,
                (state == SYSTEM_HALT) ? SYS_HALT : SYS_POWER_OFF, NULL);
        system_state = state;
        usermodehelper_disable();
        device_shutdown();
}
```

reboot_notifier 로 호출되기 위해서는  `register_reboot_notifier` 함수를 통해 등록하면 된다. device shutdown 안에서는 driver 에서 `.shutdown` 으로 등록된 것들이 불린다.

cf) `.poweroff` 는 Hibernation 용 callback 이다. "drivers/base/power/main.c" 의 `PM_EVENT_HIBERNATE` 참고

// 예시 drivers/regulator/act8945a-regulator.c

```c
static struct platform_driver act8945a_pmic_driver = {
        .driver = {
                .name = "act8945a-regulator",
                .pm = &act8945a_pm,
        },
        .probe = act8945a_pmic_probe,
        .shutdown = act8945a_pmic_shutdown,
};
module_platform_driver(act8945a_pmic_driver);
```

machine_restart 와 machine_power_off 으로 진입하면 아래와 같다.

// arch/arm64/kernel/process.c

```c
void machine_restart(char *cmd)
{
        /* Disable interrupts first */
        local_irq_disable();
        smp_send_stop();

        /*
         * UpdateCapsule() depends on the system being reset via
         * ResetSystem().
         */
        if (efi_enabled(EFI_RUNTIME_SERVICES))
                efi_reboot(reboot_mode, NULL);

        /* Now call the architecture specific reboot code. */
        if (arm_pm_restart)
                arm_pm_restart(reboot_mode, cmd);
        else
                do_kernel_restart(cmd);

        /*
         * Whoops - the architecture was unable to reboot.
         */
        printk("Reboot failed -- System halted\n");
        while (1);
}


void machine_power_off(void)
{
        local_irq_disable();
        smp_send_stop();
        if (pm_power_off)
                pm_power_off();
}
```

arm_pm_restart 와 pm_power_off 를 별도로 지정해주지 않으면 부팅 초기에 psci 가 가장 먼저 불리면서 설정된다. (그 이후에 driver 에서 새로 지정해서 덮어씌워서 사용한다.)

// drivers/firmware/psci/psci.c

```c
static void __init psci_0_2_set_functions(void)
{
        pr_info("Using standard PSCI v0.2 function IDs\n");
        psci_ops.get_version = psci_get_version;

        psci_function_id[PSCI_FN_CPU_SUSPEND] =
                                        PSCI_FN_NATIVE(0_2, CPU_SUSPEND);
        psci_ops.cpu_suspend = psci_cpu_suspend;

        psci_function_id[PSCI_FN_CPU_OFF] = PSCI_0_2_FN_CPU_OFF;
        psci_ops.cpu_off = psci_cpu_off;

        psci_function_id[PSCI_FN_CPU_ON] = PSCI_FN_NATIVE(0_2, CPU_ON);
        psci_ops.cpu_on = psci_cpu_on;

        psci_function_id[PSCI_FN_MIGRATE] = PSCI_FN_NATIVE(0_2, MIGRATE);
        psci_ops.migrate = psci_migrate;

        psci_ops.affinity_info = psci_affinity_info;

        psci_ops.migrate_info_type = psci_migrate_info_type;

        arm_pm_restart = psci_sys_reset;

        pm_power_off = psci_sys_poweroff;
}
```

### XEN Hypervisor (EL2)

psci call 을 받으면 가상화 환경이면 hypervisor에서 받게 된다. 아래 xen 코드에서 다음의 경로를 따라간다.

// xen/common/shutdown.c

```c
void hwdom_shutdown(u8 reason)
{
    switch ( reason )
    {
    case SHUTDOWN_poweroff:
        printk("Hardware Dom%u halted: halting machine\n",
               hardware_domain->domain_id);
        machine_halt();
        break; /* not reached */

    case SHUTDOWN_crash:
        debugger_trap_immediate();
        printk("Hardware Dom%u crashed: ", hardware_domain->domain_id);
        kexec_crash(CRASHREASON_HWDOM);
        maybe_reboot();
        break; /* not reached */

    case SHUTDOWN_reboot:
        printk("Hardware Dom%u shutdown: rebooting machine\n",
               hardware_domain->domain_id);
        machine_restart(0);
        break; /* not reached */
```

// xen/arch/arm/shutdown.c

```c
static void noreturn halt_this_cpu(void *arg)
{
    stop_cpu();
}

void machine_halt(void)
{
    int timeout = 10;

    watchdog_disable();
    console_start_sync();
    local_irq_enable();
    smp_call_function(halt_this_cpu, NULL, 0);
    local_irq_disable();

    /* Wait at most another 10ms for all other CPUs to go offline. */
    while ( (num_online_cpus() > 1) && (timeout-- > 0) )
        mdelay(1);

    /* This is mainly for PSCI-0.2, which does not return if success. */
    call_psci_system_off();

    /* Alternative halt procedure */
    platform_poweroff();
    halt_this_cpu(NULL);
}

void machine_restart(unsigned int delay_millisecs)
{
    int timeout = 10;
    unsigned long count = 0;

    watchdog_disable();
    console_start_sync();
    spin_debug_disable();

    local_irq_enable();
    smp_call_function(halt_this_cpu, NULL, 0);
    local_irq_disable();

    mdelay(delay_millisecs);

    /* Wait at most another 10ms for all other CPUs to go offline. */
    while ( (num_online_cpus() > 1) && (timeout-- > 0) )
        mdelay(1);

    /* This is mainly for PSCI-0.2, which does not return if success. */
    call_psci_system_reset();

    /* Alternative reset procedure */
    while ( 1 )
    {
        platform_reset();
        mdelay(100);
        if ( (count % 50) == 0 )
            printk(XENLOG_ERR "Xen: Platform reset did not work properly!\n");
        count++;
    }
}
```

### ARM Trusted Firmware (EL3)

hypervisor 에서 `call_psci_system_off();` 와 `call_psci_system_reset();` psci 를 호출하면 EL3 레벨로 넘어가게 된다. ARM Trusted Firmware 코드를 보면 아래로 간다.

https://github.com/ARM-software/arm-trusted-firmware

// lib/psci/psci_main.c

```c
u_register_t psci_smc_handler(uint32_t smc_fid,
			  u_register_t x1,
			  u_register_t x2,
			  u_register_t x3,
			  u_register_t x4,
			  void *cookie,
			  void *handle,
			  u_register_t flags)
{
    	// 생략
    	case PSCI_SYSTEM_OFF:
			psci_system_off();
			/* We should never return from psci_system_off() */
			break;

		case PSCI_SYSTEM_RESET:
			psci_system_reset();
			/* We should never return from psci_system_reset() */
			break;
```

https://github.com/ARM-software/arm-trusted-firmware/blob/master/lib/psci/psci_main.c

// lib/psci/psci_system_off.c

```c
void __dead2 psci_system_off(void)
{
	psci_print_power_domain_map();

	assert(psci_plat_pm_ops->system_off != NULL);

	/* Notify the Secure Payload Dispatcher */
	if ((psci_spd_pm != NULL) && (psci_spd_pm->svc_system_off != NULL)) {
		psci_spd_pm->svc_system_off();
	}

	console_flush();

	/* Call the platform specific hook */
	psci_plat_pm_ops->system_off();

	/* This function does not return. We should never get here */
}

void __dead2 psci_system_reset(void)
{
	psci_print_power_domain_map();

	assert(psci_plat_pm_ops->system_reset != NULL);

	/* Notify the Secure Payload Dispatcher */
	if ((psci_spd_pm != NULL) && (psci_spd_pm->svc_system_reset != NULL)) {
		psci_spd_pm->svc_system_reset();
	}

	console_flush();

	/* Call the platform specific hook */
	psci_plat_pm_ops->system_reset();

	/* This function does not return. We should never get here */
}
```

platform에서 직접 등록한 system_reset 이나 system_off 함수가 호출된다.

https://github.com/ARM-software/arm-trusted-firmware/blob/master/lib/psci/psci_system_off.c

### XEN Hypervisor (EL2)

그런데 만약 EL3 에서 저 함수들을 연결해두지 않아서 return 하고 다시 hypervisor로 넘어간다면? reboot은 `platform_reset();` 으로, shutdown은  `platform_poweroff();`로 간다. 

// xen/arch/arm/platform.c

```c
void platform_reset(void)
{
    if ( platform && platform->reset )
        platform->reset();
}

void platform_poweroff(void)
{
    if ( platform && platform->poweroff )
        platform->poweroff();
}
```

xen 코드 내에 platform 에서 등록한 `reset` 콜백과 `poweroff` 로 이어진다. 

reset 은 연결된 함수가 없어서 NULL이라면 영원히 시도하기만 하고, shutdown (poweroff)은  `halt_this_cpu(NULL);` 이 있어서 다시 EL3에 cpu off 를 요청하게 된다. 그러면 모든 cpu 가 off 된다. (사실 이거는 cpu만 off 하는 거니까 내가 원하는 power off 동작은 아닌 것 같다)

