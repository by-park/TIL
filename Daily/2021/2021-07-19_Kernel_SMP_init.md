# 2021-07-19 (Kernel SMP init)

kernel SMP init code

### start_kernel

init/main.c

```c
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
        char *command_line;
        char *after_dashes;

        set_task_stack_end_magic(&init_task);
        smp_setup_processor_id();
        debug_objects_early_init();

        cgroup_init_early();

        local_irq_disable();
        early_boot_irqs_disabled = true;

        /*
         * Interrupts are still disabled. Do necessary setups, then
         * enable them.
         */
        boot_cpu_init();
        page_address_init();
        pr_notice("%s", linux_banner);
        early_security_init();
        setup_arch(&command_line); // <========= here!
```



### smp_init_cpus

arch/arm64/kernel/setup.c

```c
void __init __no_sanitize_address setup_arch(char **cmdline_p)
{
        init_mm.start_code = (unsigned long) _text;
        init_mm.end_code   = (unsigned long) _etext;
        init_mm.end_data   = (unsigned long) _edata;
        init_mm.brk        = (unsigned long) _end;

        *cmdline_p = boot_command_line;
    // 생략
        if (acpi_disabled)
                psci_dt_init();
        else
                psci_acpi_init();

        init_bootcpu_ops();
        smp_init_cpus(); // <================ here!
        smp_build_mpidr_hash();


```



### smp_prepare_cpus

init/main.c

```c
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
        char *command_line;
        char *after_dashes;

        set_task_stack_end_magic(&init_task);
        smp_setup_processor_id();
        debug_objects_early_init();

        cgroup_init_early();

        local_irq_disable();
        early_boot_irqs_disabled = true;

        /*
         * Interrupts are still disabled. Do necessary setups, then
         * enable them.
         */
        boot_cpu_init();
        page_address_init();
        pr_notice("%s", linux_banner);
        early_security_init();
        setup_arch(&command_line);
        setup_boot_config(command_line);
        setup_command_line(command_line);
        setup_nr_cpu_ids();
        setup_per_cpu_areas();
        smp_prepare_boot_cpu(); /* arch-specific boot-cpu hooks */
        boot_cpu_hotplug_init();

    //생략
    	/* Do the rest non-__init'ed, we're now alive */
        arch_call_rest_init();  //<============== here
```

init/main.c

```c
void __init __weak arch_call_rest_init(void)
{
        rest_init();
}

noinline void __ref rest_init(void)
{
        struct task_struct *tsk;
        int pid;

        rcu_scheduler_starting();
        /*
         * We need to spawn init first so that it obtains pid 1, however
         * the init task will end up wanting to create kthreads, which, if
         * we schedule it before we create kthreadd, will OOPS.
         */
        pid = kernel_thread(kernel_init, NULL, CLONE_FS);
```
init/main.c
```c
        /*
         * init can allocate pages on any node
         */
        set_mems_allowed(node_states[N_MEMORY]);

        cad_pid = task_pid(current);

        smp_prepare_cpus(setup_max_cpus); // <========== here

        workqueue_init();

        init_mm_internals();

        do_pre_smp_initcalls();
        lockup_detector_init();

        smp_init(); // <======================= here
        sched_init_smp();

        padata_init();
        page_alloc_init_late();
        /* Initialize page ext after all struct pages are initialized. */
        page_ext_init();

        do_basic_setup();

        console_on_rootfs();

        /*
         * check if there is an early userspace init.  If yes, let it do all
         * the work
         */
        if (init_eaccess(ramdisk_execute_command) != 0) {
                ramdisk_execute_command = NULL;
                prepare_namespace();
        }

        /*
         * Ok, we have completed the initial bootup, and
         * we're essentially up and running. Get rid of the
         * initmem segments and start the user-mode stuff..
         *
         * rootfs is available now, try loading the public keys
         * and default modules
         */

        integrity_load_keys();
}

static int __ref kernel_init(void *unused)
{
        int ret;

        kernel_init_freeable();

```

