# 2021-11-16 (Yocto SDK issue)

Yocto 환경에서 사용하는 컴파일러를 로컬 환경에 가져오고 싶었다.

직접 컴파일러를 복사하였더니, No such file or directory라는 에러가 자꾸 나왔다. 해당 현상의 원인은 두 가지가 있는 것 같다.

(1) 32 bit 실행 파일을 64 bit 환경에서 실행하거나 (그 반대의 경우)

```shell
$ getconf LONG_BIT
```

위 명령어 확인으로 32 환경인지 64 환경인지 알 수 있다. 그리고 필요한 패키지를 설치하면 된다고 한다.

(2) 복사한 컴파일러 안의 정보를 확인해보면, 절대 경로들이 미처 복사되지 못한 것을 볼 수 있다. 아래의 Yocto 컴파일러 설치 방법을 사용한 경우 이렇게 경로들이 절대 경로로 설정되는 것 같다.

```shell
$ file 컴파일러경로
/컴파일러경로/x86_64-pokysdk-linux/usr/bin/aarch64-poky-linux/aarch64-poky-linux-gcc: ELF 64-bit LSB executable, x86_64, version 1 (GNU/Linux), dynamically linked, interpreter /다른 경로/x86_64-pokysdk-linux/lib/ld-linux-x86_64.so.2, BUILDID[sha1]={hash값}, for GNU/Linux 3.2.0, stripped
```



Yocto 컴파일러를 직접 받아서 설치하는 방법은 아래 링크에서 확인할 수 있다.

https://www.yoctoproject.org/docs/2.1/sdk-manual/sdk-manual.html#sdk-building-an-sdk-installer

https://www.yoctoproject.org/docs/2.1/sdk-manual/sdk-manual.html#sdk-installing-the-sdk

```shell
$ bitbake {image} -c populate_sdk
```

위 명령어를 하면 build폴더에 tmp/deploy/sdk 가 생긴다. 그 안에 있는 sh 파일을 이용하면 컴파일러를 다운받아 설치할 수 있다고 한다.



### issue

bitbake 시 자꾸 connect fail 이 발생하였다. 아래 링크를 보면 bitbake 작업 중에 ctrl+c를 누른게 원인이라고 한다.

https://github.com/3mdeb/yocto-docker/issues/27

그래서 top 을 쳤을 때 실제로 Cooker가 돌고 있는 것을 보아서 `kill -9 {pid}` 로 중단시켰지만 여전히 동일한 에러가 나왔다.

bitbake.lock 을 제거하면 된다는 검색 결과를 찾아서 `rm -rf build_dir/bitbake.lock` 를 입력하였더니 해결되었다.



### Yocto toolchain download

Yocto 홈페이지에서 직접 toolchain을 다운받으려고 하였으나, 내가 원하는 크로스 컴파일러는 없었다.

https://downloads.yoctoproject.org/releases/yocto/yocto-3.1.3/toolchain/

https://www.yoctoproject.org/software-overview/downloads/archived-releases/



### repo init시에 python error

repo init시에 python error 가 자꾸 났는데, python 3문법인데, python 2가 잡혀있을 때 난다.

```shell
TypeError: startswith first arg must be bytes or a tuple of bytes, not str
```

https://knight76.tistory.com/entry/python-subprocess-paramiko-%EC%98%88%EC%8B%9C-%EB%B6%80%EC%A0%9C-TypeError-startswith-first-arg-must-be-bytes-or-a-tuple-of-bytes-not-str-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0

python 3로 할 때는 아래 python 2 문법에서 에러가 났다.

```shell
$ repo init -u https://bitbucket.xxxx.xxxx.manifest" -b xxx
File "/xxx/.repo/repo/main.py", line 79
    file=sys.stderr)
        ^

SyntaxError: invalid syntax
```

https://coinpipe.tistory.com/96

python 2로 해도 어디선가 에러가 나고, python 3로 해도 어디선가 에러가 발생하였다. (아직 해결 진행 중)



로컬과 새로 받는 것 사이 repo 버전 충돌이 있는 것 같다.

Linux 서버 환경에서 python 기본 버전을 2와 3 중에서 선택하는 방법은 아래 명령어를 사용한다.

```shell
$ update-alternatives --config python
```

