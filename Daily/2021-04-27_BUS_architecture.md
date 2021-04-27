# 2021-04-27 (BUS architecture)

### Bus 정리

ARM 프로세서를 쓰는 경우 ARM에서 만든 버스 규격인 AMBA를 쓰게 된다. AXI 나오기 전에는 AHB (ASB 쓰다가 더 성능이 좋은 AHB가 나와서 AHB를 주로 쓰게 된 것 같다.), APB 이 두 개가 대표적이었던 것 같다.

![img](http://pds15.egloos.com/pds/200906/07/90/c0098890_4a2b967986811.jpg)

AHB는 빠른 IP용, ASB는 느린 IP 용으로 연결했다고 한다. 빠르게 통신 해야하는 IP들이 느리게 통신해야하는 IP들이랑 버스를 같이 쓰면 기다려야 하니까 둘을 분리해서 연결했다.

그러다가 성능을 더 높이기 위해 도입된 AXI가 생겼다. AXI는 다중 채널이 도입되면서 AHB에서 앞의 디바이스를 기다려야했던 것을 이젠 안 기다리고 독립적으로 처리하게 되었다.

SoC 내의 IP들을 설계할 때 그 IP 의 특성에 맞게 (동작이 느린지 빠른지?) 이미 Bus를 어떤 걸 쓸지 결정했을 것이다. 대부분 AXI인 것 같다.

그리고 이런 AHB, APB, AXI 등은 버스의 규격을 의미하는 것이 아니라 "버스 인터페이스" 를 말하는 것이다. 



### Bus 들

AHB(Advanced High-Performance Bus)/ AXI

AMBA (Advanced Microcontroller Bus Architecture)

APB (Advanced Peripheral Bus)

SPI (Serial Peripheral Interface Bus)

USB (Universal Serial Bus)

I2C BUS



cf) AXI2APB



CPU 에서 DRAM 에 접근, 혹은 다른 것에 접근할 때 공통적인 BUS가 있고, 그 다음에 개별적으로 목적지를 향해 연결된 BUS 가 있다.



### AMBA 설명 (임베디드 레시피)

> SoC내부에 IP들을 전선으로 연결하는 방법은 Bus를 이용하는 것이지요. - Bus도 뭐 여러 Device가 공유하는 전선이니까 별반 다를 건 없겠지요 - 이 Bus들을 어떻게 연결하고, IP끼리 서로 어떻게 통신 할 것이냐를 약속한 것이 AMBA에요. AMBA (Advanced Microcontroller Bus Architecture)는 영어로 한마디 정의를 하자면SoC Target On-chip bus protocol라고 할 수 있겠네요.
>
>  
>
> 역시나 ARM에서 주도하는 Bus 규격이지요. 아무래도 ARM을 SoC에 CPU로 채택하면, ARM의 성능을 최대화 하는 게 좋기 때문에 ARM社가 세상에 Open해 놓은 규격이라 할 수 있겠사옵니다.
>
> 
>
> MCU내부에서 보게 되면, Bus의 통신 방식을 잘 이해 할 수 있고, 지켜줄 수 있는 것이 필요하게 되는데, 이것을 Bus Interface라고 부르고요, 이 Bus Interface는 Bus위에 Data를 어떻게 전송할 거냐, 어떻게 받을 거냐를 잘 control해주는 interface에요.
>
>  
>
> AMBA에서는 이런 Bus Interface가 3가지 종류로 나뉘어요. 그것이 바로, AHB, ASB, APB인데요. AHB는 Advanced High Performance Bus이고요, ASB는 Advanced System Bus이고요, APB는 Advanced Peripheral Bus에요. 헥헥.

![img](http://pds15.egloos.com/pds/200906/07/90/c0098890_4a2b967986811.jpg)

> 여기에서 잠깐, 왜 AHB, APB로 회로를 나누느냐 하면, 효율성 문제이지요. AHB로 연결된 녀석들은 APB보다 훨씬 빠르니까, AHB로 모든 IP를 연결해 두게 되면, 빠른 녀석들 끼리 서로 통신을 하다가 중간에 저속의 IP에게 뭔가를 부탁하고 시키게 되었을 때, 저속의 IP가 응답을 해줄 때까지, 고속의 IP들이 Bus를 쓰지 못하고 놀게 되지요. 그러니까, 빠른 넘들 끼리는 빠른 넘들 끼리 놀게 해주고, 



> 참고로, AHB, ASB, APB외에 AXI라는 AMBA 3.0 Spec도 나왔어요. Advanced eXtensible Interface)라고 부르는데요, AXI는 Burst기반으로 이루어져있고, Write Response channel이 추가되어 있고요. Read / Write가 동시에 가능해요. 시작 주소만으로도 Burst Transfer가 가능하게 되었지요. 고속 동작이 가능하도록 설계된 이 Bus는 ARM11이상의 Core를 사용하는 MCU의 backbone bus로 사용되고 있어요.
>
>  
>
> 사족으로 이런 AMBA System덕에 SoC할 때 IP를 갖다 끼우기만 하면 계속 SoC를 확장할 수 있는 구조로 만들 수 있겠죠!

http://recipes.egloos.com/4991780



### AMBA5

최신은 AMBA 5인 듯하다.

![img](https://developer.arm.com/-/media/Arm%20Developer%20Community/Images/Block%20Diagrams/AMBA-Overview-Diagram.png?revision=f04576d8-e26c-4597-ba0a-b4ddd6f06a01&h=387&w=800&la=en&hash=3852507C4101C111687DDDF0D44C24D0DA5AB6BD)

https://developer.arm.com/architectures/system-architectures/amba



### AHB와 AXI 차이

**AHB**의 신호**와** 타이밍은 일반적인 CPU의 버스 인터페이스**와** 비슷합니다. 반면 **AXI** 사양은 성능을 높이기 위해 기존과는 상당히 다른 구조로 되어 있습니다. **AXI와 AHB**의 가장 큰 **차이**는 '채널'이라는 개념의 도입 입니다. 채널은 지금까지의 일반적인 버스 구조에는 없었던 개념입니다.

https://m.blog.naver.com/PostView.nhn?blogId=esoclab&logNo=20174360379&proxyReferer=https:%2F%2Fwww.google.com%2F

AHB는 앞의 디바이스가 버스와 데이터 전송을 하는 경우 기다려야 하는데, AXI는 다중 채널이라 독립적으로 작동할 수 있다.

https://wh00300.tistory.com/5



Bus Interface 는 IP 설계시에 정해져 있는 듯하다.



### 오늘 발생한 gcc 빌드 에러 메세지

aarch64 gcc support --fix-cortex-a53-843419

답변은 CROSS_COMPILE이라고 한다.

https://stackoverflow.com/questions/47075223/aarch64-gcc-support-fix-cortex-a53-843419



### git blame 할 때 특정 줄만 찾는 방법

```shell
$ git blame -L 69,82 Makefile
```

https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Git%EC%9C%BC%EB%A1%9C-%EB%B2%84%EA%B7%B8-%EC%B0%BE%EA%B8%B0