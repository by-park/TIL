# ARM64 초기화

### Linux Kernel

arch/arm64/kernel/head.S

https://github.com/torvalds/linux/blob/master/arch/arm64/kernel/head.S

```assembly
/*
 * The following fragment of code is executed with the MMU enabled.
 *
 *   x0 = __pa(KERNEL_START)
 */
SYM_FUNC_START_LOCAL(__primary_switched)
	adr_l	x4, init_task
	init_cpu_task x4, x5, x6

	adr_l	x8, vectors			// load VBAR_EL1 with virtual
	msr	vbar_el1, x8			// vector table address
	isb

	stp	x29, x30, [sp, #-16]!
	mov	x29, sp

	str_l	x21, __fdt_pointer, x5		// Save FDT pointer

	ldr_l	x4, kimage_vaddr		// Save the offset between
	sub	x4, x4, x0			// the kernel virtual and
	str_l	x4, kimage_voffset, x5		// physical mappings

	mov	x0, x20
	bl	set_cpu_boot_mode_flag

	// Clear BSS
	adr_l	x0, __bss_start
	mov	x1, xzr
	adr_l	x2, __bss_stop
	sub	x2, x2, x0
	bl	__pi_memset
	dsb	ishst				// Make zero page visible to PTW

#if VA_BITS > 48
	adr_l	x8, vabits_actual		// Set this early so KASAN early init
	str	x25, [x8]			// ... observes the correct value
	dc	civac, x8			// Make visible to booting secondaries
#endif

#ifdef CONFIG_RANDOMIZE_BASE
	adrp	x5, memstart_offset_seed	// Save KASLR linear map seed
	strh	w24, [x5, :lo12:memstart_offset_seed]
#endif
#if defined(CONFIG_KASAN_GENERIC) || defined(CONFIG_KASAN_SW_TAGS)
	bl	kasan_early_init
#endif
	mov	x0, x21				// pass FDT address in x0
	bl	early_fdt_map			// Try mapping the FDT early
	mov	x0, x20				// pass the full boot status
	bl	init_feature_override		// Parse cpu feature overrides
	mov	x0, x20
	bl	finalise_el2			// Prefer VHE if possible
	ldp	x29, x30, [sp], #16
	bl	start_kernel
	ASM_BUG()
SYM_FUNC_END(__primary_switched)
```

bl start_kernel 을 하면 아래가 실행됨

init/main.c

https://github.com/torvalds/linux/blob/master/init/main.c

```c
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
	char *command_line;
	char *after_dashes;

	set_task_stack_end_magic(&init_task);
	smp_setup_processor_id();
	debug_objects_early_init();
	init_vmlinux_build_id();

	cgroup_init_early();

	local_irq_disable();
	early_boot_irqs_disabled = true;
```



### Xen Hypervisor

xen/arch/arm/arm64/head.S

https://github.com/xen-project/xen/blob/master/xen/arch/arm/arm64/head.S

```assembly
GLOBAL(start)
        /*
         * DO NOT MODIFY. Image header expected by Linux boot-loaders.
         */
efi_head:
        /*
         * This add instruction has no meaningful effect except that
         * its opcode forms the magic "MZ" signature of a PE/COFF file
         * that is required for UEFI applications.
         */
        add     x13, x18, #0x16
        b       real_start           /* branch to kernel start */
```

real_start 진행

