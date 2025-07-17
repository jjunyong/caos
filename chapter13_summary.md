# Chapter 13: 교착상태 정리본

## 13-1: 교착 상태란?

### 교착상태의 정의

- **교착상태(Deadlock)**: 일어나지 않을 사건을 기다리며 진행이 멈춰 버리는 현상

### 식사하는 철학자 문제

- **상황**: 모든 철학자가 동시에 포크를 집어 들면 아무도 식사를 할 수 없고 영원히 생각만 하는 상황 발생
- **비유 관계**:
  - 철학자 → 프로세스/스레드
  - 포크 → 자원
  - 생각하는 행위 → 자원을 기다리는 것
  - 포크 → 하나의 프로세스/스레드만 접근 가능한 임계구역

### 교착상태 발생 상황

- 매우 다양한 상황에서 발생 가능
- **뮤텍스 락에서의 교착상태 예시**:
  - 프로세스 A가 lock1을 잠그고, 프로세스 B가 lock2를 잠금
  - A는 lock2가 false가 되길 기다리고, B는 lock1이 false가 되길 기다림
  - 결과: 교착 상태 발생

### 교착상태 해결을 위한 접근법

1. 발생했을 때의 상황을 정확히 표현
2. 교착 상태가 일어나는 근본 이유를 파악

### 자원 할당 그래프

- **정의**: 어떤 프로세스가 어떤 자원을 사용하고 있고, 또 어떤 프로세스가 어떤 자원을 기다리고 있는지를 표현하는 간단한 그래프
- **특징**: 교착 상태가 발생한 그래프는 원의 형태를 띰

### 교착상태 발생 조건 (4가지 모두 만족해야 발생)

#### 1. 상호배제 (Mutual Exclusion)

- 자원을 한 번에 하나의 프로세스만 이용 가능
- 만약 하나의 포크를 여러 명이 동시에 사용할 수 있었다면 교착 상태는 발생하지 않음

#### 2. 점유와 대기 (Hold and Wait)

- 자원을 보유한 채 다른 자원을 기다렸기 때문에 문제 발생

#### 3. 비선점 (Non-Preemptive)

- 포크를 강제로 빼앗을 수 없음
- 철학자들은 모두 포크의 점유가 끝나기를 기다리기만 함

#### 4. 원형 대기 (Circular Wait)

- 요청 및 할당받은 자원이 원의 형태, 즉 순환 형태를 이룸
- 자원 할당 그래프가 원의 형태, 순환 형태로 그려지면 교착 상태 발생 가능

## 13-2: 교착상태 해결방법

### 해결 방법 분류

교착 상태는 **예방**, **회피**, **검출 후 회복**을 통해서 해결

### 1. 교착상태 예방

- **개념**: 애초에 일어나지 않도록 교착 상태 발생 조건에 부합하지 않게 자원을 분배
- **방법**: 교착 상태 필요 조건 네 가지 중 하나를 충족하지 못하게 함

#### 상호배제 없애기

- 모든 실행흐름이 자원을 공유 가능하게 함
- **단점**: 현실적으로 무리

#### 점유와 대기 없애기

- 특정 프로세스에 자원을 모두 할당하거나, 아예 할당하지 않는 방식으로 배분
- **단점**:
  - 자원의 활용률이 낮아짐
  - 많은 자원을 사용하는 프로세스가 불리해짐
  - 기아 현상 야기 가능

#### 비선점 조건 없애기

- 프로세스가 선점하여 사용할 수 있는 일부 자원들에 대해서는 효과적 (예: CPU)
- **단점**: 모든 자원이 선점 가능하지 않음 (예: 프린터), 범용성 떨어짐

#### 원형 대기 없애기

- 모든 자원에 번호를 붙이고, 오름차순으로 자원을 할당하면 원형 대기 발생하지 않음
- **예시**: 식사하는 철학자 문제에서 포크에 1~5번 번호를 붙이고, 번호가 낮은 포크에서 높은 포크 순으로 집어들게 함
- **단점**:
  - 모든 자원에 번호를 붙이는 일이 간단하지 않음
  - 번호에 따라 특정 자원의 활용률이 떨어질 수 있음

