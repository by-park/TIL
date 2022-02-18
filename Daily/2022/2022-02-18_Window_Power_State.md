# 2022-02-18 (Window Power State)

Window Device Power States

https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/device-power-states



![diagram illustrating the valid device power state transitions.](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/images/dxpostates.png)



Window Driver Cellular architecture

https://docs.microsoft.com/en-us/windows-hardware/drivers/network/cellular-architecture-and-driver-model

> IHV Driver (wmbclass)
>
> The IHV-implemented "lower edge" cellular device driver implements all of the adapter-specific cellular driver functionalities that are specified by MBIM. For USB based modems, the interfaces are standardized and handled by the inbox wmbclass driver. For PCIe cellular modem devices, the IHV vendors are expected to provide an IHV client driver that translates the MBIM commands to be transmitted over the PCIe bus. 

![windows 10 cellular architecture.](https://docs.microsoft.com/en-us/windows-hardware/drivers/network/images/cellular_mbb_driver_architecture.png)



PCIe Link State Power Management

https://samoyednote.tistory.com/74

![img](https://t1.daumcdn.net/cfile/tistory/2408684B5819E30B35)