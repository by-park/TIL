# 2024-11-08 (Cortex-M Secure)

Cortex M 코어에서 Secure 모드로 전환하는 방법



A 코어처럼 익셉션을 발생시켜서 벡터로 뛰는 개념이 아니다.

non secure 상태에서 특정 함수로 분기하면 해당 함수에서 SG 를 호출해서 secure 상태로 전환한다.

그리고 secure 함수를 호출한다.

secure 에서 돌아올 때는 `BXNS` 명령어를 이용해서 돌아오면 non secure 로 복귀할 수 있다.



### SG

Secure Gate 이용 코드 예시

https://developer.arm.com/documentation/101928/0101/The-Cortex-M85-Instruction-Set--Reference-Material/Miscellaneous-instructions/SG

```assembly
SEC_ENTRY:  
PUSH    {R11, LR}  
NOP
POP     {R11, LR}  
BXNS    LR          // RETURN TO NON-SECURE

NS_ENTRY:           // VENEER SECURE GATE
SG  
BL   SEC_ENTRY  
```



가이드라인

실행 흐름에 대한 도식도 참고할 수 있다.

https://developer.arm.com/documentation/100720/0200/Switching-between-Secure-and-Non-secure-states

