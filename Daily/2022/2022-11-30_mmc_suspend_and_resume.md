# 2022-11-30 (mmc suspend and resume)

### mmc suspend and resume

kernel 에서 mmc suspend 와 resume 이 어떻게 동작하는지 확인했다. 로그를 보면서 생긴 궁금증은 mmc 초기화가 왜 suspend 를 들어갈 때 발생하는가이다. (mmc_init_card 에서 찍는 초기화 로그가 suspend 들어갈 때 초입에 찍혔다.)

1. suspend

kernel 이 suspend 에 진입할 때, `suspend_devices_and_enter(state);` 를 호출하면 그 안에서 결국 mmc_suspend 콜백이 불리게 된다. (kernel 4.14 버전에서는 sys_sync() 였는데, 아래 5.9.11 버전에서는 ksys_sync_helper로 바뀌었다. CONFIG_SUSPEND_SKIP 도 sync_on_suspend_enabled 로 바뀌었다.)

kernel/power/suspend.c

```c
        if (sync_on_suspend_enabled) {
                trace_suspend_resume(TPS("sync_filesystems"), 0, true);
                ksys_sync_helper();
                trace_suspend_resume(TPS("sync_filesystems"), 0, false);
        }

        pm_pr_dbg("Preparing system for sleep (%s)\n", mem_sleep_labels[state]);
        pm_suspend_clear_flags();
        error = suspend_prepare(state);
        if (error)
                goto Unlock;

        if (suspend_test(TEST_FREEZER))
                goto Finish;

        trace_suspend_resume(TPS("suspend_enter"), state, false);
        pm_pr_dbg("Suspending system (%s)\n", mem_sleep_labels[state]);
        pm_restrict_gfp_mask();
        error = suspend_devices_and_enter(state);
        pm_restore_gfp_mask();

 Finish:
        events_check_enabled = false;
        pm_pr_dbg("Finishing wakeup.\n");
        suspend_finish();
 Unlock:
        mutex_unlock(&system_transition_mutex);
        return error;
}
```

그러면 mmc_suspend 에 진입한다. 

drivers/mmc/core/mmc.c

```c
/*
 * Suspend callback
 */
static int mmc_suspend(struct mmc_host *host)
{
        int err;

        err = _mmc_suspend(host, true);
        if (!err) {
                pm_runtime_disable(&host->card->dev);
                pm_runtime_set_suspended(&host->card->dev);
        }

        return err;
}

static const struct mmc_bus_ops mmc_ops = {
        .remove = mmc_remove,
        .detect = mmc_detect,
        .suspend = mmc_suspend,
        .resume = mmc_resume,
        .runtime_suspend = mmc_runtime_suspend,
        .runtime_resume = mmc_runtime_resume,
        .alive = mmc_alive,
        .shutdown = mmc_shutdown,
        .hw_reset = _mmc_hw_reset,
};
```

그 내부에서 _mmc_suspend 를 호출한다. 그러면 그 안에서 `mmc_card_set_suspended(host->card);` 를 호출한다. 

drivers/mmc/core/mmc.c

```c
static int _mmc_suspend(struct mmc_host *host, bool is_suspend)
{
        int err = 0;
        unsigned int notify_type = is_suspend ? EXT_CSD_POWER_OFF_SHORT :
                                        EXT_CSD_POWER_OFF_LONG;

        mmc_claim_host(host);

        if (mmc_card_suspended(host->card))
                goto out;

        err = mmc_flush_cache(host->card);
        if (err)
                goto out;

        if (mmc_can_poweroff_notify(host->card) &&
            ((host->caps2 & MMC_CAP2_FULL_PWR_CYCLE) || !is_suspend ||
             (host->caps2 & MMC_CAP2_FULL_PWR_CYCLE_IN_SUSPEND)))
                err = mmc_poweroff_notify(host->card, notify_type);
        else if (mmc_can_sleep(host->card))
                err = mmc_sleep(host);
        else if (!mmc_host_is_spi(host))
                err = mmc_deselect_cards(host);

        if (!err) {
                mmc_power_off(host);
                mmc_card_set_suspended(host->card);
        }
out:
        mmc_release_host(host);
        return err;
}

```

