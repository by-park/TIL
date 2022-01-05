# 2022-01-05 (Xen Hypervisor Analysis)

kernel 코드는 분석한 책이 간혹 있지만, xen 은 없는 것 같아서 xen hypervisor code analysis blog 라고 검색해서 찾아보았다.

### Xen init 분석

Xen 시작점(Entry Point)

예제 A. 1 Xen 시작점(Entry Point) (xen\arch\x86\boot\head.S)

```assembly
.text
.code32

#undef bootsym_phys
#define sym_phys(sym) ((sym) - __XEN_VIRT_START)
#define bootsym_phys(sym) ((sym) - trampoline_start + BOOT_TRAMPOLINE)

#define BOOT_CS32 0x0008
#define BOOT_CS64 0x0010
#define BOOT_DS 0x0018
#define BOOT_PSEUDORM_CS 0x0020
#define BOOT_PSEUDORM_DS 0x0028

ENTRY(start)
jmp __start
```

https://cesl.tistory.com/entry/1-Xen-init-%EB%B6%84%EC%84%9D



### xen 공식 블로그

https://xenproject.org/blog/



### Xen code review process

OSCON16: Analysis of the Xen code review process: An example of software development analytics

https://www.slideshare.net/xen_com_mgr/analysis-of-the-xen-code-review-process-an-example-of-software-development-analytics

Analyzing code review in Xen

https://blog.bitergia.com/2016/03/08/analyzing-code-review-in-xen/

INTRODUCING THE XEN PROJECT CODE REVIEW DASHBOARD

https://xenproject.org/2016/06/03/introducing-the-xen-project-code-review-dashboard/



### xen on arm 발표자료

https://vdocument.in/xen-on-arm.html

https://vdocument.in/reader/full/xen-on-arm



### xen on arm 관련 논문/자료

Xen on ARM: System Virtualization using Xen Hypervisor for ARM-based Secure Mobile Phones

https://ziyang.eecs.umich.edu/esds/papers/hwang-xen.pdf

https://www.researchgate.net/publication/4318469_Xen_on_ARM_System_Virtualization_Using_Xen_Hypervisor_for_ARM-Based_Secure_Mobile_Phones

http://www-archive.xenproject.org/files/xensummit_4/Secure_Xen_ARM_xen-summit-04_07_Suh.pdf

Virtual Machine Introspection with Xen on ARM

https://www.researchgate.net/publication/282877768_Virtual_Machine_Introspection_with_Xen_on_ARM

Armv8-A virtualization (pdf)

https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Learn%20the%20Architecture/Armv8-A%20virtualization.pdf?revision=a765a7df-1a00-434d-b241-357bfda2dd31

AArch64 Virtualization (위와 동일)

https://developer.arm.com/documentation/102142/0100



xen 에서 지정한 error macro

xen/include/public/errno.h

```c
XEN_ERRNO(EPERM,         1)     /* Operation not permitted */
XEN_ERRNO(ENOENT,        2)     /* No such file or directory */
XEN_ERRNO(ESRCH,         3)     /* No such process */
#ifdef __XEN__ /* Internal only, should never be exposed to the guest. */
XEN_ERRNO(EINTR,         4)     /* Interrupted system call */
#endif
XEN_ERRNO(EIO,           5)     /* I/O error */
XEN_ERRNO(ENXIO,         6)     /* No such device or address */
XEN_ERRNO(E2BIG,         7)     /* Arg list too long */
XEN_ERRNO(ENOEXEC,       8)     /* Exec format error */
XEN_ERRNO(EBADF,         9)     /* Bad file number */
XEN_ERRNO(ECHILD,       10)     /* No child processes */
XEN_ERRNO(EAGAIN,       11)     /* Try again */
XEN_ERRNO(EWOULDBLOCK,  11)     /* Operation would block.  Aliases EAGAIN */
XEN_ERRNO(ENOMEM,       12)     /* Out of memory */
XEN_ERRNO(EACCES,       13)     /* Permission denied */
XEN_ERRNO(EFAULT,       14)     /* Bad address */
XEN_ERRNO(EBUSY,        16)     /* Device or resource busy */
(skip)
```

