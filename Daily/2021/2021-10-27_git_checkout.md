# 2021-10-27 (git checkout)

git checkout 을 git clone 직후에 특정 branch로 이동하기 위해서 많이 사용했었다. 그러나 파일의 변경 상태를 돌리기 위해서는 사용해보지 않았다.

파일의 변경 상태를 돌리기 위해서는 git reset --hard 파일이름이라고 생각해서 계속 시도했는데 되지 않았다. HEAD 나 git commit id 에 대해서만 동작하였다. 파일에 대한 변경사항 되돌리기는 git checkout 파일 이름이었다. 

checkout 에서 두 가지 뜻이 다른 것처럼 보이는데, 왜 이렇게 만들었는지는 알게 되면 업데이트하면 좋을 것 같다.

> ***\*checkout\**: 브랜치를 전환하거나 작업 파일을 복구합니다.**
>
> 1. **git checkout** (file) - 로컬의 변경 내용을 되돌릴수 있습니다.
> 2. **git checkout** -b <브랜치 이름> - <브랜치 이름> 브랜치를 만들고 갈아탑니다.
> 3. **git checkout** master.

https://velog.io/@qkqhqhrh11/Git-%EB%AA%85%EB%A0%B9%EC%96%B4-%EB%B0%8F-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%82%AC%EC%9A%A9%EB%B2%95