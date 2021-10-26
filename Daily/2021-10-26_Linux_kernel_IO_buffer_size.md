# 2021-10-26 (Linux kernel I/O buffer size)

Linux kernel 에서 sysfs 를 만들었는데, snprintf 안에 PAGE_SIZE 를 넣은 이유에 대해 질문이 들어왔다.

비슷한 코드 참고 (drivers/base/core.c)

```c
ssize_t device_show_ulong(struct device *dev,
                          struct device_attribute *attr,
                          char *buf)
{
        struct dev_ext_attribute *ea = to_ext_attr(attr);
        return snprintf(buf, PAGE_SIZE, "%lx\n", *(unsigned long *)(ea->var));
}
EXPORT_SYMBOL_GPL(device_show_ulong);
```



kernel I/O buffer size 가 검색해보니 default가 4K인 거 같다. 커널의 헤더를 찾아보면 4K로 보인다.

> **Default Buffer sizes:**
>
> - Buffer size only directly affects buffered mode
> - The default size like the kernel is based on the page size (4096 bytes on my system)
> - if stdin/stdout are connected to a terminal then default size = 1024; else size = 4096

http://www.pixelbeat.org/programming/stdio_buffering/



include/asm-generic/page.h 참고

```c
#define PAGE_SHIFT      12
#ifdef __ASSEMBLY__
#define PAGE_SIZE       (1 << PAGE_SHIFT)
#else
#define PAGE_SIZE       (1UL << PAGE_SHIFT)
#endif
#define PAGE_MASK       (~(PAGE_SIZE-1))
```



sysfs를 만들 때는 include/linux/device/class.h 참고 (이걸 이용하면 /sys/class 밑에 생긴다)

```c
#define CLASS_ATTR_RW(_name) \
        struct class_attribute class_attr_##_name = __ATTR_RW(_name)
#define CLASS_ATTR_RO(_name) \
        struct class_attribute class_attr_##_name = __ATTR_RO(_name)
#define CLASS_ATTR_WO(_name) \
        struct class_attribute class_attr_##_name = __ATTR_WO(_name)
```

include/linux/device.h 의 DEVICE_ATTR도 참고

