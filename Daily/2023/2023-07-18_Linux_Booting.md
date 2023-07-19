# 2023-07-18 (Linux Booting)

### From start_kernel() to PID 1

![Summary of early kernel boot process.](https://opensource.com/sites/default/files/u128651/linuxboot_3.png)

https://opensource.com/article/18/1/analyzing-linux-boot-process



### Initcall

- 부팅시 실행되어야 할 초기화 함수들을 목록으로 만들어 놓은 것
- init/main.c 에서 몇 단계를 거쳐 do_initcalls 를 호출
- Initcall 은 8단계의 level 이 존재함 (0~7)
  : early, core, postcore, arch, **subsys**, fs, **device**, late
- level 에 따라 do_one_initcall 이 initcall 함수들을 호출

![Initcall](https://t1.daumcdn.net/cfile/tistory/267BA43E545266AA01?original)

https://dreamlog.tistory.com/336



