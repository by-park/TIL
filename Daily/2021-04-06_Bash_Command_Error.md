# 2021-04-06 (Bash command Error)

### bash error

shell script를 작성하여 실행하였는데, 다음과 같이 command not found 에러가 발생하였다.

````
# sh test.sh
test.sh: line 41: $'\302\240': command not found
````

line 41의 경우 `cat "파일시스템"` 명령어로 위와 같은 command not found 가 난 원인을 찾을 수 없었다.

그러나, 띄어쓰기 1칸 밖에 없는 다른 라인들에서도 위와 비슷한 에러가 발견되어 확인해본 결과 띄어쓰기가 문제였다. 가장 첫번째 스페이스가 정상적인 경우 0x20 으로 들어가있는데, 에러가 발생한 라인에만 0x3f 로 들어가있었다. 복사하는 과정에서 변경이 생긴 것 같은데 스페이스가 여러 종류가 있는 것인지는 확인하지 못하였다.

가장 첫번째 스페이스를 지우고 새로 스페이스를 추가했더니 문제가 해결되었다.

(encoding 문제일 수도 있을 것 같다. 리눅스 -> 윈도우 엑셀 -> 리눅스 이런 식으로 여러 곳에 복사를 거쳐서 온 파일이기 때문에...)

비슷한 질문) https://stackoverflow.com/questions/29207246/bash-if-condition-command-not-found



### 서버 sudo shutdown

Linux 원격 접속한 서버에서 sudo shutdown now를 치는 경우, 물리적인 서버가 종료되어버렸다. 해당 서버는 직접 서버실에서 전원을 켜야 복구할 수 있었다.

https://jhnyang.tistory.com/50



### 엔지니어링 용어

하드웨어 시방이란 용어를 많이 썼는데, 해당 용어는 존재하지 않았다. 건축 분야에서 필요한 재료나 방법등을 알려주는 것을 시방이라고 한다. Specification = Spec 이다.

그동안 임베디드 용으로 사용하던 시방이라는 용어는 rework나 회로 수정으로 봐야할 것 같다.

https://blog.naver.com/dkkwon0201/120117527164