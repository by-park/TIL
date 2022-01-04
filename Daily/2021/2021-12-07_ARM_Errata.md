# 2021-12-07 (ARM Errata)

> ARM Errata는 용어 그대로 ARM 프로세서에 오류가 있을 때 이를 알리는 통지 번호를 의미합니다.
>
> 예를 들면, 'ARM Errata 1234567'과 같은 방식으로 전달되는데 보통 어셈블리 코드 포멧의 패치가 공유됩니다.

http://egloos.zum.com/rousalome/v/10017102



### Arm Cortex-A9 r4 Software Developers Errata Notice

https://developer.arm.com/documentation/uan0009/d/



### ARM Cortex-A55 Errata Notice 다운로드

https://developer.arm.com/documentation/epm137809/1400/?lang=en

왼쪽 상단의 다운로드 버튼을 누르면 최신 버전으로 다운받을 수 있다.



### cpu 버전 커널에서 확인 방법

```shell
$ cat /proc/cpuinfo
```

https://www.quora.com/How-can-I-check-which-version-of-the-ARM-CPU-is-in-my-Raspberry-Pi

https://raspberrypi.stackexchange.com/questions/9912/how-do-i-see-which-arm-cpu-version-i-have

위의 방법으로 자세한 midr 레지스터 값 등은 알 수 없었다.



### CPU Revision number

r0p1 이런 식으로 CPU revision 번호가 있는데, 이건 ARM MIDR 레지스터에서 확인 가능하다.

https://developer.arm.com/documentation/100442/0100/register-descriptions/aarch64-system-registers/midr-el1--main-id-register--el1

![img](https://documentation-service.arm.com/static/5e7e1405b471823cb9de56ce?token=)

[23:20] 이 r 번호 (major revision number) 이고, [3:0] 이 p 번호 (minor revision number) 이다.

ARM Errata Notice 에 보면, fixed in r2p0 같은 내용을 볼 수 있는데, 지금 CPU 번호를 보면 Errata 를 적용할 필요가 있는지 없는지 여부를 알 수 있다.



+

ARM Cortex Technical Reference Manual 에 보면 Appendices 에 Boot revision 마다 바뀐 내용이 있다.

https://developer.arm.com/documentation/100442/0200/appendices/revisions/revisions?lang=en