# 2021-04-05_Linux Library

lib{라이브러리 이름}.a 파일이 있는데도 불구하고 빌드시에 cannot find -l{라이브러리 이름} 이라고 뜨는 에러의 경우, 버전 등 해당 라이브러리 생성시에 설정이 잘못 되었을 수 있다.

에러 메세지 예시)

```
/usr/bin/ld -o {산출물}.elf -Bstatic -T {산출물}.lds -Ttext 0x00000000(주소) {컴파일할 c 파일들의 오브젝트 파일}.o -Llib -l{라이브러리 이름} --gc-sections -Map {산출물}.map
/usr/bin/ld: skipping incompatible lib/lib{라이브러리 이름}.a when searching for -l{라이브러리 이름}
/usr/bin/ld: cannot find -l{라이브러리 이름}
```

출처: https://aery.tistory.com/entry/usrbinld-skipping-incompatible-when-searching-for-l



커널의 버전 정보와 해당 라이브러리의 버전 정보를 비교하는 방법이 있다.

버전 설정 예시)

```
sudo ln -s /usr/lib/libz.so.1.2.8 /usr/lib/libzlib.so
```

출처: https://stackoverflow.com/questions/16710047/usr-bin-ld-cannot-find-lnameofthelibrary