`mmc_card_set_suspended` 에서는 state 에 MMC_STATE_SUSPENDED 를 체크한다.

drivers/mmc/core/card.h

```c
#define mmc_card_present(c)     ((c)->state & MMC_STATE_PRESENT)
#define mmc_card_readonly(c)    ((c)->state & MMC_STATE_READONLY)
#define mmc_card_blockaddr(c)   ((c)->state & MMC_STATE_BLOCKADDR)
#define mmc_card_ext_capacity(c) ((c)->state & MMC_CARD_SDXC)
#define mmc_card_removed(c)     ((c) && ((c)->state & MMC_CARD_REMOVED))
#define mmc_card_suspended(c)   ((c)->state & MMC_STATE_SUSPENDED)

#define mmc_card_set_present(c) ((c)->state |= MMC_STATE_PRESENT)
#define mmc_card_set_readonly(c) ((c)->state |= MMC_STATE_READONLY)
#define mmc_card_set_blockaddr(c) ((c)->state |= MMC_STATE_BLOCKADDR)
#define mmc_card_set_ext_capacity(c) ((c)->state |= MMC_CARD_SDXC)
#define mmc_card_set_removed(c) ((c)->state |= MMC_CARD_REMOVED)
#define mmc_card_set_suspended(c) ((c)->state |= MMC_STATE_SUSPENDED)
#define mmc_card_clr_suspended(c) ((c)->state &= ~MMC_STATE_SUSPENDED)
```



2. resume

kernel 이 suspend 에서 resume 하게 되면 driver 들의 resume 을 호출한다. 아래의 suspend_ops->enter 로 suspend 에 진입한 후, 그 뒤는 깨어나서 resume 하는 함수들이다.

```c
        error = syscore_suspend();
        if (!error) {
                *wakeup = pm_wakeup_pending();
                if (!(suspend_test(TEST_CORE) || *wakeup)) {
                        trace_suspend_resume(TPS("machine_suspend"),
                                state, true);
                        error = suspend_ops->enter(state);
                        trace_suspend_resume(TPS("machine_suspend"),
                                state, false);
                } else if (*wakeup) {
                        error = -EBUSY;
                }
                syscore_resume();
        }

        system_state = SYSTEM_RUNNING;

        arch_suspend_enable_irqs();
        BUG_ON(irqs_disabled());

 Enable_cpus:
        suspend_enable_secondary_cpus();

 Platform_wake:
        platform_resume_noirq(state);

 Devices_wake:
        dpm_resume_noirq(PMSG_RESUME);

 Platform_early_resume:
        platform_resume_early(state);

 Devices_early_resume:
        dpm_resume_early(PMSG_RESUME);

 Platform_finish:
        platform_resume_finish(state);
        return error;
}
```

driver 들의 resume 들이 호출되면서 mmc_resume 도 호출된다.

drivers/mmc/core/mmc.c

```c
/*
 * Callback for resume.
 */
static int mmc_resume(struct mmc_host *host)
{
        pm_runtime_enable(&host->card->dev);
        return 0;
}

```

이때는 runtime 콜백만 enable 시키는 것을 볼 수 있다. 그 외에 다른 동작은 하지 않는다. 그래서 kernel 에서 suspend & resume 을 시켜보면 mmc resume 관련 로그가 보이지 않는 것을 알 수 있다.

그러면 mmc 의 resume 동작은 언제 일어나는가? 답은 mmc 를 사용해야할 때였다.

resume 직후 바로 suspend 를 들어가게 시켜보면 다음과 같은 순서로 진행되었다. (문제 상황에서의 디버깅용이라 마지막 부분의 scheduling 은 빼고 봐야한다.) kernel 4.14 버전에서의 Call trace 이다.