### 2. 교착상태 회피

- **개념**: 교착 상태가 발생하지 않을 정도로 조금씩 자원을 할당하다가 교착 상태의 위험이 있다면 자원을 할당하지 않는 방식
- **방법**: 프로세스들에 배분할 수 있는 자원의 양을 고려하여 교착 상태가 발생하지 않을 정도의 양만큼만 자원을 배분

#### 안전상태 (Safe State)

- 교착 상태가 발생하지 않고 모든 프로세스가 정상적으로 자원을 할당받고 종료될 수 있는 상태
- 안전 순서열을 찾을 수 있는 상황

#### 불안전 상태 (Unsafe State)

- 교착상태가 발생할 수도 있는 상황
- 안전 순서열이 없는 상황

#### 안전 순서열 (Safe Sequence)

- 교착 상태 없이 안전하게 프로세스들에 자원을 할당할 수 있는 순서
- **예시**: A, B, C 프로세스가 동시에 자원을 요청한 상황에서 B → A → C 순서대로 자원을 할당하면 교착 상태가 발생하지 않는다면, B → A → C가 안전 순서열

#### 회피 방법

- 시스템 상태가 안전 상태에서 안전 상태로 움직이는 경우에만 자원을 할당
- 항상 안전 상태를 유지하도록 자원을 할당하는 방식

#### 안전 순서열 찾기 실제 예시

```
시스템 상황:
- 자원 종류: A, B, C
- 전체 가용 자원: A=10, B=5, C=7
- 현재 가용 자원: A=3, B=3, C=2

프로세스 상태:
         할당량    최대요구량    추가필요량
P1:     A=0,B=1,C=0  A=7,B=5,C=3  A=7,B=4,C=3
P2:     A=2,B=0,C=0  A=3,B=2,C=2  A=1,B=2,C=2
P3:     A=3,B=0,C=2  A=9,B=0,C=2  A=6,B=0,C=0
P4:     A=2,B=1,C=1  A=2,B=2,C=2  A=0,B=1,C=1
P5:     A=0,B=0,C=2  A=4,B=3,C=3  A=4,B=3,C=1

안전 순서열 찾기 과정:
1. 현재 가용자원(3,3,2)으로 완료 가능한 프로세스 찾기
   - P2 완료 가능 (추가필요량 1,2,2 ≤ 가용자원 3,3,2)
   - P2 완료 후 가용자원: (3,3,2) + (2,0,0) = (5,3,2)

2. 업데이트된 가용자원(5,3,2)으로 다음 프로세스 찾기
   - P4 완료 가능 (추가필요량 0,1,1 ≤ 가용자원 5,3,2)
   - P4 완료 후 가용자원: (5,3,2) + (2,1,1) = (7,4,3)

3. 업데이트된 가용자원(7,4,3)으로 다음 프로세스 찾기
   - P3 완료 가능 (추가필요량 6,0,0 ≤ 가용자원 7,4,3)
   - P3 완료 후 가용자원: (7,4,3) + (3,0,2) = (10,4,5)

4. 업데이트된 가용자원(10,4,5)으로 다음 프로세스 찾기
   - P5 완료 가능 (추가필요량 4,3,1 ≤ 가용자원 10,4,5)
   - P5 완료 후 가용자원: (10,4,5) + (0,0,2) = (10,4,7)

5. 마지막으로 P1 완료 가능 (추가필요량 7,4,3 ≤ 가용자원 10,4,7)

결과: 안전 순서열 P2 → P4 → P3 → P5 → P1 존재
→ 현재 상태는 안전 상태
```

### 3. 교착상태 검출 후 회복

- **개념**: 자원을 제약 없이 할당하다가 교착 상태가 검출되면 교착 상태를 회복하는 방법
- **방법**: 프로세스들이 자원을 요구하면 모두 할당해주고, 대신 교착 상태 발생 여부를 주기적으로 검사

#### 선점을 통한 회복

- 교착 상태가 해결될 때까지 한 프로세스씩 자원을 몰아주는 방식
- 교착 상태가 해결될 때까지 다른 프로세스로부터 자원을 강제로 빼앗고 한 프로세스에 할당

