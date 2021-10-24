# 2021-10-22 (Linux Kernel Initramfs - 32bit)

64 bit 커널의 파일 시스템이 필요한데, MMC driver를 동작시키기 전에는 rootfs 를 올릴 수가 없다. 따라서 커널에 내장되어 들어가는 initramfs 를 사용할 것이다.

그런데 사용할 대상이 ARM 64 bit chip 이나, 소수점 계산을 위한 NEON 이 없는 옵션인 경우, Neon H/W를 필요로 하는 64 bit 컴파일러로 initramfs를 빌드할 수 없다.

아래와 같은 방법으로 32 bit initramfs 를 빌드하여 64 bit kernel 에 사용할 수 있었다. (MMC driver init 후에는 그냥 32 bit rootfs 를 사용하였다)

### 1. busybox 다운로드

busybox를 이용해 initramfs를 제작한다. 여러 busybox 버전 중 v1.30.1 버전을 사용하였다.

(1-1) 직접 다운로드 링크: https://busybox.net/downloads/

(1-2) shell 명령어로 다운: `# wget https://busybox.net/downloads/busybox-1.30.1.tar.bz2`

(2) 다운 후 압축해제: `# tar -xvf busybox-1.30.1.tar.bz2`



### 2. busybox 빌드

32 bit ARM 컴파일러를 이용하여 빌드해야한다. 아래와 같은 32 bit 컴파일러를 설치하여 사용하였다.

**/usr/bin/arm-linux-gnueabi-gcc**



(1) config 설정

커널 빌드하듯이 config 설정하여 .config를 먼저 생성한 후에 컴파일을 진행해야한다. 아래의 명령어로 menuconfig를 실행한다.

`# make ARCH=arm CROSS_COMPILE=/usr/bin/arm-linux-gnueabi- menuconfig`

configuation 창이 나올텐데, 여기서 아래의 설정을 y로 변경해야 한다.

**setting -> Build static binary (no shared libs) 항목 체크**

이 후 save 하고 shell로 돌아간다.

이 상태에서 .config를 확인하면 `CONFIG_STATIC=y`로 설정되었음을 확인할 수 있다.

(2) build 하기

아래의 명령어로 빌드 진행한다.

`# make ARCH=arm CROSS_COMPILE=/usr/bin/arm-linux-gnueabi-`

(3) install 하기

아래의 명령어로 인스톨 진행한다.

`# make ARCH=arm CROSS_COMPILE=/usr/bin/arm-linux-gnueabi- install`

이제 _install 디렉토리가 생성되며 디렉토리는 아래와 같이 file system과 유사한 형태로 보인다.

```shell
~/busybox-1.30.1$ cd _install/
~/busybox-1.30.1/_install$ ls
bin linuxrc sbin usr
```

bin 디렉토리에 busybox 파일이 있는데, file 명령어로 확인 시, 32 bits로 빌드된 것을 확인할 수 있다.

 ```shell
~/busybox-1.30.1/_install/bin$ file busybox
busybox: ELF 32-bit LSB executable, ARM, EABI5 version_1 (SYSV), statistically linked, for GNU/Linux 3.2
 ```

_install 디렉토리를 파일시스템으로 만들기 위해 몇 가지 필요한 파일들을 만들어 준다.

```shell
$ mkdir -pv {bin,dev,sbin,etc,proc,sys/kernel/debug,usr/{bin,sbin},lib,lib64,mnt/root,root}
$ sudo cp -av /dev/{null,console,tty,sda1} dev/
```

이제 아래와 같이 디렉토리 및 dev 내 파일들이 생성됨을 확인할 수 있다.

```shell
~/busybox-1.30.1/_install$ ls
bin dev etc init lib lib64 linuxrc mnt proc root sbin sys usr
~/busybox-1.30.1/_install$ cd dev
~/busybox-1.30.1/_install/dev$ ls
console null sda1 tty
```



### 3. Init 파일 만들기

커널 부팅 및 마운트 후 init task가 동작하기 위해 init 파일을 만드는 것이 필수이다.

아래와 같이 간단하게 init을 위한 shell 스크립트를 작성해 주었다. (init 파일은 _install 디렉토리의 최상단에 위치해야 한다.)