suspend 를 진입할 때, sys_sync 가 호출되는 것을 볼 수 있다. 이 동작은 suspend 를 들어가기 전에 memory 에 있는 정보들을 storage 에 반영하는 (sync) 동작이다.

Call trace:

pm_suspend -> enter_state -> sys_sync -> ext4_sync_fs -> blkdev_issue_flush -> submit_bio_wait -> wait_for_completion_io -> wait_for_common_io.constprop.2 -> io_schedule_timeout -> schedule_timeout -> schedule -> \_\_schedule -> __switch_to

그러면 mmc queue 가 불리게 된다. kernel 4.14 버전에서의 Call trace 는 다음과 같다. 그리고 위의 sys_sync 와 다른 cpu 번호가 사용된다. (2개로 테스트해보면 항상 고정된 cpu 번호는 아니었다. 2개 중 하나가 1번이면 하나가 0번이었다.)

Call trace:

ret_from_fork -> kthread -> mmc_queue_thread -> mmc_blk_issue_rq -> mmc_get_card -> \_\_pm_runtime_resume -> rpm_resume -> rpm_callback -> \_\_rpm_callback -> \_\_rpm_callback -> mmc_runtime_resume -> _mmc_resume -> mmc_init_card -> mmc_select_hs400 -> mmc_set_bus_speed -> mmc_set_clock -> dw_mci_set_ios -> dw_mci_setup_bus -> _dev_info -> __\_\_dev_printk -> dev_printk_emit -> dev_vprintk_emit -> vprintk_emit -> \_\_switch_to  

kernel 5.9.11 은 함수 명이 달라진 것 같다. 

drivers/mmc/core/queue.c

```c
static blk_status_t mmc_mq_queue_rq(struct blk_mq_hw_ctx *hctx,
                                    const struct blk_mq_queue_data *bd)
{
        struct request *req = bd->rq;
        struct request_queue *q = req->q;
        struct mmc_queue *mq = q->queuedata;
        struct mmc_card *card = mq->card;
        struct mmc_host *host = card->host;
        enum mmc_issue_type issue_type;
        enum mmc_issued issued;
        bool get_card, cqe_retune_ok;
        int ret;
// 생략
        if (get_card)
            mmc_get_card(card, &mq->ctx);

```

mmc_get_card 를 호출하면 `pm_runtime_get_sync` 가 호출된다. `__mmc_claim_host` 안에서도 한번 더 호출되던데 2번이 필요한 이유는 모르겠다.

https://github.com/torvalds/linux/blob/master/drivers/mmc/core/core.c

```c
void mmc_get_card(struct mmc_card *card, struct mmc_ctx *ctx)
{
        pm_runtime_get_sync(&card->dev);
        __mmc_claim_host(card->host, ctx, NULL);
}
EXPORT_SYMBOL(mmc_get_card);
```

include/linux/pm_runtime.h

```c
static inline int pm_runtime_get_sync(struct device *dev)
{
        return __pm_runtime_resume(dev, RPM_GET_PUT);
}
```

`__pm_runtime_resume` 이 호출되면 그제서야 `_mmc_resume` 이 호출된다.

drivers/mmc/core/mmc.c

```c
/*
 * This function tries to determine if the same card is still present
 * and, if so, restore all state to it.
 */
static int _mmc_resume(struct mmc_host *host)
{
        int err = 0;

        mmc_claim_host(host);

        if (!mmc_card_suspended(host->card))
                goto out;

        mmc_power_up(host, host->card->ocr);
        err = mmc_init_card(host, host->card->ocr, host->card);
        mmc_card_clr_suspended(host->card);

out:
        mmc_release_host(host);
        return err;
}
```

내부에서 `mmc_card_suspended` 상태를 확인하고 `mmc_init_card` 를 호출하는 것을 볼 수 있다. mmc_resume 을 어떻게 시키는가가 궁금했는데, 정답은 mmc 동작이 필요할 때마다였다.



