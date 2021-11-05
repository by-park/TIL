# 2021-11-05 (Linux kernel sysfs node)

Linux kernel 에서는 파일 시스템 I/O 접근하듯이 할 수 있도록 가상의 sysfs node를 지원한다.

만드는 방법을 정리해두려고 한다.



### (1) debugfs 만드는 방법

이것은 `/sys/kernel/debug/` 경로 밑에 생긴다.

mainline 코드 예시

drivers/base/power/domain.c

```c
#include <linux/device.h>
#include <linux/debugfs.h>

static struct dentry *genpd_debugfs_dir;

DEFINE_SHOW_ATTRIBUTE(summary);

static int __init genpd_debug_init(void)
{
        struct dentry *d;
        struct generic_pm_domain *genpd;

        genpd_debugfs_dir = debugfs_create_dir("pm_genpd", NULL);

        debugfs_create_file("pm_genpd_summary", S_IRUGO, genpd_debugfs_dir,
                            NULL, &summary_fops);
// 생략
}
```

include/linux/seq_file.h

```c
static const struct file_operations __name ## _fops = {                 \
        .owner          = THIS_MODULE,                                  \
        .open           = __name ## _open,                              \
        .read           = seq_read,                                     \
        .llseek         = seq_lseek,                                    \
        .release        = seq_release,                                  \
}

#define DEFINE_SHOW_ATTRIBUTE(__name)                                   \
static int __name ## _open(struct inode *inode, struct file *file)      \
{                                                                       \
        return single_open(file, __name ## _show, inode->i_private);    \
}                                                                       \                 
```

내가 만든 코드 예시 (`/sys/kernel/debug/my_debug_dir/sample_rw`)

```c
static void __init my_debugfs_init(void) {
    struct dentry *root, *sample;
    
    root = debugfs_create_dir("my_debug_dir", NULL);
    if (!root) {
        pr_err("%s %s: couldn't create debugfs dir\n", "my_debugfs", __func__);
        return;
    }
    
    d = debugfs_create_file("sample_rw", 0644, root, NULL, &my_sample_ops); // 0644 혹은 0777
    if (!d) {
        pr_err("%s %s: couldn't create debugfs my_sample\n", "my_debugfs", __func__);
        return;
    }
}
```



### (2) class sysfs 만드는 방법

이것은 `/sys/class/` 경로 밑에 생긴다.

내가 만든 코드 예시 (`/sys/class/my_sample/my_sample_rw`)

```c
#include <linux/device.h>

static ssize_t my_sample_rw_show(struct class *d, struct class_attribute *attr, char *buf) {
    return snprintf(buf, PAGE_SIZE, "what i want to print\n");
}

// my_sample_rw_store function

static CLASS_ATTR_RW(my_sample_rw)

static struct attribute *my_class_attrs[] = {
    &class_attr_my_sample_rw.attr,
    NULL
}
ATTRIBUTE_GROUPS(my_class)

static struct class my_class = {
    .name = "my_sample",
    .class_groups = my_class_groups,
}

static void __init my_debugfs_init(void) {
    class_register(&my_class);
}
```

cf) `PAGE_SIZE`는 `include/asm-generic/page.h` 참고



### (3) device sysfs 만드는 방법

이것은 생성되는 경로는 확인하지 못하였다.

`include/linux/device.h` 의 `DEVICE_ATTR`도 참고해서 만들 수 있다.

