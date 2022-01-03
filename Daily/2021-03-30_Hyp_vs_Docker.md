# 2021-03-30 (Hypervisor vs. Docker)

### 1. Hypervisor와 Docker의 차이점

Hypervisor는 Hypervisor 위에 Guest OS가 올라가서 각 Guest OS 들이 완전히 격리되어 동작할 수 있다.

그러나 Docker를 사용하는 경우에는, Host OS를 공유하고, application 실행시에 필요한 바이너리 정도만 추가로 올라가서 완전히 환경이 격리되는 것은 아니다.

![img](https://miro.medium.com/max/700/1*wOBkzBpi1Hl9Nr__Jszplg.png)



### 전가상화 반가상화 비교 분석 (Xen vs. KVM)

> Xen의 “하이퍼바이저 위에서 모두 놀아라.” 라던지, KVM의 “커널이 곧 하이퍼바이저이다.”

=> 현재는 둘 모두 전가상화와 반가상화 기능을 지원해서 이런 구분이 의미가 없어졌다.

![img](https://www.itopening.com/wp-content/uploads/2020/01/Xen-kvm-%EB%B0%98%EA%B0%80%EC%83%81%ED%99%94-%EC%A0%84%EA%B0%80%EC%83%81%ED%99%94-3.png)

![img](https://www.itopening.com/wp-content/uploads/2020/01/Xen-kvm-%EB%B0%98%EA%B0%80%EC%83%81%ED%99%94-%EC%A0%84%EA%B0%80%EC%83%81%ED%99%94-2.png)

https://www.itopening.com/4396/



(참고)

https://medium.com/@darkrasid/docker%EC%99%80-vm-d95d60e56fdd

https://doitnow-man.tistory.com/189



### 2. B형 공부에 필요한 알고리즘

박트리님이 블로그에 정리한 STL 구현 코드는 다음의 github에서 사용할 수 있다.

https://github.com/baactree/Algorithm/tree/master/Stl

(참고)

https://baactree.tistory.com/53



### 3. 네이버 개발 폰트

네이버 github에서 네이버 개발 폰트를 확인할 수 있다. (D2 Coding 글꼴)

https://github.com/naver/d2codingfont