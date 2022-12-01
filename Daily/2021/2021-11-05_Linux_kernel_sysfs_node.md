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

`my_sample_rw_store` 내부는 아래 링크의 예시 코드를 참고하였다.

https://decdream.tistory.com/351

```c
#include <linux/device.h>

static ssize_t my_sample_rw_show(struct class *d, struct class_attribute *attr, char *buf) {
    return snprintf(buf, PAGE_SIZE, "what i want to print\n");
}

static ssize_t my_sample_rw_store(struct class *d, struct class_attribute *attr, const char *buf, size_t count) {
    int value;
    sscanf(buf, "%d", &value);
    printk("%s() value=%d\n", __func__, value);
    return count;
}

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

이것은 `/sys/devices/platform/{dt에 적힌 이름 ex.00000000.pcie}/` 밑에 생긴다.

`include/linux/device.h` 의 `DEVICE_ATTR`도 참고해서 만들 수 있다.

probe 함수 안에서 아래와 같이 file 을 생성한다.

```c
ret = device_create_file(dev, &dev_attr_my_sample);
if (ret) {
    dev_err(dev, "%s: couldn't create device file for my sample(%d)\n", __func__, ret);
	return ret;
}
```

my sample 의 attr 은 아래와 같이 만들 수 있다.

```c
static ssize_t my_sample_show(struct device *dev, struct device_attribute *attr, char *buf) {
    return snprintf(buf, PAGE_SIZE, "what i want to print\n");
}

static ssize_t my_sample_store(struct device *dev, struct device_attribute *attr, const char *buf, size_t count) {
    int val;
    if (sscanf(buf, "%d", &val) == 0) //do something
    return count;
}

static DEVICE_ATTR(my_sample, 0644, my_sample_show, my_sample_store);
```

