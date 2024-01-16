# 2024-01-16 (SOC Temperature)

주위 온도 혹은 케이스 온도로 소자 온도 계산하는 식

https://www.rohm.co.kr/electronics-basics/transistors/tr_what7

> ###### Tj=Ta+Rth(j-a)×P
>
> Ta
>
> :Temperature of ambient atmosphere (= room temperature where the measurement was done)
>
> Rth(j-a)
>
> :Thermal resistance inbetween Junction and Ambient atmosphere *
>
> P
>
> :Power dissipation　　**



반도체의 Temperature Characteristics 간단 설명

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=zelkobaray&logNo=220720139413

![img](https://mblogthumb-phinf.pstatic.net/20160526_219/zelkobaray_1464262945810L6P8W_JPEG/1.JPG?type=w2)

>위 그림에 아주 잘 나와있다..
>
>Ta : IC 주변 온도이다.  
>
>Tc : Case 표면 온도가 되겠다 
>
>Tj = ( Qja x Pd) + Ta  식으로 구할 수 있다.
>
>여기서 Qja 는 Package의 열저항이라고 해서  반도체 업체로 부터 문의해서 알아야 한다.
>
>Pd는  IC 에 인가되는 전압 X 전류 가 되겠다...



열 설계에 대한 이해 글 총정리

https://techweb.rohm.co.kr/know-how/thermal-design/