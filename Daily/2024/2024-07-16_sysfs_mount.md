# 2024-07-16 (sysfs mount)

ramfs 환경에서 sysfs 접근하려고 하는데 /sys/kernel/debug 폴더를 들어갈 수가 없었다.

`# mount -t sysfs sysfs /sys`

https://blog.naver.com/yyg1368/60131371835

위의 명령어를 치면 /sys/ 폴더는 생기지만, /sys/kernel/debug 를 갈 수는 없었다.

debug fs 는 다른 명령어로 마운트해야했다.

`# mount -t debugfs none /sys/kernel/debug`

https://infoarts.tistory.com/52





cf)

yocto build 시 에러 해결

`relocation R_X86_64_32 against '.rodata.str1.8'can not be used when making a PIE object; recompile with -fPIC`

해결 방법

`bitbake virtual/kernel -c cleanall;`

https://github.com/alexdobin/STAR/issues/1379

답변의 clean 을 보다가 시도해본 것



kernel 단독 build 시 에러 해결

`export ARCH=arm64`

`export CROSS_COMPILE=/opt/sec/2.6.1/sysroots/x86_64-pokysdk-linux/usr/bin/aarch64-poky-linux/aarch64-poky-linux`

`unset CFLAGS CPPFLAGS CXXFLAGS LDFLAGS`

`make distclean`

`make defconfig`

`make -j32`



에러 메세지가 `*** Can't find default configuration "arch/x86/configs/이름_defconfig"!` 라면

`export ARCH=arm64` 를 해야하는 것이다. 제대로 환경변수가 설정되지 않은 상태에서 빌드를 시도하면 `error: "CONFIG_이름" is not defined [-Werror=undef]` 를 볼 수 있다.

에러 메세지가 `gcc: error: unrecognized command line option '-mlittle-endian'; did you mean '-fconvert=little-endian'?` 이라면 컴파일러가 잘못 잡혔으므로 `export CROSS_COMPILE=컴파일러경로` 을 해야하는 것이다.