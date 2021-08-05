# 2021-08-05 (initramfs - embedded file system)

파일시스템을 (욕토 빌드로) 만들 수 없을 때 커널에 기본적인 파일시스템이 내장되게 할 수 있다.



initramfs를 동작시키기 위한 kernel config

```shell
CONFIG_BLK_DEV_INITRD=y
CONFIG_INITRAMFS_SOURCE="./initramfs.cpio.gz"
```

https://chlrbgh0.tistory.com/entry/initramfs-in-kernel



ramdisk, ramfs, rootfs, initramfs, tmpfs 차이

https://blog.daum.net/tlos6733/156



initramfs와 initrd의 구분

http://egloos.zum.com/furmuwon/v/11145268



initramfs 원하는 디렉토리 만들 때 사용하는 명령어

```shell
find . | cpio -o -H newc | gzip > ../initramfs_data.cpio.gz
```

마운트하길 원하는 디렉토리에서 위와 같이 명령어를 실행하면
상위 디렉토리에 initramfs_data.cpio.gz  파일이 만들어집니다.

http://forum.falinux.com/zbxe/index.php?document_srl=791185&mid=lecture_tip