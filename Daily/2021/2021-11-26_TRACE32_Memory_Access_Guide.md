# 2021-11-26 (TRACE32 Memory Access Guide)

### T32 로 Memory Access 할 때 적는 명령어 설명

![img](http://trace32.com/data/wiki/2015-04-14/1429009553.png)

**3)**  **DAP****의 AXI/AHB Master를 통한 physical Address Access**

ARM SoC설계시 Debug Component중 DAP(Debug Access Port)을 설계할 경우 설계자가 DAP내에 AXI 또는 AHB master와 같은 MEM-AP(Memory Access Port)를 설계한 경우 TRACE32 Debugger는 CPU를 통하지 않고 내장된 BUS master를 통해 바로(직접) System BUS에 Access할 수 있습니다.

아래 Block Diagram은 DAP의 Memory Access Port 연결도를 보여 주고 있습니다. DAP은 최대 256 개의 MEM-AP를 가질 수 있으며 보통은 3~5개 정도를 설계하고 있습니다. 경우에 따라 AXI/AHB Access Port를 둘다 설계하는 경우도 있으며 CPU Core를 통하지 않고 DAP 내 MEM-AP인 AXI-AP또는 AHB-AP를 통해 System BUS를 Access하고자 하는 경우 사용하게 됩니다. 따라서 여러 개의 MEM-AP 중 어떤 MEM-AP를 통해 System BUS에 Access할것인지에 대한 설정이 필요하며 이의 구분을 위해 “Memory Class”라는 것을 두어 속성을 정의하게 되었습니다.

 

**ZAXI:[address]**   : Secure 권한(Z)으로 AXI-AP를 통해 해당 BUS를 Access함

**NAXI:[addresss]**  : Non-Secure 권한(N)으로 AXI-AP를 통해 해당 BUS를 Access함

**ZAHB:[address]**  : Secure 권한(Z)으로 AHB-AP를 통해 해당 BUS를 Access함

**NAHB:[address]**  : Non-Secure 권한(N)으로 AHB-AP를 통해 해당 BUS를 Access함

![img](http://trace32.com/data/wiki/2015-04-14/1429009634.png)

예를 들면 만약 CPU core를 통하지 않고 DAP에 내장된 AHB-AP를 통해 0x2020000 번지의 값을 읽어 확인하고 싶다면 아래와 같이 Access 할 수 있습니다.

**Data.dump ZAHB:0x2020000**    ; Scure권한으로 AHB-AP를 통해 0x2020000번지 읽기

아래 그림은 CPU core를 통하지 않고 DAP내의 AHB-AP를 통해 바로 0x2020000번지의 메모리 값을 Secure권한으로 읽어 낸 예입니다.

![img](http://trace32.com/data/wiki/2015-04-14/1429009731.png)

만약 AHB-AP나 AXI-AP를 통해 특정 번지에 특정 값을 쓰고자 한다면

**Data.Set AXI:[Address] %[Long|Word|Byte] [Value]**

**Data.Set AHB:[Address] %[Long|Word|Byte] [Value]**

Ex) Data.Set ZAHB: 0x406D1ED0 %Long 0x7654321

: Secure 권한으로 Access하되 AHB를 통해 0x406D1ED0번지에 32bit Bus size로 0x7654321 쓰기

 

그러나 DAP이 가질 수 있는 MEM-AP의 개수가 최대 256개다 보니DAP의 어느AP를 통해 Access할 것인지 그리고 그 AP의 Port 번호는 몇인지 TRACE32에게 알려줄 필요가 있겠습니다. 다음 그림은 이에 대한 설정 윈도우를 보여주고 있습니다. CPU 메뉴 >> SYStem Setting…. 클릭 후 SYStem 윈도우가 열리면 “CONFIG” 버튼을 클릭하고 우측과 같이 “SYStem.CONFIG” 윈도우가 열리면 DAP 탭을 선택하여 MEM-AP 에 대한 설정을 하시면 되겠습니다. 아래 설정의 경우 AHB-AP는 0번 포트를, AXI-AP는 5번 포트를, Debugging을 위한 Debug Component들이 위치한 APB-AP는 1번 포트를 사용하도록 설정하고 있습니다. 이는 해당 SoC의 AP설계 정보를 TRACE32에게 알려주는 역할을 하게 됩니다.

만약 Memory Class를 ZAXI:0x2020000으로 정의하여 Access하게 되며5번 포트인 AXI-AP를 통해 BUS에 Access하는 동작을 하게 됩니다.

![img](http://trace32.com/data/wiki/2015-04-14/1429009764.png)



### MMU table 보는 방법

현재의 Kernel/User Space 전영역의 MMU Table확인 명령은

**MMU.List.PageTable**

입니다. MMU.List.PageTable은 Default로 Virtual Address가 0x0이므로 Kernel/User Space의 전 영역을 보여줍니다. 아래 그림은 MMU.List.PT에 의한 현재 TTBRx가 가리키는 Current MMU Table을 보이고 있습니다. 즉 User Space는 현재 TTBR0가 가리키는 Table을 보입니다.

![img](http://trace32.com/data/wiki/2015-04-14/1429012295.png)



[출처]

http://trace32.com/wiki/index.php/Accessing_System_Data#.C2.A0_.C2.A0_.C2.A0kernel_.EC.98.81.EC.97.AD_MMU_Table_.ED.99.95.EC.9D.B8