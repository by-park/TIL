# 2022-04-22 (GIC v3 Register)

### GIC v3 Read Only 레지스터

SGIs are always treated as edge-triggered, and therefore GICR_ICFGR0 behaves as Read-As-One, Writes Ignored (RAO/WI) for these interrupts.

GICR_ICFGR0 는 xen 코드를 보면 GIC v3 버전에서 write 를 하지 않도록 되어있는데, 어차피 edge-triggered 라서 쓸 수 없는 영역이기 때문에 코드에도 들어가 있지 않다.

https://github.com/xen-project/xen/blob/master/xen/arch/arm/vgic-v3.c

```c
// read 는 가능
static int vgic_v3_rdistr_sgi_mmio_read(struct vcpu *v, mmio_info_t *info,
                                        uint32_t gicr_reg, register_t *r)
{
    struct hsr_dabt dabt = info->dabt;

    switch ( gicr_reg )
    {
    case VREG32(GICR_IGROUPR0):
    case VREG32(GICR_ISENABLER0):
    case VREG32(GICR_ICENABLER0):
    case VREG32(GICR_ISACTIVER0):
    case VREG32(GICR_ICACTIVER0):
    case VRANGE32(GICR_IPRIORITYR0, GICR_IPRIORITYR7):
    case VRANGE32(GICR_ICFGR0, GICR_ICFGR1):
         /*
          * Above registers offset are common with GICD.
          * So handle in common with GICD handling
          */
        return __vgic_v3_distr_common_mmio_read("vGICR: SGI", v, info,
                                                gicr_reg, r);

// write는 불가능 (GICR_ICFGR0 없음)
static int vgic_v3_rdistr_sgi_mmio_write(struct vcpu *v, mmio_info_t *info,
                                         uint32_t gicr_reg, register_t r)
{
    struct hsr_dabt dabt = info->dabt;

    switch ( gicr_reg )
    {
    case VREG32(GICR_IGROUPR0):
    case VREG32(GICR_ISENABLER0):
    case VREG32(GICR_ICENABLER0):
    case VREG32(GICR_ISACTIVER0):
    case VREG32(GICR_ICACTIVER0):
    case VREG32(GICR_ICFGR1):
    case VRANGE32(GICR_IPRIORITYR0, GICR_IPRIORITYR7):
    case VREG32(GICR_ISPENDR0):
         /*
          * Above registers offset are common with GICD.
          * So handle common with GICD handling
          */
        return __vgic_v3_distr_common_mmio_write("vGICR: SGI", v,
                                                 info, gicr_reg, r);
```

혹시나 접근하게 되면 default 로 처리되므로 아래와 같은 메세지를 찍게 된다.

```c
    default:
        printk(XENLOG_G_ERR
               "%pv: vGICR: SGI: unhandled write r%d offset %#08x\n",
               v, dabt.reg, gicr_reg);
        goto write_ignore;
    }

// 메세지 예시
d1v0: vGICR: SGI: unhandled write r2 offset 0x000c00
```





### 참고는 ARM 의 GIC v3 문서 (pdf 다운 경로)

Arm Generic Interrupt Controller Architecture Specification
GIC architecture version 3 and version 4

https://developer.arm.com/documentation/ihi0069/c