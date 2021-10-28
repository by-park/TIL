# 2021-10-28 (vim space to tab)

vim 편집기에서 4 space를 탭 하나로 바꿀 수 있는 방법을 찾아보았다.

스페이스 4칸을 쓰고, 그 다음에는 역슬래시 t를 쓰면 된다.

(탭을 스페이스로 바꾸고 싶으면 둘의 위치를 바꾸면 된다)

https://skylit.tistory.com/273

```shell
:%s/    /\t/g
```

