# 2021-10-14 (initramfs)

fp 가 없는데 64 bit 인 임베디드 환경에서, 64 bit kernel & 64 bit file system 을 쓰면 부팅하다가 문제가 발생하였다. file system에서 소수점 연산이 필요한 것으로 보인다. 그래서 fp 제거 빌드를 하려고 하였으나 컴파일시 에러가 너무 많이 발생해서 32 bit file system 으로 사용하려고 한다.

그리고 initramfs 를 이용하려고 한다.



### binfmt-464c 에러

32 bit initramfs 빌드시 아래와 같은 에러 발생

64 bit kernel & 32 bit initramfs 에서 발생할 수 있다고 한다.

```shell
request_module: runaway loop modprobe binfmt-464c
```

https://stackoverflow.com/questions/12223258/cross-compiled-kernel-unable-to-boot-because-of-a-strange-module-loop?rq=1

https://stackoverflow.com/questions/33993245/unable-to-load-initramfs-on-my-x86-board



multilib 32 library도 지원하도록 하는 방법

(결과적으로 위에 발생한 bin format 을 해결하는 방법은 아니었다)

```shell
MULTILIBS = "multilib:lib32"
DEFAULTTUNE_virtclass-multilib-lib32 = "x86"
```

https://docs.windriver.com/bundle/Wind_River_Linux_Platform_Developers_Guide_LTS_21/page/mmo1433778583499.html

https://stackoverflow.com/questions/37271509/how-to-get-initramfs-libraries-for-32-and-64-bit-using-yocto



### Yocto Project 개요

Yocto 에 관해 알고 싶을 때 찬찬히 읽어보면 좋을 것 같다.

https://slowbootkernelhacks.blogspot.com/2016/12/yocto-project.html



### Rootfs 내용 수정하는 방법 (initramfs 포함)

https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842473/Build+and+Modify+a+Rootfs