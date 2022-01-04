# 2021-04-12 (ARMv8 Exception Level)

### ARM 문서 참고 Documentation: https://developer.arm.com/documentation



1. Exception Level

ARMv8 에서 수행되는 S/W는 4가지 exception level을 가진다.



**EL0**: User applications. Lowest privilege level.

**EL1**: An operating system kernel (e.g., Linux kernel)

**EL2**: Hypervisor (e.g., KVM virtualization)

**EL3**: Low-level firmware, including the Secure Monitor (e.g., baremetal assembly/C code)



아래로 갈 수록 Execution privilege가 높아진다.



2. Security State

ARMv8에는 2가지 Security State가 있다. ARM에서는 Trustzone을이용하여 system을 normal world와 secure world로 분리하여 system을 보호한다고 한다.

Normal world (Non-secure state) 에서는 특정 memory나 register를 접근하기 위해 trustzone을 이용한다.

또한 EL3에서 동작하는 Secure monitor는 Normal world와 Secure world 사이의 gateway 역할을 한다.

![img](https://blog.kakaocdn.net/dn/rJQ2a/btqC85cY4BW/A1xyWTmzMJB5Lc0Shl6XHK/img.png)



3. Execution state

ARMv8 은 AArch64 (ARM64) 와 AArch32 로 2가지의 Execution state를 제공한다. ARMv7과 달리 ARMv8 에서는 AArch64 state를 사용할 수 있으며, 64-bit general purpose register (GPR) 를 사용한다. AArch32는 ARMv7 과 호환성을 유지하기 위해 제공하는 것이며, 32-bit general purpose register를 사용한다.

64비트는 Xn 이고, 32 비트는 Wn이다.

![img](https://blog.kakaocdn.net/dn/p7oBx/btqC86iBO3B/KxKnpGUgKplpMWLDYBdArK/img.png)

출처: https://gongpd.tistory.com/6



(2021-04-13 내용 추가)

ARMv8 Instruction Set Overview 에서 Wn 레지스터와 Xn 레지스터의 의미를 확인하였다.

The letter W is shorthand for a 32-bit word, and X for a 64-bit extended word.

[출처] https://www.element14.com/community/servlet/JiveServlet/previewBody/41836-102-1-229511/ARM.Reference_Manual.pdf



(2021-08-20 내용 추가)

ARM 관련 지식 습득을 위해서 ARM 문서 메뉴얼로 공부하면 좋다는 추천을 받았는데, ARM Cortex-M3 한글 번역한 블로그 링크를 저장해둔다.

https://dhpark1212.tistory.com/entry/ARM-Product-%EA%B4%80%EB%A0%A8-%EB%AC%B8%EC%84%9C