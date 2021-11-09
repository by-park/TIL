# 2021-11-09 (Linux schedule command - AT)

Linux 에서 일정 시간 이후 실행되게 하는 방법

1. at command 이용

https://mentha2.tistory.com/85

```shell
$ at now + 1 hour -f my_script.sh
```

대량의 파일을 삭제하고자 my_script.sh 에는 rm -rf 명령어들을 적어두었다.

2. sleep 이용

그러나 서버에 at command 가 없어서 이런 명령어를 만들었다.

```shell
$ sleep 1h && sh my_script.sh
```



(참고)

삭제가 너무 오래 걸리고 리소스를 잡아먹는데, rsync 가 더 빠르다고 해서 해봤으나 똑같았다.

https://blog.asamaru.net/2016/04/26/efficiently-delete-large-directory-containing-thousands-of-files/

```shell
$ rsync -a --delete _empty/ target_directory/
```

