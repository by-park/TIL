# 2021-04-07 (Shell Script)

shell script 문법 중 일부 정리

### 1. 배열 순환

```shell
for NAME in "ME" "YOU" "THEM" "ALL"; do
    echo "Name is ${NAME}"
done

PLANETS=( "EARTH" "MARS" "VINUS" )
for PLANET in ${PLANETS[@]}; do
    echo "This is ${PLANET}"
done
for (( i=0; i<${#PLANETS[@]}; i++ )); do
    echo "Planet #$i is ${PLANETS[i]}"
done
```



[출처] https://blog.leocat.kr/notes/2018/02/17/shell-looping-list



### 2. 산술식

```
let res1=$num/2 [내장명령어]
echo $((res1++))
```

echo 안에 쓸 때는 (()) 괄호를 2번 써야한다.

[출처] https://rim0621.tistory.com/90