# 2023-11-05 (check patch)

check patch 에러 잡기

**trailing statements should be on next line**

이 에러는 while (조건문); 이렇게 한 줄만 쓰면 발생한다.

에러를 해결하기 위해서는 ; 를 아랫줄로 써야한다.

```c
/* Before */
while (1);
/* After */
while (1)
;
```

https://patchwork.kernel.org/project/linux-fbdev/patch/20210904063127.11142-1-srivathsa729.8@gmail.com/



이번주에 헷갈렸던 것

gerrit 에 hook 을 연결한 code review bot 이 coding-style -1 점을 줬는데,

checkpatch 와 cspell 에러를 모두 해결해야하는 줄 알았다. 결과적으로 checkpatch 의 ERROR 항목만 해결하면 되는 것이었다. (check patch 의 warning 과 cspell 의 error 를 해결하지 않아도 됨)