```c
real_start:
        /* BSS should be zeroed when booting without EFI */
        mov   x26, #0                /* x26 := skip_zero_bss */

real_start_efi:
        msr   DAIFSet, 0xf           /* Disable all interrupts */

        /* Save the bootloader arguments in less-clobberable registers */
        mov   x21, x0                /* x21 := DTB, physical address  */

        /* Find out where we are */
        ldr   x0, =start
        adr   x19, start             /* x19 := paddr (start) */
        sub   x20, x19, x0           /* x20 := phys-offset */

        /* Using the DTB in the .dtb section? */
.ifnes CONFIG_DTB_FILE,""
        load_paddr x21, _sdtb
.endif

        /* Initialize the UART if earlyprintk has been enabled. */
#ifdef CONFIG_EARLY_PRINTK
        bl    init_uart
#endif
        PRINT("- Boot CPU booting -\r\n")

        bl    check_cpu_mode
        bl    cpu_init
        bl    create_page_tables
        bl    enable_mmu

        /* We are still in the 1:1 mapping. Jump to the runtime Virtual Address. */
        ldr   x0, =primary_switched
        br    x0
primary_switched:
        /*
         * The 1:1 map may clash with other parts of the Xen virtual memory
         * layout. As it is not used anymore, remove it completely to
         * avoid having to worry about replacing existing mapping
         * afterwards.
         */
        bl    remove_identity_mapping
        bl    setup_fixmap
#ifdef CONFIG_EARLY_PRINTK
        /* Use a virtual address to access the UART. */
        ldr   x23, =EARLY_UART_VIRTUAL_ADDRESS
#endif
        bl    zero_bss
        PRINT("- Ready -\r\n")
        /* Setup the arguments for start_xen and jump to C world */
        mov   x0, x20                /* x0 := Physical offset */
        mov   x1, x21                /* x1 := paddr(FDT) */
        ldr   x2, =start_xen
        b     launch
ENDPROC(real_start)
            
launch:
        ldr   x3, =init_data
        add   x3, x3, #INITINFO_stack /* Find the boot-time stack */
        ldr   x3, [x3]
        add   x3, x3, #STACK_SIZE     /* (which grows down from the top). */
        sub   x3, x3, #CPUINFO_sizeof /* Make room for CPU save record */
        mov   sp, x3

        /* Jump to C world */
        br    x2
ENDPROC(launch)
```

br x2 를 통해 start_xen 이 호출됨

xen/arch/arm/setup.c

https://github.com/xen-project/xen/blob/master/xen/arch/arm/setup.c

