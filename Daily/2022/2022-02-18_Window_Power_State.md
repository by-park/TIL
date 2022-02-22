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



PCIe State 설명

https://gigglehd.com/zbxe/2068851

> 다음의 확장은Latency Tolerance Reporting이다.위에서 조금 썼지만,PCI Express디바이스의 경우, 디바이스 그 자체에는
>
> **D0:풀 가동 상태(필수)
> D1/D2:가동하고 있지만 일부의 패킷을 처리하지 않아도 좋다(옵션)
> D3hot/D3cold:대기 상태(필수)**
>
> 하지만 있어, 또Link에는
>
> **L0:Full Active Link
> L0s:Standby.L0에 곧 복귀 가능
> L1:Low Power Standby.L0에는(L2/L3보다는)비교적 고속으로 복귀 가능
> L2/L3 Ready:L2혹은L3에 이행 준비.LDn경유로L0에 복귀
> L2:Low Power Sleep.LDn경유로L0에 복귀
> L3:0 Power.LDn경유로L0에 복귀
> LDn:L0에의 복귀**
>
> 하지만 설정되어 있다.그런데, 문제는 이L2/L3이다.PCI Express 디바이스가D0이면, 당연Link도L0없고L0s, 혹은L1(을)를 홀드 하고 있는 것이 보통이니까, 이것은 문제 없다.그런데 전력 절약 모드에 들어가면,Link도 상응하게 전력 다운을 도모하고 싶은 곳이다.그런데L2/L3의 경우, 지금의PCI Express그럼 반드시LDn(을)를 경유한다.즉1도Link Down하고, 재차 초기화를 걸치고 나서 접속을 복귀시킨다고 하는 일이 된다.이 결과,1도L2되어L3에 들어갔을 경우, 복귀까지 대단하게 시간이 걸리는 일이 된다.이 점은 당초부터 문제가 되어 있던 이야기로, 특히 루트 컴플렉스와 엔드 포인트가 직접 연결되고 있으면 문제 없지만, 사이에 스윗치가 들어가거나 하면 매우 귀찮은 것이 된다.
>
> 　L2/L3(을)를 고려에 넣지 않아도,L1에 들어가 있는 것만으로 실은 상당히 귀찮다.PCI Express에는ASPM(Active State Power Management)그렇다고 하는 기능이 있어,L0s/L1에 관해서는Exit Latency(L0s/L1(으)로부터L0에 복귀할 때까지의Latency)(을)를 보관 유지해 둘 수가 있지만, 이것이 나누기와 조잡하다.
>
> 출처 : [기글 하드웨어 뉴스 리포트 - PCI Express 3.0의 특징 4 - https://gigglehd.com/zbxe/newsreport/2068851](https://gigglehd.com/zbxe/newsreport/2068851)
> by 낄낄



PCI configuration space 개념 정리

http://melonicedlatte.com/computerarchitecture/2020/01/22/140000.html

![image](http://melonicedlatte.com/assets/images/202001/AA2C7852-8969-4940-A83D-10869851EAD1.png)



ASPM (Active State Power Management) 가 해주는 PCIe Link 상태 관리

https://www.sevenforums.com/tutorials/292971-pcie-link-state-power-management-turn-off-windows.html

![PCIe Link State Power Management - Turn On or Off in Windows-advanced_power_options.jpg](https://www.sevenforums.com/attachments/tutorials/271645d1486603536t-pcie-link-state-power-management-turn-off-windows-advanced_power_options.jpg?s=01beb946ca72c43e7509b696827c7a97)



Linux Kernel PCI Power Management

https://docs.kernel.org/power/pci.html

> PCI devices supporting the PCI PM Spec can be programmed to go to any of the supported low-power states (except for D3cold). 

| Current State \| New State |
| -------------------------- |
| D0 \| D1, D2, D3           |
| D1 \| D2, D3               |
| D2 \| D3                   |
| D1, D2, D3 \| D0           |

> PCI devices supporting the PCI PM Spec can be programmed to generate PMEs while in any power state (D0-D3), but they are not required to be capable of generating PMEs from all supported power states. In particular, the capability of generating PMEs from D3cold is optional and depends on the presence of additional voltage (3.3Vaux) allowing the device to remain sufficiently active to generate a wakeup signal.



window driver 가이드 - PCI Power Management

https://docs.microsoft.com/en-us/windows-hardware/drivers/pci/pci-power-management-and-device-drivers

> Scenario 2: PCI power management and device drivers
>
> 1. **ACPI driver**: Runs ASL code (_PS0 and _ON for any OnNow required power resources) to control the state external to the chip.
> 2. **PCI driver**: Puts the device in D0 using PCI-PM registers and restores Plug and Play configuration (interrupts and BARs--these might be different from what the device was previously on).
> 3. **Device driver**: Restores proprietary context in the device.



window driver 가이드 - A Device Enters a Low-Power State

https://docs.microsoft.com/en-us/windows-hardware/drivers/wdf/a-device-enters-a-low-power-state-umdf

> For each UMDF-based function and filter driver that supports the device, the framework does the following, in sequence, one driver at a time, starting with the driver that is highest in the driver stack:
>
> 1. If the driver is using self-managed I/O, the framework calls the driver's [**IPnpCallbackSelfManagedIo::OnSelfManagedIoSuspend**](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wudfddi/nf-wudfddi-ipnpcallbackselfmanagedio-onselfmanagediosuspend) callback function.
> 2. The framework stops all of the device's power-managed I/O queues and calls their [**IPnpCallbackSelfManagedIo::OnSelfManagedIoStop**](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wudfddi/nf-wudfddi-ipnpcallbackselfmanagedio-onselfmanagediostop) callback functions (if they exist).
> 3. If the driver is the device's power policy owner, the framework calls its [**IPowerPolicyCallbackWakeFromS0::OnArmWakeFromS0**](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wudfddi/nf-wudfddi-ipowerpolicycallbackwakefroms0-onarmwakefroms0) or [**IPowerPolicyCallbackWakeFromSx::OnArmWakeFromSx**](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wudfddi/nf-wudfddi-ipowerpolicycallbackwakefromsx-onarmwakefromsx) callback function.
> 4. The framework calls the driver's [**IPnpCallback::OnD0Exit**](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wudfddi/nf-wudfddi-ipnpcallback-ond0exit) callback function (if it exists).



ACPI 구조

https://kbench.com/?q=node/696

![img](https://images.kbench.com/korean/mouse_potato/hw_lec/1999_04/acpi/acpistr.gif)



window 10 development for absolute beginners

https://blogs.windows.com/windowsdeveloper/2015/09/30/windows-10-development-for-absolute-beginners/