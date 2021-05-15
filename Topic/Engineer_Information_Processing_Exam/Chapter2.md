---
typora-copy-images-to: images
---

# Engineer Information Processing Exam (정보처리기사 시험)

## 목차

[2. 소프트웨어 개발 (데이터 입출력 구현)](#2.-소프트웨어-개발-(데이터-입출력-구현))

[2. 소프트웨어 개발 (통합 구현)](#2.-소프트웨어-개발-(통합-구현))

[2. 소프트웨어 개발 (제품 소프트웨어 패키징)](#2.-소프트웨어-개발-(제품-소프트웨어-패키징))

[2. 소프트웨어 개발 (애플리케이션 테스트 관리)](#2.-소프트웨어-개발-(애플리케이션-테스트-관리))

[2. 소프트웨어 개발 (인터페이스 구현)](#2.-소프트웨어-개발-(인터페이스-구현))



## 2. 소프트웨어 개발 (데이터 입출력 구현)

### 자료구조 & 알고리즘

- 리스트, 스택, 큐, 트리, 그래프



> 35. 알고리즘 시간복잡도 O(1) 이 의미하는 것은? 답 (3)
>
>     (1) 컴퓨터 처리가 불가
>
>     (2) 알고리즘 입력 데이터 수가 한 개
>
>     (3) 알고리즘 수행시간이 입력 데이터 수와 관계없이 일정
>
>     (4) 알고리즘 길이가 입력 데이터보다 작음
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



> 21. 정렬된 N개의 데이터를 처리하는데 O(Nlog_2 N) 의 시간이 소요되는 정렬 알고리즘은? 답 (4)
>
>     (1) 선택정렬 (2) 삽입정렬 (3) 버블정렬 (4) 합병정렬
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



> 31. 다음 트리의 차수 (degree) 와 단말 노드 (terminal node) 의 수는? 답 (2)
>
>     ![img](http://www.comcbt.com/cbt/data/j4/j420160821/j420160821m4.gif)
>
>     (1) 차수: 4, 단말 노드: 4
>
>     (2) 차수: 2, 단말 노드: 4
>
>     (3) 차수: 4, 단말 노드: 8
>
>     (4) 차수: 2, 단말 노드: 8
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 트리의 순회 방법

- 중위 순회 : 왼쪽 서브트리 -> 중간 노드 -> 오른쪽 서브트리
- 전위 순회 : 중간 노드 -> 왼쪽 서브트리 -> 오른쪽 서브트리
- 후위 순회 : 왼쪽 서브트리 -> 오른쪽 서브트리 -> 중간 노드



> 26. 다음 트리를 전위 순회 (preorder traversal) 한 결과는? 답 (4)
>
>     ![img](images/preorder_200606.png)
>
>     (1) +\*AB/\*CBE 
>
>     (2) AB/C\*D\*E+ 
>
>     (3) A/B\*C\*D+E
>
>     (4) +\*\*/ABCDE
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제
>
> [그림 출처] https://q.fran.kr/%EB%AC%B8%EC%A0%9C/3664



### 그래프

- 무방향, 방향 그래프



> 26. 제어흐름 그래프가 다음과 같을 때 McCabe의 cyclomatic 수는 얼마인가? 답 ()
>
>     ![img](data:image/png;base64,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)
>
>     (1) 3     (2) 4     (3) 5     (4) 6
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제
>
> [그림 출처] https://q.fran.kr/%EB%AC%B8%EC%A0%9C/6802



### 데이터 베이스

- 데이터베이스 특징

- DBMS 장단점



### 스키마

- 정의
- 외부 스키마
- 개념 스키마
- 내부 스키마



### 데이터 베이스 설계

- 요구조건 분석
- 개념적 설계
- 논리적 설계
- 물리적 설계
- 데이터베이스 구현



### SQL

- DDL
- DML
- DCL

데이터 접속 (ORM), 트랜잭션

절차형 SQL 프로시저, 트리거, 사용자 정의함수



## 2. 소프트웨어 개발 (통합 구현)

> 22. 소프트웨어 공학의 기본 원칙이라고 볼 수 없는 것은? 답 (4)
>
>     (1) 품질 높은 소프트웨어 상품 개발
>
>     (2) 지속적인 검증 시행
>
>     (3) 결과에 대한 명확한 기록 유지
>
>     (4) 최대한 많은 인력 투입
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제



### 단위 모듈

- 디바이스 드라이버
- 네트워크
- 파일
- 메모리
- 프로세스



### 단위 모듈 테스트

- 화이트 박스, 블랙 박스 테스트
- 테스트 케이스



White Box Testing 종류: Condition Testing, Loop Testing, Data Flow Testing

Black Box Testing 종류: Equivalence Partitioning Testing, Boundary Value Testing, Cause-Effect Graphic Testing, Error Guessing, Comparison Testing



> 22. White Box Testing에 대한 설명으로 옳지 않은 것은? 답 (1)
>
>     (1) Base Path Testing, Boundary Value Analysis 가 대표적인 기법이다.
>
>     (2) Source Code의 모든 문장을 한번 이상 수행함으로서 진행된다.
>
>     (3) 모듈 안의 작동을 직접 관찰할 수 있다.
>
>     (4) 산출물의 각 기능별로 적절한 프로그램의 제어구조에 따라 선택, 반복 등의 부분들을 수행함으로써 논리적 경로를 점검한다.
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 개발 지원 도구

- IDE
- 빌드도구
- 협업 도구



## 2. 소프트웨어 개발 (제품 소프트웨어 패키징)

### 소프트웨어 패키징



> 39. SW 패키징 도구 활용시 고려사항과 거리가 먼 것은? 답 (3)
>
>     (1) 패키징 시 사용자에게 배포되는 SW이므로 보안을 고려한다.
>
>     (2) 사용자 편의성을 위한 복합성 및 비효율성 문제를 고려한다.
>
>     (3) 보안상 단일 기종에서만 사용할 수 있도록 해야 한다.
>
>     (4) 제품 SW 종류에 적합한 암호화 알고리즘을 적용한다.
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 릴리즈 노트



### 디지털 저작권 관리

- 클리어링 하우스, 콘텐츠 분배자, DRM 컨트롤러, 보안 컨테이너
- 디지털 저작권 관리의 기술 요소



> 32. 디지털 저작권 관리 (DRM) 의 기술 요소가 아닌 것은? 답 (4)
>
>     (1) 크랙 방지 기술 (2) 정책 관리 기술 (3) 암호화 기술 (4) 방화벽 기술
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 소프트웨어 설치 메뉴얼



### 소프트웨어 사용자 메뉴얼



### 국제 표준 제품 품질

- ISO/IEC 9126: 소프트웨어 품질 특성 및 거도에 대한 표준화
- ISO/IEC 14598: 소프트웨어 제품 평가
- ISO/IEC 12119: 패키지 소프트웨어 평가



> 23. 패키지 소프트웨어의 일반적인 제품 품질 요구사항 및 테스트를 위한 국제 표준은? 답 (3)
>
>     (1) ISO/IEC 2196 (2) IEEE 19554
>
>     (3) ISO/IEC 12119 (4) ISO/IEC 14959
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제



> 23. 소프트웨어 품질 측정을 위해 개발자 관점에서 고려해야할 항목으로 거리가 먼 것은? 답 (4)
>
>     (1) 정확성 (2) 무결성 (3) 사용성 (4) 간결성
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 소프트웨어 버전 등록

- 형상 관리

소프트웨어 버전관리 도구



> 34. 소프트웨어 형상 관리의 의미로 적절한 것은? 답 (2)
>
>     (1) 비용에 관한 사항을 효율적으로 관리하는 것
>
>     (2) 개발 과정의 변경 사항을 관리하는 것
>
>     (3) 테스트 과정에서 소프트웨어를 통합하는 것
>
>     (4) 개발 인력을 관리하는 것
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 빌드 자동화 도구

- 젠킨스
- 그래들



## 2. 소프트웨어 개발 (애플리케이션 테스트 관리)

### 애플리케이션 테스트

- 정적, 동적 테스트
- 명세, 구조, 경험 기반
- 검증, 확인 테스트
- 회복, 안전, 강도, 성능, 구조, 회귀 테스트



> 33. 소프트웨어 테스트에서 오류의 80%는 전체 모듈의 20% 내에서 발견된다는 법칙은?
>
>     (1) Brooks 의 법칙 (2) Boehm의 법칙 (3) Pareto 의 법칙 (4) Jackson의 법칙



### 테스트 기법에 따른 애플리케이션 테스트

- 화이트박스
  - 기초 경로 검사
  - 제어구조 검사
    - 조건 검사
    - 루프 검사
    - 데이터 흐름 검사
- 블랙박스 테스트
  - 동치 분할 검사
  - 경계값 분석
  - 원인-효과 그래프 검사
  - 오류 예측 검사
  - 비교 검사



> 25. 블랙박스 테스트의 유형으로 틀린 것은? 답 (4)
>
>     (1) 경계값 분석 (2) 오류 예측 (3) 동등 분할 기법 (4) 조건, 루프 검사
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제



> 28. 평가 점수에 따른 성적부여는 다음 표와 같다. 이를 구현한 소프트웨어를 경계값 분석 기법으로 테스트하고자 할 때 다음 중 테스트 케이스의 입력 값으로 옳지 않은 것은? 답 (3)
>
>     ![image-20210515022435666](images/image-20210515022435666.png)
>
>     (1) 59 (2) 80 (3) 90 (4) 101
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 개발 단계에 따른 애플리케이션 테스트

- 단위 테스트
- 통합 테스트
- 검증 테스트: 형상, 알파, 베타
- 시스템 테스트: 복구, 보안, 강도, 성능



> 37. 검증 검사 기법 중 개발자의 장소에서 사용자가 개발자 앞에서 행하는 기법이며, 일반적으로 통제된 환경에서 사용자와 개발자가 함께 확인하면서 수행되는 검사는? 답 (3)
>
>     (1) 동치 분할 검사 (2) 형상 검사 (3) 알파 검사 (4) 베타 검사
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



> 38. 하향식 통합에 있어서 모듈 간의 통합 시험을 위해 일시적으로 필요한 조건만을 가지고 임시로 제공되는 시험용 모듈을 무엇이라고 하는가?
>
>     (1) Stub (2) Driver (3) Procedure (4) Function
>
> [출처 ] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 애플리케이션 테스트 프로세스



### 테스트 케이스 / 테스트 시나리오 / 테스트 오라클



### 테스트 자동화 도구



> 24. 인터페이스 구현 검증도구 중 아래에서 설명하는 것은? 답 (2)
>
>     - 서비스 호출, 컴포넌트 재사용 등 다양한 환경을 지원하는 테스트 프레임워크
>     - 각 테스트 대상 분산 환경에 데몬을 사용하여 테스트 대상 프로그램을 통해 테스트를 수행하고, 통합하여 자동화하는 검증 도구
>
>     (1) xUnit (2) STAF (3) FitNesse (4) RubyNode
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 결함 관리



### 애플리케이션 성능 분석

- 처리량, 응답시간, 경과시간, 자원 사용률



### 애플리케이션 성능 개선

- 소스코드 최적화 (클린 코드, 나쁜 코드)
- 소스코드 품질 분석 도구: pmd, cppcheck, checkstyle, ccm, avalanche, valgrind



> 24. 다음 중 클린 코드 작성원칙으로 거리가 먼 것은? 답 (2)
>
>     (1) 누구든지 쉽게 이해하는 코드 작성
>
>     (2) 중복이 최대화된 코드 작성
>
>     (3) 다른 모듈에 미치는 영향 최소화
>
>     (4) 단순, 명료한 코드 작성
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제



> 36. 소스코드 품질분석 도구 중 정적분석 도구가 아닌 것은? 답 (3)
>
>     (1) pmd (2) cppcheck (3) valMeter (4) checkstyle
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



> 40. 외계인코드 (Alien Code) 에 대한 설명으로 옳은 것은? 답 (2)
>
>     (1) 프로그램의 로직이 복잡하여 이해하기 어려운 프로그램을 의미한다.
>
>     (2) 아주 오래되거나 참고문서 또는 개발자가 없어 유지보수 작업이 어려운 프로그램을 의미한다.
>
>     (3) 오류가 없어 디버깅 과정이 필요 없는 프로그램을 의미한다.
>
>     (4) 사용자가 직접 작성한 프로그램을 의미한다.
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



## 2. 소프트웨어 개발 (인터페이스 구현)

### EAI (Enterprise Application Integration)

- EAI는 기업 내 각종 애플리케이션 및 플랫폼 간의 정보 전달, 연계 통합 등 상호 연동이 가능하게 해주는 솔루션이다.

![img](https://hyeonukdev.github.io/images/%EC%A0%95%EB%B3%B4%EC%B2%98%EB%A6%AC%EA%B8%B0%EC%82%AC/0520_01.png)

> 25. EAI (Enterprise Application Integration) 의 구축 유형으로 옳지 않은 것은? 답 (4)
>
>     (1) Point-to-Point (2) Hub&Spoke (3) Message Bus (4) Tree
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



### 인터페이스 보안

- IPSec: 네트워크 계층에서 IP 패킷 단위의 데이터 변조 방지 및 은닉 기능 제공
- SSL: TCP / IP 계층과 애플리케이션 계층 사이에서 인증, 암호화, 무결성을 보장하는 프로토콜
- S-HTTP: 클라이언트와 서버 간 전송되는 모든 메시지를 암호화하는 프로토콜



> 27. 인터페이스 보안을 위해 네트워크 영역에 적용될 수 있는 솔루션과 거리가 먼 것은? 답 (2)
>
>     (1) IPSec (2) SMTP (3) SSL (4) S-HTTP
>
> [출처] 전자문제집 CBT 2020년 6월 6일 필기 기출문제



> 21. 인터페이스 보안을 위해 네트워크 영역에 적용될 수 있는 솔루션과 거리가 먼 것은? 답 (3)
>
>     (1) IPSec (2) SSL (3) SMTP (4) S-HTTP
>
> [출처] 전자문제집 CBT 2020년 8월 22일 필기 기출문제



## 참고 자료

- 정보처리기사 필기 요약정리: https://shlee1990.tistory.com/category/%EC%9E%90%EA%B8%B0%EA%B3%84%EB%B0%9C/%EC%9E%90%EA%B2%A9%EC%A6%9D
- 정보처리기사 필기 요약 더 자세한 정리: https://narup.tistory.com/72
- 정보처리기사 필기 전자문제집 CBT: https://www.comcbt.com/xe/iz

