# 2022-06-08 (Linux Wakelock)

Linux Kernel 에 있는 wakelock API

> **안드로이드 Suspend**
> 안드로이드용으로 패치된 커널에서는 Suspend가 되기 전(enter_state() 함수를 호출하기 전)에 kernel/power/earlysuspend.c 파일의 request_suspend_state()함수를 호출한다(안드로이드는 Early Suspend와 Wake Lock 기능을 커널에 추가했다). 이를 좀 더 자세한 이해를 위해 먼저 안드로이드가 추가한 새로운 기능에 대해서 소개하겠다.
>
> **파일:
> **  - linux_source/kernel/power/main.c
>   \- linux_source/kernel/power/earlysuspend.c
>   \- linux_source/kernel/power/wakelock.c

http://taehyo.egloos.com/v/4091452



리눅스 커널에서 잡혀있는 wakelock 리스트 보는 방법

```shell
# cat /sys/power/wake_lock
wakelock이름 wakelock이름 wakelock이름
```



// kernel/power/main.c

```c
#ifdef CONFIG_PM_WAKELOCKS
static ssize_t wake_lock_show(struct kobject *kobj,
                              struct kobj_attribute *attr,
                              char *buf)
{
        return pm_show_wakelocks(buf, true);
}
```

위의 command 를 입력하면 이와 같은 함수를 호출한다. 참고로 wakelock 기능을 사용하기 위해서는 위의 코드에서 보이듯이 `CONFIG_PM_WAKELOCKS` 를 enable 해야한다.



// kernel/power/wakelock.c

```c
ssize_t pm_show_wakelocks(char *buf, bool show_active)
{
        struct rb_node *node;
        struct wakelock *wl;
        char *str = buf;
        char *end = buf + PAGE_SIZE;

        mutex_lock(&wakelocks_lock);

        for (node = rb_first(&wakelocks_tree); node; node = rb_next(node)) {
                wl = rb_entry(node, struct wakelock, node);
                if (wl->ws->active == show_active)
                        str += scnprintf(str, end - str, "%s ", wl->name);
        }
        if (str > buf)
                str--;

        str += scnprintf(str, end - str, "\n");

        mutex_unlock(&wakelocks_lock);
        return (str - buf);
}
```

그러면 위와 같이 wakelock 을 확인하는 함수를 호출한다.