```c
/* C entry point for boot CPU */
void __init start_xen(unsigned long boot_phys_offset,
                      unsigned long fdt_paddr)
{
    size_t fdt_size;
    const char *cmdline;
    struct bootmodule *xen_bootmodule;
    struct domain *d;
    int rc, i;

    dcache_line_bytes = read_dcache_line_bytes();

    percpu_init_areas();
    set_processor_id(0); /* needed early, for smp_processor_id() */

    setup_virtual_regions(NULL, NULL);
    /* Initialize traps early allow us to get backtrace when an error occurred */
    init_traps();

    setup_pagetables(boot_phys_offset);

    smp_clear_cpu_maps();

    device_tree_flattened = early_fdt_map(fdt_paddr);
    if ( !device_tree_flattened )
        panic("Invalid device tree blob at physical address %#lx.\n"
              "The DTB must be 8-byte aligned and must not exceed 2 MB in size.\n\n"
              "Please check your bootloader.\n",
              fdt_paddr);

    /* Register Xen's load address as a boot module. */
    xen_bootmodule = add_boot_module(BOOTMOD_XEN,
                             (paddr_t)(uintptr_t)(_start + boot_phys_offset),
                             (paddr_t)(uintptr_t)(_end - _start), false);
    BUG_ON(!xen_bootmodule);

    fdt_size = boot_fdt_info(device_tree_flattened, fdt_paddr);

    cmdline = boot_fdt_cmdline(device_tree_flattened);
    printk("Command line: %s\n", cmdline);
    cmdline_parse(cmdline);

    setup_mm();

    /* Parse the ACPI tables for possible boot-time configuration */
    acpi_boot_table_init();

    end_boot_allocator();

    /*
     * The memory subsystem has been initialized, we can now switch from
     * early_boot -> boot.
     */
    system_state = SYS_STATE_boot;

    vm_init();

    if ( acpi_disabled )
    {
        printk("Booting using Device Tree\n");
        device_tree_flattened = relocate_fdt(fdt_paddr, fdt_size);
        dt_unflatten_host_device_tree();
    }
    else
    {
        printk("Booting using ACPI\n");
        device_tree_flattened = NULL;
    }

    init_IRQ();

    platform_init();

    preinit_xen_time();

    gic_preinit();

    arm_uart_init();
    console_init_preirq();
    console_init_ring();

    processor_id();

    smp_init_cpus();
    nr_cpu_ids = smp_get_max_cpus();
    printk(XENLOG_INFO "SMP: Allowing %u CPUs\n", nr_cpu_ids);

    /*
     * Some errata relies on SMCCC version which is detected by psci_init()
     * (called from smp_init_cpus()).
     */
    check_local_cpu_errata();

    check_local_cpu_features();

    init_xen_time();

    gic_init();

    tasklet_subsys_init();

    if ( xsm_dt_init() != 1 )
        warning_add("WARNING: SILO mode is not enabled.\n"
                    "It has implications on the security of the system,\n"
                    "unless the communications have been forbidden between\n"
                    "untrusted domains.\n");

    init_maintenance_interrupt();
    init_timer_interrupt();

    timer_init();

    init_idle_domain();

    rcu_init();

    setup_system_domains();

    local_irq_enable();
    local_abort_enable();

    smp_prepare_cpus();

    initialize_keytable();

    console_init_postirq();

    do_presmp_initcalls();

    for_each_present_cpu ( i )
    {
        if ( (num_online_cpus() < nr_cpu_ids) && !cpu_online(i) )
        {
            int ret = cpu_up(i);
            if ( ret != 0 )
                printk("Failed to bring up CPU %u (error %d)\n", i, ret);
        }
    }

    printk("Brought up %ld CPUs\n", (long)num_online_cpus());
    /* TODO: smp_cpus_done(); */

    /* This should be done in a vpmu driver but we do not have one yet. */
    vpmu_is_available = cpu_has_pmu;

    /*
     * The IOMMU subsystem must be initialized before P2M as we need
     * to gather requirements regarding the maximum IPA bits supported by
     * each IOMMU device.
     */
    rc = iommu_setup();
    if ( !iommu_enabled && rc != -ENODEV )
        panic("Couldn't configure correctly all the IOMMUs.\n");

    setup_virt_paging();

    do_initcalls();

    /*
     * It needs to be called after do_initcalls to be able to use
     * stop_machine (tasklets initialized via an initcall).
     */
    apply_alternatives_all();
    enable_errata_workarounds();
    enable_cpu_features();

    /* Create initial domain 0. */
    if ( !is_dom0less_mode() )
        create_dom0();
    else
        printk(XENLOG_INFO "Xen dom0less mode detected\n");

    if ( acpi_disabled )
    {
        create_domUs();
        alloc_static_evtchn();
    }

    /*
     * This needs to be called **before** heap_init_late() so modules
     * will be scrubbed (unless suppressed).
     */
    discard_initial_modules();

    heap_init_late();

    init_trace_bufs();

    init_constructors();

    console_endboot();

    /* Hide UART from DOM0 if we're using it */
    serial_endboot();

    if ( (rc = xsm_set_system_active()) != 0 )
        panic("xsm: unable to switch to SYSTEM_ACTIVE privilege: %d\n", rc);

    system_state = SYS_STATE_active;

    for_each_domain( d )
        domain_unpause_by_systemcontroller(d);

    /* Switch on to the dynamically allocated stack for the idle vcpu
     * since the static one we're running on is about to be freed. */
    memcpy(idle_vcpu[0]->arch.cpu_info, get_cpu_info(),
           sizeof(struct cpu_info));
    switch_stack_and_jump(idle_vcpu[0]->arch.cpu_info, init_done);
}
```



### Little Kernel

arch/arm64/start.S

https://github.com/littlekernel/lk/blob/master/arch/arm64/start.S

