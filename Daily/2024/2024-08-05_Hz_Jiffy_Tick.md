# 2024-08-05 (Hz, Jiffy, Tick)

Hz, Jiffy, Tick 정리

코드 및 자료 찾아서 잘못된 설명 있는지 확인 필요



리눅스 커널 config 를 보면 CONFIG_HZ_250 = y 로 되어있음

Hz 를 250으로 잡은 것이고 1초에 250번이라는 뜻

커널에서는 시간 관리를 Jiffy 로 한다. (참고로 64비트에서는 Jiffy64를 쓴다.)

1초에 250 번 Jiffy count 를 업데이트해주는 것이다.

그래서 Hz 와 Jiffy 를 거의 같은 것처럼 생각해도 된다.



Hrtimer 와 대비해서 일반 Mode timer 는 Timer expired 값을 Jiffy 값으로 넣는다.

1초 뒤를 원할 때 1 * 250 Hz 이런 식이다.

Jiffy가 250번 count 가 up 되면 그 timer 가 작동하는 것이다.



Jiffy 는 언제 업데이트가 되는가?

Arm 은 arch timer 가 있고 PPI 인터럽트를 통해서 인터럽트가 뜬다.

이게 뜰 때 Jiffy 가 같이 업데이트 된다.

그런데 1초에 250번 인터럽트가 꼬박꼬박 뜬다면 그게 다 비용이고 손실이 된다.

그래서 모아뒀다가 뜰 수도 있다고 한다.

한번 뜰 때 그걸 tick 이라고 하는데, 0에서 1로 값이 올라갈 때 그 값을 말한다.



Hrtimer 는 Jiffy 를 이용하지 않는다.

더 정밀한 timer 를 쓰려고 (resolution) 사용하는게 arch timer 다.



Interrupt 를 띄워서 Jiffy 를 업데이트한다.

Arch timer 가 26 Mhz 면 정밀도가 26Mhz 가 된다.

Arch timer 에는 clock 공급 소스가 있어야 하는데 그건 SoC 벤더가 제공해줘야 한다.

위의 예시에서는 그걸 26 Mhz 로 준 것

26 Mhz 로 tick 을 치면 arch timer 의 count 가 업데이트 된다.

커널은 그 arch timer 를 이용한다.

인터럽트가 26 Mhz 마다 뜨는 건 아니다. 별개로 설정된 timer 값에 따라 뜬다.



스케줄링도 arch timer 를 이용한다.

Timer 인터럽트가 뜨면 스케줄링을 하는데, 1초에 250번이면 ms 단위로 해상도가 낮다.

Arch timer count 값 보고 task 띄우는 시기 아니까 task 스케줄링이 일어난다.