# 2021-07-14 (Little Kernel Developer)

Little Kernel의 community나 code 수정 히스토리를 파악하고 싶었는데, 따로 찾지 못했다. LK의 최초 commit을 확인해보았는데, 2008 년 아래 "initial commit of lk (little kernel) project" 가 최초 commit이었다. (commit 1d0df6996457273367e6d9d9d08bf6adb0fc9b65)

https://github.com/littlekernel/lk/commit/1d0df6996457273367e6d9d9d08bf6adb0fc9b65

https://github.com/littlekernel/lk/commits/master?after=77fa084cd05459a1f360a2b825e14ea6e60518e5+2204&branch=master

LK는 Travis Geiselbrecht 라는 분이 혼자서 만든 것이었고, 지금도 혼자서 유지보수 중인 것으로 보인다. (최신 commit이 혼자밖에 없어서...) 2015년 쯤에는 contributor가 많았다.



Fushia나 Zircon 이라는 이름은 치면 정보가 많은 것 같은데, 그것의 기반이 된 LK는 정보가 잘 없는 것 같다. 그래도 LK 제작자가 Travis 님이라는 건 몇 글에서 찾을 수 있었다.

> Fuchsia is based on a new microkernel called Zircon. Zircon is derived from Little Kernel, a small operating system intended for embedded systems. “Little Kernel” was developed by Travis Geiselbrecht, a creator of the NewOS kernel used by Haiku. [Forbes ](https://www.forbes.com/)describes the evolution of Zircon as ... (생략)

https://medium.com/@shahiandr/operating-system-is-coming-from-google-beautiful-and-clean-8e82acb4a815



LK는 android bootloader 로 많이 사용된다고 한다.

>  Bootloader는 다양한 종류가 있는데, Linux Embedded System에서는 uBoot(Universal Boot Loader)를, Android System에서는 LK Boot(Little Kernel based Android Boot Loader)를 주로 사용합니다. Bootloader는 다양한 종류가 있으며 그 용도가 각각 다릅니다.

https://m.blog.naver.com/jyoun/221691860304

![img](https://mblogthumb-phinf.pstatic.net/MjAxOTEwMjlfMTU3/MDAxNTcyMzE1ODk2MDE4.jWZw6WX37koi0izD6g2yTdmqvLcdassX-sPthKCfyFAg.-xtwp_nAcc4Uw5K8YyjmyV_5kMwVwmywbI0hnlIcJ6og.PNG.jyoun/image.png?type=w800)

Travis Geiselbrecht 의 개인 홈페이지

> ### Projects
>
> Hacked on a few projects in my day, one of which is pretty well known in certain circles.
>
> - **[NewOS](http://newos.org/)** - A OS project I started years ago and didn't know when to stop. It's reasonably well mature, and even scored a slahdotting and offer to write an article for Dr Dobbs magazine. It has been forked years ago and now forms the kernel for the [Haiku](http://haiku-os.org/) project.
> - **lk embedded kernel** - little embedded style kernel for some ARM devices. source at **git://github.com/travisg/lk.git**. Web browseable [here](https://github.com/travisg/lk).
> - **Arm Emulator** - Hacked a fun little arm emulator. Still needs some work, but it currently emulates a "generic" ARM based system. Good for prototyping a kernel. SVN depot at **git://github.com/travisg/armemu.git**. Web browseable [here](https://github.com/travisg/armemu).
> - **Apple ][ Emulator** - A cheezy little Apple ][ emulator I wrote a long time ago for fun. Source [here](http://git.tkgeisel.com/?p=apple.git). [Here](https://tkgeisel.com/pics/apple2-macosx.jpg) and [here](https://tkgeisel.com/pics/apple2-macosx-80.jpg) are a couple of screenshots.

https://tkgeisel.com/



과거에는 LK forum 이 있었던 것 같다. 아래 이솝 커뮤니티에서 LK 스터디 인원을 모집하면서 올린 참고자료에 forum 이 있었다.

http://www.aesop.or.kr/index.php?mid=Board_Community_Notice&document_srl=76116

Android for MSM git 목록에 보면 kernel/lk 라고 lk 코드가 있는 것 같은데, 이건 2016년 이후로 commit이 없다.

https://www.codeaurora.org/project-category/mobile-connectivity