```assembly
.section .text.boot
FUNCTION(_start)
.globl arm_reset
arm_reset:
    /* keep track of the boot EL */
    mrs     boot_el, currentel

    /* if we came in at higher than EL1, drop down to EL1 */
    bl      arm64_elX_to_el1

    /* disable EL1 FPU traps */
    mov     tmp, #(0b11<<20)
    msr     cpacr_el1, tmp

#if WITH_KERNEL_VM
    /* enable caches so atomics and spinlocks work */
    mrs     tmp, sctlr_el1
    orr     tmp, tmp, #(1<<12) /* Enable icache */
    orr     tmp, tmp, #(1<<2)  /* Enable dcache/ucache */
    orr     tmp, tmp, #(1<<3)  /* Enable Stack Alignment Check EL1 */
    orr     tmp, tmp, #(1<<4)  /* Enable Stack Alignment Check EL0 */
    bic     tmp, tmp, #(1<<1)  /* Disable Alignment Checking for EL1 EL0 */
    msr     sctlr_el1, tmp

    /* set up the mmu according to mmu_initial_mappings */

    /* load the base of the translation table and clear the table */
    adrp    page_table1, arm64_kernel_translation_table
    add     page_table1, page_table1, #:lo12:arm64_kernel_translation_table

#if WITH_SMP
    /* if the cpu id is != 0 it's a secondary cpu */
    mrs     cpuid, mpidr_el1
    ubfx    cpuid, cpuid, #0, #SMP_CPU_ID_BITS
    cbnz    cpuid, .Lmmu_enable_secondary

    /* this path forward until .Lmmu_enable_secondary is the primary cpu only */

#endif /* WITH_SMP */
#endif /* WITH_KERNEL_VM */

// 생략

#if WITH_SMP
    cbnz    cpuid, .Lsecondary_boot
#endif
#endif /* WITH_KERNEL_VM */

    /* load the stack pointer */
    ldr tmp, =__stack_end
    mov sp, tmp

    /* clear bss */
.L__do_bss:
    /* clear out the bss excluding the stack and kernel translation table  */
    /* NOTE: relies on __post_prebss_bss_start and __bss_end being 8 byte aligned */
    ldr     tmp, =__post_prebss_bss_start
    ldr     tmp2, =__bss_end
    sub     tmp2, tmp2, tmp
    cbz     tmp2, .L__bss_loop_done
.L__bss_loop:
    sub     tmp2, tmp2, #8
    str     xzr, [tmp], #8
    cbnz    tmp2, .L__bss_loop
.L__bss_loop_done:

    /* load the boot args we had saved previously */
    adrp    tmp, arm64_boot_args
    add     tmp, tmp, :lo12:arm64_boot_args
    ldp     x0, x1, [tmp], #16
    ldp     x2, x3, [tmp]

    bl  lk_main
    b   .
```

lk_main 으로 이동

top/main.c

https://github.com/littlekernel/lk/blob/master/top/main.c

