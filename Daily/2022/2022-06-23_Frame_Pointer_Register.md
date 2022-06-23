# 2022-06-23 (Frame Pointer Register)

x29 가 훼손되어서 디버깅했던 기록

아래와 같이 ARM 규약에 따르면 x29 는 함수 호출 전후로 훼손되면 안 된다. 그런데 suspend 에 들어갔다가 resume 했더니 x29 (frame pointer) 에서 꺼내온 lr 값이 이상했다.



Caller 가 저장하는 레지스터 (x9 - x15)

Callee 가 저장하는 레지스터 (x19 - x29)

https://developer.arm.com/documentation/den0024/a/The-ABI-for-ARM-64-bit-Architecture/Register-use-in-the-AArch64-Procedure-Call-Standard/Parameters-in-general-purpose-registers

Caller 가 호출하는 쪽, Callee 가 호출되는 쪽

![img](http://1.bp.blogspot.com/-JKw3ByhdFhs/VRJHfmBfbfI/AAAAAAAAA6E/IT5aAQoITxQ/s1600/caller.png)

http://cr3denza.blogspot.com/2015/03/caller-callee.html

https://developer.arm.com/documentation/102374/0100/Procedure-Call-Standard



x29 에 대해 알아보니 Frame pointer라고 한다.

아래 경로의 정수 레지스터 부분 참고 (x29/fp 비휘발성 프레임 포인터)

https://docs.microsoft.com/ko-kr/cpp/build/arm64-windows-abi-conventions?view=msvc-170

아래 경로의 Register names 라는 테이블 참고

https://doc.rust-lang.org/nightly/reference/inline-assembly.html

> 스택이 자라난다는 소리가 뭐냐구요? 스택 공간에는 로컬 변수를 할당 받아 쓰고 가장 중요한 함수가 호출될 때 돌아갈 주소(Linked Register)와 돌아갈 주소가 있는 주소(Frame Pointer 주소)를 스택에 저장하거든요. 스택에 어떤 값을 푸쉬하면 자연히 스택 주소가 점점 작아져요. 이걸 스택이 자란다고 할 수 있죠. 높은 주소에서 낮은 주소로요. 

http://egloos.zum.com/rousalome/v/9966225



컴파일할 때 옵션으로 `-fomit-frame-pointer` 을 주면 x29 를 frame pointer로 사용하지 않는다. frame pointer가 필요없는 경우 사용하지 않는다고 한다. `-fno-omit-frame-pointer` 를 주면 x29 를 frame pointer로 사용한다.

https://developer.arm.com/documentation/101754/0616/armclang-Reference/armclang-Command-line-Options/-fomit-frame-pointer---fno-omit-frame-pointer

xen 이나 linux kernel 코드를 보면 `CONFIG_FRAME_POINTER` 라는 config 가 있다. 이걸 켜면 위와 같은 컴파일 옵션이 붙어서 빌드되는 것이다.

```c
// xen/Makefile
ifeq ($(CONFIG_FRAME_POINTER),y)
CFLAGS += -fno-omit-frame-pointer
else
CFLAGS += -fomit-frame-pointer
endif
```

이 옵션을 켜주지 않아서 x29 자체에도 이상한 값이 들어있었다.



x29 과 lr 을 저장했다가 꺼낼 때 stp, ldp 를 사용하는데 이는 원래 2개의 값을 스택에 저장하는 명령어라고 한다. 

![img](https://postfiles.pstatic.net/MjAyMTA2MDZfMTEw/MDAxNjIyOTUxNTk1NzI3.Sl4_41UqCMz0XS9S8R9zTX_6veCATc8Gfgoewch_xzkg.i1Br4iOUmo5kCfN7u8BPJUEXyzX4DkU4ZX2ExdMWQEYg.PNG.crushhh/%EA%B7%B8%EB%A6%BC-7-5.png?type=w580)

https://blog.naver.com/PostView.naver?blogId=crushhh&logNo=222386913294&parentCategoryNo=&categoryNo=212&viewDate=&isShowPopularPosts=true&from=search

linux kernel 코드를 보면 다음과 같다.

https://github.com/torvalds/linux/blob/master/arch/arm64/kernel/sleep.S

```assembly
SYM_FUNC_START(__cpu_suspend_enter)
	stp	x29, lr, [x0, #SLEEP_STACK_DATA_CALLEE_REGS]
	stp	x19, x20, [x0,#SLEEP_STACK_DATA_CALLEE_REGS+16]
	stp	x21, x22, [x0,#SLEEP_STACK_DATA_CALLEE_REGS+32]
	stp	x23, x24, [x0,#SLEEP_STACK_DATA_CALLEE_REGS+48]
	stp	x25, x26, [x0,#SLEEP_STACK_DATA_CALLEE_REGS+64]
	stp	x27, x28, [x0,#SLEEP_STACK_DATA_CALLEE_REGS+80]
```

```assembly
SYM_FUNC_START(_cpu_resume)
    // 생략
    ldp	x19, x20, [x29, #16]
	ldp	x21, x22, [x29, #32]
	ldp	x23, x24, [x29, #48]
	ldp	x25, x26, [x29, #64]
	ldp	x27, x28, [x29, #80]
	ldp	x29, lr, [x29]
	mov	x0, #0
	ret
SYM_FUNC_END(_cpu_resume)
```

참고로 kernel 에서 넣을 때는 x0 에 넣은 다음에 그걸 sleep_save_stash 에 넣는 것으로 보이는데, 꺼낼 때는 x29 에서 꺼내는 걸 볼 수 있다. cpu_resume 을 보면 x0 와 x29 를 sleep_save_stash 에서 꺼내온다.

```assembly
	/* x7 contains hash index, let's use it to grab context pointer */
	ldr_l	x0, sleep_save_stash
	ldr	x0, [x0, x7, lsl #3]
	add	x29, x0, #SLEEP_STACK_DATA_CALLEE_REGS // <---------- 여기!
	add	x0, x0, #SLEEP_STACK_DATA_SYSTEM_REGS
	/* load sp from context */
	ldr	x2, [x0, #CPU_CTX_SP]
	mov	sp, x2
	/*
	 * cpu_do_resume expects x0 to contain context address pointer
	 */
	bl	cpu_do_resume
```

내 코드의 잘못되었던 점은 x0 에 넣고, x29 에서 꺼내는 건 똑같이 가져왔는데, x29 를 저 주소로 맞춰주지 않았던 것이다. 그런데 x29 가 frame pointer 로 빌드되었을 때는 이전 lr 과 x29 가 그 주소에 들어있어서 비슷하게 돌아왔다. 수정하는 건 x29 신경 안 쓰고 x0 에 넣고 x0 에서 꺼내는 거로 바꿨다. (그게 더 save/restore 로직에 맞아보여서)



iamroot 스터디 자료

> cpu_suspend() 함수의 context 복원 문제
> \- suspend 후 resume할 때 caller-saved register가 온전히 보전되는가?
> \- __cpu_suspend_enter() 함수 내에서 caller-saved register 외의 모든 register context는 구조체 sleep_stack_data 내부에 보존되고 resume시에 복원되는 것이 확인됨
> \- 그러나 register를 저장하는 함수 __cpu_suspend_enter()와 실제 SMC를 통해 suspend를 수행하는 함수 fn()이 분리되어 있기 때문에 caller-saved register가 항상 보존되는 것인지 불분명함
> \- 현재는 두 함수 사이에 아무것도 없기 때문에 __cpu_suspend_enter() 함수에서 리턴하더라도 sp 아래쪽 값들(__cpu_suspend_enter() 함수에 진입했을 때 스택에 올린 caller-saved register들 정보)이 살아있고 보존되는 것으로 생각되며 그래서 문제 없이 동작하는 것으로 보임
> \- 그러나 엄밀히 따지면 하나의 함수에서 context를 저장하고 SMC를 호출하는 것이 정석으로 생각됨
> \- 실제로 이전 패치에서는 하나의 함수 __cpu_suspend_enter()에서 context 저장과 SMC 호출이 모두 이루어지므로 문제가 없음

http://www.iamroot.org/xe/index.php?mid=Note&document_srl=216792