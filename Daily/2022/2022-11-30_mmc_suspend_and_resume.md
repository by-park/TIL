# 2022-11-30 (mmc suspend and resume)

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

kernel 4.14 버전에서의 Call trace 는 다음과 같다.

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