### CPU Hard Lock up

CPU 0 번에서 hard lock up 이 걸렸다. (20초 동안 아무 동작을 하지 않으면 다른 CPU 가 이를 detect 할 수 있다.)

resume 직후 suspend 를 들어가던 상황이었고, 출력된 로그로 미루어 보면 suspend 동작이 CPU 1번, mmc 동작이 CPU 0번에서 이루어지고 있었다. 그러다가 인터럽트가 발생해 CPU 1번에 할당된 task 가 돌고 나서 CPU 0번에서 돌던 mmc 동작의 dev_info 에서 문제가 발생하였다. (mmc 동작을 CPU 1번의 call trace 에도 CPU 0번의 call trace 에도 보이는 것 같은데, 스케줄링 되면서 옮겨간 것 같다. 어느 CPU 에서 먼저 돌았는지 아직 정확히 모르겠다.) mmc 의 dev_info 로 printk 를 찍고 나면 unlock 을 해줘야하는데, 그 전에 다른 코어로 스케줄링이 되어버린 것이 문제인 것 같다. 코드상으로 unlock 은 내부 어딘가에서 CPU 마다 카운트를 관리하는 것처럼 보인다. 그래서 스케줄링 되면 안 되는 타이밍에 스케줄링 되어버려서 unlock 을 할 수 없게 된 것이 아닐까 싶다.

include/linux/device.h

https://github.com/torvalds/linux/blob/master/include/linux/device.h

```c
#define dev_info(dev,fmt,arg...) _dev_info(dev,fmt,##arg)
```

dev_info 안에 __dev_printk 가 호출된다.

```c
#define define_dev_printk_level(func, kern_level)		\
void func(const struct device *dev, const char *fmt, ...)	\
{								\
	struct va_format vaf;					\
	va_list args;						\
								\
	va_start(args, fmt);					\
								\
	vaf.fmt = fmt;						\
	vaf.va = &args;						\
								\
	__dev_printk(kern_level, dev, &vaf);			\
								\
	va_end(args);						\
}								\
EXPORT_SYMBOL(func);

define_dev_printk_level(_dev_emerg, KERN_EMERG);
define_dev_printk_level(_dev_alert, KERN_ALERT);
define_dev_printk_level(_dev_crit, KERN_CRIT);
define_dev_printk_level(_dev_err, KERN_ERR);
define_dev_printk_level(_dev_warn, KERN_WARNING);
define_dev_printk_level(_dev_notice, KERN_NOTICE);
define_dev_printk_level(_dev_info, KERN_INFO);

```

vprintk_emit 을 보면 console_unlock 전에 preempt_disable 을 하는 것을 볼 수 있다. 하지만 그 전에 스케줄링 되어버릴 가능성이 있다.

kernel/printk/printk.c

https://github.com/torvalds/linux/blob/master/kernel/printk/printk.c

```c
asmlinkage int vprintk_emit(int facility, int level,
			    const struct dev_printk_info *dev_info,
			    const char *fmt, va_list args)
{
	int printed_len;
	bool in_sched = false;

	/* Suppress unimportant messages after panic happens */
	if (unlikely(suppress_printk))
		return 0;

	if (unlikely(suppress_panic_printk) &&
	    atomic_read(&panic_cpu) != raw_smp_processor_id())
		return 0;

	if (level == LOGLEVEL_SCHED) {
		level = LOGLEVEL_DEFAULT;
		in_sched = true;
	}

	printk_delay(level);

	printed_len = vprintk_store(facility, level, dev_info, fmt, args);

	/* If called from the scheduler, we can not call up(). */
	if (!in_sched) {
		/*
		 * The caller may be holding system-critical or
		 * timing-sensitive locks. Disable preemption during
		 * printing of all remaining records to all consoles so that
		 * this context can return as soon as possible. Hopefully
		 * another printk() caller will take over the printing.
		 */
		preempt_disable();
		/*
		 * Try to acquire and then immediately release the console
		 * semaphore. The release will print out buffers. With the
		 * spinning variant, this context tries to take over the
		 * printing from another printing context.
		 */
		if (console_trylock_spinning())
			console_unlock();
		preempt_enable();
	}

	wake_up_klogd();
	return printed_len;
}
EXPORT_SYMBOL(vprintk_emit);
```

