# 2021-06-27 (Microarchitecutre)

- microprocessor (CPU)

- Intel Microarchitecture 시기별 정리: https://en.wikichip.org/wiki/intel/microarchitectures

arm 및 intel 모두 wikichip 에 잘 정리되어있었다.

- Inside the Machine 책 내용: http://www.inside-the-machine.com/
- Intel Michroarchitecture 발전 과정 정리 (한글): https://udteam.tistory.com/57

![현대 CPU의 구조 : 백엔드 편 - IYD - Everything Inside Your Device](https://t1.daumcdn.net/cfile/tistory/202600414D39E70F20)

- How FPGAs Work When They Don't (ppt): https://www.slideshare.net/InfinITnetvaerk/how-fpgas-work-when-they-dont

문서관리원 모델 관련된 내용이 있어서 참고하였다.

- Computer Architecture: A Quantitative Approach (John L. Hennessy & David A. Patterson)

Inside the Machine 책에서 언급하는 더 상위 책 (완전 초보를 위한 책과 이 책 사이의 간극을 메워주는 책이 Inside the Machine이다)

- 파이프라인 설명하는 이미지:https://techdecoded.intel.io/resources/understanding-the-instruction-pipeline/#gs.4n6odn

![img](http://simplecore-ger.intel.com/techdecoded/wp-content/uploads/sites/11/figure-2-3.png)

cf) Inside the machine 책에 의하면 SUV 자동차 공장이나 햄버거 가게 같은 서비스업을 비유로 들기 좋다고 한다.

- ISA 란 무엇인지 잘 설명한 내용 (한글): https://it-eldorado.tistory.com/21

> **ISA(Instruction Set Architecture)**의 진짜 정체는 무엇일까? ISA도 결국 특정 규칙과 체계를 가지고 있는 하나의 언어이다. 다만 한국말이나 영어와 같은 자연어가 아니고, C 언어나 자바와 같은 고급 언어도 아니며, CPU가 이해할 수 있는 0과 1로 이뤄진 기계어일 뿐이다.
>
> ISA는 일종의 **CPU 설계도**와 같다. 해당 CPU가 어떤 데이터들을 대상으로 어떤 연산들을 수행할 수 있는지, 어떤 종류의 레지스터들을 몇 개 사용하는지, 어떤 구조의 메모리와 호환이 가능한지 등을 명시한 규칙과 체계이다. 이러한 맥락에서 ISA를 **소프트웨어(프로그램)와 하드웨어(CPU) 사이의 인터페이스**라고도 부른다. C 언어 등으로 작성된 프로그램이 ISA의 규칙에 맞게 적절히 기계어로 번역이 되면 그것을 CPU가 읽고 해석해서 지시된 동작을 수행할 수 있기 때문이다.
>
> **하나의 CPU는 반드시 하나의 ISA를 사용하므로, 어떤 CPU를 설계하기 위해서는 그 CPU가 사용할 ISA에 대한 이해가 선행돼야 한다.** 예를 들어, 내가 인텔의 x86이라는 ISA를 사용하는 CPU를 만들어야 한다고 해보자. 그러면 나는 x86이 정의하고 있는 사항들을 충분히 숙지한 뒤에 CPU를 구성하는 논리 회로들을 설계해야 한다. 가령 x86이 명령어들을 64비트로 표현하고 그것의 상위 16비트를 opcode로 사용한다면, 64비트의 입력 중 상위 16비트만 Decoder에 입력시켜서 어떤 동작을 수행할지 판단하게 하는 논리 회로를 설계해야 한다.

