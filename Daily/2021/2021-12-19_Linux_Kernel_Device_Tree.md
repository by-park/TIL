# 2021-12-19 (Linux Kernel Device Tree)

https://elinux.org/Device_Tree_Usage

https://elinux.org/Device_Tree_What_It_Is

https://elinux.org/Device_Tree_Reference

https://jung-max.github.io/2019/10/22/Device_Tree_%EB%AC%B8%EB%B2%95/

http://junyelee.blogspot.com/2015/07/a-tutorial-on-device-tree.html

https://kernel.bz/boardPost/118684/3



### Discoverable vs Non-discoverable device

- Discoverability mechanism: 실시간으로 장치를 식별 가능한 메커니즘
- Non-discoverability mechanism: 실시간으로 장치 식별이 불가능한 메커니즘

- Platform Device: Non-discoverable device. 하드웨어 버스에 무엇이 연결되어 있고, 어떻게 연결되어 있는지 알아야한다. 이러한 정보를 device tree로 제공한다.

Device tree에 실시간으로 장치가 식별 불가능한 H/W 들의 속성을 적어준다.

> A device tree (DT) is a data structure of named nodes and properties that describe non-discoverable hardware.

https://source.android.com/devices/architecture/dto

(커널에서 실시간으로 장치 식별되는 것들도 있다.)



### Before / After

Device tree가 없으면 c 파일에 일일히 입력

