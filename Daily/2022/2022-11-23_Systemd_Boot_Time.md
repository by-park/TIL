# 2022-11-23 (Systemd Boot Time)

systemd analyze 를 이용하면 부팅에 걸리는 시간을 확인할 수 있다.

ex) dev-mmcblk0p2.device 가 오래 걸리는 거에 대한 해결책은

(1) eMMC 나 빠른 SD Card 사용

(2) 커널 버전 업 혹은 -ti 대신 -bone 커널 사용

(3) 병목 현상을 일으키는 패키지 disable 이 있다.

아래 링크 말고도 optimization 을 문의한 글들이 많이 보인다. 해결책은 대개 비슷하다.

https://forum.beagleboard.org/t/boot-time-optimization/2540



분석 명령어

```shell
$ systemd-analyze time
Startup finished in 1.564s (kernel) + 1min 44.906s (userspace) = 1min 46.471s 
graphical.target reached after 1min 33.385s in userspace
```

https://www.lesstif.com/system-admin/systemd-analyze-booting-time-93127400.html

각 서비스별 걸리는 시간 보는 명령어

```shell
$ systemd-analyze blame
1min 34.446s dev-mmcblk0p2.device
1min 2.259s generic-board-startup.service
50.664s dev-loop8.device
49.671s dev-loop7.device
49.351s dev-loop6.device
48.843s dev-loop4.device
48.804s dev-loop5.device
48.462s dev-loop3.device
48.420s dev-loop1.device
48.392s dev-loop0.device
48.197s dev-loop2.device
24.250s snapd.service
```

그림으로 보는 명령어

```shell
$ systemd-analyze plot > boot.svg
```

