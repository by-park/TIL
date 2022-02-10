# 2022-02-10 (GNU MAKE argument)

Make를 할 때 인자를 넣는 방법

```shell
$ make TARGET VAR=VALUE VAR2=VALUE2
```

이런 식으로 뒤에 쭉 넣으면 `VAR=VALUE`가 `Makefile` 안에 `변수=값` 이런 식으로 들어가게 된다.

그러면 그걸 이용해서 if 문으로 VAR 값을 검사해서 내가 원하는 방식으로 쓸 수 있다.

동일한 코드를 `#ifdef` 를 이용하여 특정 config 가 enable되었을 때 빌드 되고, 그렇지 않을 때 빌드되지 않게 한다면 이걸 이용하면 빌드시에 서로 다른 config 가 적용되도록 할 수 있다. (어떤 경로의 config 파일을 가리킬 것인지 결정하게 해서)

https://kldp.org/node/93529