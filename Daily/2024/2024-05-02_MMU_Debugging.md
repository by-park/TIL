# 2024-05-02 (MMU Debugging)

ARM Trust Firmware 에서 특정 주소에 접근했을 때 panic 발생

접근은 mmio_write_32 함수로 레지스터에 값을 쓰려고 한 것임



panic 이 발생할 때와 안 할 때의 차이는 MMU enable 을 했을 때와 안 했을 때였다.

MMU 를 enable 했다고 해도 어차피 주소가 1:1 매핑이므로 문제가 생길 일은 없다고 생각했는데, MMU table 에 추가한 주소가 전체가 아니라 일부였다. 아래와 같이 mmap_add 에 주소를 추가해줘야한다. 나는 아래에 poplar_mmap 처럼 배열에 추가해서 해결했다.



// plat/hisilicon/poplar/aarch64/platform_common.c

```c
#define MAP_DEVICE      MAP_REGION_FLAT(DEVICE_BASE,                    \
                                        DEVICE_SIZE,                    \
                                        MT_DEVICE | MT_RW | MT_SECURE)

static const mmap_region_t poplar_mmap[] = {
        MAP_DDR,
        MAP_DEVICE,
        MAP_TSP_MEM,
#ifdef SPD_opteed
        MAP_OPTEE_PAGEABLE,
#endif
        {0}
};

#define DEFINE_CONFIGURE_MMU_EL(_el)                                    \
        void plat_configure_mmu_el##_el(unsigned long total_base,       \
                                  unsigned long total_size,             \
                                  unsigned long ro_start,               \
                                  unsigned long ro_limit,               \
                                  unsigned long coh_start,              \
                                  unsigned long coh_limit)              \
        {                                                               \
                mmap_add_region(total_base, total_base,                 \
                                total_size,                             \
                                MT_MEMORY | MT_RW | MT_SECURE);         \
                mmap_add_region(ro_start, ro_start,                     \
                                ro_limit - ro_start,                    \
                                MT_MEMORY | MT_RO | MT_SECURE);         \
                mmap_add_region(coh_start, coh_start,                   \
                                coh_limit - coh_start,                  \
                                MT_DEVICE | MT_RW | MT_SECURE);         \
                mmap_add(poplar_mmap);                                  \
                init_xlat_tables();                                     \
                                                                        \
                enable_mmu_el##_el(0);                                  \
        }

DEFINE_CONFIGURE_MMU_EL(3)
DEFINE_CONFIGURE_MMU_EL(1)

```

