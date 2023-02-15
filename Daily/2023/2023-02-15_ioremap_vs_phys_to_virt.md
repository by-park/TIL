# 2023-02-15 (ioremap_vs_phys_to_virt)

### ioremap vs phys_to_virt

https://rootfriend.tistory.com/entry/ioremap-VS-phystovirt

> ioremap()함수와 phys_to_virt()함수는, 둘다 물리주소를 가상주소로 바꿀때 쓰인다.
>
> 하지만 **차이점**은 존재한다.
>
> ioremap()함수는 요구된 물리 주소로 시작하는 영역을 커널 모드에서 사용할 수 있도록 가상 주소 공간으로 **등록** 하지만,
>
> phys_to_virt() 함수는 PAGE_OFFSET과 같은 값을 이용하여 **변환 처리만 계산**하기 때문이다.
>
> void* ioremap(unsigned long offset , unsigned long size);
> 반환 값: 가상 주소의 선두 주소
> offset : 물리 주소의 시작 주소
> size : 크기
>
> void* phys_to_virt(unsigned long address);
> 반환 값 : 변환된 가상주소
> address : 물리 주소
>
>
> \* **PAGE_OFFSET** 매크로 상수값 : 물리 주소와 가상 주소간에 변환을 위해 쓰이며, #include <asm/page.h>에 정의된다.

http://chammoru.egloos.com/v/4168312

>virt_to_phys() 함수는 가상 주소를 물리 주소로 바꾸고, phys_to_virt() 함수는 물리 주소를 가상 주소로 바꾼다. 그러나 phys_to_virt() 함수는 ioremap()함수와 같은 기능을 수행하지 않는다. ioremap()함수는 요구된 물리 주소로 시작하는 영역을 커널 모드에서 사용할 수 있도록 가상 주소 공간으로 등록하지만, phys_to_virt()함수는 PAGE_OFFSET과 같은 값을 이용하여 변환 처리만 하기 때문이다. 그래서 virt_to_phys()와 phys_to_virt()함수는 커널에서 사용할 수 있도록 등록된 가상 주소와 물리 주소에 대해 사용해야 한다.

요약하면 virt_to_phys 는 이미 kernel 에 등록된 가상 주소 영역 (ex. 커널 코드가 올라가있는 DRAM 영역) 에 사용하는 것이다. ioremap 은 외부 접근을 위한 i/o 영역을 remap 을 해주는 것이므로, kernel 에서 등록된 가상 주소가 아닌 다른 영역 (ex. special function register 영역) 에 사용된다.



### ioremap 설명 참고

제목: How does ioremap convert physical address to virtual address?

출처: https://www.quora.com/How-does-ioremap-convert-physical-address-to-virtual-address

Follow through the function declaration:

[Linux Cross Reference](http://lxr.free-electrons.com/source/arch/x86/mm/ioremap.c#L229) - ioremap()

[Linux Cross Reference - ioremap_nocache()](http://lxr.free-electrons.com/source/arch/x86/include/asm/io.h#L192)

[Linux Cross Reference - ioremap_caller()](http://lxr.free-electrons.com/source/arch/x86/mm/ioremap.c#L83)

The page range allocated for ioremap is listed here:![img](https://qph.cf2.quoracdn.net/main-qimg-786cae8c4b98ab663a75a52559b9ed5f-lq)

And a good graphic summary is here (it is just formula - one byte to one byte linear mapping from MMIO physical space to the virtual address space of kernel) :![img](https://qph.cf2.quoracdn.net/main-qimg-876a1d3864e31e6b36c9d40687c0b357-pjlq)Or if you want to follow through the whole course:[EMBEDDED LINUX KERNEL AND DRIVER DEVELOPMENT](http://www.makelinux.net/books/embedded_linux_kernel_and_drivers/img0)

In case pgd, pud, pmd are foreign words for you:

![img](https://qph.cf2.quoracdn.net/main-qimg-d5d51babf5982ad7152fdb803e646d36)

Or you can look up "intel page table" ( PGD, PTE, PMD are generic concept, but each CPU implement its own page table internals):

[Wiki Page table](https://en.wikipedia.org/wiki/Page_table)

For eg, this is PowerPC page table:![img](https://qph.cf2.quoracdn.net/main-qimg-c3e0d5bd5b77f111f30d41169b2baf88)



### kernel 의 메모리 영역 보기

```shell
$ cat /proc/iomem
```

질문

메모리 영역 중 APIC 영역이 reserved 로 잡혀있다는 질문

https://stackoverflow.com/questions/6984916/help-with-apic-functions-in-linux

답변

The APIC address is a physical memory address, but you are trying to access it as a linear memory address - that's why your first approach doesn't work. The memory is marked as "reserved" precisely *because* it belongs to the APIC, rather than real memory.

You should use the internal kernel functions. To do so, you should include `` and use:

```
apic->send_IPI_allbutself(vector);
```