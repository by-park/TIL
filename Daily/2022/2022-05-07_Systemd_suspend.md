# 2022-05-07 (Systemd suspend)

## systemd 란

> init 프로세스를 대체하여 도입된 PID 1번 프로세스
>
> 2010년도에 발표하여 2015년부터 본격적으로 도입됐다.
>
> 시스템 관리, 로그 관리, 서비스 관리, 초기화 스크립트 관리 등의 역할을 하고 있다.
>
> Systemd의 d는 Daemon 을 의미한다.
>
> init 프로세스: 유닉스 기반 컴퓨터 운영 체제에서 부팅 과정 중 최초로 실행되는 프로세스
>
> Daemon: 리눅스가 처음 가동될 때 실행되는 백그라운드 프로세스로 요청이 생기면 반응하는 리스너 같은 역할을 한다.
>
> PID: 프로세스를 구분하기 위한 식별자
>
> 유닛파일 종류: 서비스 유닛, 장치 유닛, 마운트 유닛, 자동마운트 유닛, 스왑 유닛, 타겟 유닛, 경로 유닛, 타이머 유닛, 스냅샷 유닛, 소켓 유닛, 범위 유닛, 슬라이스 유닛
>
> 유닛 파일 구성
>
> 유닛 제어 명령

https://www.slideshare.net/DataUs/systemd-241729316

> 근 몇 년 안에 리눅스를 조금이라도 깊게 사용해봤다면 systemd라는 미지의 프로세스가 있다는 것을 눈치채셨을 겁니다. 오래전에 사용하셨던 분이라면 init이 사라져버리고 systemd가 떡하니 PID 1를 차지하고 있는 것을 보고 의아해 하였을 분도 계실겁니다.
>
> systemd가 PID 1을 차지하는 것을 보면 init이 퇴출당하고 init이 하던 작업을 물려받은 더 대단한 systemd가 나타났다는 것을 쉽게 유추할 수 있습니다. 도대체 뭐가 그리 좋길래 상당수의 배포판들이 오래전부터 사용해오던 init을 버리고 systemd로 갈아탔는지 알아봅시다.
>
> 예전의 PID 1이었던 init은 현재로부터 수 십 년 전에 처음 소개된 프로그램인데 그 때의 구조를 거의 바꾸지 않고 계속 기능이 추가되며 날이 갈수록 복잡해지는 프로그램들로 인해 효율이 떨어졌습니다. 그리고 그 구조라는 것이 시작할 프로그램을 구동하는 쉘 스크립트를 특정 run-level의 rc 디렉토리에 추가하는 것이고, init은 부팅 과정에서 단계적으로 run-level을 올려가며 해당 run-level에 포함된 스크립트들을 순차적으로 실행시키니 설정의 난잡함 뿐만 아니라 속도마저 느렸습니다.

https://stackframe.tistory.com/12

### service 란? service 와 target의 차이?

> Service: 흔히 생각하는 시스템 서비스는 모두 Service 유닛에 해당한다.
>
> Target: Target 은 Service 유닛의 집합이다.

https://greencloud33.tistory.com/26

### Daemon 이란?

> 유닉스에서 데몬이란 사용자가 직접적으로 제어하지 않고, 백그라운드에서 돌면서 여러 작업을 하는 프로그램을 말한다. 일반적으로 프로세스로 실행된다.

https://blog.daum.net/servant2342/2076534

> 데몬은 시스템의 첫 프로세스인 (pid가 1인) systemd 의 바로 하위 프로세스가 된다.
>
> 원래 Init이 PID 1인 루트 프로세스였는데 이를 개선한 것이 systemd 이다.
>
> 데몬은 부모 프로세스가 없어서 PPID (부모 프로세스 ID) 는 1이다.
>
> systemd 는 프로세스 트리에서 가장 상위의 프로세스이고 모든 프로세스의 직간접 부모인 데몬이다.
>
> 리눅스는 PID 1번 프로세스가 가장 먼저 실행되어 OS 에 필요한 각종 데몬들을 init 한다.

https://slow-and-steady-wins-the-race.tistory.com/147

백그라운드 프로세스와 데몬이 같은 의미는 아니다. 백그라운드로 돌다가 끝나면 사라지는 건 데몬이라고 할 수 없다. 데몬은 부팅 때 자동으로 켜져 백그라운드에서 계속 실행된다는 뜻이라 꺼지지 않고 실시간으로 클라이언트와 통신을 계속 해야하는 프로세스들이 주로 데몬이라고 불린다.

https://mamu2830.blogspot.com/2020/04/blog-post_18.html

### init process 와 systemd 프로세스

init process

init 을 복사해서 자식 프로세스를 만든다.

