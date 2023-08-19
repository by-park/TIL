# 2023-08-18 (GPIO Drive Strength)

GPIO 를 설정할 때 input/output, low/high, pull-up/down 설정은 하지만, drive strength 설정은 잘 하지 않아서 언제 사용하는지 찾아보았다. low/high 사이 변환하는 시간이 짧아지도록 할 수 있는 설정이라 빠른 장치에서 drive strength 를 설정한다면 연결된 부분도 설정해주는 것으로 보인다.

> **1.5 PAD Drive Strength**
>
>  다른 많은 CPU의 데이터시트에서는 Drive Strength라고 표현을 합니다. STM32F746의 경우 "I/O switching Current"라고 표현을 합니다. Drive Strength의 전류가 높다는 것은 0V에서 3.3V로 3.3V에서 0V로 변환하는 시간이 짧아지는 대신에 그 만큼 전류의 소비는 커 집니다. 데이터시트에 보면 I/O 토글을 2MHz로(Cext가 22pF 일때) 약 0.3mA이지만, 90MHz일때는 19.6mA가 됩니다. STM32F746의 "GPIO port output speed register"의 00(Low speed), 01(Medium speed), 10(High speed), 11(Very high speed)는 PAD의 Drive Strength를 설정하는 것으로써 00(Low speed)는 전류가 적게 소모되는 대신에 0V에서 3.3V 스위칭하는데 11(Very high speed)보다는 길다는 의미 입니다.
>
>  Drive Strength의 설정하는 경우는 매우 빠른 장치(보통 DDR, USB High Speed)와 통신시에 해당 장치의 데이터시트에서 요구하는 Drive Strength와 맞출 때입니다.

https://blog.naver.com/duvallee/221442039474