# 2021-06-05 (yocto conf)

### yocto conf 파일 설명 (빌드 환경설정파일)

> 환경설정파일은 *.conf 형식이며, 빌드시 전역적으로 적용된다.
>
> ### /build/conf/bblayers.conf
>
> “bitbake core-image-minimal “ 과 같이 이미지 빌드 명령을 주면 이 파일을 가장 먼저 파싱한다.
>
> 아래처럼 BBLAYERS에 */conf/layer.conf가 있는 디렉토리를 등록한다.
>
> BBLAYERS ?= " poky/meta /poky/meta-poky /meta-qt5 … “
>
> ### /meta-xx/conf/layer.conf
>
> 아래처럼 레시피파일(*.bb, *.bbappend)이 위치를 등록한다. 등록되지 않은 경로의 레시피는 포함되지 않는다.
>
> BBFILES += "${LAYERDIR}/recipes*/*/*.bb ${LAYERDIR}/recipes*/*/*.bbappend"
>
> ### /build/conf/local.conf
>
> oe-init-build-env 스크립트를 수행하면 local.conf가 자동으로 생성된다. 필요에 따라 이파일을 수정하면 된다.
>
> 이 파일을 수정하여 빌드타겟, 크로스컴파일러, CFLAG세팅 등 필요한 빌드설정을 변경할 수 있다.
>
> 이 파일을 통해 메타데이터의 설정을 추가하거나 덮어쓸 수 있다.

http://shincdevnote.blogspot.com/2017/08/yocto.html

conf 파일이 없으면 빌드할 때 default value로 자동 생성된다고 한다.

```shell
chyi@earth:~/Atmel/yocto_rootfs/test/poky$ source oe-init-build-env
You had no conf/local.conf file. This configuration file has therefore been
created for you with some default values. You may wish to edit it to use a
different MACHINE (target hardware) or enable parallel build options to take
advantage of multiple cores for example. See the file for more information as
common configuration options are commented.
```

https://slowbootkernelhacks.blogspot.com/2016/12/yocto-project.html



### kernel Kconfig

depends 말고 if를 사용해서 Kconfig를 설정할 수도 있다.

kernel/arch/arm/config 폴더 밑에 defconfig 파일에 명시되어있다고 한다.

```
if MSM_DEVICES
  config GPIO
  bool
  default y
endif
```

혹은 `select` 로 역종속성을 쓸 수도 있다고 한다. 그러나 이건 depends on 처럼 켜지면 켜지에 하는 것이지 반전의 역할로는 못 쓸 것 같다.

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=lyjzzz&logNo=220484264943