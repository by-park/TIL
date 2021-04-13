# 2021-03-26 (Linux Mainline cherry-pick)

### 1. Linux Mainline code의 commit을 확인하는 법

Linux Mainline의 commit 정보를 다음의 웹사이트에서 확인할 수 있다.

https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/



### 2. 내 개발 브랜치에 Mainline code를 cherry-pick 하는 방법

Mainline code를 git clone 하는 명령어는 다음과 같다.

(참고: https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux.git)

```
git clone https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux
```

Branch는 master branch 가 존재한다.



따라서 Mainline code의 commit id를 알고 있다면 다음과 같이 cherry-pick이 가능하다.

```
git fetch https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux master && git cherry-pick db24726
```



(2021-04-13 내용 추가)

방화벽 등의 문제로 위의 링크가 접속이 안 되는 경우에 다음과 같이 github 링크로도 사용이 가능하다. (위와 commit id가 동일하지 않으니 다시 확인이 필요하다)

```
git fetch https://github.com/torvalds/linux.git master && git cherry-pick 53fedcc0
```



방화벽 문제로 링크가 문제가 생긴 경우 에러 메세지는 다음과 같다.

```shell
$ git fetch https://kernel.googlesource.com/pub/scm/linux/kernel/git/torvalds/linux master && git cherry-pick db24726
error: RPC failed; HTTP 403 curl 22 The requested URL returned error: 403
fatal: the remote end hung up unexpectedly
```

아래 링크들에서는 `--depth 1` 이나 `--unshallow` 옵션을 사용하라는 내용이었는데, git cherry-pick 의 경우는 원래 이런 문제가 발생하지 않는지 동일한 상황은 찾지 못했다.

https://dukeyang.tistory.com/16

https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining

