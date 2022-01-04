# 2021-04-13 (Git format-patch)

push 하지 않은 commit message까지 임시 파일로 저장하려면 git format patch를 사용해야한다.

```shell
$ git format-patch {git commit id}
0001-changes.patch
0002-changes2.patch

$ git am -3 0001-changes.patch
```

am -3 옵션에 대한 설명

: 충돌이 났을 때 Git에게 좀 더 머리를 써서 Patch를 적용하도록 하려면 `-3` 옵션을 사용한다. 이 옵션은 Git에게 3-way Patch를 적용해 보라고 하는 것이다. Patch가 어느 시점에서 갈라져 나온 것인지 알 수 없기 때문에 이 옵션은 기본적으로 비활성화돼 있다. 하지만 같은 프로젝트의 커밋이라면 기본 옵션보다 훨씬 똑똑하게 충돌 상황을 해결한다.

[출처] https://git-scm.com/book/ko/v2/%EB%B6%84%EC%82%B0-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-Git-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0



cf)

1. commit 내용만 필요한 경우:

```git
git reset --mixed HEAD^
```

[출처] https://gmlwjd9405.github.io/2018/05/25/git-add-cancle.html



2. commit을 작성하지 않은 경우

```
git diff > temp.patch
git apply temp.patch
```

혹은 git stash를 사용하는 방법도 있다. (stack 처럼 동작한다)

```
git stash // 저장
git stash list // 저장 목록 확인
git stash pop // 꺼내고 적용
```

[출처] https://gmlwjd9405.github.io/2018/05/18/git-stash.html