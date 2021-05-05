# 2021-05-05 (Linux objdump)

### vmlinux와 system map 파일의 차이

stack overflow에 올라온 질문을 보면 system.map 파일에서 보이는 주소와 vmlinux.o를 disassemble 한 주소 결과가 다르다는 질문이 있다. vmlinux.o 는 link하기 위한 파일이고, 최종 결정된 주소는 system.map이 맞다고 한다.

> `vmlinux.o` is a **relocatable** file, not **executable** file.
>
> Use `file vmlinux.o` to see its type, or more detailed `readelf -h vmlinux.o`
>
> Relocatable files need to be linked together to produce a final executable ( like `a.out`, for your case it's `vmlinux` ).
>
> The address in a relocatable file is not the final address, they will be relocated during linking. See [linking](https://en.wikipedia.org/wiki/Linker_(computing)), [ELF format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format). The addresses in `System.map` are the final one.

https://stackoverflow.com/questions/48553386/memory-addresses-in-the-system-map-and-vmlinux-o-after-building-linux-kernel



### system.map 파일

> - **System.map 파일**
>
> - - 리눅스 커널에 들어 있는 심벌에 대한 정보를 담고 있음.
>   - 이 파일은 커널 부팅과정에서 사용되지 않고, 부팅 이후 디버깅을 하는 프로그램 등에 의해 사용됨

[출처] https://blog.naver.com/aka_handa/10187321614



### vmlinux로 kernel symbol 확인

`readelf -s vmlinux` To view the kernel symbols.

[출처] https://www.programmersought.com/article/70204330130/



### vmlinux

vmliux는 최종 완성본이 아니고 최종본은 Image가 바이너리 형태의 파일로 만들어지는 듯하다. execute는 가능하지만 실제로 execute용으로 사용하지는 않고 디버깅 목적으로 많이 사용한다는 것 같다. 또한 vmlinux는 elf format이다. 그런데 kernel은 우리가 프로그램처럼 실행하는 게 아니고, 처음부터 boot 시켜서 그 주소로 뛰게 하는 것이니까 elf 같이 이런 정보들이 이미지 상에 필요하지는 않은 듯하다.

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/Elf-layout--en.svg/260px-Elf-layout--en.svg.png)

> **vmlinux**:
>
> A non-compressed and non-bootable Linux kernel file format, just an intermediate step to producing `vmlinuz`.
>
> **vmlinuz**:
> A compressed and bootable Linux kernel file. It is actually `zImage` or `bzImage` file.
>
> **zImage**:
> For old kernels, just fit `640k` ram size.
>
> **bzImage**:
> `Big zImage`, no `640k` ram size limit, can much larger.
>
> Please refer this document: [vmlinuz Definition](http://www.linfo.org/vmlinuz.html).

[출처] https://unix.stackexchange.com/questions/5518/what-is-the-difference-between-the-following-kernel-makefile-terms-vmlinux-vml

> vmlinux is a ELF format based file which is nothing but the uncompressed version of kernel image which can be used for debugging. The zImage or bzImage are the compressed version of kernel image which is normally used for booting.
>
> The vmlinux as such directly cannot be used by UBoot. However, by addition of metadata info in the process of creation of uImage for vmlinux, it is possible to boot via UBoot.

> The vmlinux is the boot file in ELF format, and then the initrd file (ram disk) is run in the same directory (/boot).
>
> The vmlinux file is practically the kernel itself.

[출처] https://stackoverflow.com/questions/41326607/what-is-the-use-of-vmlinux-file-generated-when-we-compile-linux-kernel



### objdump

>  objdump의 파일명은 architecture 별로 상이할 수 있으며, 주로 이용되는 사용법은 다음과 같다
>
> **objdump [-d] [-S] [-l] [obj 파일명] > [output 파일명]**
>
>  -d 옵션은 disassemble, 즉 어셈블리어 코드 목록을 생성한다.
>
>  -S 옵션은 소스코드를 최대한 보여주는 옵션이다. optimization 된 object file의 경우 소스코드가 모두 보이지 않을 수 있다.
>
>  -l 옵션은 소스코드에서의 line을 보여주는 옵션이다.
>
>  '> [output 파일명]' 을 하게 되면 파일명으로 export 하게된다. 이를 생략하면 화면에 즉시 출력된다.

[출처] https://log1.tistory.com/3



> objdump -d vmlinux 명령어를 입력하면 vmlinux에서 어셈블리 코드를 출력할 수 있습니다.
>
> root@raspberrypi:/home/pi/kernel_obj# objdump -d vmlinux
>
> vmlinux:   file format elf32-littlearm
>
> 80008000 <stext>:
>
> 80008000:	eb0430de bl	80114380 <__hyp_stub_install>
>
> 80008004:	e10f9000 mrs	r9, CPSR
>
> 80008008:	e229901a eor	r9, r9, #26
>
> 8000800c:	e319001f tst	r9, #31
>
> 80008010:	e3c9901f bic	r9, r9, #31
>
> 80008014:	e38990d3 orr	r9, r9, #211	; 0xd3

[출처] http://egloos.zum.com/rousalome/v/10009180