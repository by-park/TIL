# 2022-11-14 (spin lock)

### spin_lock, spin_lock_irq, spin_lock_irqsave란 무엇인가

http://egloos.zum.com/nimhaplz/v/5301468

- mutex lock 은 퍼포먼스 낭비가 있기 때문에 spin lock 이 등장하였다.
- spin lock 은 dead lock 이 발생할 수 있다 (오늘 경험하였다!! 동시에 2개 인터럽트가 발생했는데, 인터럽트 핸들러 안에 spin_lock, spin_unlock 을 잡았더니 첫번째 인터럽트가 다 처리되지 않았을 때 같은 코어에 두 번째 인터럽트가 발생하면 spin_lock 에 걸려버렸다. softirq 로 같은 코어에 그 인터럽트가 발생하도록 한 것이기 때문에 무조건 dead lock 이 생길 수 밖에 없었다.)
- spin_lock_irq 함수를 사용하면 인터럽트를 disable 시키기 때문에 dead lock 문제가 해결된다. 그러나 unlock 할 때 irq 를 enable 시켜야하는게 맞는지 알 수 없다. (disable 이 2번 들어왔을 수도 있다. 그러면 enable 을 2번 해줘야 진짜 enable 될 것)
- spin_lock_irqsave 함수를 사용하면 irq 가 현재 enable 상태인지 disable 상태인지를 저장한다. 따라서 위의 문제가 해결된다. 그러나 메모리에 저장하는 동작이 추가되어있으므로 퍼포먼스 오버헤드가 있다.

