# 2023-01-16 (Static vs Dynamic Library)

### 라이브러리

라이브러리란 (+.a 와 .so 파일 빌드하는 명령어)

> 라이브러리(Library)는 다른 프로그램들과 링크되기 위하여 존재하는, 하나 이상의 서브루틴(Subroutine)이나 함수(Function)들의 집합 파일 말하는데 함께 링크(link)될 수 있도록 보통 **미리 컴파일된 형태인 오브젝트코드(Object code)** 형태로 존재한다.(Object들은 자주 사용하는 함수의 소스를 컴파일하여 만들 수 있음.) 라이브러리는 코드 재사용을 위해 조직화된 오래된 기법 중의 하나이며, 많은 다른 프로그램들에서 사용할 수 있도록, 운영체계나 소프트웨어 개발 환경 제공자들에 의해 제공되는 경우가 많다.
>
> 라이브러리라는 기술이 생긴 이유는 코드의 재사용(**자주 사용하는 함수들의 쉽게 사용 가능**) 및 부품화 실현, 소스를 제공하지 않음으로서 중요 기술의 유출을 방지할 수 있고, 라이브러리는 사용하는 개발자들로서는 대형 어플리케이션 개발 시간을 단축시킬 수 있다는 장점들이 주어지기 때문이다. 라이브러리 내에 있는 루틴들은 두루 쓸 수 있는 범용일 수도 있지만, 3차원 애니메이션 그래픽 등과 같이 특별한 용도의 함수로 설계될 수도 있다.

https://sens.tistory.com/33

라이브러리의 장점

> 라이브러리가 가지는 장점은 다음과 같다.
>
> *1. 코드를 **재사용**하기 쉽다.*
>
> *2. 코드의 내용을 숨겨 **기술 유출을 방지**할 수 있다.*
>
> *3. 이미 구현되어 있는 기능들을 가져다 쓸 수 있어 **개발 시간을 단축**할 수 있다.*
>
> *4. **컴파일 시간을 단축**할 수 있다. (라이브러리는 미리 컴파일되어 있어 링킹만 하면 바로 사용 가능하다)*

라이브러리 확장자 비교

> ***\*.dll : 윈도우 환경의 동적 라이브러리***
>
> ***\*.lib : 윈도우 환경의 정적 라이브러리***
>
> ***\*so : 리눅스 환경의 동적 라이브러리***
>
> ***\*a : 리눅스 환경의 정적 라이브러리***

https://bradbury.tistory.com/224

동적 라이브러리 (shared library)와 Linker/Loader 이해하기

https://www.lesstif.com/software-architect/shared-library-linker-loader-12943542.html

라이브러리의 entry point는 어디인가

https://stackoverflow.com/questions/14375977/do-dynamic-linked-libraries-dll-so-etc-have-an-entry-point

- windows 는 DllMain에 구현
- linux 는 아래와 같이 구현

```c
void __attribute__ ((constructor)) my_init(void);
void __attribute__ ((destructor)) my_fini(void);
```

Static vs Dynamic library 성능 비교

: 정적 라이브러리가 최적화가 가능함 그래서 더 빠름

https://stackoverflow.com/questions/4384752/static-vs-dynamic-library-performance

Static vs Shared Library 특징 비교

| Properties | Static library                                               | Shared library                                               |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Time       | Takes longer to execute, because loading into the memory happens every time while executing. | It is faster because shared library code is already in the memory. |

https://www.geeksforgeeks.org/difference-between-static-and-shared-libraries/

object 파일과 static library 파일의 차이

: static library 는 object 파일과 static library 를 합친 것 뿐이다! (archive라고 표현함)

https://stackoverflow.com/questions/6177498/whats-the-difference-between-object-file-and-static-libraryarchive-file

https://softwareengineering.stackexchange.com/questions/389383/what-is-the-difference-between-a-static-library-and-an-archive-library

참고 그림 (static library vs. dynamic library)

