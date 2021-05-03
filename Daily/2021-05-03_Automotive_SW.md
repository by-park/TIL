# 2021-05-03 (Automotive SW)



자동차 소프트웨어 개발 관련 블로그

http://www.yocto.co.kr/



아래는 해당 블로그의 몇 내용들이다.



### Yocto Rolling master model for production

> Platform 및 플랫폼 개발을 하면서 오픈소스 처럼 개발 하기는 정말 어렵다. 여기서 오픈소스 처럼 개발 한다는 의미는 Yocto master branch를 계속 쫓아가면서 production branch (필자의 회사는 dunfell branch)를 유지하는 것이다. 즉 2개의 branch를 운영하고 개발자는 코드를 제출할 때는 master와 dunfell 모두 올려야하고 테스트도 물론 모두 함께 해야한다는 것을 의미한다.
>
> 필자의 회사에서는 Yocto 로 개발하는 Infortainment 플랫폼 및 제품에서 이를 시도하고 있다. 개발자들은 잘 지켜가고 있지만 현재 리소스 부족으로 테스트는 자동화만 돌리고 일부 매뉴얼 테스트는 대응하기 어렵지만, 필요할 때마다 요청해서 돌리고 있는 상황이다.
>
> 이 모델의 효용성에 대해서는 필자도 아래와 같이 유지하기 어려운 점들, 오픈소스와 플랫폼 개발/양산 개발 모델을 같이 할 수 있는지 등 때문에 아직 100% 확신을 하지 못하는 상황이라 일단 지켜보고 판단해도 늦지 않을 것 같다. 

