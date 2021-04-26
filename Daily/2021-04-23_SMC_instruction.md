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



### Linux Kernel SMC 와 HVC

arch/arm64/kernel/smccc-call.S

(https://elixir.bootlin.com/linux/latest/source/arch/arm64/kernel/smccc-call.S)

```c
/* SPDX-License-Identifier: GPL-2.0-only */
/*
 * Copyright (c) 2015, Linaro Limited
 */
#include <linux/linkage.h>
#include <linux/arm-smccc.h>

#include <asm/asm-offsets.h>
#include <asm/assembler.h>

	.macro SMCCC instr
	\instr	#0
	ldr	x4, [sp]
	stp	x0, x1, [x4, #ARM_SMCCC_RES_X0_OFFS]
	stp	x2, x3, [x4, #ARM_SMCCC_RES_X2_OFFS]
	ldr	x4, [sp, #8]
	cbz	x4, 1f /* no quirk structure */
	ldr	x9, [x4, #ARM_SMCCC_QUIRK_ID_OFFS]
	cmp	x9, #ARM_SMCCC_QUIRK_QCOM_A6
	b.ne	1f
	str	x6, [x4, ARM_SMCCC_QUIRK_STATE_OFFS]
1:	ret
	.endm

/*
 * void arm_smccc_smc(unsigned long a0, unsigned long a1, unsigned long a2,
 *		  unsigned long a3, unsigned long a4, unsigned long a5,
 *		  unsigned long a6, unsigned long a7, struct arm_smccc_res *res,
 *		  struct arm_smccc_quirk *quirk)
 */
SYM_FUNC_START(__arm_smccc_smc)
	SMCCC	smc
SYM_FUNC_END(__arm_smccc_smc)
EXPORT_SYMBOL(__arm_smccc_smc)

/*
 * void arm_smccc_hvc(unsigned long a0, unsigned long a1, unsigned long a2,
 *		  unsigned long a3, unsigned long a4, unsigned long a5,
 *		  unsigned long a6, unsigned long a7, struct arm_smccc_res *res,
 *		  struct arm_smccc_quirk *quirk)
 */
SYM_FUNC_START(__arm_smccc_hvc)
	SMCCC	hvc
SYM_FUNC_END(__arm_smccc_hvc)
EXPORT_SYMBOL(__arm_smccc_hvc)
```

include/linux/arm-smccc.h

(https://elixir.bootlin.com/linux/latest/source/include/linux/arm-smccc.h)

```c
#define ARM_SMCCC_STD_CALL	        _AC(0,U)
#define ARM_SMCCC_FAST_CALL	        _AC(1,U)
#define ARM_SMCCC_TYPE_SHIFT		31

#define ARM_SMCCC_SMC_32		0
#define ARM_SMCCC_SMC_64		1
#define ARM_SMCCC_CALL_CONV_SHIFT	30

/**
 * __arm_smccc_smc() - make SMC calls
 * @a0-a7: arguments passed in registers 0 to 7
 * @res: result values from registers 0 to 3
 * @quirk: points to an arm_smccc_quirk, or NULL when no quirks are required.
 *
 * This function is used to make SMC calls following SMC Calling Convention.
 * The content of the supplied param are copied to registers 0 to 7 prior
 * to the SMC instruction. The return values are updated with the content
 * from register 0 to 3 on return from the SMC instruction.  An optional
 * quirk structure provides vendor specific behavior.
 */
asmlinkage void __arm_smccc_smc(unsigned long a0, unsigned long a1,
			unsigned long a2, unsigned long a3, unsigned long a4,
			unsigned long a5, unsigned long a6, unsigned long a7,
			struct arm_smccc_res *res, struct arm_smccc_quirk *quirk);

/**
 * __arm_smccc_hvc() - make HVC calls
 * @a0-a7: arguments passed in registers 0 to 7
 * @res: result values from registers 0 to 3
 * @quirk: points to an arm_smccc_quirk, or NULL when no quirks are required.
 *
 * This function is used to make HVC calls following SMC Calling
 * Convention.  The content of the supplied param are copied to registers 0
 * to 7 prior to the HVC instruction. The return values are updated with
 * the content from register 0 to 3 on return from the HVC instruction.  An
 * optional quirk structure provides vendor specific behavior.
 */
asmlinkage void __arm_smccc_hvc(unsigned long a0, unsigned long a1,
			unsigned long a2, unsigned long a3, unsigned long a4,
			unsigned long a5, unsigned long a6, unsigned long a7,
			struct arm_smccc_res *res, struct arm_smccc_quirk *quirk);

#define arm_smccc_smc(...) __arm_smccc_smc(__VA_ARGS__, NULL)

#define arm_smccc_smc_quirk(...) __arm_smccc_smc(__VA_ARGS__)

#define arm_smccc_hvc(...) __arm_smccc_hvc(__VA_ARGS__, NULL)

#define arm_smccc_hvc_quirk(...) __arm_smccc_hvc(__VA_ARGS__)

/* SMCCC v1.1 implementation madness follows */
#ifdef CONFIG_ARM64

#define SMCCC_SMC_INST	"smc	#0"
#define SMCCC_HVC_INST	"hvc	#0"

#elif defined(CONFIG_ARM)
#include <asm/opcodes-sec.h>
#include <asm/opcodes-virt.h>

#define SMCCC_SMC_INST	__SMC(0)
#define SMCCC_HVC_INST	__HVC(0)

#endif
```



### XEN의 SMC와 HVC

xen/arch/arm/arm64/smc.S

(https://github.com/xen-project/xen/blob/master/xen/arch/arm/arm64/smc.S)

```c
/*
 * xen/arch/arm/arm64/smc.S
 *
 * Wrapper for Secure Monitors Calls
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation; either version 2 of the License.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 */

/*
 * void __arm_smccc_1_0_smc(register_t a0, register_t a1, register_t a2,
 *                          register_t a3, register_t a4, register_t a5,
 *                          register_t a6, register_t a7,
 *                          struct arm_smccc_res *res)
 */
ENTRY(__arm_smccc_1_0_smc)
        smc     #0
        ldr     x4, [sp]
        cbz     x4, 1f          /* No need to store the result */
        stp     x0, x1, [x4, #SMCCC_RES_a0]
        stp     x2, x3, [x4, #SMCCC_RES_a2]
1:
        ret
```

xen/include/asm-arm/smccc.h

(https://github.com/xen-project/xen/blob/master/xen/include/asm-arm/smccc.h)

```c
/*
 * This file provides common defines for ARM SMC Calling Convention as
 * specified in
 * http://infocenter.arm.com/help/topic/com.arm.doc.den0028a/index.html
 */

#define ARM_SMCCC_STD_CALL              _AC(0,U)
#define ARM_SMCCC_FAST_CALL             _AC(1,U)
#define ARM_SMCCC_TYPE_SHIFT            31

#define ARM_SMCCC_CONV_32               _AC(0,U)
#define ARM_SMCCC_CONV_64               _AC(1,U)
#define ARM_SMCCC_CONV_SHIFT            30

/*
 * arm_smccc_1_1_smc() - make an SMCCC v1.1 compliant SMC call
 *
 * This is a variadic macro taking one to eight source arguments, and
 * an optional return structure.
 *
 * @a0-a7: arguments passed in registers 0 to 7
 * @res: result values from registers 0 to 3
 *
 * This macro is used to make SMC calls following SMC Calling Convention v1.1.
 * The content of the supplied param are copied to registers 0 to 7 prior
 * to the SMC instruction. The return values are updated with the content
 * from register 0 to 3 on return from the SMC instruction if not NULL.
 *
 * We have an output list that is not necessarily used, and GCC feels
 * entitled to optimise the whole sequence away. "volatile" is what
 * makes it stick.
 */
#define arm_smccc_1_1_smc(...)                                  \
    do {                                                        \
        __declare_args(__count_args(__VA_ARGS__), __VA_ARGS__); \
        asm volatile("smc #0\n"                                 \
                     __constraints(__count_args(__VA_ARGS__))); \
        if ( ___res )                                           \
        *___res = (typeof(*___res)){r0, r1, r2, r3};            \
    } while ( 0 )

#define arm_smccc_smc(...)                                      \
    do {                                                        \
        if ( cpus_have_const_cap(ARM_SMCCC_1_1) )               \
            arm_smccc_1_1_smc(__VA_ARGS__);                     \
        else                                                    \
            arm_smccc_1_0_smc(__VA_ARGS__);                     \
    } while ( 0 )
#endif /* CONFIG_ARM_64 */

#endif /* __ASSEMBLY__ */
```

