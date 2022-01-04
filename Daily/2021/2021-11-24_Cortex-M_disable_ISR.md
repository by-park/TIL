# 2021-11-24 (Cortex-M disable ISR)

cortex-M core를 사용할 때 ISR 이 실행되지 않게 막으려면 prioriy mask의 bit를 1로 설정해서 enable 하면 ISR은 disable된다. 이걸 설정하면 WFI 에서는 깨어나지만 exception vector로 점프하지는 않았다. 그래서 while(1) 로 polling 방식으로 구현하고 WFI 에서 깨어나서 pending bit를 보고 어떤 interrupt 가 온 건지 확인하도록 코드를 구현할 수 있다.

![img](https://documentation-service.arm.com/static/5ea823e69931941038df1ae6?token=)

https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/programmers-model/core-registers

register 내부 설명은 다음과 같다.

![img](https://documentation-service.arm.com/static/5ea823e69931941038df1ae7?token=)

> ##### Priority Mask Register
>
> The PRIMASK register prevents activation of all exceptions with configurable priority. See the register summary in [Table 2.2](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/programmers-model/core-registers?lang=en) for its attributes. The bit assignments are:
>
> 0 = no effect
>
> 1 = prevents the activation of all exceptions with configurable priority.