# 2021-06-18 (Kernel Module)

kernel device driver 파일 아랫 부분에 보면 MODULE_AUTHOR() 같은 매크로들이 보인다.

### Linux Kernel Module Programming Guide 번역본

2.5 챕터 부분을 보면 2.4 버전 이후 커널을 사용하고 있으면 모듈을 로딩했을 때 커널 에러 메세지를 볼 수 있다고 보여주고 있다. 커널 2.4 버전 이후에 코드의 라이센스를 명시하는 메커니즘이 고안되었기 때문이다. 

 http://hisjournal.net/doc/The_Linux_Kernel_Module_Programming_Guide_v2.6_by_YoonMin_Nam.pdf



### Module 매크로 목록

```c
MODULE_AUTHOR(author);
MODULE_DESCRIPTION(description);
MODULE_VERSION(version_string);
MODULE_DEVICE_TABLE(table_info);
MODULE_ALIAS(alternate_name);
```

https://titanwolf.org/Network/Articles/Article?AID=2255f552-d747-4475-a9b2-9675310d8d43#gsc.tab=0



### Module 관련 에러 메시지

```shell
[    16.904365] <86> (2)[1:init]Disabling lock debugging due to kernel taint
[    16.904892] <a6> (2)[1:init][name:module&]module1: module license 'unspecified' - tainting kernel.
```

http://egloos.zum.com/rousalome/v/9966490