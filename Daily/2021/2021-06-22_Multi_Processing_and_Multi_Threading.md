# 2021-06-22 (Multi Processing and Multi Threading)

Multi processing과 Multi threading을 설명하는 그림

![img](https://miro.medium.com/max/1526/1*F8ckVaR__PlBssnf-mn76A.png)

![Multiprocessing vs Multithreading](https://i.imgur.com/eO0ux4b.jpg)

> ## What is MultiProcessing?
>
> - Multiprocessing allows you to **spawn multiple processes within a program**.
> - It allows you to **leverage multiple CPU cores** on your machine
> - **Multiple processes within a program do not share the memory**
> - **Side steps the GIL(Global Interpreter Lock) limitation of Python** which allows only one thread to hold control of the Python interpreter
> - **Used for computation or CPU intensive programs**
>
> **then what is Multi-threading and when to use it?**
>
> **A Thread is the**
>
> - **Smallest set of independent commands executed in a program**
> - Multiple threads within an application can execute simultaneously on a CPU referred to as **MultiThreading**
> - **Runs always within a program** and cannot run on its own
> - **Used when programs are network bound or there is heavy I/O operation**
> - **Memory is shared between multiple threads within a process** and hence has lower resources consumption

![img](https://miro.medium.com/max/747/1*6Y7JWcTJUS4v4_CVHx8CCA.png)

> `멀티프로세싱`은 fork를 통해 프로세스를 다수개로 늘려 **여러 개의 프로그램**들을 병렬로 처리하며, `멀티스레딩`은 **하나의 프로그램 안**에서 병렬 처리를 한다.
> 멀티 스레딩은 멀티 프로세싱보다 훨씬 적은 자원을 소모하기 때문에 더 효율적이나 안정성 측면에서는 멀티 프로세싱이 안정적이다.
>
> 멀티 프로세싱은 **다수의 프로세서가 서로 협력적으로 일을 처리하는 것을 의미**한다.
> 컴퓨터는 1대인데 프로세서(CPU)는 2개 이상이다. 보통, 멀티코어시스템(multi-core system)을 포함한다.

![img](https://media.vlpt.us/images/chy0428/post/64d411b0-0f60-48e0-897e-f535e98ee9d0/image.png)



> [`multiprocessing`](https://docs.python.org/ko/3/library/multiprocessing.html#module-multiprocessing) 은 [`threading`](https://docs.python.org/ko/3/library/threading.html#module-threading) 모듈과 유사한 API를 사용하여 프로세스 스포닝(spawning)을 지원하는 패키지입니다. [`multiprocessing`](https://docs.python.org/ko/3/library/multiprocessing.html#module-multiprocessing) 패키지는 지역과 원격 동시성을 모두 제공하며 스레드 대신 서브 프로세스를 사용하여 [전역 인터프리터 록](https://docs.python.org/ko/3/glossary.html#term-global-interpreter-lock) 을 효과적으로 피합니다. 이것 때문에, [`multiprocessing`](https://docs.python.org/ko/3/library/multiprocessing.html#module-multiprocessing) 모듈은 프로그래머가 주어진 기계에서 다중 프로세서를 최대한 활용할 수 있게 합니다. 유닉스와 윈도우에서 모두 실행됩니다.

[출처] https://docs.python.org/ko/3/library/multiprocessing.html



Q. 프로세스 하나가 CPU 하나에 완전 대응되는 것인가? fork 하면 다른 CPU 에 배정되는 것인가?

=> 위의 설명들이 섞여 있어서 그런데 위키 백과에 보면 설명이 따로 나뉘어있다.

> (1) **Multiprocessing** is the use of two or more [central processing units](https://en.wikipedia.org/wiki/CPU) (CPUs) within a single [computer system](https://en.wikipedia.org/wiki/Computer_system).[[1\]](https://en.wikipedia.org/wiki/Multiprocessing#cite_note-Rajagopal1999-1)[[2\]](https://en.wikipedia.org/wiki/Multiprocessing#cite_note-EbbersKettner2012-2) The term also refers to the ability of a system to support more than one processor or the ability to allocate tasks between them. 
>
> (2) At the [operating system](https://en.wikipedia.org/wiki/Operating_system) level, *multiprocessing* is sometimes used to refer to the execution of multiple concurrent [processes](https://en.wikipedia.org/wiki/Process_(computing)) in a system, with each process running on a separate CPU or core, as opposed to a single process at any one instant.[[6\]](https://en.wikipedia.org/wiki/Multiprocessing#cite_note-MorleyParker2012-6)[[7\]](https://en.wikipedia.org/wiki/Multiprocessing#cite_note-Shibu-7) When used with this definition, multiprocessing is sometimes contrasted with [multitasking](https://en.wikipedia.org/wiki/Computer_multitasking), which may use just a single processor but switch it in time slices between tasks (i.e. a [time-sharing system](https://en.wikipedia.org/wiki/Time-sharing_system)). 

https://en.wikipedia.org/wiki/Multiprocessing

어느 레벨에서 보냐에 따라 설명에 차이가 나는 것이다. 프로세서를 여러 개 쓴다는 뜻이 가장 맞는 설명인 것 같다.



참고

leverage 뉘앙스: https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=yp2k&logNo=80204601806

참고 Python GIL: https://burningrizen.tistory.com/252