![Kernel Recipes 2013 - ARM support in the Linux kernel](https://image.slidesharecdn.com/2013kernelrecipes-arm-support-kernel-130926023120-phpapp01/95/kernel-recipes-2013-arm-support-in-the-linux-kernel-13-638.jpg?cb=1380163241)

커널 안에 들어있던 디바이스 데이터를 커널과 분리하였다. 매번 커널까지 새로 컴파일할 필요 없도록 하기 위해서.



### DTS / DTC / DTB

Device Tree Source (DTS) -> Device Tree Compiler (DTC) -> DTB (Device Tree Binary = blob)

![img](https://source.android.com/devices/architecture/images/treble_dto_bootloader.png)



### Device Tree Structure (dtsi)

칩에 대한 dtsi 를 include 해서 보드에 대한 dts 에 넣음. 공통되는 부분이 더 많으면 dtsi 를 더 만들 수 있음.

https://rabbitthief37.github.io/post/trans-el-introduction-to-device-trees-06

![Introduction to Device Trees - 6](https://rabbitthief37.github.io/assets/article_images/2020-02-07/1.png)

https://wiki.vlee.kr/doku.php?id=device_tree

![device_tree [Seed&#39;s Tale]](https://wiki.vlee.kr/lib/exe/fetch.php?media=dtb_inclusion.jpg)

### Node names

- node-name@unit-address
- node-name: node name, should describe the general class of device
- unit-address: component of name, is specific to the bus type on which the node sits.

![Device Tree Customization](https://docs.toradex.com/102450-device-tree-anatomy.png)

https://developer.toradex.com/device-tree-customization

node name 은 일반적으로 통용되는 class name을 쓰는 걸 권장한다고 한다. unit address는 해당 버스에서 device를 구별하기 위해 들어가있다. register 시작 주소와 unit address가 매칭되는 것이 일반적이다.

### Properties

- describe the characteristics of the node (device). consist of "name" and "value".
- property name: property value

property는 해당 디바이스가 어떤 특성을 갖고 있는지를 설명한다. property name = "property value" 조합으로 작성이 된다. property value 가 없는 property도 있다.



### Standard Properties

- model: specified the manufacturer's model number of device
- compatible: define the specific programming model for the device. allowing a single device driver to match against several devices.
- reg: describes the address of the device's resource within the address space in the bus . means the offsets and length of memory-mapped IO register blocks commonly.

![Linux Device Tree](https://image.slidesharecdn.com/04-devicetree-191014151755/95/linux-device-tree-2-638.jpg?cb=1571066368)

https://www.slideshare.net/itembedded/linux-device-tree

model = model number

compatible = manufacturer, model

reg = <address_offset length address_offset length>



property 는 크게 나눠서 standard property 와 non standard property 로 2개로 나뉜다. standard 중 중요한 일부분을 보면, model 이라는 property 가 있는데, 이건 해당 디바이스의 모델 넘버를 명시하는데 사용이 된다.

model = "model number" 모델명을 명명

compatible = "manufacturer,model" 은 권장되는 방식이고 디바이스를 특정할 수 있게 명시하는 것이다.

reg = 는 버스의 타입에 따라 다를 수 있지만 일반적으로 memory-mapped IO 라고 하면

reg = <address_offset length> 이렇게 쓴다.

만약에 다수의 블락으로 정의가 되어있으면 이렇게 중복해서 설정할 수도 있다.

reg = <address_offset length address_offset length>



- status: indicates the operational status of a device.

![img](https://i0.wp.com/upload-images.jianshu.io/upload_images/21160298-574376c3b91499d2.png)

https://www.codenong.com/jscc3788716fa0/

status property 는 디바이스의 동작상태를 의미한다. okay, disabled, reserved, fail, fail-sss 5개의 값 중 하나를 갖는데, 동작이 가능한지 사용중인지로 5개의 값이 선택된다. 

해당 디바이스가 동작 가능한데 사용하지 않음(used) = reserved 이다. 

해당 디바이스가 동작 불가능하고 사용도 하지 않음 = disabled 이다. 

해당 디바이스가 동작 가능하고 사용하고 있음 = okay 이다.



### Interrupt Properties

- Interrupt tree represents the hierarchy and routing of interrupts in the platform devices.
- An interrupt source to an interrupt controller is represented in the device tree using interrupt properties.
- interrupt-parent: devices generating interrupt send interrupts to the specific device, typically interrupt controller.
- interrupts: describes a value of one or more interrupt sources for the device generating interrupt.
- interrupt-controller: specifies the device either a interrupt controller or not.

![Original] Linux Interrupt Subsystem (2) - Universal Framework - Programmer  All](https://www.programmerall.com/images/197/36/3694186e09d145e478a7307e118f2f0d.png)

예시를 보면 interrupt-controller 가 최상위 GIC 하나만 있는게 아니고, gpio 등 여러 개가 존재 (interrupt-parent 는 무조건 컨트롤러인가?)



인터럽트를 생성하는 디바이스라면 인터럽트 소스 정보가 무엇이고, 어떤 컨트롤러로 전달되는지 정의할 필요가 있다. 인터럽트 컨트롤러라면 형태랑 해당 노드가 인터럽트 컨트롤러인지 아닌지 명시해야함. 디바이스 트리처럼 hierarchy 가 있다.

interrupt-parent 는 어디로 전달되는지를 나타낸다. 컨트롤러가 레퍼런스로 들어간다.

interrupt 에 데이터 구성 형태는 interrupt 를 받는 parent 로 설정된 노드에 interrupt cell 이라는 property 에 의해 전달되는 소스의 형태가 결정된다.

interrupt-controller; 는 값을 갖고 있진 않고 어떤 노드 안에 정의되어있으면 해당 디바이스는 인터럽트 컨트롤러임을 명시하는 형태로 동작한다.



device tree와 interrupt tree는 다르다. 부모 자식 관계가 다르게 연결되어있다.



i2c parent 가 / 루트 노드인데, interrupt-parent = <&sample>; 이 있으면 sample 은 / 루트 노드에서 가서 찾아보면 interrupt-controller; 라고 되어있다. 그러면 i2c 에서 흘러나오는 건 sample 로 흘러가는 형태로 동작하는 것이다. 그리고 sample 의 interrupt-parent= <&sample2>; 가 또 있으면 여기서는 sample2 쪽으로 가게 된다는 걸 알수 있다. sample2 device 또한 interrupt-controller이다.

![Embedded Linux Beginners Guide | Documentation | RocketBoards.org](https://rocketboards.org/foswiki/pub/Documentation/EmbeddedLinuxBeginnerSGuide/6-exampledts.png)

![Device Tree Customization](https://docs.toradex.com/102450-device-tree-anatomy.png)