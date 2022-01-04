# 2021-06-02 (Kernel GPIO driver)

### 1. device tree에 GPIO 선언

device tree에서 다음과 같이 선언한다. 아래 예시의 nand_sel 과 같은 것은 GPIO 관련 device tree node로 미리 선언되어있어야한다.

```c
/{
    pinctrl@address{
        gpiogroup0: gpiogroup0 {
            gpio-controller;
            #gpio-cells = <2>;
        }
    }

	driver이름: driver이름 {
                ...
        gpios = <&gpiogroup0 1 1>;
		pinctrl-names = "default";
		pinctrl-0 = <&nand_sel &uart3_rx &sdio0_d4>;
                ...
        }
}
```

http://jake.dothome.co.kr/pinctrl-2/

http://qwooowp.blogspot.com/2018/08/linux-gpio-control.html



### 2. Driver 코드에서 GPIO 번호 얻어옴

그리고 driver 코드에서 `of_get_gpio(of_node 구조체, pinctrl 번호)` 함수 사용 (아래와 같이 2가지 방법 중 방법 1에 해당한다)

> **방법 1)** gpio number를 직접 계산한 후 사용하는 방법
>    gpio_nb = of_get_gpio(pdev->dev.of_node, 0);  /* GPIO4_16 => 92번 */
>    irq = gpio_to_irq(gpio_nb);                   /* 92번 gpio pin -> 151 irq 획득 */
>
> **방법 2)** device tree 내용을 활용하는 방법
>    irq = platform_get_irq(pdev, 0);     /* device tree 내용으로 부터 151 irq 획득 */

https://slowbootkernelhacks.blogspot.com/2017/05/yocto-project-linux-device-driver.html

필요한 헤더는 `#include <linux/of_gpio.h>` 이다.



### 3. Driver 코드에서 GPIO 번호 valid 여부 확인

위의 함수에서 ret 값으로 받은 번호로 `gpio_is_valid(번호)` 를 확인할 수 있다.

(커널 버전 5.9.11)

```c
static inline bool gpio_is_valid(int number)
{
        return number >= 0 && number < ARCH_NR_GPIOS;
}
```

참고로 13년 전에는 is_valid_gpio 라는 define이 있었던 듯하다.

```c
--- a/include/asm-generic/gpio.h
+++ b/include/asm-generic/gpio.h
@@ -16,6 +16,8 @@
#define ARCH_NR_GPIOS 256
#endif
+#define is_valid_gpio(gpio) ((unsigned int)(gpio) < ARCH_NR_GPIOS)
```

https://linux.kernel.narkive.com/8PBZgoym/patch-introduce-is-valid-gpio-predicate-and-use-it-in-gpiolib-c

이걸 사용해서 다음과 같이 코드를 작성하면 된다.

```c
data.gpio = of_get_gpio(pdev->dev.of_node, 0);
if (!gpio_is_valid(data.gpio)) {
    dev_err(&pdev->dev, "failed to get gpio\n");
    return -EINVAL;
}
```

필요한 헤더는 `#include <linux/gpio.h>` 이다.



### 4. Driver 코드에서 GPIO 번호로 필요한 세팅 (in/out 설정 혹은 get, set value 호출)

gpio 설정 함수 리스트

```c
int gpio_direction_input(unsigned gpio)
int gpio_direction_output(unsigned gpio)
int gpio_get_value(unsigned gpio)
int gpio_set_value(unsigned gpio, int value)
```

http://jake.dothome.co.kr/gpio-1/

https://blog.dasomoli.org/446/

https://blog.naver.com/PostView.nhn?blogId=crushhh&logNo=221562780494



+참고

gpio sysfs 인터페이스

```shell
$ ls /sys/class/gpio/gpio24/
$ echo in >/sys/class/gpio/gpio24/direction
$ echo out >/sys/class/gpio/gpio24/direction
$ cat /sys/class/gpio/gpio24/direction
$ cat /sys/class/gpio/gpio24/value
```

https://www.ics.com/blog/gpio-programming-using-sysfs-interface