```shell
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys
mount -t debugfs none /sys/kernel/debug

echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"

exec /bin/sh
exec /etc/systemd/system/내_daemon.service
```

작성 후에 아래와 같이 실행 권한을 조절해 준다.

`# chmod +x init`

**이제 원하는 application이 있다면 32 bits로 빌드한 후, _install 디렉토리에 포함시켜 주면 된다.**



### 4. initramfs 생성

initramfs는 cpio archive 형태의 gz 압축 파일이다. 따라서 아래와 같이 _install 디렉토리를 압축해야 한다.

아래 예제에서 현재 디렉토리는 _install 디렉토리이다.

`# find . | cpio -H newc -o | gzip -9 > ../initramfs-debug-image-32bits.cpio.gz`

**생성되는 파일의 확장자는 반드시 \*.cpio.gz 로 해야한다.**

 

이제 상위 디렉토리에 가면 initramfs-debug-image-32bits.cpio.gz라는 initramfs 가 생성되었음을 확인할 수 있다.



### 5. 커널 빌드하기

initramfs를 만들었으니, 커널에 내장하여 커널 컴파일을 해야 한다. 커널은 64bits 컴파일러를 이용하면 된다.

먼저 생성된 initramfs를 커널 디렉토리의 최상단에 위치 시킨다. 그리고 make menuconfig를 통해 아래와 같이 config를 변경한다.

General setup -> Initial RAM filesystem and RAM disk (initramfs/initrd) support    ---> *로 설정 하고, source file 이름은 initramfs 파일 이름으로 설정 (./initramfs-debug-image-32bits.cpio.gz)

config 저장하고 .config 확인하면 아래와 같이 initramfs가 사용되도록 변경됨을 확인할 수 있다.

 ```shell
CONFIG_BLK_DEV_INITRD=y
CONFIG_INITRAMFS_SOURCE="./initramfs_32bits.cpio.gz"
 ```

 

64bits 커널에서 32bits user space를 이용하기 위해서는 아래의 config도 설정되어야 한다.

**Kernel Features -> Kernel support for 32-bit EL0   ---> \*로 설정한다.** 

config 저장하고 .config에서 아래의 옵션이 설정됨을 확인한다.

```shell
CONFIG_COMPAT_BINFMT_ELF=y
```

config가 설정되었으니 make 명령어로 빌드하면 initramfs가 내장된 커널 이미지를 얻을 수 있다.



참고로 이 config 결과물을 저장할 때는 내 config 파일을 직접 수정하지 않고, make menuconfig로 .config를 생성한 후 make saveddefconfig를 통해 최종 만들어진 savedefconfig 로 내 config와 차이를 비교해서 수정한다.

```shell
$ make 내 config
$ make menuconfig
$ make savedefconfig
$ vimdiff defconfig 내 config
```



###  참고. 32bit static linking daemon build 방법

3번에 원하는 application 을 빌드해서 넣는다는 부분의 application 빌드 관련이다.

1. CMakeLists.txt 에 static linking 옵션을 준다.

```text
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -1,6 +1,7 @@
 cmake_minimum_required(VERSION 2.8.10)
 set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -std=c++0x")
-
+set(BUILD_SHARED_LIBS OFF)
+set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
#ADD_DEFINITIONS(-DDEBUG_KERNEL_MSG)
```



CMakefiles.txt에 아래와 같이 link 가 되어있어야한다

```text
TARGET_LINK_LIBRARIES(내_app 내_app2 pthread)
```



2. 해당 daemon directory에서 build 폴더를 만든다.

```shell
~/내_app$ mkdir build
~/내_app$ cd build
```

 

3. build 폴더에서 cmake 실행한다. (옵션으로 32bit compiler 지정)

```shell
cmake .. -DCMAKE_C_COMPILER=/usr/bin/arm-linux-gnueabi-gcc -DCMAKE_CXX_COMPILER=/usr/bin/arm-linux-gnueabi-g++
```



4. **make**로 build

```shell
~/내_app/build$ make
```



5. **build** 결과 확인

```shell
~/내_app/build$ file 내_app
내_app: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=0ca020662dabbc6a2d70e53d1c83ae803494b20d, not stripped
```

