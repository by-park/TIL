# 2021-09-03 (remove callback of linux driver)

Linux driver에서 .remove로 연결된 remove 함수가 호출되는 경우는 다음과 같다. 모듈 형태로 넣었을 때, 그 모듈을 삭제하거나 driver에 연결된 device를 삭제했을 때였다. 그러면 내가 내장형으로 커널과 같이 빌드되기 옵션을 선택하면 remove 함수는 탈 수 없는 것 같다. (.remove는 .prove 처럼 platform driver ops 로 연결된다.)

Difference between remove and shutdown in the linux driver

> (1) When uninstalling the driver, remove calls
>
> (2) When the device is removed, the driver associated with the device needs to be removed, and the remove call

https://titanwolf.org/Network/Articles/Article?AID=3df3cb7e-1b17-4e7c-b936-408b4696123a#gsc.tab=0



질문 답변에도 역시 자동으로 호출된다는 답변을 볼 수 있었다. (위에서처럼 저런 경우에 자동으로 .remove 콜백이 불리게 된다.)

Linux Platform device "remove" method

> Your platform driver's remove method should get called automatically.

https://stackoverflow.com/questions/45305098/linux-platform-device-remove-method