```c
/* called from arch code */
void lk_main(ulong arg0, ulong arg1, ulong arg2, ulong arg3) {
    // save the boot args
    lk_boot_args[0] = arg0;
    lk_boot_args[1] = arg1;
    lk_boot_args[2] = arg2;
    lk_boot_args[3] = arg3;

    // get us into some sort of thread context
    thread_init_early();

    // early arch stuff
    lk_primary_cpu_init_level(LK_INIT_LEVEL_EARLIEST, LK_INIT_LEVEL_ARCH_EARLY - 1);
    arch_early_init();

    // do any super early platform initialization
    lk_primary_cpu_init_level(LK_INIT_LEVEL_ARCH_EARLY, LK_INIT_LEVEL_PLATFORM_EARLY - 1);
    platform_early_init();

    // do any super early target initialization
    lk_primary_cpu_init_level(LK_INIT_LEVEL_PLATFORM_EARLY, LK_INIT_LEVEL_TARGET_EARLY - 1);
    target_early_init();

#if WITH_SMP
    dprintf(INFO, "\nwelcome to lk/MP\n\n");
#else
    dprintf(INFO, "\nwelcome to lk\n\n");
#endif
    dprintf(INFO, "boot args 0x%lx 0x%lx 0x%lx 0x%lx\n",
            lk_boot_args[0], lk_boot_args[1], lk_boot_args[2], lk_boot_args[3]);

    // bring up the kernel heap
    lk_primary_cpu_init_level(LK_INIT_LEVEL_TARGET_EARLY, LK_INIT_LEVEL_HEAP - 1);
    dprintf(SPEW, "initializing heap\n");
    heap_init();

    // deal with any static constructors
    dprintf(SPEW, "calling constructors\n");
    call_constructors();

    // initialize the kernel
    lk_primary_cpu_init_level(LK_INIT_LEVEL_HEAP, LK_INIT_LEVEL_KERNEL - 1);
    kernel_init();

    lk_primary_cpu_init_level(LK_INIT_LEVEL_KERNEL, LK_INIT_LEVEL_THREADING - 1);

    // create a thread to complete system initialization
    dprintf(SPEW, "creating bootstrap completion thread\n");
    thread_t *t = thread_create("bootstrap2", &bootstrap2, NULL, DEFAULT_PRIORITY, DEFAULT_STACK_SIZE);
    thread_set_pinned_cpu(t, 0);
    thread_detach(t);
    thread_resume(t);

    // become the idle thread and enable interrupts to start the scheduler
    thread_become_idle();
}
```



### ARM trusted firmware

bl31/aarch64/bl31_entrypoint.S

https://github.com/renesas-rcar/arm-trusted-firmware/blob/rcar-s4_v2.5/bl31/aarch64/bl31_entrypoint.S

```assembly

	.globl	bl31_entrypoint
	.globl	bl31_warm_entrypoint

	/* -----------------------------------------------------
	 * bl31_entrypoint() is the cold boot entrypoint,
	 * executed only by the primary cpu.
	 * -----------------------------------------------------
	 */

func bl31_entrypoint
	/* ---------------------------------------------------------------
	 * Stash the previous bootloader arguments x0 - x3 for later use.
	 * ---------------------------------------------------------------
	 */
	mov	x20, x0
	mov	x21, x1
	mov	x22, x2
	mov	x23, x3

#if !RESET_TO_BL31
	/* ---------------------------------------------------------------------
	 * For !RESET_TO_BL31 systems, only the primary CPU ever reaches
	 * bl31_entrypoint() during the cold boot flow, so the cold/warm boot
	 * and primary/secondary CPU logic should not be executed in this case.
	 *
	 * Also, assume that the previous bootloader has already initialised the
	 * SCTLR_EL3, including the endianness, and has initialised the memory.
	 * ---------------------------------------------------------------------
	 */
	el3_entrypoint_common					\
		_init_sctlr=0					\
		_warm_boot_mailbox=0				\
		_secondary_cold_boot=0				\
		_init_memory=0					\
		_init_c_runtime=1				\
		_exception_vectors=runtime_exceptions		\
		_pie_fixup_size=BL31_LIMIT - BL31_BASE
#else

	/* ---------------------------------------------------------------------
	 * For RESET_TO_BL31 systems which have a programmable reset address,
	 * bl31_entrypoint() is executed only on the cold boot path so we can
	 * skip the warm boot mailbox mechanism.
	 * ---------------------------------------------------------------------
	 */
	el3_entrypoint_common					\
		_init_sctlr=1					\
		_warm_boot_mailbox=!PROGRAMMABLE_RESET_ADDRESS	\
		_secondary_cold_boot=!COLD_BOOT_SINGLE_CPU	\
		_init_memory=1					\
		_init_c_runtime=1				\
		_exception_vectors=runtime_exceptions		\
		_pie_fixup_size=BL31_LIMIT - BL31_BASE

	/* ---------------------------------------------------------------------
	 * For RESET_TO_BL31 systems, BL31 is the first bootloader to run so
	 * there's no argument to relay from a previous bootloader. Zero the
	 * arguments passed to the platform layer to reflect that.
	 * ---------------------------------------------------------------------
	 */
	mov	x20, 0
	mov	x21, 0
	mov	x22, 0
	mov	x23, 0
#endif /* RESET_TO_BL31 */

	/* --------------------------------------------------------------------
	 * Perform BL31 setup
	 * --------------------------------------------------------------------
	 */
	mov	x0, x20
	mov	x1, x21
	mov	x2, x22
	mov	x3, x23
	bl	bl31_setup

#if ENABLE_PAUTH
	/* --------------------------------------------------------------------
	 * Program APIAKey_EL1 and enable pointer authentication
	 * --------------------------------------------------------------------
	 */
	bl	pauth_init_enable_el3
#endif /* ENABLE_PAUTH */

	/* --------------------------------------------------------------------
	 * Jump to main function
	 * --------------------------------------------------------------------
	 */
	bl	bl31_main
```

