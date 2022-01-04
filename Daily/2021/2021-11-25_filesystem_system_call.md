# 2021-11-25 (filesystem system call)

reboot 나 poweroff 함수가 어떻게 호출되는지 알고 싶었다.

kernel 코드에 보면 `__attribute__ ((interrupt("IRQ")))` 이렇게 표현되어있었는데, 이건 빌드 시점에 interrupt handler 형태에 맞게 함수 등을 컴파일해준다고 한다.

> interrupt
>
> 지정된 속성이 인터럽트 핸들러임을 나타내려면이 속성을 사용하십시오. 컴파일러는이 속성이 존재하는 경우 인터럽트 핸들러에 사용하기에 적합한 함수 입력 및 종료 시퀀스를 생성합니다.
>
> 다음과 같이 인터럽트 속성에 선택적 매개 변수를 추가하여 처리 할 인터럽트 종류를 지정할 수 있습니다.

https://runebook.dev/ko/docs/gcc/arm-function-attributes



syscall 이 저장되는 파일 (`include/linux/syscall.h` `arch/arm64/kernel/sys.c` `include/uapi/asm-generic/unistd.h`) 등 관련 정보는 이전에 찾아서 정리한 글이 있다.

(2021-05-27_kernel_reboot_syscall.md)



파일 시스템에서 `reboot`과 `reboot -f` 동작이 달랐다. 마찬가지로 `poweroff`와 `poweroff -f`의 동작이 달랐다. f 옵션을 붙이면 스크립트에서 문제가 생겨도 계속 진행하기 때문이어서 그런 것 같다. f 옵션을 붙이지 않으면 `Starting Reboot...` 이라는 로그 혹은 `Starting Power-Off...` 이 보인다. f 옵션을 붙이면 `Rebooting.` 혹은 `Powering off.` 가 보인다. 그래도 결과적으로 syscall 을 호출하는 것은 맞는 듯하다.



https://stackoverflow.com/questions/29656136/is-there-a-system-call-service-routine-in-the-interrupt-vector

![syscall table](https://i.stack.imgur.com/VJ4mY.png)

Linux kernel syscall table 관련된 설명

```c
asmlinkage const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
    [0 ... __NR_syscall_max] = &sys_ni_syscall,
    #include <asm/syscalls_64.h>
};
```

https://0xax.gitbooks.io/linux-insides/content/SysCall/linux-syscall-2.html



Linux kernel syscall table

https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md



이건 조금 다른 주제이지만 file 을 열고 닫을 때 file descriptor 번호를 어떻게 file system 과 kernel 사이 주고 받느냐는 질문에 대한 내용이다.

https://stackoverflow.com/questions/29224775/how-linux-identify-a-particular-file-system-to-execute-system-call



reboot은 `reboot` 명령어를 사용하는데 suspend to ram은 왜 `echo mem > /sys/power/state` 일까 해서 찾아보았는데, 코드상에서 suspend는 syscall 로 되어있지 않았다. (`kernel/power/main.c`)

https://rnathsus.tistory.com/263