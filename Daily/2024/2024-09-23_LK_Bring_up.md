# 2024-09-23 (LK Bring up)

LK Bring up 과정 정리

1. LK 빌드를 위해서는 먼저 project 를 만들어야한다. 그리고 target, platform 도 필요하다.

빌드 명령어는 `make 프로젝트명` 이 된다.

예를 들면, `make armemu-test` 로 빌드 할 수 있다.

make 만 치면 나오는 에러는 다음과 같다

```bash
$ make
make[1]: Entering directory '/home/user/lk'
engine.mk:50: *** No project specified. User 'make list' for a list of projects or 'make help' for additional help. Stop.
make[1]: Leaving directory '/home/user/lk'
makefile:34: recipe for target '_top' failed
make: *** [_top] Error 2
```

makefile 을 열어보면

```makefile
_top:
        @$(MAKE) -C $(LKMAKEROOT) -rR -f $(LKROOT)/engine.mk $(addprefix -I,$(LKINC)) $(MAKECMDGOALS)
```

engine.mk

```makefile
# try to include the project file
-include project/$(PROJECT).mk
ifndef TARGET
$(error couldn't find project or project doesn't define target)
endif
include target/$(TARGET)/rules.mk
ifndef PLATFORM
$(error couldn't find target or target doesn't define platform)
endif
include platform/$(PLATFORM)/rules.mk

ifndef ARCH
$(error couldn't find arch or platform doesn't define arch)
endif
include arch/$(ARCH)/rules.mk
ifndef TOOLCHAIN_PREFIX
$(error TOOLCHAIN_PREFIX not set in the arch rules.mk)
endif
```

project 를 먼저 include 하고, target 을 include 하고, platform 을 include 하고, arch 를 include 한다.

https://stackoverflow.com/questions/16981464/difference-between-include-and-include-in-a-makefile

-include 는 에러 메세지를 발생시키지 않는다. (project 에러는 이미 위에서 처리해서 막은 듯하다.)



makefile 추가 문법 - .PHONY 는 어떤 용도로 쓰일까? => 실제 파일 이름과의 충돌 해결

https://jusths.tistory.com/226

makefile 문법 - 의존성 처리

https://80000coding.oopy.io/1c295f7b-cfbc-41df-8e8a-23a66f6266d3

$(MAKE) 가 선언되어있지 않아서 확인해보니, make 명령어에 붙은 옵션을 재귀로 합치는 문법이라고 함

https://stackoverflow.com/questions/38978627/what-is-the-variable-make-in-a-makefile



project/qemu-virt-arm64-test.mk

여기에 target 폴더에 mk 를 또 추가함

```makefile
MODULES += \
        app/shell

include project/target/qemu-virt-arm64.mk
```

project/target/qemu-virt-arm64.mk

```makefile
ARCH := arm64
ARM_CPU := cortex-a53

include project/target/qemu-virt-arm.mk
```

project/target/qemu-virt-arm.mk

project 를 거치면 target 이 정의되어있음 (engine.mk 에서 이 TARGET 을 이용해서 다음 makefile 이 있는 폴더를 찾음)

```makefile
TARGET := qemu-virt-arm
```

target/qemu-virt-arm/rules.mk

platform 이 정의됨

```makefile
LOCAL_DIR := $(GET_LOCAL_DIR)

GLOBAL_INCLUDES += \
    $(LOCAL_DIR)/include

PLATFORM := qemu-virt-arm

#include make/module.mk
```

platform/qemu-virt-arm/rules.mk

arch 가 arm64 로 결정됨

