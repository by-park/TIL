# 2022-01-16 (Kernel PSCI CPU OFF contribution)

```c
static int psci_0_2_cpu_off(u32 state)
{
	return __psci_cpu_off(PSCI_0_2_FN_CPU_OFF, state);
}
```

psci_0_2 인데 cpu_off 만 PSCI id가  `PSCI_FN_NATIVE(0_2, CPU_OFF)` 가 아닌 이유를 찾지 못하였다.

CPU_OFF 외에 다른 PSCI id 들이 변경된 패치는 아래에서 확인 가능하다.

https://github.com/torvalds/linux/commit/029180b1c99046831c33ed43fdbdb620506cb15b

https://www.spinics.net/lists/arm-kernel/msg426468.html



위의 내용을 수정할 경우, 아래 파일 위치에 있는 것들도 같이 수정해야할 것 같다.

```c
arch/arm64/kvm/psci.c:  case PSCI_0_2_FN64_CPU_ON:
arch/arm64/kvm/psci.c:  case PSCI_0_2_FN64_CPU_ON:
arch/arm64/kvm/psci.c:          case PSCI_0_2_FN64_CPU_ON:
include/uapi/linux/psci.h:#define PSCI_0_2_FN64_CPU_ON                  PSCI_0_2_FN64(3)
```



기본 linux kernel contribution 방법은 아래에서 참고

https://github.com/torvalds/linux/pull/805