![img](https://miro.medium.com/max/700/0*cL47CWcJO1HaTET3.png)

https://medium.com/@estebandelahoz/static-libraries-vs-dynamic-libraries-8496accd2c15

object file 외에 library 가 필요한 이유

https://stackoverflow.com/questions/23615282/object-files-vs-library-files-and-why

Static vs Dynamic Libraries Tutorial

![static_vs_dynamic_libraries2](https://www.bogotobogo.com/cplusplus/images/libraries/static_vs_dynamic_2.png)

https://www.bogotobogo.com/cplusplus/libraries.php

static library vs. dynamic library

![img](https://miro.medium.com/max/471/1*MvSE8agQXoC-FbAsYnS8IQ.png)

https://medium.com/@wendymayorgasegura/dynamic-library-vs-static-library-8b28bae039de

Dynamic loading 은 OS 의 역할이 필요하다 (dynamic linking 과는 구별이 필요함)

https://gateoverflow.in/299786/dynamic-loading-require-special-support-operating-system

Dynamic loading 과 Dynamic linking 차이

https://www.educative.io/answers/dynamic-loading-vs-dynamic-linking

OS 없이 Dynamic linking 기능 만들 수 있을까? : 어렵다

https://www.embeddedrelated.com/showthread/comp.arch.embedded/200239-1.php

OS 에서 static vs. dynamic linking (OS tutorial 공부하기 좋은 사이트)

https://prepinsta.com/operating-systems/static-vs-dynamic-linking/

Window 와 Linux 에서 dynamic linking 동작의 차이

https://stackoverflow.com/questions/58603797/dynamic-linking-linux-vs-windows

Dynamic linker란

> In [computing](https://en.wikipedia.org/wiki/Computing), a **dynamic linker** is the part of an [operating system](https://en.wikipedia.org/wiki/Operating_system) that [loads](https://en.wikipedia.org/wiki/Loader_(computing)) and [links](https://en.wikipedia.org/wiki/Linker_(computing)) the [shared libraries](https://en.wikipedia.org/wiki/Shared_libraries) needed by an [executable](https://en.wikipedia.org/wiki/Executable) when it is executed (at "[run time](https://en.wikipedia.org/wiki/Run_time_(program_lifecycle_phase))"), by copying the content of libraries from [persistent storage](https://en.wikipedia.org/wiki/Persistent_storage) to [RAM](https://en.wikipedia.org/wiki/RAM), filling [jump tables](https://en.wikipedia.org/wiki/Jump_table) and relocating [pointers](https://en.wikipedia.org/wiki/Pointer_(computer_programming)). The specific operating system and executable format determine how the dynamic linker functions and how it is implemented.

https://en.wikipedia.org/wiki/Dynamic_linker

Dynamic Linking 의 장단점

https://lemp.io/the-advantages-and-disadvantages-of-dynamic-linking/



### Linux Library

static library 와 dynamic library 설명 및 명령어 정리 잘 됨

https://www.linkedin.com/pulse/difference-between-static-dynamic-libraries-carlos-araque-baleta

확장자별 의미

> .a — stands for “archive”
> .so — stands for “shared object”

https://www.baeldung.com/linux/a-so-extension-files

static library 의 내부

https://stackoverflow.com/questions/3757108/contents-of-a-static-library

object file 과 symbol (object file 내부에 symbol 이 있다. elf 파일이기 때문에)

http://nickdesaulniers.github.io/blog/2016/08/13/object-files-and-symbols/

https://blog.naver.com/PostView.naver?blogId=techref&logNo=222221869206&from=search&redirect=Log&widgetTypeCall=true&directAccess=false

Procedure Linkage Table 의 역할 (동적 라이브러리는 주소를 알려주기 위한 PLT 가 필요하다)

https://stackoverflow.com/questions/20486524/what-is-the-purpose-of-the-procedure-linkage-table

https://ctf101.org/binary-exploitation/what-is-the-got/

https://bpsecblog.wordpress.com/2016/03/07/about_got_plt_1/

dynamic loading 과 plt 그림

![figure 2](https://d3i71xaburhd42.cloudfront.net/cfbc65b2dec34a8fdf8ca8b909d0c358cb55ccf2/5-Figure2-1.png)

https://www.semanticscholar.org/paper/The-inside-story-on-shared-libraries-and-dynamic-Beazley-Ward/cfbc65b2dec34a8fdf8ca8b909d0c358cb55ccf2



### Windows Library

윈도우 dll 동적 라이브러리 동작 원리

> 다시 시작하는 윈도우 프로그래밍
>
> 동적 연결 라이브러리 동작 원리
>
> 항공기 사고의 74%는 이륙 후 3분, 착륙 전 8분 동안 발생한다. 이를 흔히 ‘마의 11분’이라고 부르는데 크리티컬한 작업에서 시작과 끝이 얼마나 중요하고 위험한지를 의미한다. DLL을 사용할 때도 대부분의 오류가 항공기 사고와 유사하게 DLL을 메모리에 적재하는 시점이나 해제하는 과정에서 발생하며, 이는 윈도우가 DLL 목록을 처리하는 특수한 방식 때문이다. 이 글에서는 바로 이러한 특수성에 대해 살펴보고 DLL로 인한 오류를 예방하기 위해 주의해야 할 점들을 살펴본다.

https://dataonair.or.kr/db-tech-reference/d-lounge/expert-column/?mod=document&uid=52627

http://www.jiniya.net/wp/archives/10447



### 요약

라이브러리란 소스코드를 공개하지 않거나, 코드를 재사용하기 위해서 미리 컴파일한 오브젝트 파일이다. 라이브러리는 정적 라이브러리와 동적 라이브러리 (공유 라이브러리) 가 있다. Linux 의 정적 라이브러리는 '.a' 파일로 빌드되며 빌드될 때 필요한 부분마다 삽입하기 때문에 코드 중복이 있다. 그러나 코드 최적화가 가능하고 동적으로 연결된 것보다 속도가 빠르다. 반면, 공유 라이브러리는 '.so' 파일로 빌드되고, 여러 프로그램들이 공유해서 사용한다. 코드 최적화가 안 되고 속도가 느리다. 운영 체제에서 Dynamic Linker 가 스캔해서 공유 라이브러리를 관리해야하기 때문에 운영체제가 없으면 사용하기 어렵다. 최초로 호출할 때 공유 라이브러리의 함수 위치를 찾아서 PLT 에 저장해두고, 그 다음부터는 호출할 때 함수 위치를 새로 찾지 않고 기존에 찾아둔 테이블의 위치를 참조하여 접근한다.



### Library 테스트 방법

1. main.c 를 작성한다.

```c
#include <stdio.h>

extern int lib_test(void);
int main(void){
        printf("This is main!\n");
        lib_test();
        return 0;
}
```



2. 라이브러리로 사용할 lib_test.c 를 작성한다.

```c
#include <stdio.h>

int lib_test(void)
{
        printf("This is library!\n");
        return 0;
}
```



3. lib_test.c 를 라이브러리로 만들기 위해 우선, lib_test.o 오브젝트 파일로 컴파일한다.

```shell
$ cc -o lib_test.o -c lib_test.c
```



4. 그리고 lib_test.o 오브젝트 파일을 이용해서 libtest.a 라이브러리 파일을 생성한다. 생성되는 라이브러리 파일의 이름은 접두사가 lib 여야하고, 접미사는 .a 여야한다.

```shell
$ ar -rc libtest.a lib_test.o
```



5. 만들어진 라이브러리를 활용하여 최종 프로그램을 컴파일합니다.

아래 명령어의 의미는 만들어질 산출물 이름은 (-o 옵션은 결과물 이름을 의미함) final_program.out 이며, main.c 를 컴파일해서 만들 것이고, 라이브러리가 있는 디렉토리 위치는 현재 위치 (-L 옵션에 현재 위치를 뜻하는 . 을 붙임), 라이브러리 명은 libtest.a (접두사, 접미사를 제외하고 -l 옵션에 test 만 쓰면 이렇게 인식됨) 이다.

```shell
$ cc -o final_program.out main.c -L. -ltest
```



6. 출력 결과는 다음과 같다.

```shell
$ ./final_program.out
This is main!
This is library!
```



[참고] https://docs.oracle.com/cd/E19120-01/open.solaris/819-0690/chapter2-9/index.html

[참고] https://jihadw.tistory.com/133



### 기존에 Library 가 있는 경우

1.  위의 절차 중 3번부터 진행한다. 컴파일러를 사용하여 라이브러리 파일을 오브젝트 파일로 빌드한다. (-o 옵션 뒤의 인자가 생성될 결과물의 이름) 컴파일러의 이름은 긴 앞부분은 생략하고 뒷부분만 표기했다.

```shell
$ ~/aarch64-poky-linux-gcc -c lib_test.c -o lib/lib_test.o
```



2. 위에서 생성된 오브젝트 파일을 사용하여 라이브러리 파일을 생성한다. 생성되는 라이브러리명은 lib_test.a 이다.

```shell
$ ~/aarch64-poky-linux-ar -rc lib/lib_test.a lib/lib_test.o
```



3. 기존에 라이브러리가 있기 때문에 2개의 라이브러리를 하나로 합치는 작업을 추가로 진행한다. (기존 라이브러리명은 libprev.a 라고 가정한다.)

```shell
$ ~/aarch64-poky-linux-ar -rcT lib/liblib_test.a lib/libprev.a lib/lib_test.a
```



옵션 설명: -L 은 라이브러리가 존재하는 디렉토리명, -l 은 라이브러리명, -I 는 헤더파일 디렉토리, -i는 헤더파일명 

https://www.rapidtables.com/code/linux/gcc/gcc-l.html

https://www.ibm.com/docs/en/zos/2.2.0?topic=descriptions-ar-create-maintain-library-archives

https://seamless.tistory.com/2 (옵션 표로 잘 정리되어있음)



컴파일러 전체 주소 예시: opt/poky/sysroots/x86_64-pokysdk-linux/usr/bin/aarch64-poky-linux/aarch64-pocky-linux-

https://discourse.cmake.org/t/ccmake-doesnt-automatically-found-the-toolchain-file-where-cmake-does/2887



### 라이브러리 사용시 겪었던 에러들

1. 정적 라이브러리 (.a) 들만 가지고 합쳐서 하나의 정적 라이브러리 (.a) 를 만들고자 했을 때 아래 에러 발생

에러 내용: could not read symbols: Archive has no index; run randlib to add one

https://stackoverflow.com/questions/2765240/could-not-read-symbols-archive-has-no-index-run-ranlib-to-add-one

https://kldp.org/node/116896

.a 와 .o 를 합치면 괜찮을 거라 생각했는데 그것도 아니었다. 라이브러리를 만들 때는 .o 파일들만 합쳐야했다.

에러 내용: undefined symbol

https://stackoverflow.com/questions/33851045/gcc-a-static-library-with-undefined-symbols

해결 방법: libtool 을 쓰거나 ar crsT 옵션을 쓰면 된다고 한다.

https://stackoverflow.com/questions/3821916/how-to-merge-two-ar-static-libraries-into-one