bl bl31_main 이 호출되면 아래로 이동

bl31/bl31_main.c

https://github.com/renesas-rcar/arm-trusted-firmware/blob/rcar-s4_v2.5/bl31/bl31_main.c

```c
void bl31_main(void)
{
	NOTICE("BL31: %s\n", version_string);
	NOTICE("BL31: %s\n", build_message);

#ifdef SUPPORT_UNKNOWN_MPID
	if (unsupported_mpid_flag == 0) {
		NOTICE("Unsupported MPID detected!\n");
	}
#endif

	/* Perform platform setup in BL31 */
	bl31_platform_setup();

	/* Initialise helper libraries */
	bl31_lib_init();

#if EL3_EXCEPTION_HANDLING
	INFO("BL31: Initialising Exception Handling Framework\n");
	ehf_init();
#endif

	/* Initialize the runtime services e.g. psci. */
	INFO("BL31: Initializing runtime services\n");
	runtime_svc_init();

	/*
	 * All the cold boot actions on the primary cpu are done. We now need to
	 * decide which is the next image (BL32 or BL33) and how to execute it.
	 * If the SPD runtime service is present, it would want to pass control
	 * to BL32 first in S-EL1. In that case, SPD would have registered a
	 * function to initialize bl32 where it takes responsibility of entering
	 * S-EL1 and returning control back to bl31_main. Once this is done we
	 * can prepare entry into BL33 as normal.
	 */

	/*
	 * If SPD had registered an init hook, invoke it.
	 */
	if (bl32_init != NULL) {
		INFO("BL31: Initializing BL32\n");

		int32_t rc = (*bl32_init)();

		if (rc == 0)
			WARN("BL31: BL32 initialization failed\n");
	}
	/*
	 * We are ready to enter the next EL. Prepare entry into the image
	 * corresponding to the desired security state after the next ERET.
	 */
	bl31_prepare_next_image_entry();

	console_flush();

	/*
	 * Perform any platform specific runtime setup prior to cold boot exit
	 * from BL31
	 */
	bl31_plat_runtime_setup();
}
```



### U-boot

arch/arm/cpu/armv8/start.S

https://github.com/u-boot/u-boot/blob/master/arch/arm/cpu/armv8/start.S

