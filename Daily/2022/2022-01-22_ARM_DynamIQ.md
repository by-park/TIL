# 2022-01-22 (ARM DynamIQ)

### DSU (DynamIQ Shared Unit)

ARM v8 은 cluster 정보가 있었는데, ARM v8.2 부터는 cluster 가 통합되는 DSU 가 도입되었다.

그래서 mpidr 포맷이 다르다!!!

Aff1 자리에 ARMv8 은 cluster 번호가 있었지만, ARMv8.2 는 core 번호가 있다.

※ DSU 지원시 L3 cache 에 대해서 모든 CPU가 shared 가능하며, 물리적 cluster의 구분이 사라짐

| ARM 세대 | Core 명    | DSU 지원여부 | CPU  | mpidr    | Aff3[32:39] | Aff2[16:23] | Aff1[8:15] | Aff0[0:7]           |
| -------- | ---------- | ------------ | ---- | -------- | ----------- | ----------- | ---------- | ------------------- |
|          |            |              |      |          |             |             | cluster    | core                |
| ARMv8    | Cortex-A53 | x            | CPU0 | 81000000 | 0           | 0           | 0          | 0                   |
|          |            |              | CPU1 | 81000001 | 0           | 0           | 0          | 1                   |
|          |            |              | CPU2 | 81000002 | 0           | 0           | 0          | 2                   |
|          |            |              | CPU3 | 81000003 | 0           | 0           | 0          | 3                   |
|          |            |              | CPU4 | 81000100 | 0           | 0           | 1          | 0                   |
|          |            |              | CPU5 | 81000101 | 0           | 0           | 1          | 1                   |
|          |            |              | CPU6 | 81000102 | 0           | 0           | 1          | 2                   |
|          |            |              | CPU7 | 81000103 | 0           | 0           | 1          | 3                   |
|          |            |              |      |          |             | cluster     | core       | thread (single = 0) |
| ARMv8.2  | Cortex-A55 | o            | CPU0 | 81000000 | 0           | 0           | 0          | 0                   |
|          |            |              | CPU1 | 81000100 | 0           | 0           | 1          | 0                   |
|          |            |              | CPU2 | 81000200 | 0           | 0           | 2          | 0                   |
|          |            |              | CPU3 | 81000300 | 0           | 0           | 3          | 0                   |
|          |            |              | CPU4 | 81000400 | 0           | 0           | 4          | 0                   |
|          |            |              | CPU5 | 81000500 | 0           | 0           | 5          | 0                   |
|          |            |              | CPU6 | 81000600 | 0           | 0           | 6          | 0                   |
|          |            |              | CPU7 | 81000700 | 0           | 0           | 7          | 0                   |



ARM core 별 Revision 정보 (ex. ARM v8.2 등)

https://en.wikipedia.org/wiki/Comparison_of_ARMv8-A_cores



Cortex A75, Cortex A55 에서 DSU 도입 (한글 자료)

> DynamIQ에서는 ARM의 헤테로지니어스 멀티 코어 구성인 big.LITTLE을 보다 유연하고 효율적으로 실현할 수 있습니다. 최대 8코어 구성에, 빅코어 1개에 리틀코어 7개 같은 조합도 가능합니다. 또한 CPU 코어 이외에 다른 가속 장치와의 연결도 효율적이고 쉬워집니다. 또한 클러스터 아키텍처의 변화에 따라 캐시 계층이 효율적으로 바뀝니다.

https://gigglehd.com/gg/mobile/1202388



ARM DynamIQ 관련 자료

https://www.anandtech.com/show/11213/arm-launches-dynamiq-biglittle-to-eight-cores-per-cluster



DynamIQ 구체적 설명 (한글 자료)

> 기본적으로 HMP와 같은 동작 원리이지만, 기존의 big.LITTLE은 부하가 높은 작업을 리틀코어에서 빅코어로 옮기거나, 그 반대로 빅에서 리틀로 옮겨야 하는 상황이 번번히 발생하며, 이 경우 big.LITTLE 구성 요소 중 CCI(Cache Coherent Interconnect)를 이용하여 각각의 클러스터의 캐시를 단순히 동기화하여 big.LITTLE의 작업 전환을 구현한다. 하지만 각각의 클러스터는 서로 다른 계층의 캐시를 가지고 있고 클러스터의 범위를 넘어선 캐시의 공유는 불가하기 때문에 이렇게 작업을 넘겨주는 과정에서 효율이 떨어지게 된다. DynamIQ 방식은 서로 다른 종류의 코어를 하나의 클러스터로 묶고 거대한 3차 캐시를 능동적으로 공유함으로 이 작업 전환 과정의 리소스 소모를 최대한으로 줄일 수 있게 된다. 특히 설계면에서도 기존에는 클러스터로 묶여있던 코어의 구성을 매우 능동적으로 바꿀 수 있다. 1+3 구조의 쿼드코어 구성이나 4+4+4 구조의 12코어 구성의 big.LITTLE도 기존보다 쉽게 구현할 수 있게 되어서 하드웨어 벤더들이 좀 더 라인업의 구성을 용이하게 할 수 있다.

https://namu.wiki/w/ARM%20big.LITTLE%20%EC%86%94%EB%A3%A8%EC%85%98