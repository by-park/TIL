# 2022-02-22 (Xen initcall)

xen 에서 late_initcall 이 필요해서 직접 만들어보았다.

(1) 이런 패치도 반영이 가능한지 문의할 community 찾기

(2) x86 에 추가가 필요할지?



https://github.com/BY1994/xen/commit/6b745daa4a8b8d5af2d84b57aec13246a1947cdc

```c
commit 6b745daa4a8b8d5af2d84b57aec13246a1947cdc (HEAD -> master, origin/master, origin/HEAD)
Author: BY1994 <lovelychoco100@gmail.com>
Date:   Tue Feb 22 22:20:39 2022 +0900

    xen: add late init call in start_xen

    This patch added late_initcall section in init.data.
    The late initcall would be called after initcall
    in the start_xen function.

    Some initializing works on priority should be run
    in do_initcalls and other non-prioritized works
    would be run in do_late_initcalls.

    To call some functions by late_initcall,
    then it is possible by using
    __late_initcall(/*Function Name*/);

    Signed-off-by: BY1994 <lovelychoco100@gmail.com>

diff --git a/xen/arch/arm/setup.c b/xen/arch/arm/setup.c
index d5d0792ed4..ae07d84c73 100644
--- a/xen/arch/arm/setup.c
+++ b/xen/arch/arm/setup.c
@@ -1048,6 +1048,8 @@ void __init start_xen(unsigned long boot_phys_offset,
     /* Hide UART from DOM0 if we're using it */
     serial_endboot();

+    do_late_initcalls();
+
     system_state = SYS_STATE_active;

     for_each_domain( d )
diff --git a/xen/arch/arm/xen.lds.S b/xen/arch/arm/xen.lds.S
index 08016948ab..75fd582729 100644
--- a/xen/arch/arm/xen.lds.S
+++ b/xen/arch/arm/xen.lds.S
@@ -166,6 +166,8 @@ SECTIONS
        __presmp_initcall_end = .;
        *(.initcall1.init)
        __initcall_end = .;
+       *(.initcalllate.init)
+       __late_initcall_end = .;

        . = ALIGN(4);
        __alt_instructions = .;
diff --git a/xen/common/kernel.c b/xen/common/kernel.c
index e119e5401f..033811eb38 100644
--- a/xen/common/kernel.c
+++ b/xen/common/kernel.c
@@ -357,7 +357,7 @@ void add_taint(unsigned int flag)
 }

 extern const initcall_t __initcall_start[], __presmp_initcall_end[],
-    __initcall_end[];
+    __initcall_end[], __late_initcall_end[];

 void __init do_presmp_initcalls(void)
 {
@@ -373,6 +373,13 @@ void __init do_initcalls(void)
         (*call)();
 }

+void __init do_late_initcalls(void)
+{
+    const initcall_t *call;
+    for ( call = __initcall_end; call < __late_initcall_end; call++ )
+        (*call)();
+}
+
 #ifdef CONFIG_HYPFS
 static unsigned int __read_mostly major_version;
 static unsigned int __read_mostly minor_version;
diff --git a/xen/include/xen/init.h b/xen/include/xen/init.h
index bfe789e93f..4586b0d4a3 100644
--- a/xen/include/xen/init.h
+++ b/xen/include/xen/init.h
@@ -65,11 +65,14 @@ typedef void (*exitcall_t)(void);
     const static initcall_t __initcall_##fn __init_call("presmp") = fn
 #define __initcall(fn) \
     const static initcall_t __initcall_##fn __init_call("1") = fn
+#define __late_initcall(fn) \
+    const static initcall_t __initcall_##fn __init_call("late") = fn
 #define __exitcall(fn) \
     static exitcall_t __exitcall_##fn __exit_call = fn

 void do_presmp_initcalls(void);
 void do_initcalls(void);
+void do_late_initcalls(void);

 #endif /* __ASSEMBLY__ */
```