```assembly
/*************************************************************************
 *
 * Startup Code (reset vector)
 *
 *************************************************************************/

.globl	_start
_start:
#if defined(CONFIG_LINUX_KERNEL_IMAGE_HEADER)
#include <asm/boot0-linux-kernel-header.h>
#elif defined(CONFIG_ENABLE_ARM_SOC_BOOT0_HOOK)
/*
 * Various SoCs need something special and SoC-specific up front in
 * order to boot, allow them to set that in their boot0.h file and then
 * use it here.
 */
#include <asm/arch/boot0.h>
#else
	b	reset
#endif

// 생략

reset:
	/* Allow the board to save important registers */
	b	save_boot_params
.globl	save_boot_params_ret
save_boot_params_ret:

#if CONFIG_POSITION_INDEPENDENT
	/* Verify that we're 4K aligned.  */
	adr	x0, _start
	ands	x0, x0, #0xfff
	b.eq	1f
0:
	/*
	 * FATAL, can't continue.
	 * U-Boot needs to be loaded at a 4K aligned address.
	 *
	 * We use ADRP and ADD to load some symbol addresses during startup.
	 * The ADD uses an absolute (non pc-relative) lo12 relocation
	 * thus requiring 4K alignment.
	 */
	wfi
	b	0b
1:

	/*
	 * Fix .rela.dyn relocations. This allows U-Boot to be loaded to and
	 * executed at a different address than it was linked at.
	 */
pie_fixup:
	adr	x0, _start		/* x0 <- Runtime value of _start */
	ldr	x1, _TEXT_BASE		/* x1 <- Linked value of _start */
	subs	x9, x0, x1		/* x9 <- Run-vs-link offset */
	beq	pie_fixup_done
	adrp    x2, __rel_dyn_start     /* x2 <- Runtime &__rel_dyn_start */
	add     x2, x2, #:lo12:__rel_dyn_start
	adrp    x3, __rel_dyn_end       /* x3 <- Runtime &__rel_dyn_end */
	add     x3, x3, #:lo12:__rel_dyn_end
pie_fix_loop:
	ldp	x0, x1, [x2], #16	/* (x0, x1) <- (Link location, fixup) */
	ldr	x4, [x2], #8		/* x4 <- addend */
	cmp	w1, #1027		/* relative fixup? */
	bne	pie_skip_reloc
	/* relative fix: store addend plus offset at dest location */
	add	x0, x0, x9
	add	x4, x4, x9
	str	x4, [x0]
pie_skip_reloc:
	cmp	x2, x3
	b.lo	pie_fix_loop
pie_fixup_done:
#endif

#if defined(CONFIG_ARMV8_SPL_EXCEPTION_VECTORS) || !defined(CONFIG_SPL_BUILD)
.macro	set_vbar, regname, reg
	msr	\regname, \reg
.endm
	adr	x0, vectors
#else
.macro	set_vbar, regname, reg
.endm
#endif
	/*
	 * Could be EL3/EL2/EL1, Initial State:
	 * Little Endian, MMU Disabled, i/dCache Disabled
	 */
	switch_el x1, 3f, 2f, 1f
3:	set_vbar vbar_el3, x0
	mrs	x0, scr_el3
	orr	x0, x0, #0xf			/* SCR_EL3.NS|IRQ|FIQ|EA */
	msr	scr_el3, x0
	msr	cptr_el3, xzr			/* Enable FP/SIMD */
	b	0f
2:	mrs	x1, hcr_el2
	tbnz	x1, #HCR_EL2_E2H_BIT, 1f	/* HCR_EL2.E2H */
	orr	x1, x1, #HCR_EL2_AMO_EL2	/* Route SErrors to EL2 */
	msr	hcr_el2, x1
	set_vbar vbar_el2, x0
	mov	x0, #0x33ff
	msr	cptr_el2, x0			/* Enable FP/SIMD */
	b	0f
1:	set_vbar vbar_el1, x0
	mov	x0, #3 << 20
	msr	cpacr_el1, x0			/* Enable FP/SIMD */
0:
	msr	daifclr, #0x4			/* Unmask SError interrupts */

#if CONFIG_COUNTER_FREQUENCY
	branch_if_not_highest_el x0, 4f
	ldr	x0, =CONFIG_COUNTER_FREQUENCY
	msr	cntfrq_el0, x0			/* Initialize CNTFRQ */
#endif

4:	isb

	/*
	 * Enable SMPEN bit for coherency.
	 * This register is not architectural but at the moment
	 * this bit should be set for A53/A57/A72.
	 */
#ifdef CONFIG_ARMV8_SET_SMPEN
	switch_el x1, 3f, 1f, 1f
3:
	mrs     x0, S3_1_c15_c2_1               /* cpuectlr_el1 */
	orr     x0, x0, #0x40
	msr     S3_1_c15_c2_1, x0
	isb
1:
#endif

	/* Apply ARM core specific erratas */
	bl	apply_core_errata

	/*
	 * Cache/BPB/TLB Invalidate
	 * i-cache is invalidated before enabled in icache_enable()
	 * tlb is invalidated before mmu is enabled in dcache_enable()
	 * d-cache is invalidated before enabled in dcache_enable()
	 */

	/* Processor specific initialization */
	bl	lowlevel_init

#if defined(CONFIG_ARMV8_SPIN_TABLE) && !defined(CONFIG_SPL_BUILD)
	branch_if_master x0, master_cpu
	b	spin_table_secondary_jump
	/* never return */
#elif defined(CONFIG_ARMV8_MULTIENTRY)
	branch_if_master x0, master_cpu

	/*
	 * Slave CPUs
	 */
slave_cpu:
	wfe
	ldr	x1, =CPU_RELEASE_ADDR
	ldr	x0, [x1]
	cbz	x0, slave_cpu
	br	x0			/* branch to the given address */
#endif /* CONFIG_ARMV8_MULTIENTRY */
master_cpu:
	msr	SPSel, #1		/* make sure we use SP_ELx */
	bl	_main

```

