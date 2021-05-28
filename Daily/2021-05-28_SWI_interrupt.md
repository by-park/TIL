# 2021-05-28 (SWI interrupt)

exception이 발생하면 exception vector 로 뛰게 된다.

https://kimbyungnam.github.io/os/2019/11/07/3.2/

exception vector는 vbar_el1 레지스터에 들어간다.

arch/arm64/kernel/head.S

```c
        adr_l   x8, vectors                     // load VBAR_EL1 with virtual
        msr     vbar_el1, x8                    // vector table address
```

vector table은 이렇게 만들어져있다

arch/arm64/kernel/entry.S

```c
/*
 * Exception vectors.
 */
        .pushsection ".entry.text", "ax"

        .align  11
SYM_CODE_START(vectors)
        kernel_ventry   1, sync_invalid                 // Synchronous EL1t
        kernel_ventry   1, irq_invalid                  // IRQ EL1t
        kernel_ventry   1, fiq_invalid                  // FIQ EL1t
        kernel_ventry   1, error_invalid                // Error EL1t

        kernel_ventry   1, sync                         // Synchronous EL1h
        kernel_ventry   1, irq                          // IRQ EL1h
        kernel_ventry   1, fiq_invalid                  // FIQ EL1h
        kernel_ventry   1, error                        // Error EL1h

        kernel_ventry   0, sync                         // Synchronous 64-bit EL
```

arch/arm64/kernel/entry.S

```c
/*
 * EL1 mode handlers.
 */
        .align  6
SYM_CODE_START_LOCAL_NOALIGN(el1_sync)
        kernel_entry 1
        mov     x0, sp
        bl      el1_sync_handler
        kernel_exit 1
SYM_CODE_END(el1_sync)

```
arch/arm64/kernel/entry-common.c
```c
asmlinkage void notrace el1_sync_handler(struct pt_regs *regs)
{
        unsigned long esr = read_sysreg(esr_el1);

        switch (ESR_ELx_EC(esr)) {
        case ESR_ELx_EC_DABT_CUR:
        case ESR_ELx_EC_IABT_CUR:
                el1_abort(regs, esr);
                break;
        /*
         * We don't handle ESR_ELx_EC_SP_ALIGN, since we will have hit a
         * recursive exception when trying to push the initial pt_regs.
         */
        case ESR_ELx_EC_PC_ALIGN:
                el1_pc(regs, esr);
                break;
        case ESR_ELx_EC_SYS64:
        case ESR_ELx_EC_UNKNOWN:
                el1_undef(regs);
                break;
        case ESR_ELx_EC_BREAKPT_CUR:
        case ESR_ELx_EC_SOFTSTP_CUR:
        case ESR_ELx_EC_WATCHPT_CUR:
        case ESR_ELx_EC_BRK64:
                el1_dbg(regs, esr);
                break;
        default:
                el1_inv(regs, esr);
        }
}
NOKPROBE_SYMBOL(el1_sync_handler);

```

arch/arm64/kernel/syscall.c:    el0_svc_common(regs, regs->regs[8], __NR_syscall
s, sys_call_table);

```c
static void el0_svc_common(struct pt_regs *regs, int scno, int sc_nr,
                           const syscall_fn_t syscall_table[])
{
        unsigned long flags = current_thread_info()->flags;

        regs->orig_x0 = regs->regs[0];
        regs->syscallno = scno;
...
        invoke_syscall(regs, scno, sc_nr, syscall_table);

        /*
         * The tracing status may have changed under our feet, so we have to
         * check again. However, if we were tracing entry, then we always trace
         * exit regardless, as the old entry assembly did.
         */
        if (!has_syscall_work(flags) && !IS_ENABLED(CONFIG_DEBUG_RSEQ)) {
                local_daif_mask();
                flags = current_thread_info()->flags;
                if (!has_syscall_work(flags) && !(flags & _TIF_SINGLESTEP)) {
                        /*
                         * We're off to userspace, where interrupts are
                         * always enabled after we restore the flags from
                         * the SPSR.
                         */
                        trace_hardirqs_on();
                        return;
                }
                local_daif_restore(DAIF_PROCCTX);
        }

trace_exit:
        syscall_trace_exit(regs);
}


void do_el0_svc(struct pt_regs *regs)
{
        sve_user_discard();
        el0_svc_common(regs, regs->regs[8], __NR_syscalls, sys_call_table);
}
```

arch/arm64/kernel/entry-common.c:       do_el0_svc(regs);

```c
static void notrace el0_svc(struct pt_regs *regs)
{
        if (system_uses_irq_prio_masking())
                gic_write_pmr(GIC_PRIO_IRQON | GIC_PRIO_PSR_I_SET);

        do_el0_svc(regs);
}
NOKPROBE_SYMBOL(el0_svc);
```

arch/arm64/kernel/entry-common.c

```c
asmlinkage void notrace el0_sync_handler(struct pt_regs *regs)
{
        unsigned long esr = read_sysreg(esr_el1);

        switch (ESR_ELx_EC(esr)) {
        case ESR_ELx_EC_SVC64:
                el0_svc(regs);
                break;
        case ESR_ELx_EC_DABT_LOW:
                el0_da(regs, esr);
                break;
        case ESR_ELx_EC_IABT_LOW:
                el0_ia(regs, esr);
                break;
```

코드를 보면 el0 에서 sys call 이 불리는 것으로 보인다.