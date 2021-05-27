# 2021-05-27 (kernel reboot syscall)

### reboot syscall

kernel에서 reboot할 때는 다음의 함수가 실행된다.

출처: http://egloos.zum.com/rousalome/v/9966588

```c
SYSCALL_DEFINE4(reboot, int, magic1, int, magic2, unsigned int, cmd,
		void __user *, arg)
{
	struct pid_namespace *pid_ns = task_active_pid_ns(current);
	char buffer[256];
	int ret = 0;

	/* We only trust the superuser with rebooting the system. */
	if (!ns_capable(pid_ns->user_ns, CAP_SYS_BOOT))
		return -EPERM;

	/* For safety, we require "magic" arguments. */
	if (magic1 != LINUX_REBOOT_MAGIC1 ||
			(magic2 != LINUX_REBOOT_MAGIC2 &&
			magic2 != LINUX_REBOOT_MAGIC2A &&
			magic2 != LINUX_REBOOT_MAGIC2B &&
			magic2 != LINUX_REBOOT_MAGIC2C))
		return -EINVAL;

	/*
	 * If pid namespaces are enabled and the current task is in a child
	 * pid_namespace, the command is handled by reboot_pid_ns() which will
	 * call do_exit().
	 */
	ret = reboot_pid_ns(pid_ns, cmd);
	if (ret)
		return ret;

	/* Instead of trying to make the power_off code look like
	 * halt when pm_power_off is not set do it the easy way.
	 */
	if ((cmd == LINUX_REBOOT_CMD_POWER_OFF) && !pm_power_off)
		cmd = LINUX_REBOOT_CMD_HALT;

	mutex_lock(&reboot_mutex);
	switch (cmd) {
	case LINUX_REBOOT_CMD_RESTART:
		kernel_restart(NULL);
		break;
```



여기서 사용된 `SYSCALL_DEFINE4(reboot, int, magic1, int, magic2, unsigned int, cmd,
		void __user *, arg)` 함수를 찾아보니 다음과 같다. sys_reboot() 함수로 만들어준다.

출처: http://rousalome.egloos.com/v/9991442

```c
14 #define SYSCALL_DEFINEx(x, sname, ...)				\
15	SYSCALL_METADATA(sname, x, __VA_ARGS__)			\
16	__SYSCALL_DEFINEx(x, sname, __VA_ARGS__)
17
18 #define __PROTECT(...) asmlinkage_protect(__VA_ARGS__)
19 #define __SYSCALL_DEFINEx(x, name, ...)					\
...
20                      asmlinkage long sys##name(__MAP(x,__SC_DECL,__VA_ARGS__))	\
21		__attribute__((alias(__stringify(SyS##name))));		\
```



system call 이 발생하면 sys_call_table 벡터로 뛰게 된다.

![img](http://thumbnail.egloos.net/600x0/http://pds20.egloos.com/pds/201906/16/57/a0386257_5d05d83ae3e17.png)

그 테이블이 sys_reboot도 있는 것이다.

```
d.v %y.l sys_call_table
________address|value_______|symbol
    NSD:80107FC4|0x8012D4E0   \\vmlinux\kernel/signal\sys_restart_syscall
1   NSD:80107FC8|0x80121E08   \\vmlinux\exit\sys_exit
2   NSD:80107FCC|0x8011C6D0   \\vmlinux\fork\sys_fork
3   NSD:80107FD0|0x802844FC   \\vmlinux\read_write\sys_read
4   NSD:80107FD4|0x8028459C   \\vmlinux\read_write\sys_write
5   NSD:80107FD8|0x80281788   \\vmlinux\open\sys_open
6   NSD:80107FDC|0x80280380   \\vmlinux\open\sys_close
```

(전체 sys call table: https://filippo.io/linux-syscall-table/)

이 시스템콜 번호와 핸들러는 리눅스 POSIX 표준으로 이미 정해져있다고 한다.



syscall 번호는 아래 header에서 확인할 수 있다. (모든 번호가 다 보이지 않아서 확인 중..)

`arch/arm/include/asm/unistd.h`

`include/linux/syscalls.h`

https://stackoverflow.com/questions/28126204/simple-system-call-implementation-example



### Boot process (MIT)

https://web.mit.edu/rhel-doc/3/rhel-rg-en-3/s1-boot-init-shutdown-process.html

### hook을 이용한 커널 해킹

출처: https://umbum.dev/522

출처: https://m.cafe.daum.net/opensupport.xyz/psGH/14?

```c

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/syscalls.h>
#include <linux/string.h>
#include <linux/init.h>
#include <linux/sched.h>
#include <linux/thread_info.h>
#define SYSCALL_TABLE_BASE_ADDR (0x8000fc28)
#define MANAGER_PERMISSION (0xff)
unsigned int ** g_puSysTableAddr = (unsigned int**)SYSCALL_TABLE_BASE_ADDR;
unsigned int g_uPrevAP = 0x00;
unsigned int g_uNewAP = MANAGER_PERMISSION;
unsigned int (* sys_write_orig)(int fd, char *byf, size_t count);
int is_match_PID(void)
{
unsigned long pid = 0;
struct task_struct *task;
//struct task_struct *current;
if(strcmp(current->comm, "hello_world") == 0 ){ //"target"
pid = task->pid;
return true;
}
/* 2nd how to:
for_each_process(task){
if(strcmp(task->comm, "hello_world") == 0 ){ //"target"
pid = task->pid;
return true;
}
}
*/
/* 3rd how to:
//printk("My current process id/pid is %d\n", current->pid);
for_each_process(task){
if(strcmp(task->comm, "hello_world") == 0 ){ //"target"
pid = task->pid;
if(current->pid == pid)
return true;
}
}
*/
}
//return pid;
return false;
}
unsigned int sys_write_hooked(int nFD, char * pBuf, size_t nCnt)
{
if ((nFD==1) && is_match_PID())
{
memset(pBuf, 0, nCnt);
strcpy(pBuf, "Hacked!!!\n");
return sys_write_orig(nFD, pBuf, nCnt);
}
else
return sys_write_orig(nFD, pBuf, nCnt);
}
int __init Hook_Init(void)
{
sys_write_orig = (void *)g_puSysTableAddr[__NR_write];
__asm__ __volatile__("mrc p15, 0, %0, c3, c0" : "=r"(g_uPrevAP));
__asm__ __volatile__("mcr p15, 0, %0, c3, c0" : :"r"(g_uNewAP));
g_puSysTableAddr[__NR_write] = (unsigned int *) sys_write_hooked;
__asm__ __volatile__("mcr p15, 0, %0, c3, c0" : :"r"(g_uPrevAP));
return 0;
}
void __exit Hook_Exit(void)
{
__asm__ __volatile__("mrc p15, 0, %0, c3, c0" : "=r"(g_uPrevAP));
__asm__ __volatile__("mcr p15, 0, %0, c3, c0" : :"r"(g_uNewAP));
g_puSysTableAddr[__NR_write] = (unsigned int *) sys_write_orig;
__asm__ __volatile__("mcr p15, 0, %0, c3, c0" : :"r"(g_uPrevAP));
}
module_init(Hook_Init);
module_exit(Hook_Exit);
```





### tig = text-mode interface for git

![img](https://t1.daumcdn.net/cfile/tistory/2265E74A574B8E2936)

git blame이나 git grep을 이용하듯이 tig blame이나 tig grep을 쓰면 gui처럼 되게 보기 좋게 정리되어 나온다.