#### 프로세스 강제 종료를 통한 회복

- 교착 상태에 놓인 프로세스를 모두 강제 종료
- 또는 교착 상태가 없어질 때까지 한 프로세스씩 강제 종료
- **단점**: 교착 상태가 없어졌는지 여부를 확인하는 과정에서 오버헤드 야기

### 4. 교착상태 무시

- 교착 상태를 아예 무시하는 방법도 존재

## 실무에서의 교착상태

### OS 레벨에서의 실제 교착상태 핸들링

실제 엔지니어가 OS 레벨에서 교착상태를 직접 핸들링하는 경우는 **매우 드물다**. 현대 운영체제는 이미 교착상태 방지 메커니즘을 내장하고 있기 때문이다.

#### OS 레벨에서 엔지니어가 마주하는 실제 상황들:

**1. 시스템 튜닝 상황**

- **문제**: 서버에서 특정 프로세스들이 응답 없음 상태로 멈춤
- **진단**: `ps aux`, `top`, `htop` 명령어로 좀비 프로세스나 D state 프로세스 확인
- **해결**:

  ```bash
  # 데드락 의심 프로세스 확인
  ps aux | grep " D "

  # 시스템 호출 추적
  strace -p <PID>

  # 강제 종료
  kill -9 <PID>
  ```

**2. 커널 모듈 개발 시**

- **상황**: 드라이버 개발 중 커널 패닉 발생
- **원인**: 스핀락이나 뮤텍스 사용 시 잘못된 락 순서
- **해결**: 락 순서를 일관되게 유지하는 코딩 컨벤션 적용

**3. 시스템 모니터링**

- **도구**: `sar`, `iostat`, `vmstat` 등으로 시스템 리소스 모니터링
- **패턴**: I/O 대기 시간이 비정상적으로 높으면 교착상태 의심
- **조치**: 시스템 재시작이나 특정 서비스 재시작

### 애플리케이션 레벨에서의 실제 교착상태 (훨씬 자주 발생)

엔지니어가 실제로 자주 마주하는 것은 애플리케이션 레벨에서의 교착상태다.

#### 1. 데이터베이스 교착상태 (가장 흔한 사례)

**실제 상황:**

```sql
-- 세션 1
BEGIN;
UPDATE users SET balance = balance - 100 WHERE id = 1;
UPDATE users SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- 세션 2 (동시 실행)
BEGIN;
UPDATE users SET balance = balance - 50 WHERE id = 2;
UPDATE users SET balance = balance + 50 WHERE id = 1;
COMMIT;
```

**에러 메시지:**

```
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
```

**실전 해결책:**

```python
# 파이썬 코드 예시
import time
import random

def transfer_money(from_id, to_id, amount):
    max_retries = 3
    for attempt in range(max_retries):
        try:
            with db.transaction():
                # 항상 작은 ID부터 락을 걸어 데드락 방지
                if from_id < to_id:
                    from_account = Account.objects.select_for_update().get(id=from_id)
                    to_account = Account.objects.select_for_update().get(id=to_id)
                else:
                    to_account = Account.objects.select_for_update().get(id=to_id)
                    from_account = Account.objects.select_for_update().get(id=from_id)

                from_account.balance -= amount
                to_account.balance += amount
                from_account.save()
                to_account.save()
                return True

        except DatabaseError as e:
            if 'deadlock' in str(e).lower():
                # 지수 백오프로 재시도
                time.sleep(random.uniform(0.1, 0.5) * (2 ** attempt))
                continue
            raise

    raise Exception("Transfer failed after max retries")
```

#### 2. 멀티스레드 프로그래밍에서의 교착상태

**실제 상황:**

```java
// 은행 계좌 이체 시스템
public class BankAccount {
    private final Object lock = new Object();
    private double balance;

    // 위험한 코드 - 데드락 발생 가능
    public void transfer(BankAccount target, double amount) {
        synchronized(this.lock) {
            synchronized(target.lock) {  // 데드락 위험!
                this.balance -= amount;
                target.balance += amount;
            }
        }
    }
}
```