[출처] [Yocto Rolling master model for production](http://www.yocto.co.kr/2020/08/yocto-rolling-master-model-for.html)

![img](https://martinfowler.com/articles/branching-patterns/production-branch.png)



### Build dlt-viewer for MacOS

> [DLT](https://at.projects.genivi.org/wiki/display/PROJ/Diagnostic+Log+and+Trace) (Diagnostic Log and Trace)는 [GENIVI](http://genivi.org/)의 로그 관련 오픈소스 프로젝트이고 BMW에서 기여하였다. 차량의 로그 취합 및 추적을 위해 사용하고 [dlt-daemon](https://github.com/GENIVI/dlt-daemon)과 [dlt-viewer ](https://github.com/GENIVI/dlt-viewer)프로젝트가 있다.
>
> - dlt-daemon: 디바이스에 설치하여 로그를 수집하는 데몬이다. 로그를 파일로 저장하여 기록하기도 하고 실시간으로 네트웍을 사용하여 전송하기도 한다.
> - dlt-viewer: dlt 포맷으로 저장된 파일을 볼 수 있고, 실시간으로 네트웍으로 차량에서 보내는 로그 정보를 볼 수도 있다.

[출처] [Build dlt-viewer for MacOS](http://www.yocto.co.kr/2020/04/build-dlt-viewer-for-macos.html)



### Android Automotive

> 우선 Android Auto와 Android Automotive에 대한 용어 정의부터 명확하게 하고 시작해야할 것 같다.
>
> Android Auto: Android 기반 스마트폰의 Projection 을 지원하기 위한 기존 OS에 설치된 애플리케이션이다.
> Android Automotive: Android Automotive OS, Native Android OS 등 많은 용어로 불리고 있는 안드로이드 리눅스 커널에서 부터 서비스, 애플리케이션이 모두 올라간 OS 이다.
>
> 이 글에서Android Automotive를 이야기한다.
>
> 아래 링크에서와 같이 점점 더 많은 OEM들이 Android Automotive를 Infotainment OS로 채택하고 있는 추세이고, 특히 자체 OS를 만들기 어려운 OEM들로부터 점차 많이 각광받는 OS로 보인다.
>
> \- Volvo (2020): https://group.volvocars.com/company/innovation/android
> \- GM (2021): https://www.theverge.com/2019/9/5/20851021/general-motors-android-auto-google-infotainment
> \- Renault-Nissan-Mitsubishi Alliance (2021): https://www.autonews.com/article/20180918/COPY01/309189948/renault-nissan-mitsubishi-lets-google-s-android-into-its-dashboards
>
> 하지만 필자가 일하고 있는 Mercedes나 BMW, 현대자동차와 같이 자체 OS를 개발할 능력이 있는 OEM들은 검토는 하고 있겠지만 실제로 사용하려고 있는 움직임은 보이고 있는 것 같지 않다.
>
> Android Automotive의 장점은 Google에서 플랫폼에 관련된 모든 기능들을 구현하고 OEM들은 필요한 서비스와 애플리케이션을 구현하여 추가해 제품을 만들어 내기 때문에 플랫폼을 개발하기 위한 리소스 투입이 적고 빨리 제품을 만들어 낼 수 있다는 것이다. 또한 스마트폰에 있는 안드로이드 생태계를 이어 받을 수 있다는 것도 하나의 장점이다.
>
> 단점으로는 Google에 종속된다는 것이다. 이는 스마트폰 및 TV OS에서 기존에 했던 것들을 파악해 보면 어느정도 느낌이 온다.

[출처] [Android Automotive](http://www.yocto.co.kr/2020/01/android-automotive.html)



### Heading for the yocto project

> Yocto 프로젝트를 처음 시작하는 독자들을 위한 50 page 정도의 입문서가 오픈 소스로 github(https://github.com/CollaborativeWritersHub/heading-for-the-yocto-project)에서 작업되고 있다.
>
> https://github.com/CollaborativeWritersHub/heading-for-the-yocto-project/releases 에 가보면 첫번째 버전을 볼 수 있고 아주 기초적인 내용들을 볼 수 있다. 안타깝지만 영어로 되어 있고, 곧 필자가 시간이 되면 번역하여 한국어판으로도 github에서 볼 수 있게 할 예정이다.

[출처] [Heading for the yocto project (Yocto 프로젝트를 처음 시작하는 개발자들을 위한 오픈 소스 책)](http://www.yocto.co.kr/2018/01/heading-for-yocto-project-yocto.html)

![Product Details](https://images-na.ssl-images-amazon.com/images/I/41qvkPmIgcL._AC_US218_.jpg)



### Yocto Project vs Buildroot

> Yocto Project와 Buildroot 모두 Embedded Linux를 쉽게 만들기 위한 빌드 프레임워크이고, 많은 프로젝트에서 사용중에 있다. 필자는 Yocto Project 경험이 있지만 Buildroot 경헙은 거의 없고 단지 문서만 읽고 둘간의 차이를 간단히 비교해 보고자 한다.
>
> Yocto Project/OpenEmbedded는 Python 기반의 Task scheduler인 bitbake와 metadata로 이루어져 있고, Buildroot는 make 문법 기반이고 대부분 shell script로 이루어져 있다. 빌드 및 환경 설정은 kernel 과 유사하다.
>
> 가볍고 확장성이 적은 프로젝트를 한다면 Buildroot를, 대규모 프로젝트이며 다양한 SoC, 확장성을 고려하면 Yocto Project를 사용하는 것이 좋을 것같다. Buildroot는 쉽게 익혀 사용할 수 있는 반면, Yocto Project는 처음에 접하는데 시간이 약간 걸릴 수 있다.
>
> 아래 그림은 ELCE 2016에서 두가지 프로젝트 중 어떤 경우에 각 프로젝트를 선택하면 좋은지에 대해 보여준다.

[출처] [Yocto Project vs Buildroot](http://www.yocto.co.kr/2017/07/yocto-project-vs-buildroot.html)

![img](https://1.bp.blogspot.com/-Sejuab1d0Tk/WV-l_o4jADI/AAAAAAAHNmY/hQCdruBCM7s0jJmFfMBvevafuJuUsmMYwCLcBGAs/s640/yb.PNG)