해당 파일의 git log 들을 보면 이 이슈와 관련한 해결 노력이 있었던 것 같다.

https://github.com/torvalds/linux/commit/a699449bb13b70b8bd10dc03ad7327ea3993221e



이번 이슈와 비슷하게 스케줄링으로 인해 printk 에 문제가 생겨서 적용된 해결패치들이다.

리뷰: https://lore.kernel.org/lkml/20180414030145.26304-1-sergey.senozhatsky@gmail.com/

패치: https://github.com/torvalds/linux/commit/43a17111c2553925f65e7be9b9c3f9d90cf29a8b

Commit 명: printk: wake up klogd in vprintk_emit



리뷰: https://lore.kernel.org/stable/20191127203116.455029038@linuxfoundation.org/

패치: https://github.com/torvalds/linux/commit/3ac37a93fa9217e576bebfd4ba3e80edaaeb2289

Commit 명: printk: lock/unlock console only for new logbuf entries

하지만 위 패치는 의미가 없어서 revert 되었다.

https://github.com/torvalds/linux/commit/8749efc0c0c325bf0c948c0b11d77bd3e497ead5



kernel hard lock up 시 보이는 panic 디버깅 로그는 왜 Call trace 가 무수히 많이 나오는지 잘 모르겠다.



CPU 0의 Call trace

el1_irq (panic시 로그 출력을 위한 것으로 추정된다.)

console_unlock.part.8

vprintk_emit

dev_vprintk_emit

dev_printk_emit

\_\_dev_printk

_dev_info

dw_mci_setup_bus

dw_mci_set_ios

mmc_set_clock

mmc_set_bus_speed

mmc_select_hs400

mmc_init_card

_mmc_resume

mmc_runtime_resume

mmc_runtime_resume

\_\_rpm_callback

rpm_callback

rpm_resume

__pm_runtime_resume

mmc_get_card

mmc_blk_issue_rq

mmc_queue_thread

kthread

ret_from_fork



CPU 1의 Call trace

\_\_switch_to

vprintk_emit

dev_vprintk_emit

dev_printk_emit

\_\_dev_printk

_dev_info

dw_mci_setup_bus

dw_mci_set_ios

mmc_set_clock

mmc_set_bus_speed

mmc_select_hs400

mmc_init_card

_mmc_resume

mmc_runtime_resume

mmc_runtime_resume (왜 2번이지?)

\_\_rpm_callback

rpm_callback

rpm_resume

\_\_pm_runtime_resume

mmc_get_card

mmc_blk_issue_rq

mmc_queue_thread

kthread

ret_from_fork



CPU 1의 다른 Call trace

\_\_switch_to

\_\_schedule

schedule

schedule_timeout

io_schedule_timeout

wait_for_common_io.constprop.2

wait_for_completion_io

submit_bio_wait

blkdev_issue_flush

ext4_sync_fs

sync_fs_one_sb

iterate_supers

sys_sync

enter_state

pm_suspend

state_store

kobj_attr_store

sysfs_kf_write

kernfs_fop_write

\_\_vfs_write

SyS_write



CPU 1번 다른 Call trace

el1_irq

\_\_switch_to

\_\_schedule

schedule

_synchronize_rcu_expedited

synchronize_rcu_expedited

synchronize_net

dev_deactivate_many

\_\_dev_close_many

dev_close_many

dev_close.part.19

dev_close

stmmac_service_task

process_one_work

worker_thread

kthread

ret_from_fork

