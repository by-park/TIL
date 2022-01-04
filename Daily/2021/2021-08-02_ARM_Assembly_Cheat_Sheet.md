# 2021-08-02 (ARM Assembly Cheat Sheet)

### ARM Assembly Cheat Sheet from Azeria Labs

예문과 해석이 있어서 명령어 이해하기가 좋다. 한 장 안에 많은 개념이 표현되는 것도 나중에 공부하기 좋을 것 같다.

https://azeria-labs.com/assembly-basics-cheatsheet/

![img](https://azeria-labs.com/downloads/cheatsheetv1.3-1920x1080.png)



### Cheatography 의 ARM assembly

https://cheatography.com/syshella/cheat-sheets/arm-assembly/

pdf로도 뽑을 수 있다. 다만 각 명령어의 예문은 없다.



### Github Arm Cheatsheet

명령어에 destination과 source 표시가 구분이 안 되어있어서 아쉽다. 직접 적은 게 아니라 pdf 형식으로 되어있는 것도 아쉽다.

https://github.com/oowekyala/arm-cheatsheet/blob/master/arm-cheatsheet.pdf



### ARM Assembly 기초

전체적인 ARM assembly 설명이 이해하기 쉽게 되어있다.

https://kyuhyuk.kr/article/simple-arm-operating-system/2019/03/04/Simple-ARM-Operating-System-Chapter-2



### ARM Instruction Set 중 Branch

명령어 format과 branch 관련 조건이 표로 잘 정리되어있다.

eq는 CPSR 레지스터의 Z가 1일 때 점프하는 것이고, ne는 Z 가 0일 때 점프하는 것이다.

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=mayooyos&logNo=220830008603



### ARM 각 명령어 검색 사전 (ARM KEIL)

https://www.keil.com/support/man/docs/armasm/armasm_dom1361289878994.htm



### ARM 어셈블리 가이드

ARM 어셈블리 기초 연산들 설명이 있고, CMP 하면 if (Rn-RM ==0) 이라는 설명이 있다. 이 결과를 NZCV에 저장하는데, Z는 연산결과가 0인 경우 1 로 SET한다.

https://blog.naver.com/PostView.naver?blogId=gangst11&logNo=145839687&redirect=Dlog&widgetTypeCall=true&directAccess=false



### 컴퓨터 구조 정리

전반적인 구조 관점과 어셈블리를 설명하는데, BGT, BGE 가 "왼쪽이 오른쪽보다 크면, 크거나 같으면" 설명이 명확하게 되어있는 듯하다.

https://themangs.tistory.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EA%B5%AC%EC%A1%B0-%EC%A0%95%EB%A6%AC-2%EB%B2%88%EC%A7%B8



### 주요 어셈블리 명령어 요약

mov, add, b 등 기초 명령어 요약이 되어있다.

http://kaludin.egloos.com/v/2668154



### TST 명령과 ARM CPSR 레지스터

TST 명령을 하면 AND 비트 연산을 수행하고, 결과가 1인 경우 CPSR 레지스터의 Z 비트는 0이 된다. 

(이게 CMP 처럼 생각하는 것과는 반대되는 부분이라 헷갈렸다. B.NE 면 tst로 결과가 1이 나오면 점프하는 것이 맞다.)

http://rousalome.egloos.com/v/9991439



### Table of Branch Instructions (MIPS)

Branch instruction 전체 종류 정리

자세한 설명을 없으나 한 눈에 보기 좋다.

https://chortle.ccsu.edu/assemblytutorial/Chapter-24/ass24_4.html



### ARM MPIDR

ARM의 CPU 번호를 알 수 있는 레지스터. Cluster 번호와 CPU 번호가 있다.

![img](https://documentation-service.arm.com/static/5e7de0ffcbfe76649ba535b2?token=)

8 비트부터는 CPU 번호가 있고, 16 비트부터는 Cluster 번호가 있다. 예를 들어 0번 CPU는 0000 이고, 1번 CPU는 0100 이다.

https://developer.arm.com/documentation/100403/0200/register-descriptions/aarch32-system-registers/mpidr--multiprocessor-affinity-register