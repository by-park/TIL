# 2021-11-02 (git squash)

git patch 를 하나로 합치고 싶을 때

https://dev-yakuza.posstree.com/ko/git/git-squash/



```shell
$ git rebase -i 합칠 커밋보다 밑에 있는 커밋 id
```

-i 가 나왔을 때 아래 화면에서 pick 이라고 되어있는 것을 맨 위만 남기고 (맨 위가 가장 아래 쌓인 commit이다) s로 바꾼다

```shell
pick 커밋id 커밋message
pick 커밋id 커밋message
pick 커밋id 커밋message
```

이것을

```shell
pick 커밋id 커밋message
s 커밋id 커밋message
s 커밋id 커밋message
```

하면 모든 패치 message 와 내용이 하나로 합쳐진다. 나는 commit message 내용을 완전히 지우고 새로 썼다. (새 커밋으로 올릴 거라서)

그리고 change-id를 지워서 올렸다. (어디 부분인지 몰라서 git commit --amend 해서 보이는 change-id를 지워줬다) 그러면 새 커밋으로 sqaush 된 패치가 올라가게 된다.