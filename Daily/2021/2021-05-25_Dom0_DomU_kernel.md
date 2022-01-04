# 2021-05-25 (Dom0 DomU kernel)

### Dom0 와 DomU 커널이 분리되어있다고 한다. (para-virtualization을 위해서)

> 가상화 관련 일하는 친구에게 징징대서 물어보니 이상한 용어들 속출 -_-
> 아무튼 xen 서버가 될 녀석은 Dom0 커널을 설치해야 하고
> xen 의 guest가 될 녀석은 DomU 커널을 설치해야 한다.(반가상화시 para-virtualization)
> 전가상화(full virtualization)할 경우 win7을 guest로 쓸수 있다고 한다.
>
> 출처: https://minimonk.net/3360 [구차니의 잡동사니 모음]



### Dom0 Kernel

https://wiki.xenproject.org/wiki/Dom0_Kernels_for_Xen



### DomU kernel

https://wiki.xenproject.org/wiki/DomU_Support_for_Xen



### DomU 용 Kernel 빌드 방법

Configuring the Kernel for domU Support

1. If building x86 32 bit kernel make sure you have CONFIG_X86_PAE enabled (which is set by selecting CONFIG_HIGHMEM64G)
   - non-PAE mode doesn't work in 2.6.25, and has been dropped altogether from 2.6.26 and newer kernel versions.
2. For x86 , enable these core options (Processor type and features| Paravirtualized guest support]
   - CONFIG_HYPERVISOR_GUEST=y (3.10+ only)
   - CONFIG_PARAVIRT=y
   - CONFIG_XEN=y
   - CONFIG_PARAVIRT_GUEST=y
   - CONFIG_PARAVIRT_SPINLOCKS=y
3. For ARM, enable these core options (Kernel Features)
   - CONFIG_XEN=y
   - CONFIG_PARAVIRT=y
4. And Xen pv console device support (Device Drivers|Character devices
   - CONFIG_HVC_DRIVER=y
   - CONFIG_HVC_XEN=y
5. And Xen disk and network support (Device Drivers|Block devices and Device Drivers|Network device support)
   - CONFIG_XEN_FBDEV_FRONTEND=y
   - CONFIG_XEN_BLKDEV_FRONTEND=y
   - CONFIG_XEN_NETDEV_FRONTEND=y
6. And the rest (Device Drivers|Xen driver support)
   - CONFIG_XEN_PCIDEV_FRONTEND=y
   - CONFIG_INPUT_XEN_KBDDEV_FRONTEND=y
   - CONFIG_XEN_FBDEV_FRONTEND=y
   - CONFIG_XEN_XENBUS_FRONTEND=y
   - CONFIG_XEN_SAVE_RESTORE=y
   - CONFIG_XEN_GRANT_DEV_ALLOC=m
7. And for tmem support (x86 only):
   - CONFIG_XEN_TMEM=y
   - CONFIG_CLEANCACHE=y
   - CONFIG_FRONTSWAP=y
   - CONFIG_XEN_SELFBALLOONING=y

[출처] https://wiki.xenproject.org/wiki/Mainline_Linux_Kernel_Configs#Configuring_the_Kernel_for_domU_Support



### Dom0 용 Kernel 빌드 방법

Configuring the Kernel for dom0 Support

NOTE: Xen dom0 support depends on ACPI support. Make sure you enable ACPI support or you won't see Dom0 options at all.

In addition to the config options above you also need to enable:

- CONFIG_XEN_DOM0=y
- CONFIG_XEN_DEV_EVTCHN=y
- CONFIG_XENFS=y
- CONFIG_XEN_COMPAT_XENFS=y
- CONFIG_XEN_SYS_HYPERVISOR=y
- CONFIG_XEN_GNTDEV=y
- CONFIG_XEN_BACKEND=y
- CONFIG_XEN_NETDEV_BACKEND=m
- CONFIG_XEN_BLKDEV_BACKEND=m
- CONFIG_XEN_BALLOON=y
- CONFIG_XEN_SCRUB_PAGES=y

For x86, you will also need to enable:

- CONFIG_X86_IO_APIC=y
- CONFIG_ACPI=y
- CONFIG_ACPI_PROCFS=y (optional)
- CONFIG_PCI_XEN=y
- CONFIG_XEN_PCIDEV_BACKEND=m

If you're using RHEL5 or CentOS5 as a dom0 (ie. you have old udev version), make sure you enable the following options as well:

- CONFIG_SYSFS_DEPRECATED=y
- CONFIG_SYSFS_DEPRECATED_V2=y

[출처] https://wiki.xenproject.org/wiki/Mainline_Linux_Kernel_Configs#Configuring_the_Kernel_for_domU_Support