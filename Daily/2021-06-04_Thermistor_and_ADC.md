# 2021-06-04 (Thermistor & ADC)

### ADC를 사용해서 Thermistor를 읽는 방법



전압 분배법칙을 이용하기 위해 R1에 Thermistor용 저항을 단다. 그러면 이것은 온도가 낮아지면 저항이 커지고, 온도가 높아지면 저항이 작아진다. R2는 고정이다. 100K 등 값이 정해져있다. R25 = 100 K 이런 스펙이 주어지면 이건 R1을 말한다. 25도일 때 저항이 100000 옴이란 뜻이다.

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8f/Voltage_divider.svg/125px-Voltage_divider.svg.png)

(R2랑 Vout 부분은 물이 아래로 떨어지는 걸 생각할 때 전류는 R2는 지나가기 힘든 길이라 Vout에 더 많이 지나갈 수 있는 걸 상상할 수 있다)

murata thermistor 제품을 사용한다면 murata data sheet를 참고하면 아래와 같이 온도에 따른 저항값 표를 이용할 수 있다.

(18 페이지)

그림 추가 예정

https://www.murata.com/~/media/webrenewal/support/library/catalog/products/thermistor/ntc/r44e.ashx

그리고 만약 1.8 V Vin일 때 10 bit ADC로 이 Vout을 읽는다면 값 * 1023 / 1.8 로 계산할 수 있다.





참고 (위 내용들과 비슷한 내용임)

https://openstory.tistory.com/230