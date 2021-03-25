# 2021-03-25 (Linux Question)

Linux kernel 코드에 관련된 질문을 올릴 수 있는 사이트를 찾고 가입해보았다.

https://www.linuxquestions.org/



코드 리뷰로 받은 질문으로 `return 0;` 을 macro로 선언하는 것이 어떻겠냐는 피드백이 있었는데,

SUCCESS 등의 macro를 linux mainline에 찾지 못하여서 다음과 같이 질문을 게시하였다.

https://www.linuxquestions.org/questions/linux-kernel-70/why-don%27t-we-use-success-status-macro-in-kernel-4175692633/

```
Title: Why don't we use success status macro in kernel?

Hi, all.
It's my first question in here and I'm newbie in kernel development.
I was curious when I saw that error codes are defined in macros
while success status is not expressed by macros.

For example, error codes are expressed by macros like this.
Code:
return -EINVAL;
However, success is not defined but expressed by constant number.
Code:
return 0;
Why don't we use a macro for success status for unity with fail status (error code)?
(I think that one of possible reasons are clarity of constant number (0)
or that success status is not always expressed by 0)

I tried to use similar macros such as SUCCESS or NO_ERROR,
but it seems that they are not defined in official mainline codes.
```

