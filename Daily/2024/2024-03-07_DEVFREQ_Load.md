# 2024-03-07 (DEVFREQ Load)

Linux Kernel 6.1 기준 exynos5422의 load 활용법 조사



// drivers/memory/samsung/exynos5422-dmc.c

```c
static struct devfreq_dev_profile exynos5_dmc_df_profile = {
        .timer = DEVFREQ_TIMER_DELAYED,
        .target = exynos5_dmc_target,
        .get_dev_status = exynos5_dmc_get_status,
        .get_cur_freq = exynos5_dmc_get_cur_freq,
};
```

cpufreq 이 아닌 devfreq 임



// drivers/devfreq/governor_simpleondemand.c

ondemand governor 를 사용하면 정기적으로 업데이트하도록 설정할 수 있음

```c
static int devfreq_simple_ondemand_func(struct devfreq *df,
                                        unsigned long *freq)
{
        int err;
        struct devfreq_dev_status *stat;
        unsigned long long a, b;
        unsigned int dfso_upthreshold = DFSO_UPTHRESHOLD;
        unsigned int dfso_downdifferential = DFSO_DOWNDIFFERENCTIAL;
        struct devfreq_simple_ondemand_data *data = df->data;

        err = devfreq_update_stats(df);
```



devfreq_update_stats 에서 get_dev_status 콜백 호출

// drivers/devfreq/governor.h

```c
static inline int devfreq_update_stats(struct devfreq *df)
{
        if (!df->profile->get_dev_status)
                return -EINVAL;

        return df->profile->get_dev_status(df->dev.parent, &df->last_status);
}
```



exynos5_dmc_get_status

=> exynos5_counters_get

=> devfreq_event_get_event (load_count 와 total_count 를 얻음)



// drivers/memory/samsung/exynos5422-dmc.c

```c
static int exynos5_counters_get(struct exynos5_dmc *dmc,
                                unsigned long *load_count,
                                unsigned long *total_count)
{
        unsigned long total = 0;
        struct devfreq_event_data event;
        int ret, i;

        *load_count = 0;

        /* Take into account only read+write counters, but stop all */
        for (i = 0; i < dmc->num_counters; i++) {
                if (!dmc->counter[i])
                        continue;

                ret = devfreq_event_get_event(dmc->counter[i], &event);
                if (ret < 0)
                        return ret;

                *load_count += event.load_count;

                if (total < event.total_count)
                        total = event.total_count;
        }

        *total_count = total;

        return 0;
}

```



devfreq_event_get_event 호출 했을 때 PPMU 를 멈추고 total count 와 load count 를 읽는다

// .drivers/devfreq/event/exynos-ppmu.c

```c
static int exynos_ppmu_v2_get_event(struct devfreq_event_dev *edev,
                                    struct devfreq_event_data *edata)
{
        struct exynos_ppmu *info = devfreq_event_get_drvdata(edev);
        int id = exynos_ppmu_find_ppmu_id(edev);
        int ret;
        unsigned int pmnc, cntenc;
        unsigned int pmcnt_high, pmcnt_low;
        unsigned int total_count, count;
        unsigned long load_count = 0;

        /* Disable PPMU */
        ret = regmap_read(info->regmap, PPMU_V2_PMNC, &pmnc);
        if (ret < 0)
                return ret;

        pmnc &= ~PPMU_PMNC_ENABLE_MASK;
        ret = regmap_write(info->regmap, PPMU_V2_PMNC, pmnc);
        if (ret < 0)
                return ret;

        /* Read cycle count and performance count */
        ret = regmap_read(info->regmap, PPMU_V2_CCNT, &total_count);
        if (ret < 0)
                return ret;
        edata->total_count = total_count;

        switch (id) {
        case PPMU_PMNCNT0:
        case PPMU_PMNCNT1:
        case PPMU_PMNCNT2:
                ret = regmap_read(info->regmap, PPMU_V2_PMNCT(id), &count);
                if (ret < 0)
                        return ret;
                load_count = count;
                break;
```



exynos5_dmc_get_status 에서는 exynos5_counters_get 이후에 exynos5_counters_set_event 를 호출한다.



exynos5_counters_set_event

=> devfreq_event_set_event

=> edev->desc->ops->set_event(edev);

=> exynos_ppmu_v2_set_event



// drivers/devfreq/devfreq-event.c

```c
int devfreq_event_set_event(struct devfreq_event_dev *edev)
{
        int ret;

        if (!edev || !edev->desc)
                return -EINVAL;

        if (!edev->desc->ops || !edev->desc->ops->set_event)
                return -EINVAL;

        if (!devfreq_event_is_enabled(edev))
                return -EPERM;

        mutex_lock(&edev->lock);
        ret = edev->desc->ops->set_event(edev);
        mutex_unlock(&edev->lock);

        return ret;
}
EXPORT_SYMBOL_GPL(devfreq_event_set_event);
```



// drivers/devfreq/event/exynos-ppmu.c

```c
static int exynos_ppmu_v2_set_event(struct devfreq_event_dev *edev)
{
        struct exynos_ppmu *info = devfreq_event_get_drvdata(edev);
        unsigned int pmnc, cntens;
        int id = exynos_ppmu_find_ppmu_id(edev);
        int ret;

        /* Enable all counters */
        ret = regmap_read(info->regmap, PPMU_V2_CNTENS, &cntens);
        if (ret < 0)
                return ret;

        cntens |= (PPMU_CCNT_MASK | (PPMU_ENABLE << id));
        ret = regmap_write(info->regmap, PPMU_V2_CNTENS, cntens);
        if (ret < 0)
                return ret;

        /* Set the event of proper data type monitoring */
        ret = regmap_write(info->regmap, PPMU_V2_CH_EVx_TYPE(id),
                           edev->desc->event_type);
        if (ret < 0)
                return ret;

        /* Reset cycle counter/performance counter and enable PPMU */
        ret = regmap_read(info->regmap, PPMU_V2_PMNC, &pmnc);
        if (ret < 0)
                return ret;
```

