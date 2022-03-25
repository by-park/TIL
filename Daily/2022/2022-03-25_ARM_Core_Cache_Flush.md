# 2022-03-25 (ARM Core Cache Flush)

### ARM Cortex-A core cache flush

ARM Cortex-A 코어들이 power down 될 때 Cache Flush가 되는지 알아보았다.

결론부터 보면, Cortex-A55 는 Core가 꺼질 때 하드웨어적으로 자동으로 Flush 되고, Cortex-A53 은 Flush 동작을 코드로 실행해줘야한다.

- ARM Cortex-A55

> The L1 and L2 caches are disabled, flushed and the core is removed from coherency automatically on transition to Off mode.

https://developer.arm.com/documentation/100442/0100/functional-description/power-management/power-modes/off?lang=en

- ARM Cortex-A53

dsb sy 등 필요한 작업들이 가이드되어있다.

> 5. Execute a `DSB SY` instruction to ensure that all cache, TLB and branch predictor maintenance operations issued by any core in the cluster device before the SMPEN bit was cleared have completed.

https://developer.arm.com/documentation/ddi0500/e/functional-description/power-management/power-modes



### Flush, Clean, Clear, Invalidate

ARM Cortex-A 가이드를 보면 flush 란 단어는 대개 clean 과 invalidate를 의미하는데,  ARM 에서는 clean 와 invalidate 용어만 사용한다고 되어있다.

- Invalidation: cache 의 valid bit 를 clear 시켜줌 (무효화)
- clean: dirty cache line 인 내용들을 메모리에 반영하고 dirty bit 를 clear 시켜줌

(ARMv7-A) https://developer.arm.com/documentation/den0013/d/Caches/Invalidating-and-cleaning-cache-memory

(ARMv8-A) https://developer.arm.com/documentation/den0024/a/Caches/Cache-maintenance



Cache Flush와 Invalidate

- Flush: Cache 의 내용을 저장 공간에 다시 써준다. Data 일치화
- Invalidate: Cache의 내용을 무효화한다. 지워버린다고 생각하면 된다. DMA Controller 를 사용하는 경우 CPU 가 원래 값이 바뀐 줄 모르고 Cache에서 읽어오지 않게 Cache 를 무효화한다.

https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=sadbiker&logNo=90016135740



Cache Flush 와 Clean

: Flush 0 으로 변경, 버림 / Clean 메모리에 Data 업데이트

http://blog.skby.net/cache-clean-flush/

Cache Flush 와 Clear

: Flush 그냥 지움 / Clear 주메모리에 옮기고 난 후 지움

http://www.iamroot.org/xe/index.php?mid=Kernel&document_srl=17266



### Barrier

DSB 설명

> DSB(Data Synchronization Barrier, called DWB)
>
> - 아래의 항목들(DMB 명령이 하는일 포함)이 완료될 때까지 추가 명령이 실행되는 것을 멈추고 기다리므로 DMB보다 훨씬 느리다. 인수의 사용은 DMB와 동일하다.
>   - Instruction 캐시 및 Data 캐시 조작
>   - Branch predictor 캐시 flush
>   - 지연된 load/store 명령의 처리 <- DMB 명령이 하는 일
>   - TLB 캐시 조작 완료

https://outoftheblackbox.site/Assembly-Barrier-instruction/



[참고]

ARM Cortex-A9 기준 L1 Cache size는 32 KB이다.

https://stackoverflow.com/questions/69676614/why-does-dsb-not-flush-the-cache



[참고]

ARM P-Channel 사용하면, ARM Cortex-A55 도 reset 시에 cache flush 를 막을 수 있는 것 같다.

https://developer.arm.com/documentation/100442/0200/functional-description/power-management/power-modes/debug-recovery?lang=en