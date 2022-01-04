# 2021-07-05 (Interrupts in multicore)

> Interrupts on multi-core systems On a multi-core system, each interrupt is directed to one (and only one) CPU, although it doesn't matter which. How this happens is under control of the programmable interrupt controller chip(s) on the board. When you initialize the PICs in your system's startup, you can program them to deliver the interrupts to whichever CPU you want to; on some PICs you can even get the interrupt to rotate between the CPUs each time it goes off.

https://stackoverflow.com/questions/33955582/how-do-interrupts-work-in-multi-core-system

>  For the startups we write, we typically program things so that all interrupts (aside from the one(s) used for interprocessor interrupts) are sent to CPU 0. 

http://www.qnx.com/developers/docs/qnxcar2/index.jsp?topic=%2Fcom.qnx.doc.neutrino.prog%2Ftopic%2Finthandler_Multicore.html

> https://www.kernel.org/doc/Documentation/IRQ-affinity.txt suggests that on modern kernels, all CPU cores are allowed to handle IRQs equally by default. But this might not be the optimal solution, as it may lead to e.g. inefficient use of CPU cache lines with frequent IRQ sources. It is the job of `irqbalance` to fix that.

https://unix.stackexchange.com/questions/516115/whats-the-policy-determining-which-cpu-handles-which-interrupt-in-the-linux-ker

> To prevent multiple devices from sending the same interrupt, Linux provides an interrupt request system so that each device in the computer system is assigned an interrupt number to ensure the uniqueness of its interrupt request.
>
> In kernel 2.4 and later, Linux improves the capability of assigning specific interrupts to the specified processors (or processor groups). This is called the SMP IRQ affinity, which controls how the system responds to various hardware events. You can limit or redistribute the server workload so that the server can work more efficiently.

https://www.alibabacloud.com/blog/understanding-cpu-interrupts-in-linux_597128