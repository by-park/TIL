# 2021-10-13 (Linux Kernel Version History)

Linux Kernel version history 이미지

https://en.wikipedia.org/wiki/Linux_kernel_version_history

![linux_kernel](https://upload.wikimedia.org/wikipedia/en/timeline/sv0hk4cdjcvwtjrzmms5ch4rqck0lea.png)



Linux kernel release versions - Longterm

| Version | Maintainer                       | Released   | Projected EOL |
| ------- | -------------------------------- | ---------- | ------------- |
| 5.10    | Greg Kroah-Hartman & Sasha Levin | 2020-12-13 | Dec, 2026     |
| 5.4     | Greg Kroah-Hartman & Sasha Levin | 2019-11-24 | Dec, 2025     |
| 4.19    | Greg Kroah-Hartman & Sasha Levin | 2018-10-22 | Dec, 2024     |
| 4.14    | Greg Kroah-Hartman & Sasha Levin | 2017-11-12 | Jan, 2024     |
| 4.9     | Greg Kroah-Hartman & Sasha Levin | 2016-12-11 | Jan, 2023     |
| 4.4     | Greg Kroah-Hartman & Sasha Levin | 2016-01-10 | Feb, 2022     |

https://www.kernel.org/category/releases.html



Linux kernel version 별 release 날짜와 maintainer

https://handwiki.org/wiki/Software:Linux_kernel_version_history



### Linux version 별 상세한 변경 사항 ★

https://kernelnewbies.org/LinuxVersions



Linux version 별 대표 변경 사항

https://www.thomas-krenn.com/en/wiki/Linux_Kernel_Versions



### 특정 버전의 Linux kernel git 받는 법

(1) git clone 을 특정 branch로

https://unix.stackexchange.com/questions/46077/where-to-download-linux-kernel-source-code-of-a-specific-version

```shell
// 1 - 1 방법
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

cd linux-stable
git checkout v2.6.36.2

// update repository
git fetch

// 1 - 2 방법
git clone --depth 1 --single-branch --branch v4.5  git://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack
```



(2) kernel 에서 공식 제공하는 longterm 코드 받기 (git 정보 없음)

https://www.kernel.org/