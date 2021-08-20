# 2021-08-19 (shell and BSOD)

mobaXterm 에 이상한 글자가 출력될 때, Blue screen (BSOD) 으로 가는 현상을 경험하였다. (Suspend To RAM 이후에 Uart 가 깨지는 현상이 Resume 초반에 간혹 보이는 것 같다.) terraterm에서는 컴퓨터가 멈추거나 Blue screen으로 가는 현상이 나타나지 않은 것으로 볼 때, mobaXterm의 버그이지 않을까 싶어서 비슷한 에러들을 찾아보았다. 



- ssh causes system halt, BSOD after KB4489868

https://github.com/microsoft/WSL/issues/3916



### Terminal Debug Color

터미널 메세지에 보이는 색깔을 설정할 수 있는데, 방법을 정확히 파악하지는 못했지만 나중에 시도해보기 위해 남겨둔다.

> 붉은 색 예시 ([0m 은 색깔 초기화이다)
>
> RETAILMSG(1, (TEXT("\033[31m+  SAMPLE MESSAGE  \033[0m\r\n")));

https://bstyle.tistory.com/61