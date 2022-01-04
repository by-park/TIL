# 2021-05-01 (Codejam Round 1C)

### Codejam Round 1C 참여

3 문제 중에 2 번째 문제가 풀 수 있을 것 같아서, 그걸 위주로 풀었는데

접근 방법이 잘못된 것 같은데, Sample 만 맞고 반례는 찾지 못했다.

해설을 보니 이분 탐색을 이용하라고 되어있었다.

이분 탐색을 이용한 다른 분의 답안을 찾았다.



### 33등 firefly 님 2번 Roaring Years 답안 코드

```python
def cons(x, y):
    ret = ""
    for i in range(x, x+y):
        ret = ret + str(i)
    return int(ret)

t = int(input())
for tc in range(1, t+1):
    n = int(input())
    ans = 10000000000000000000000
    strlen = len(str(n))
    
    for i in range(1, 11):
        number = (strlen + i - 1) // i
        if number==1:
            continue
        MIN = 10**(i-1)
        MAX = 10**i - 1
        ANS = -1
        #print(MIN, MAX, number, i, n)
        
        while MIN <= MAX:
            MID = (MIN + MAX) // 2
            #print(MIN, MAX)
            if cons(MID, number) > n:
                ANS = MID
                MAX = MID-1
            else:
                MIN = MID+1
        #print(i, ANS)
        if ANS != -1:
            ans = min(ans, cons(ANS, number))
            #print(ans)
    
    for i in range(1, 11):
        for st in range(max(1, 10**i-20), 10**i):
            string = str(st)
            for j in range(st+1, st+100):
                string += str(j)
                if int(string) > n:
                    break
            ans = min(ans, int(string))
    
    print(f"Case #{tc}: {ans}")

```

