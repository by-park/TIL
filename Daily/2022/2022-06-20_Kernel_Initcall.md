# 2022-06-20 (Kernel Initcall)

Kernel 의 Initcall 8 level 정리

- `early`;
- `core`;
- `postcore`;
- `arch`;
- `subsys`;
- `fs`;
- `device`;
- `late`.

```c
#define early_initcall(fn)        __define_initcall(fn, early)
#define core_initcall(fn)        __define_initcall(fn, 1)
#define postcore_initcall(fn)        __define_initcall(fn, 2)
#define arch_initcall(fn)        __define_initcall(fn, 3)
#define subsys_initcall(fn)        __define_initcall(fn, 4)
#define fs_initcall(fn)            __define_initcall(fn, 5)
#define device_initcall(fn)        __define_initcall(fn, 6)
#define late_initcall(fn)        __define_initcall(fn, 7)
```

https://0xax.gitbooks.io/linux-insides/content/Concepts/linux-cpu-3.html