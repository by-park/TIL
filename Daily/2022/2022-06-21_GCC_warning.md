# 2022-06-21 (GCC warning)

gcc compiler 새 버전에서 switch case 문에서 break 를 걸어주지 않으면 warning 이 출력되도록 변경되었다.

little kernel 의 commit 을 보면 아래와 같이 이 warning 을 피하기 위해 `/* fallthrough */` 라는 주석을 추가한 것을 확인할 수 있다.

https://github.com/littlekernel/lk/commit/5dea3e193308344aafc46b46350f4d79e6791f39