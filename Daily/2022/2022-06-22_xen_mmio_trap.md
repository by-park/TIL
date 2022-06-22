# 2022-06-22 (xen mmio trap)

xen hypervisor 에서 mmio 사용법은 정확히 모르지만, OS 들이 특정 메모리 주소를 접근하고자 할 때 xen 에서 trap 하는 방법이 있다. 해당 영역을 등록하면 그 영역을 접근하였을 때, xen 에서 대신 처리해서 결과값을 리턴해줄 수 있는 것 같다.

xen/arch/arm/include/asm/mmio.h

xen/arch/arm/vuart.c

```c
    register_mmio_handler(d, &vuart_mmio_handler,
                          d->arch.vuart.info->base_addr,
                          d->arch.vuart.info->size,
                          NULL);
```

```c
static const struct mmio_handler_ops vuart_mmio_handler = {
    .read  = vuart_mmio_read,
    .write = vuart_mmio_write,
};
```



mmio 로 접근하는 주소는 보통 32 bit 이지만, OS 에서 구조체 형식으로 선언해서 접근하는 경우, 컴파일러가 32 bit 보다 작게 접근하기도 한다. 그러면 `ldrb`, `ldrh`, `ldr` 등으로 구분된다. write도 `strb`, `strh`, `str` 이 있다. 

이걸 info 에서 dabt size를 이용해서 확인할 수 있었다. 

// xen/arch/arm/include/asm/mmio.h

```c
typedef struct
{
    struct hsr_dabt dabt;
    paddr_t gpa;
} mmio_info_t;
```

// xen/arch/arm/include/asm/hsr.h

```c
/* HSR data abort size definition */
enum dabt_size {
    DABT_BYTE        = 0,
    DABT_HALF_WORD   = 1,
    DABT_WORD        = 2,
    DABT_DOUBLE_WORD = 3,
};
```

```c
    struct hsr_dabt {
        unsigned long dfsc:6;  /* Data Fault Status Code */
        unsigned long write:1; /* Write / not Read */
        unsigned long s1ptw:1; /* Stage 2 fault during stage 1 translation */
        unsigned long cache:1; /* Cache Maintenance */
        unsigned long eat:1;   /* External Abort Type */
        unsigned long fnv:1;   /* FAR not Valid */
#ifdef CONFIG_ARM_32
        unsigned long sbzp0:5;
#else
        unsigned long sbzp0:3;
        unsigned long ar:1;    /* Acquire Release */
        unsigned long sf:1;    /* Sixty Four bit register */
#endif
        unsigned long reg:5;   /* Register */
        unsigned long sign:1;  /* Sign extend */
        unsigned long size:2;  /* Access Size */
        unsigned long valid:1; /* Syndrome Valid */
        unsigned long len:1;   /* Instruction length */
        unsigned long ec:6;    /* Exception Class */
    } dabt; /* HSR_EC_DATA_ABORT_* */

```

dabt size 를 이용해서 처리했다.

```c
/**
 @brief mmio read/write

 @param v vcpu
 @param info mmio information
 @param r register value pointer
 @return 1 on success, 0 on failure
**/

static int my_mmio_read(struct vcpu *v, mmio_info_t *info, register_t *r) {
    struct arm_smccc_res res;
    int read = 0x0;
    paddr_t addr = info->gpa;
    u32 dabt_size = info->dabt.size;
    
    arm_smccc_smc(SMC_CMD_ACCESS, read, addr, 0, dabt_size, &res);
    *r = vreg_reg32_extract(res.a0, info);
    return 1;
}

static int my_mmio_write(struct vcpu *v, mmio_info_t *info, register_t r) {
    struct arm_smccc_res res;
    int write = 0x1;
    u32 val = 0;
    paddr_t addr = info->gpa;
    u32 dabt_size = info->dabt.szie;
    
    vreg_reg32_setbits(&val, r, info);
    
    arm_smccc_smc(SMC_CMD_ACCESS, write, addr, val, dabt_size, &res);
    return 1;
}
```



받는 쪽에서 필요한 크기 만큼만 읽어오는 방법

```c
// smc handler 부분에서

switch(cmd) {
    case READ:
        switch (len) {
            case BYTE:
                ret = mmio_read_8(addr);
                break;
            case HALF_WORD:
                ret = mmio_read_16(addr);
                break;
            case WORD:
                ret = mmio_read_32(addr);
                break;
        }
        break;
    case WRITE:
        switch (len) {
            case BYTE:
                ret = mmio_write_8(addr);
                break;
            case HALF_WORD:
                ret = mmio_write_16(addr);
                break;
            case WORD:
                ret = mmio_write_32(addr);
                break;  
        }
        break;
    default:
        ret = -1;
}
```

https://github.com/ARM-software/arm-trusted-firmware/blob/master/include/lib/mmio.h

```c
#ifndef MMIO_H
#define MMIO_H

#include <stdint.h>

static inline void mmio_write_8(uintptr_t addr, uint8_t value)
{
	*(volatile uint8_t*)addr = value;
}

static inline uint8_t mmio_read_8(uintptr_t addr)
{
	return *(volatile uint8_t*)addr;
}

static inline void mmio_write_16(uintptr_t addr, uint16_t value)
{
	*(volatile uint16_t*)addr = value;
}

static inline uint16_t mmio_read_16(uintptr_t addr)
{
	return *(volatile uint16_t*)addr;
}
```



\+ Data Abort 는 DABT, Prefetch Abort 는 PABT 라고 한다.

https://developer.arm.com/documentation/ddi0460/d/Programmers-Model/Exceptions/Aborts

\+ Microsoft MMIO Access 관련 설명 페이지

https://docs.microsoft.com/ko-kr/virtualization/api/hypervisor-instruction-emulator/funcs/mmioaccessie