```makefile
LOCAL_DIR := $(GET_LOCAL_DIR)

MODULE := $(LOCAL_DIR)

ifeq ($(ARCH),)
ARCH := arm64
endif

LK_HEAP_IMPLEMENTATION ?= dlmalloc

MODULE_SRCS += \
    $(LOCAL_DIR)/debug.c \
    $(LOCAL_DIR)/platform.c \
    $(LOCAL_DIR)/secondary_boot.S \
    $(LOCAL_DIR)/uart.c

MEMBASE := 0x40000000
MEMSIZE ?= 0x08000000   # 512MB
KERNEL_LOAD_OFFSET := 0x100000 # 1MB

MODULE_DEPS += \
    lib/cbuf \
    lib/fdtwalk \
    dev/interrupt/arm_gic \
    dev/timer/arm_generic \

GLOBAL_DEFINES += \
    MEMBASE=$(MEMBASE) \
    MEMSIZE=$(MEMSIZE) \
    PLATFORM_SUPPORTS_PANIC_SHELL=1 \
    CONSOLE_HAS_INPUT_BUFFER=1

GLOBAL_DEFINES += MMU_WITH_TRAMPOLINE=1 \

LINKER_SCRIPT += \
    $(BUILDDIR)/system-onesegment.ld

include make/module.mk
```

참고로 MEMBASE 와 MEMSIZE 가 정의됨

arch/arm64/rules.mk

```makefile
LOCAL_DIR := $(GET_LOCAL_DIR)

MODULE := $(LOCAL_DIR)

GLOBAL_DEFINES += \
        ARM64_CPU_$(ARM_CPU)=1 \
        ARM_ISA_ARMV8=1 \
        IS_64BIT=1

MODULE_SRCS += \
        $(LOCAL_DIR)/arch.c \
        $(LOCAL_DIR)/asm.S \
        $(LOCAL_DIR)/exceptions.S \
        $(LOCAL_DIR)/exceptions_c.c \

GLOBAL_DEFINES += \
        ARCH_DEFAULT_STACK_SIZE=4096

# if its requested we build with SMP, arm generically supports 4 cpus
ifeq ($(WITH_SMP),1)
SMP_MAX_CPUS ?= 4
SMP_CPU_CLUSTER_SHIFT ?= 8
SMP_CPU_ID_BITS ?= 24 # Ignore aff3 bits for now since they are not next to aff2

GLOBAL_DEFINES += \
    WITH_SMP=1 \
    SMP_MAX_CPUS=$(SMP_MAX_CPUS) \
    SMP_CPU_CLUSTER_SHIFT=$(SMP_CPU_CLUSTER_SHIFT) \
    SMP_CPU_ID_BITS=$(SMP_CPU_ID_BITS)

ARCH_OPTFLAGS := -O2

# we have a mmu and want the vmm/pmm
WITH_KERNEL_VM ?= 1

ifeq ($(WITH_KERNEL_VM),1)

MODULE_SRCS += \
        $(LOCAL_DIR)/mmu.c

KERNEL_ASPACE_BASE ?= 0xffff000000000000
KERNEL_ASPACE_SIZE ?= 0x0001000000000000
USER_ASPACE_BASE   ?= 0x0000000001000000
USER_ASPACE_SIZE   ?= 0x0000fffffe000000

KERNEL_BASE ?= $(KERNEL_ASPACE_BASE)
KERNEL_LOAD_OFFSET ?= 0

GLOBAL_DEFINES += \
    KERNEL_BASE=$(KERNEL_BASE) \
    KERNEL_LOAD_OFFSET=$(KERNEL_LOAD_OFFSET)

else

KERNEL_BASE ?= $(MEMBASE)
KERNEL_LOAD_OFFSET ?= 0

endif

# try to find the toolchain
include $(LOCAL_DIR)/toolchain.mk
```

arch/arm64/toolchain.mk

```makefile
ifndef ARCH_arm64_TOOLCHAIN_INCLUDED
ARCH_arm64_TOOLCHAIN_INCLUDED := 1

ifndef ARCH_arm64_TOOLCHAIN_PREFIX
ARCH_arm64_TOOLCHAIN_PREFIX := aarch64-elf-
FOUNDTOOL=$(shell which $(ARCH_arm64_TOOLCHAIN_PREFIX)gcc)
ifeq ($(FOUNDTOOL),)
ARCH_arm64_TOOLCHAIN_PREFIX := aarch64-linux-android-
FOUNDTOOL=$(shell which $(ARCH_arm64_TOOLCHAIN_PREFIX)gcc)
ifeq ($(FOUNDTOOL),)
$(error cannot find toolchain, please set ARCH_arm64_TOOLCHAIN_PREFIX or add it to your path)
endif
endif
endif

#ARCH_arm64_COMPILEFLAGS := -mgeneral-regs-only -DWITH_NO_FP=1
```



