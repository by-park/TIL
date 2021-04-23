# 2021-04-23 (SMC instruction)

xen은 HVC 뿐만 아니라 SMC도 trap 해서 사용한다. 

보안을 위해서 SMC로 직접 접근하지 못하게 VM이 중간에 trap한다. VM이 직접 처리할 지 혹은 넘길지를 결정하게 된다. 

SiP Service Calls 규약을 따르는 smc handler를 작성해서 사용한다면 (EL3 용 프로그램으로) 0x8200_0000 이런 식으로 SMC instruction 을 쓸 때 argument가 넘어가게 된다.



### Trapping use of the `SMC` instruction

https://developer.arm.com/documentation/ddi0406/c/System-Level-Architecture/The-System-Level-Programmers--Model/Traps-to-the-hypervisor/Trapping-use-of-the-SMC-instruction

HCR.TSC 가 1로 세팅되면 SMC 가 hypervisor에서 trap된다.



### ARM Trusted Firmware-A

https://gommaro.tistory.com/7

![img](https://blog.kakaocdn.net/dn/bxyawO/btqyYQ39rGH/9p3ZDkdnlC0ubP5qYQrMI1/img.png)



### Arm SiP Services

https://trustedfirmware-a.readthedocs.io/en/latest/components/arm-sip-service.html

> This document enumerates and describes the Arm SiP (Silicon Provider) services.



### SMC and HVC handling in hypervisor

https://www.mail-archive.com/xen-devel@lists.xen.org/msg96210.html

> Requirements 목록
>
> ```
> 0. There are no much difference between SMC and HVC handling (at least
> according to SMCCC).
> 1. Hypervisor should at least provide own UUID and version while
> called by SMC/HVC
> 2. Hypervisor should forward some calls from dom0 directly to Secure
> Monitor (Xilinx use case)
> 3. Hypervisor should virtualize PSCI calls, CPU service calls, ARM
> architecture service calls, etc.
> 4. Hypervisor should handle TEE calls in a secure way (e.g. no
> untrusted handlers in Dom0 userspace).
> 5. Hypervisor should support multiple TEEs (at least at compilation time).
> 6. Hypervisor should do this as fast as possible (DRM playback use case).
> 7. All domains (including dom0) should be handled in the same way.
> 8. Not all domains will have right to issue certain SMCs.
> 9. Hypervisor will issue own SMCs in some cases.
> ```



### ARM Trusted Firmware Design

https://chromium.googlesource.com/chromiumos/third_party/arm-trusted-firmware/+/factory-strago-7458.B/docs/firmware-design.md