**실전 해결책:**

```java
// 해결책: 락 순서 정하기
public class BankAccount {
    private static final Object tieLock = new Object();
    private final Object lock = new Object();
    private final int id;
    private double balance;

    public void transfer(BankAccount target, double amount) {
        if (this.id < target.id) {
            synchronized(this.lock) {
                synchronized(target.lock) {
                    this.balance -= amount;
                    target.balance += amount;
                }
            }
        } else if (this.id > target.id) {
            synchronized(target.lock) {
                synchronized(this.lock) {
                    this.balance -= amount;
                    target.balance += amount;
                }
            }
        } else {
            // 같은 ID인 경우 추가 락 사용
            synchronized(tieLock) {
                synchronized(this.lock) {
                    synchronized(target.lock) {
                        this.balance -= amount;
                        target.balance += amount;
                    }
                }
            }
        }
    }
}
```

#### 3. 마이크로서비스 아키텍처에서의 분산 교착상태

**실제 상황:**

```
주문 서비스 → 재고 서비스 → 결제 서비스 → 주문 서비스
(서비스 간 순환 호출로 인한 교착상태)
```

**실전 해결책:**

```python
# 서킷 브레이커 패턴과 타임아웃 적용
import asyncio
import aiohttp
from datetime import datetime, timedelta

class ServiceClient:
    def __init__(self, service_url, timeout=5.0):
        self.service_url = service_url
        self.timeout = timeout
        self.circuit_breaker = CircuitBreaker()

    async def call_service(self, endpoint, data):
        if self.circuit_breaker.is_open():
            raise ServiceUnavailableError("Circuit breaker is open")

        try:
            async with aiohttp.ClientSession() as session:
                async with session.post(
                    f"{self.service_url}/{endpoint}",
                    json=data,
                    timeout=aiohttp.ClientTimeout(total=self.timeout)
                ) as response:
                    if response.status == 200:
                        self.circuit_breaker.record_success()
                        return await response.json()
                    else:
                        self.circuit_breaker.record_failure()
                        raise ServiceError(f"Service returned {response.status}")

        except asyncio.TimeoutError:
            self.circuit_breaker.record_failure()
            raise ServiceTimeoutError("Service call timed out")
```

#### 4. 실무에서의 교착상태 디버깅 도구들

**데이터베이스:**

```sql
-- MySQL 데드락 정보 확인
SHOW ENGINE INNODB STATUS;

-- PostgreSQL 현재 락 상태 확인
SELECT * FROM pg_locks;
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

**Java 애플리케이션:**

```bash
# 스레드 덤프 생성
jstack <PID> > thread_dump.txt

# 데드락 모니터링 JVM 옵션
-XX:+PrintGCDetails -XX:+PrintConcurrentLocks
```

**Python 애플리케이션:**

```python
# 데드락 감지 데코레이터
import threading
import time
from functools import wraps

def deadlock_detector(timeout=10):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = [None]
            exception = [None]

            def target():
                try:
                    result[0] = func(*args, **kwargs)
                except Exception as e:
                    exception[0] = e

            thread = threading.Thread(target=target)
            thread.start()
            thread.join(timeout)

            if thread.is_alive():
                # 스레드가 여전히 살아있으면 데드락 의심
                raise TimeoutError(f"Possible deadlock detected in {func.__name__}")

            if exception[0]:
                raise exception[0]

            return result[0]
        return wrapper
    return decorator
```

### 실무 핵심 요약

1. **OS 레벨 교착상태**: 현대 시스템에서는 매우 드물며, 주로 시스템 모니터링과 문제 해결 차원에서 접근
2. **애플리케이션 레벨 교착상태**: 매우 자주 발생하며, 개발자가 직접 예방하고 해결해야 함
3. **핵심 해결책**: 락 순서 정하기, 타임아웃 설정, 재시도 로직, 서킷 브레이커 패턴
4. **디버깅**: 로그 분석, 스레드 덤프, 데이터베이스 상태 확인 등의 도구 활용

실제 개발 현장에서는 이론적 지식보다는 **실용적인 패턴과 도구**를 활용한 문제 해결이 더 중요하다.
