# 2023-03-07 (ARM Cortex-M)

### ARM Cortex M 시리즈 중 Cortex-M23

Cortex-M23 이나 Cortex-M33 은 Power 를 덜 사용하는 이점이 있는 아키텍처이지만, 캐시가 없고 분기 예측도 없다. 최근에 나오는 시리즈보다 Performance 는 떨어지는 듯하다.

- two-stage pipeline [ARM M23 Specifications](https://developer.arm.com/Processors/Cortex-M23)

- 명령어 별 사이클 [ARM Cortex-M23 Processor Technical Reference Manual](https://developer.arm.com/documentation/ddi0550/c/CHDCICDF)

ex) MOV 1 cycle, LDR 1 or 2 cycle, STR 1 or 2 cycle



### 참고

ARM Assembly 전체적으로 정리된 블로그

https://kyuhyuk.kr/article/raspberry-pi/2019/05/15/ARM-Assembly

https://kyuhyuk.kr/article/simple-arm-operating-system/2019/03/04/Simple-ARM-Operating-System-Chapter-2

충북대 Cortex M Processor 정리 자료

http://ael.chungbuk.ac.kr/lectures/undergraduate/%EC%9E%84%EB%B2%A0%EB%94%94%EB%93%9CSW%EC%8B%A4%EC%8A%B5/%EA%B0%95%EC%9D%98%EB%85%B8%ED%8A%B8/01/1_0_ARM%20Cortex-M3%20%ED%94%84%EB%A1%9C%EC%84%B8%EC%84%9C.pdf

Zephyr Cortex M Developer Guide

https://docs.zephyrproject.org/3.2.0/hardware/arch/arm_cortex_m.html

ldr 와 str 명령어에 sp 나 pc 의 offset 활용하는 예시

https://developer.arm.com/documentation/dui0068/b/Thumb-Instruction-Reference/Thumb-memory-access-instructions/LDR-and-STR--pc-or-sp-relative