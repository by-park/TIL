# 2022-03-17 (xen suspend)

xen mainline 에 suspend 를 추가해보기

```c
commit 89f6af2a8aab0c44ad35369cb24fe4aa27e8efa2 (HEAD -> suspend, origin/suspend)
Author: BY1994 <lovelychoco100@gmail.com>
Date:   Thu Mar 17 22:52:26 2022 +0900

    xen: add machine suspend for entering suspend

    This patch added machine_suspend function
    and platform suspend function inside that.
    Those are called when xen entering suspend
    by psci suspend.

    Signed-off-by: BY1994 <lovelychoco100@gmail.com>

diff --git a/xen/arch/arm/include/asm/platform.h b/xen/arch/arm/include/asm/platform.h
index 997eb25216..3fe084fdc0 100644
--- a/xen/arch/arm/include/asm/platform.h
+++ b/xen/arch/arm/include/asm/platform.h
@@ -25,6 +25,8 @@ struct platform_desc {
     void (*reset)(void);
     /* Platform power-off */
     void (*poweroff)(void);
+    /* Platform suspend */
+    void (*suspend)(void);
     /* Platform specific SMC handler */
     bool (*smc)(struct cpu_user_regs *regs);
     /*
@@ -58,6 +60,7 @@ int platform_cpu_up(int cpu);
 #endif
 void platform_reset(void);
 void platform_poweroff(void);
+void platform_suspend(void);
 bool platform_smc(struct cpu_user_regs *regs);
 bool platform_has_quirk(uint32_t quirk);
 bool platform_device_is_blacklisted(const struct dt_device_node *node);
diff --git a/xen/arch/arm/platform.c b/xen/arch/arm/platform.c
index 4db5bbb4c5..0bce820a4d 100644
--- a/xen/arch/arm/platform.c
+++ b/xen/arch/arm/platform.c
@@ -127,6 +127,12 @@ void platform_poweroff(void)
         platform->poweroff();
 }

+void platform_suspend(void)
+{
+    if ( platform && platform->suspend )
+        platform->suspend();
+}
+
 bool platform_smc(struct cpu_user_regs *regs)
 {
     if ( likely(platform && platform->smc) )
diff --git a/xen/arch/arm/shutdown.c b/xen/arch/arm/shutdown.c
index 3dc6819d56..d578d53279 100644
--- a/xen/arch/arm/shutdown.c
+++ b/xen/arch/arm/shutdown.c
@@ -66,6 +66,11 @@ void machine_restart(unsigned int delay_millisecs)
     }
 }

+void machine_suspend(void)
+{
+    platform_suspend();
+}
+
 /*
  * Local variables:
  * mode: C
diff --git a/xen/common/shutdown.c b/xen/common/shutdown.c
index abde48aa4c..250e915a1a 100644
--- a/xen/common/shutdown.c
+++ b/xen/common/shutdown.c
@@ -34,6 +34,12 @@ void hwdom_shutdown(u8 reason)
 {
     switch ( reason )
     {
+    case SHUTDOWN_suspend:
+        printk("Hardware Dom%u suspended: suspending machine\n",
+               hardware_domain->domain_id);
+        machine_suspend();
+        break;
+
     case SHUTDOWN_poweroff:
         printk("Hardware Dom%u halted: halting machine\n",
                hardware_domain->domain_id);
diff --git a/xen/include/xen/shutdown.h b/xen/include/xen/shutdown.h
index b3f7e30cde..85bbb74e32 100644
--- a/xen/include/xen/shutdown.h
+++ b/xen/include/xen/shutdown.h
@@ -6,9 +6,10 @@
 /* opt_noreboot: If true, machine will need manual reset on error. */
 extern bool_t opt_noreboot;

-void noreturn hwdom_shutdown(u8 reason);
+void hwdom_shutdown(u8 reason);

 void noreturn machine_restart(unsigned int delay_millisecs);
 void noreturn machine_halt(void);
+void machine_suspend(void);

 #endif /* __XEN_SHUTDOWN_H__ */
```



### github force push 가능

local repository 상태를 github repository 에 반영 가능하다.

master branch 에 넣은 정보를 없애고 (git reset --hard 이전 commit id), git branch 새 이름 으로 branch 를 만들고, git checkout 새 이름 으로 들어가서 git cherry-pick 옮길 commit id 하고 git push 했더니, up stream 설정하라는 메세지가 나와서 설정했더니 branch 가 github에 반영되었다. git checkout master 로 이동한 후, git push origin master --force 를 했더니 지금 상태가 반영되었다.

```shell
$ git reset --hard {이전 commit id}
$ git branch new_branch
$ git checkout new_branch
$ git cherry-pick {없어진 commit id}
$ git push --set-upstream origin new_branch
$ git checkout master
$ git push origin master --force
```

