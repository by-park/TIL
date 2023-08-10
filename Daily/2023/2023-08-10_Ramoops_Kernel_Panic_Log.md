# 2023-08-10 Ramoops (Kernel Panic Log)

### Kernel Panic 이 났을 때 로그를 저장해두는 방법

=> Ramoops 사용하기

재부팅했을 때, 해당 영역에 로그가 있으면 파일로 저장하는 스크립트를 구동시켜서 활용할 수 있다고 한다.

https://xenostudy.tistory.com/599

https://www.kernel.org/doc/html/v4.14/admin-guide/ramoops.html



### 사용 방법

- device tree 에 ramoops 노드를 추가한다.

```c
 ramoops: ramoops@8f000000 {
     compatible = "ramoops";
     reg = <0 0x8f000000 0 0x100000>;
     record-size = <0x4000>;
     console-size = <0x4000>;
 };
```

- kernel config 에서 `CONFIG_PSTORE` 와 `CONFIG_PSTORE_RAM` 을 켠다.

- 커널 발생시 `/sys/fs/pstore` 에 로그가 저장된다고 한다. (ramfs) 저장장치에 저장된 것은 아니기 때문에 재부팅시에 램이 날아가지 않아서 이 위치에 로그가 있다면 복사해와야한다고 한다. (복사하지 않아도 재부팅했을 때 눈으로 바로 확인할 수 있다는 점이 일단 편리할 것 같다....)



### OOPS 란

> In [computing](https://en.wikipedia.org/wiki/Computing), an **oops** is a serious but non-fatal error in the [Linux kernel](https://en.wikipedia.org/wiki/Linux_kernel). An oops may precede a [kernel panic](https://en.wikipedia.org/wiki/Kernel_panic), but it may also allow continued operation with compromised [reliability](https://en.wikipedia.org/wiki/Reliability_engineering). The term does not stand for anything, other than that it is a simple mistake.
>
> 
>
> **웁스**(oops)는 [리눅스 커널](https://ko.wikipedia.org/wiki/리눅스_커널)이 정확한 행위를 위반한 것을 말하며, 결과로 특정한 오류 로그를 만들어 낸다. 잘 알려진 [커널 패닉](https://ko.wikipedia.org/wiki/커널_패닉)은 웁스들의 여러 종류들 중에서 야기되는 것이지만 웁스의 다른 종류들은(커널 패닉과는 달리) 절충된 신뢰성과 함께 계속된 동작을 허용한다. 이 용어는 간단한 실수 외에 다른 것을 의미하지는 않는다.

https://en.wikipedia.org/wiki/Linux_kernel_oops

https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%EC%BB%A4%EB%84%90_%EC%9B%81%EC%8A%A4