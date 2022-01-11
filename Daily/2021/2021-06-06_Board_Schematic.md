# 2021-06-06 (Board Schematic)

임베디드 보드 회로도

![img](https://mblogthumb-phinf.pstatic.net/data5/2005/1/4/107/%B1%D7%B8%B21-4-kingseft.jpg?type=w210)

> 핀 특성을 나타내는 이름과 기호를 쓰기도 한다. 그림1은 다양한 핀을 제공하는 소자
> 도면을 예로 보여준다.
>
> 핀 1은 일반적인 편이다.
>
> 핀 2는 로우에서 활성(active low)이라는 표시로 이름 위에 줄이 있다. 이는 논리값 0
> 에서 핀을 활성화하고, 논리값 1에서 비활성화한다는 뜻이다. 핀 2의 이름은 CS로, 대개
> 칩 선택(chip select)을 의미한다. 칩 선택은 칩을 활성화하기 위해 사용하는 핀이며,
> 주변기기와 메모리 칩 대부분은 칩 선택 입력이 있다. 컴퓨터 시스템 내부에는 메모리와
> 주변기기가 많이 존재하기 때문에 칩 선택이 중요하다. 프로세서는 칩 선택을 통해서 데
> 이터를 읽고 쓸 칩을 활성화한다. 일부 디바이스에는 칩 활성화(chip enable)를 뜻하는
> CE라는 입력 핀이 있다. 이 핀은 같은 기능을 하지만, 단지 다른 이름을 사용할 뿐이다.
>
> 핀 3에서 작은 삼각형은 모서리 변이에 따른 입력(edge-triggered input)이라는 표시다.
>
> 핀 4와 8은 각각 접지(GND)와 전원(VCC)이다. VCC 또는 VDD는 칩 전원 공급원을 표시하
> 는 데 사용한다. 변압기와 고체 전자공학에서 흔히 쓰는 용어인 Collectors에서 C(VCC),
> Drain에서 D(VDD)를 각각 따왔다. 용어가 무슨 뜻인지 모른다고 걱정할 필요는 없다.
> VCC나 VDD는 전원 공급과 관련이 있다는 사실만 알면 된다.
>
> 핀 7은 하이에서 활성인 출력 핀이며, 핀 6은 로우에서 활성인 출력 핀이다. 핀 6에 있는
> 원을 주목하기 바란다. 이 원은 반전출력(inverted output)을 나타낸다. 핀 7과 동일한
> 이름을 사용하여 핀 6이 핀 7 출력을 반전한다는 표시를 한다.
>
> 마지막으로, 핀 5는 NC라고 표시되어 있다. '연결 없음(No Connect)'이라는 말로, 핀에
> 아무런 기능이 없다는 뜻이다. 이 핀에 선을 연결해서는 안 된다(매우 드물지만'연결하지
> 마시오(Do Not Wire)'라고 표시한 핀도 있다. 같은 뜻이다). 그러나, NC라는 이름이 붙었
> 다고 해서 무조건 연결 없음이라고 생각해야 한다는 뜻은 아니다. 칩 제조업체가 다른
> 이유로 핀을 NC라고 부를 수도 있다. 마찬가지로, 각 디바이스에 따라오는 데이터시트를
> 주의깊게 확인해야 한다.
>
> 두 소자 사이를 선으로 연결하거나, 그냥 선 이름(net label)만을 표시할 수 도 있다.
> 선 이름은 같은 이름을 가진 다른 선 전부와 연결되어있음을 뜻한다. 복잡한 회로도에서
> 는 연결해야 할 선 전부를 일일이 나타내면 오히려 실용적이지 못할 수 도 있다. 선이
> 여기저기로 복잡하게 뻗어있다면, 최종 회로도를 쉽게 이해하기는 불가능할 것이다. 그래
> 서 단순히 선에 이름을 붙이는 방법을 널리 사용한다. 이 방법만으로도 어느 소자를 어느
> 소자에 연결해야 하는지 표시하는 데는 충분하다.

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=agnazz&logNo=100010559519