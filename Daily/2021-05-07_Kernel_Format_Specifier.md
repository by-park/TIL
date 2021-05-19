# 2021-05-07 (Kernel special format specifier)

Kernel's `printk()` supports special `%p` format specifiers:

> # Symbols/Function Pointers
>
> ```
>     %pF     versatile_init+0x0/0x110
>     %pf     versatile_init
>     %pS     versatile_init+0x0/0x110
>     %pSR    versatile_init+0x9/0x110
>             (with __builtin_extract_return_addr() translation)
>     %ps     versatile_init
>     %pB     prev_fn_of_versatile_init+0x88/0x88
> ```

See https://www.kernel.org/doc/Documentation/printk-formats.txt for full list.

or https://www.kernel.org/doc/html/latest/core-api/printk-formats.html

For your example, setting the `initcall_debug=1` kernel cmdline option might be a better way, than adding `printk()` manually.

[출처] https://stackoverflow.com/questions/49347664/how-to-print-function-name-with-function-pointer-in-linux-kernel