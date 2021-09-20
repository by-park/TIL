# 2021-09-20 (Python Print)

python 으로 list 로 관리하다가 print할 때는 [] 이런 괄호가 보이지 않고 숫자만 출력하고 싶을 때,

```python
" ".join(list)
```

와 같이 쓰려면 안에 list가 string 배열이어야한다.

```python
print(*list, sep=" ")
```

와 같이 * 을 붙이면 여러 인자를 넘겨준 효과가 난다.

https://www.daleseo.com/python-lists-print/

백준 2845를 풀면서 연습했다.