2. 컴파일러가 필요하다.

아래는 컴파일러 path 가 잡혀있지 않아 발생하는 에러 메세지의 초반 일부이다.

```bash
$ make qemu-virt-arm64-test
make[1]: Entering directory '/home/user/lk'
make[2]: Entering directory '/home/user/lk'
arch/arm64/toolchain.mk:11:cannot find toolchain in path, assuming aarch64-elf- prefix
TOOLCHAIN_PREFIX = aarch64-elf-
make[2]: aarch64-elf-gcc: Command not found
LINKER_TYPE=bfd
COMPILER_TYPE=gcc
```



3. uart driver 포팅 필요

platform/ 에 debug.c 에 platform_dputc (쓰는 거) 와 platform_dgetc (받는 거)구현 필요

```c
void platform_dputc(char c) {
    if (c == '\n') {
        _dputc('\r');
    }

    _dputc(c);
}

int platform_dgetc(char *c, bool wait) {
    return dgetc(c, wait);
}
```



4. 운영체제를 위한 메모리 설정

(1) KERNEL_BASE 를 원래 MEMBASE 로 하면 1:1 매핑인데 `KERNEL_BASE = $(MEMBASE)`

가상 주소를 사용하기 위해 0xFFFF~ 로 시작하게 매핑함 `KERNEL_BASE := 0xFFFF000056000000`

(2) WITH_KERNEL_VM config enable

arch 의 rules.mk 에서 `WITH_KERNEL_VM ?= 0` 을 하기 때문에 그 전에 WITH_KERNEL_VM 을 1로 설정해줘야한다.

(3) mmu table

platform/ 안에 platform.c 파일 만들어서 추가함

```c
struct mmu_initial_mapping mmu_initial_mappings[] = {
    /* 2GB of DRAM */
    {
        .phys = 0x80000000,
        .virt = 0xFFFF000080000000,
        .size = 0x80000000,
        .flags = 0,
        .name = "dram"
    },
    /* SFR */
    {
        .phys = 0x10000000,
        .virt = 0xFFFF000010000000,
        .size = 0x10000000,
        .flags = MMU_INITIAL_MAPPING_FLAG_DEVICE,
        .name = "device1"
    },
    /* null entry to terminate the list */
    {0}
}
```

heap 에서 메모리 영역 관리할 때 사용하도록 arena 를 추가함. malloc 할 때 이 주소에서 할당해준다.

```c
static pmm_arena_t arena = {
    .name = "dram",
    .base = 0x40000000,
    .size = 0x40000000,
    .flags = PMM_ARENA_FLAG_KMAP,
}

void platform_early_init(void) {
    uart_init();
    arm_gic_init();
    arm_generic_timer_init(); // HOOK 이 있어서 여기서 호출할 필요 없을 것 같다.
    pmm_add_arena(&arena);
}
```



가상 주소를 사용하지 않으면 (`WITH_KERNEL_VM`), arm64_fpu_pre_context_switch 에서 문제 발생

그래서 config 를 켜기 전에 해당 함수를 잠깐 주석처리했다가 다시 켰다.



5. arm gic 설정 필요

주소 설정 필요함



※ strict-align 옵션 필요한지 확인하기 (필요하지 않음) 속도를 빠르게 하기 위해 align 시켜서 memory access 하는 것

※ floating 연산을 하려면 general-regs-only 옵션 제거해야함

※ project / target / platform / arch / app 이 각각 어떤 기준으로 구분된 건지 찾기