![img](https://lh3.googleusercontent.com/-bwA-u76f5js/X_LFvu9imtI/AAAAAAAAF64/Mt8r9cK-uFIAAzuTguPOK5q7XHPCSvctwCLcBGAsYHQ/w640-h408/20210104_163506.png)

systemd

![img](https://lh3.googleusercontent.com/-cjkbMX7vTOk/X_LGUUyL6HI/AAAAAAAAF7A/g6vfpUevwo8M5xxDqdj0giByZBqIHN_XACLcBGAsYHQ/w640-h392/20210104_163525.png)

init 프로세스는 부팅 때 모든 서비스 (데몬) 을 순서대로 차례차례 킨다.

단점은 시스템에 필수적인 특정 프로세스가 켜지는게 오래 걸리면, 나머지 프로세스들은 대기해야하기 때문에 부팅이 오래 걸릴 수 있다.

systemd 는 부팅에 필요한 프로세스들만 먼저 병렬적으로 시작한다. 부팅 시간이 init 에 비해 훨씬 빠르다.

init 이 사용하던 `/etc/rc.d/rc3.d/` 디렉토리 내부를 보면 텅 비어있다. 원래는 init 에서 실행시킬 파일들이 순서대로 들어있었다.

기존 init 은 런 레벨에 맞는 특정 디렉토리에 실행한 데몬들을 집어넣어 놓고 init 이 특정 디렉토리를 선택 후 내부의 프로그램들을 순서대로 실행하는 방식이었다면,

systemd 는 어떤 데몬을 실행할지, 그 데몬은 어떤 다른 프로그램과 의존성을 갖는지 등등을 적은 `.target` 이란 확장자가 붙은 설정파일을 이용하여 런레벨과 데몬들을 실행한다.

>  systemd 시스템부터는 기존 init 기능 이외에도 시스템 전반적인 것들을 관리하는 기능이 더 있기에, 그만큼 사용해야 할 파일들도 많아졌고 그래서 효율적으로 사용하기 위해, systemd 가 사용하는 파일들을 그 용도에 따라 명확히 구분(세분화) 했습니다. 세분화된 systemd 가 사용하는 파일들을 단위(unit) 이라고 부르기로 했죠. 세분화했다는 것은 용도에 따라 파일 이름 끝에 붙이는 확장자가 더욱 다양해졌다는 겁니다.
>
> 예를 들어, 우리가 알고 있는 데몬(서비스) 은 파일 이름 끝에 확장자 `.service` 를 붙여 구분하고, 데몬 프로그램에서 사용하는 소켓 파일은 확장자 `.socket` 을 붙이듯이 말이죠

https://mamu2830.blogspot.com/2021/07/init-systemd-difference.html



## systemd 의 suspend service 설명

> Note that scripts or binaries dropped in /usr/lib/systemd/system-sleep/ are intended for local use only and should be considered hacks. If applications want to react to system suspend/hibernation and resume, they should rather use the **Inhibitor interface**[1].

Inhibitor interface로 Lock을 잡을 수 있다고 한다. (https://www.freedesktop.org/wiki/Software/systemd/inhibit/)

```c
fd = Inhibit("shutdown:idle", "Package Manager", "Upgrade in progress...", "block");
/* ...
      do your work
                 ... */
close(fd);
```

> Internally, this service will echo a string like "mem" into /sys/power/state, to trigger the actual system suspend. What exactly is written where can be configured in the [Sleep] section of /etc/systemd/sleep.conf or a sleep.conf.d file. See [systemd-sleep.conf(5)](https://man.archlinux.org/man/systemd-sleep.conf.5.en).

sleep 안에서 `echo mem > /sys/power/state` 가 실행된다고 한다.

https://man.archlinux.org/man/systemd-suspend.service.8.en



## suspend 시 실행되는 스크립트 작성법

system suspend 전과 resume 때 스크립트를 실행하고자 하면, system-sleep folder 안에 넣으면 된다고 한다. (folder 경로: `/lib/systemd/system-sleep/`)

pre, post 로 인자 넘어오는 걸 처리하게 해주면 된다.

```c
#!/bin/sh
if [ "${1}" == "pre" ]; then
  # Do the thing you want before suspend here, e.g.:
  echo "we are suspending at $(date)..." > /tmp/systemd_suspend_test
elif [ "${1}" == "post" ]; then
  # Do the thing you want after resume here, e.g.:
  echo "...and we are back from $(date)" >> /tmp/systemd_suspend_test
fi
```

https://blog.christophersmart.com/2016/05/11/running-scripts-before-and-after-suspend-with-systemd/ (Running scripts before and after suspend with systemd)

https://forum.manjaro.org/t/systemd-not-running-a-script-in-system-sleep-folder/69603 (Systemd not running a script in system-sleep folder)

https://unix.stackexchange.com/questions/337853/how-can-i-trigger-a-systemd-unit-on-suspend-before-networking-is-shut-down (How can I trigger a systemd unit on suspend before networking is shut down?)

https://askubuntu.com/questions/1313479/correct-way-to-execute-a-script-on-resume-from-suspend (Correct way to execute a script on resume from suspend)

https://bbs.archlinux.org/viewtopic.php?id=152942 (my script in /usr/lib/systemd/system-sleep/ was erroneous)

https://askubuntu.com/questions/1054273/script-in-lib-systemd-system-sleep-deleted-but-still-runs (script in /lib/systemd/system-sleep/ deleted but still runs)



## Systemd 의 suspend 소스코드

### service

https://github.com/systemd/systemd/blob/main/units/systemd-suspend.service.in

```c
[Unit]
Description=System Suspend
Documentation=man:systemd-suspend.service(8)
DefaultDependencies=no
Requires=sleep.target
After=sleep.target

[Service]
Type=oneshot
ExecStart={{ROOTLIBEXECDIR}}/systemd-sleep suspend
```

### target

https://github.com/systemd/systemd/blob/main/units/sleep.target

```c
[Unit]
Description=Sleep
Documentation=man:systemd.special(7)
DefaultDependencies=no
RefuseManualStart=yes
StopWhenUnneeded=yes
```

sleep 내부 동작 (`echo mem > /sys/power/state` 호출하는 부분. ExecStart 부분 호출되면 불리는 것으로 추정됨)

https://github.com/systemd/systemd/blob/main/src/sleep/sleep.c



### 소스코드 관계

```c
suspend.target
|-systemd-suspend.service
  |-sleep.target
```

https://unix.stackexchange.com/questions/337853/how-can-i-trigger-a-systemd-unit-on-suspend-before-networking-is-shut-down