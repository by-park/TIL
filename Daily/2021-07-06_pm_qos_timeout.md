# 2021-07-06 (pm qos timeout)

Linux kernel 4.14 버전에서 pm_qos_update_request_timeout 함수를 사용해보았다.

pm qos 를 사용하는 driver에서 pm_qos_add_request 후에 다른 driver에서 pm qos를 잡으면 pm qos 를 풀고 싶은데,  그때 사용할 수 있다. timeout 에 지정한 시간 후에는 지정한 시간 마이크로초 이후에 0으로 잡힌다. 

Linux kernel 5.9.11에서는 함수 형태가 좀 달라져서 해당 함수는 없었다. (대체재가 생긴 걸까?)

Q. 이런 버전별 변경 히스토리는 어디서 볼까?

> The changelogs are on [kernel.org](https://www.kernel.org/).
>
> The URLs have a predictable pattern. The current kernel change log is at: https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.17.8
>
> So, to read the changes from 4.17.1, you would go to: https://cdn.kernel.org/pub/linux/kernel/v4.x/ChangeLog-4.17.2

https://unix.stackexchange.com/questions/457345/where-to-find-the-linux-changelog-of-minor-versions

```c
/**
 * pm_qos_work_fn - the timeout handler of pm_qos_update_request_timeout
 * @work: work struct for the delayed work (timeout)
 *
 * This cancels the timeout request by falling back to the default at timeout.
 */
static void pm_qos_work_fn(struct work_struct *work)
{
	struct pm_qos_request *req = container_of(to_delayed_work(work),
						  struct pm_qos_request,
						  work);
	__pm_qos_update_request(req, PM_QOS_DEFAULT_VALUE);
}

void pm_qos_add_request(struct pm_qos_request *req,
			int pm_qos_class, s32 value)
{
	if (!req) /*guard against callers passing in null */
		return;
	if (pm_qos_request_active(req)) {
		WARN(1, KERN_ERR "pm_qos_add_request() called for already added request\n");
		return;
	}
	req->pm_qos_class = pm_qos_class;
	INIT_DELAYED_WORK(&req->work, pm_qos_work_fn);
	trace_pm_qos_add_request(pm_qos_class, value);
	pm_qos_update_target(pm_qos_array[pm_qos_class]->constraints,
			     &req->node, PM_QOS_ADD_REQ, value);
}
EXPORT_SYMBOL_GPL(pm_qos_add_request);

/**
 * pm_qos_update_request_timeout - modifies an existing qos request temporarily.
 * @req : handle to list element holding a pm_qos request to use
 * @new_value: defines the temporal qos request
 * @timeout_us: the effective duration of this qos request in usecs.
 *
 * After timeout_us, this qos request is cancelled automatically.
 */
void pm_qos_update_request_timeout(struct pm_qos_request *req, s32 new_value,
				   unsigned long timeout_us)
{
	if (!req)
		return;
	if (WARN(!pm_qos_request_active(req),
		 "%s called for unknown object.", __func__))
		return;
	cancel_delayed_work_sync(&req->work);
	trace_pm_qos_update_request_timeout(req->pm_qos_class,
					    new_value, timeout_us);
	if (new_value != req->node.prio)
		pm_qos_update_target(
			pm_qos_array[req->pm_qos_class]->constraints,
			&req->node, PM_QOS_UPDATE_REQ, new_value);
	schedule_delayed_work(&req->work, usecs_to_jiffies(timeout_us));
}
```

https://code.woboq.org/linux/linux/kernel/power/qos.c.html