_main 이 실행되면 아래로 이동

```assembly
ENTRY(_main)

/*
 * Set up initial C runtime environment and call board_init_f(0).
 */
#if defined(CONFIG_TPL_BUILD) && defined(CONFIG_TPL_NEEDS_SEPARATE_STACK)
	ldr	x0, =(CONFIG_TPL_STACK)
#elif defined(CONFIG_SPL_BUILD) && defined(CONFIG_SPL_STACK)
	ldr	x0, =(CONFIG_SPL_STACK)
#elif defined(CONFIG_INIT_SP_RELATIVE)
#if CONFIG_POSITION_INDEPENDENT
	adrp	x0, __bss_start     /* x0 <- Runtime &__bss_start */
	add	x0, x0, #:lo12:__bss_start
#else
	adr	x0, __bss_start
#endif
	add	x0, x0, #CONFIG_SYS_INIT_SP_BSS_OFFSET
#else
	ldr	x0, =(SYS_INIT_SP_ADDR)
#endif
	bic	sp, x0, #0xf	/* 16-byte alignment for ABI compliance */
	mov	x0, sp
	bl	board_init_f_alloc_reserve
	mov	sp, x0
	/* set up gd here, outside any C code */
	mov	x18, x0
	bl	board_init_f_init_reserve

#if defined(CONFIG_DEBUG_UART) && CONFIG_IS_ENABLED(SERIAL)
	bl	debug_uart_init
#endif

	mov	x0, #0
	bl	board_init_f
```

uboot 코드 분석

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kangyunmoon&logNo=220735281091

		Timer summary in microseconds:
		       Mark    Elapsed  Stage
			  0          0  reset
		  3,575,678  3,575,678  board_init_f start
		  3,575,695         17  arch_cpu_init A9
		  3,575,777         82  arch_cpu_init done
		  3,659,598     83,821  board_init_r start
		  3,910,375    250,777  main_loop
		 29,916,167 26,005,792  bootm_start
		 30,361,327    445,160  start_kernel
### * QEMU 로 디버깅

https://m.blog.naver.com/rlawlxo1064/221621552737

https://elinux.org/Kernel_Debugging_Tips

http://issaris.blogspot.com/2007/12/download-linux-kernel-sourcecode-from.html

![img](http://4.bp.blogspot.com/_5GXxEBP1qng/R2KpI7bw5kI/AAAAAAAAAEg/fnAIu_T3TeI/s1600/screenshot025.png)