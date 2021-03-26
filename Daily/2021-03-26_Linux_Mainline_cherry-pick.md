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

