# 2023-04-12 (ARMv9 Architecture)

ARMv8 Architecture

https://phdbreak99.github.io/blog/arch/2019-03-18-armv8-architecture/

![AArch64 Priviledge and Security Model](https://phdbreak99.github.io/blog/arch/2019-03-18-armv8-architecture/aarch64-priviledge-security-model.png)

ARMv9 Architecture 는 ARMv8 의 Trustzone 을 기반으로 하였기 때문에 위와 같은 Secure, Non-secure 영역이 구분되어있다. 여기에 추가로 Realm 영역이 추가되었다. OS 나 하이퍼바이저에서도 보지 못하는 영역이기 때문에 운영체제가 해킹되어도 해당 영역의 데이터는 안전하게 보호할 수 있다. 해당 영역은 신뢰할 수 있는 소수의 관리 소프트웨어만 실행할 수 있다.

https://blog.lablup.com/posts/2021/05/07/ARMv9/

![img](https://blog.lablup.com/assets/ARMv9/armv9-07.png)

