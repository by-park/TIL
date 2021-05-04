# 2021-05-04 (LK init hook)

Little kernel의 초기화 과정 중 init hook 에 대하여 정리해본다.

오늘 헷갈렸던 부분은 platform_init() 과 target_init() 사이 lk_primary_cpu_init_level이다. 여기서 hook에 등록된 함수들을 호출한다.

(Level이 뭐였는지 다시 확인이 필요할 것 같다!!)



top/main.c

(https://github.com/littlekernel/lk/blob/master/top/main.c)

```c
static int bootstrap2(void *arg) {
    dprintf(SPEW, "top of bootstrap2()\n");

    lk_primary_cpu_init_level(LK_INIT_LEVEL_THREADING, LK_INIT_LEVEL_ARCH - 1);
    arch_init();

    // initialize the rest of the platform
    dprintf(SPEW, "initializing platform\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_ARCH, LK_INIT_LEVEL_PLATFORM - 1);
    platform_init();

    // initialize the target
    dprintf(SPEW, "initializing target\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_PLATFORM, LK_INIT_LEVEL_TARGET - 1);
    target_init();

    dprintf(SPEW, "initializing apps\n");
    lk_primary_cpu_init_level(LK_INIT_LEVEL_TARGET, LK_INIT_LEVEL_APPS - 1);
    apps_init();

    lk_primary_cpu_init_level(LK_INIT_LEVEL_APPS, LK_INIT_LEVEL_LAST);

    return 0;
}
```



hook 등록은 여기서 이루어진다. Level은 threading으로 들어갔다.

lib/sysparam/sysparam.c

(https://github.com/littlekernel/lk/blob/master/lib/sysparam/sysparam.c)

```c
static void sysparam_init(uint level) {
    list_initialize(&params.list);
}

LK_INIT_HOOK(sysparam, &sysparam_init, LK_INIT_LEVEL_THREADING);
```



init level 은 다음과 같은 순서로 되어있다.

top/include/lk/init.h

(https://github.com/littlekernel/lk/blob/master/top/include/lk/init.h)

```c
enum lk_init_level {
    LK_INIT_LEVEL_EARLIEST = 1,

    LK_INIT_LEVEL_ARCH_EARLY     = 0x10000,
    LK_INIT_LEVEL_PLATFORM_EARLY = 0x20000,
    LK_INIT_LEVEL_TARGET_EARLY   = 0x30000,
    LK_INIT_LEVEL_HEAP           = 0x40000,
    LK_INIT_LEVEL_VM             = 0x50000,
    LK_INIT_LEVEL_KERNEL         = 0x60000,
    LK_INIT_LEVEL_THREADING      = 0x70000,
    LK_INIT_LEVEL_ARCH           = 0x80000,
    LK_INIT_LEVEL_PLATFORM       = 0x90000,
    LK_INIT_LEVEL_TARGET         = 0xa0000,
    LK_INIT_LEVEL_APPS           = 0xb0000,

    LK_INIT_LEVEL_LAST = UINT_MAX,
};
```



lk_primary_cpu_init_level은 lk_init_level 함수를 호출한다.

top/include/lk/init.h

(https://github.com/littlekernel/lk/blob/master/top/include/lk/init.h)

```c
static inline void lk_primary_cpu_init_level(uint start_level, uint stop_level) {
    lk_init_level(LK_INIT_FLAG_PRIMARY_CPU, start_level, stop_level);
}
```



lk_init_level에서는 hook 을 호출한다.

```c
void lk_init_level(enum lk_init_flags required_flag, uint start_level, uint stop_level) {
    LTRACEF("flags %#x, start_level %#x, stop_level %#x\n",
            required_flag, start_level, stop_level);

    ASSERT(start_level > 0);
    uint last_called_level = start_level - 1;
    const struct lk_init_struct *last = NULL;
    for (;;) {
        /* search for the lowest uncalled hook to call */
        LTRACEF("last %p, last_called_level %#x\n", last, last_called_level);

        const struct lk_init_struct *found = NULL;
        bool seen_last = false;
        for (const struct lk_init_struct *ptr = &__start_lk_init; ptr != &__stop_lk_init; ptr++) {
            LTRACEF("looking at %p (%s) level %#x, flags %#x, seen_last %d\n", ptr, ptr->name, ptr->level, ptr->flags, seen_last);

            if (ptr == last)
                seen_last = true;

            /* reject the easy ones */
            if (!(ptr->flags & required_flag))
                continue;
            if (ptr->level > stop_level)
                continue;
            if (ptr->level < last_called_level)
                continue;
            if (found && found->level <= ptr->level)
                continue;

            /* keep the lowest one we haven't called yet */
            if (ptr->level >= start_level && ptr->level > last_called_level) {
                found = ptr;
                continue;
            }

            /* if we're at the same level as the last one we called and we've
             * already passed over it this time around, we can mark this one
             * and early terminate the loop.
             */
            if (ptr->level == last_called_level && ptr != last && seen_last) {
                found = ptr;
                break;
            }
        }

        if (!found)
            break;

#if TRACE_INIT
        if (found->level >= EARLIEST_TRACE_LEVEL) {
            printf("INIT: cpu %d, calling hook %p (%s) at level %#x, flags %#x\n",
                   arch_curr_cpu_num(), found->hook, found->name, found->level, found->flags);
        }
#endif
        found->hook(found->level);
        last_called_level = found->level;
        last = found;
    }
}
```





little kernel hook 관련 게시글 (중국 블로그)

https://www.mrbluyee.com/2018/11/15/Little-Kernel-05/