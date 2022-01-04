# 2021-12-16 (Runtime PM)

https://elinux.org/images/0/08/ELC-2010-Hilman-Runtime-PM.pdf



### Power 소모

- 전압과 주파수 등에 의해 결정된다.
- Static Power (동작이 없을 때에도 회로에 전원을 공급해주기 위해 기본적으로 소모되는 전류) 와 Dynamic Power (동작 중일 때 = 회로에 신호가 바뀔 때 소모되는 전류) 로 나뉜다.

### Power 절약 방법

- Clock 이나 Power 를 통해 전력 소모를 조절한다.
- Frequency Scaling / Clock Gating / Power Gating / Power off 방법을 사용한다.

Frequency Scaling 은 동작 주파수를 조절하는 것으로, dynamic power를 줄인다.

Clock gating은 공급되는 clock 을 차단하여 dynamic power 를 줄인다. https://www.kernel.org/doc/htmldocs/kernel-api/clk.html

Power gating은 내부적으로 공급되는 전원의 스위치를 차단하여 static power를 줄인다. https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=beahey&logNo=90185364333&redirect=Dlog&widgetTypeCall=true&directAccess=false

Power off 는 외부로부터 공급되는 전원을 차단하여 소모량이 거의 없게 된다.

### Power Control 종류

- Runtime power management https://www.kernel.org/doc/html/v4.14/driver-api/pm/devices.html
- System-wide power management https://www.kernel.org/doc/html/latest/admin-guide/pm/index.html

Working state level은 Device 등을 Power gating으로 조절

System level 은 필수적인 건 남기고 나머지는 모두 전원 차단 (sleep)

### Power Domain

https://semiengineering.com/knowledge_centers/low-power/techniques/power-gating/

Power domain 으로 묶인 것들은 power gating 가능

clock 은 domain 단위로 가는 것이 아니므로 clock gating 도 위에 덧붙여서 필요한 상황에 활용해야할 것이다.



