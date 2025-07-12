# Chapter 12: 프로세스 동기화

## 12-1: 동기화

### 동기화의 개념

프로세스들은 서로 협력하며 영향을 주고받는데, 이때 프로세스들은 실행 순서와 자원의 일관성을 보장해야 하기 때문에 반드시 동기화되어야 한다.

### 동기화의 의미

**프로세스들 사이의 수행 시기를 맞추는 것**

동기화는 두 가지 목적으로 나뉜다:

- **실행순서 제어**: 프로세스를 올바른 순서로 실행
- **상호배제(Mutual Exclusion)**: 동시에 접근해서는 안 되는 자원에 하나의 프로세스만 접근하게 하기

> 참고: 스레드도 동기화 대상이다.

### 1. 실행순서 제어를 위한 동기화

Reader-Writer 프로세스 예시:

- Reader는 파일에 값이 있을 때만 실행을 이어나갈 수 있음
- 따라서 Writer → Reader 순서가 중요

### 2. 상호배제를 위한 동기화

공유가 불가능한 자원의 동시 사용을 피하기 위함

**은행 계좌 예시:**

- 기존 계좌: 10만원
- 프로세스 A: 2만원 입금
- 프로세스 B: 5만원 입금
- 기대 결과: 17만원
- 실제 결과: 12만원 또는 15만원 (동기화 실패 시)

### 생산자-소비자 문제

**Mutual Exclusion의 유명한 문제**

- **생산자**: 버퍼에 물건 추가 + 총합 변수 +1
- **소비자**: 버퍼에서 물건 제거 + 총합 변수 -1
- **초기 상태**: 물건 10개
- **실행**: 생산자와 소비자를 동시에 10만번 실행
- **기대 결과**: 총합 = 10
- **실제 결과**: 예상과 다른 값 (동기화 실패)

### 공유 자원과 임계 구역

#### 공유 자원 (Shared Resource)

- 전역 변수
- 파일
- 입출력장치
- 보조기억장치 등

#### 임계 구역 (Critical Section)

동시에 실행 시 문제가 발생하는 자원에 접근하는 코드 영역

#### 경쟁 조건 (Race Condition)

여러 프로세스가 동시에 임계 구역 코드를 실행하여 문제가 발생하는 상황

### 상호배제 동기화의 3원칙

1. **Mutual Exclusion**: 한 프로세스가 임계 구역에 진입했다면 다른 프로세스는 들어올 수 없음
2. **Progress**: 임계 구역에 어떤 프로세스도 진입하지 않았다면 진입하고자 하는 프로세스는 들어갈 수 있어야 함
3. **Bounded Waiting**: 한 프로세스가 임계 구역에 진입하고 싶다면 그 프로세스는 언젠가는 임계 구역에 들어올 수 있어야 함 (무한정 대기 X)

---

## 12-2: 동기화 기법

### 뮤텍스 락 (Mutex Lock)

**탈의실 자물쇠와 같은 개념**

상호배제를 위한 동기화 도구로, 프로세스는 임계 구역에 있음을 알리기 위해 뮤텍스 락을 이용해 잠근다.

#### 뮤텍스 락의 구성 요소

1. **자물쇠**: 프로세스들이 공유하는 전역 변수 `lock`
2. **acquire 함수**: 임계 구역 잠그는 역할
   - 프로세스가 임계 구역 진입 전 호출
   - 이미 잠겨 있다면 계속 무한루프를 돌다가 lock이 풀리면 바로 잠금
   - 이렇게 쉴 새 없이 무한루프로 잠겨있는지 반복 확인하는 것을 **Busy Wait**라 함
3. **release 함수**: 임계 구역 잠금 해제
   - 임계 구역에서의 작업이 끝난 후 호출
   - lock을 false로 변경

#### 뮤텍스 락 기초 코드

```c
// 전역 변수
bool lock = false;

// acquire 함수 (임계 구역 잠금)
acquire() {
    while (lock == true) {
        // busy wait: 계속 대기
    }
    lock = true;
}

// release 함수 (임계 구역 잠금 해제)
release() {
    lock = false;
}
```

#### 뮤텍스 락 사용법

```c
acquire()     // 임계 구역 잠금
// 임계 구역 코드
release()     // 임계 구역 잠금 해제
```

### 세마포 (Counting Semaphore)

**좀 더 일반화된 방식의 동기화 도구**

뮤텍스 락은 하나의 공유 자원에 접근하는 것을 상정한 것이지만, 세마포는 여러 공유 자원이 있는 상황에서도 적용 가능하다.

#### 세마포의 구성 요소

1. **세마포 변수 S**: 공유 자원의 개수
2. **wait 함수**: 자원 요청 시 호출
   - S가 0보다 크면 S를 1 감소시키고 진입
   - S가 0이면 대기
3. **signal 함수**: 자원 반납 시 호출
   - S를 1 증가시킴

#### 세마포 기초 코드

```c
// 세마포 변수 (공유 자원의 개수)
int S = n;  // n은 사용 가능한 자원의 개수

// wait 함수 (자원 요청)
wait() {
    while (S <= 0) {
        // busy wait: 자원이 없으면 계속 대기
    }
    S--;
}

// signal 함수 (자원 반납)
signal() {
    S++;
}
```

**문제점: Busy Wait**

- 자원이 없을 때 프로세스가 무한루프를 돌며 CPU 낭비
- 대기 중인 프로세스가 계속 CPU를 점유하여 비효율적

#### 세마포 개선 코드 (Busy Wait 문제 해결)

```c
wait() {
    S--;
    if (S < 0) {
        add this process to Queue;  /* 대기 큐에 삽입 */
        sleep();                    /* 대기 상태로 전환 */
    }
}

signal() {
    S++;
    if (S <= 0) {
        remove a process p from Queue;  /* 대기 큐에서 프로세스 제거 */
        wakeup(p);                     /* 프로세스 p를 대기 상태에서 준비 상태로 전환 */
    }
}
```

**개선 이유:**

- 자원이 없을 때 프로세스를 대기 큐에 보내고 sleep 상태로 전환
- CPU를 다른 프로세스가 사용할 수 있어 효율성 향상
- 자원이 반납되면 대기 중인 프로세스만 선택적으로 깨움

#### 기존 vs 개선된 세마포의 S 값 해석

**기존 세마포 (Busy Wait):**

- **S**: 단순히 사용 가능한 자원의 개수
- **S > 0**: 자원 있음, 즉시 사용 가능
- **S = 0**: 자원 없음, busy wait으로 대기

**개선된 세마포:**

- **S > 0**: 사용 가능한 자원의 개수
- **S = 0**: 자원 없음, 대기 프로세스 없음
- **S < 0**: 자원 없음 + **|S| = 대기 중인 프로세스 개수**

**예시: 자원 2개인 세마포**

```
초기: S = 2
P1 wait(): S = 1 (자원 1개 사용)
P2 wait(): S = 0 (자원 2개 모두 사용)
P3 wait(): S = -1 (자원 없어서 대기, 대기 프로세스 1개)
P4 wait(): S = -2 (대기 프로세스 2개)
P1 signal(): S = -1 (P3 깨움, 아직 P4는 대기 중)
P2 signal(): S = 0 (P4 깨움, 모든 대기 해소)
```

**signal()에서 if (S <= 0)인 이유:**

- S를 증가시켰는데도 S ≤ 0이면 아직 대기 중인 프로세스가 있다는 의미
- 반납된 자원을 대기 중인 프로세스에게 즉시 할당해야 함

### Busy Wait 문제와 해결책

#### 문제점

- 뮤텍스 락과 기본 세마포에서 가용한 공유자원이 없는 경우
- 프로세스는 Busy Wait을 무작정 해야 함
- CPU 주기 낭비 발생

#### 해결책

세마포에서 더 나은 방법을 사용:

1. 자원이 없을 때 프로세스를 대기 큐에 삽입
2. 해당 프로세스를 대기 상태로 전환
3. 자원이 반납되면 대기 중인 프로세스를 깨움

### 세마포를 이용한 프로세스 실행 순서 제어

세마포를 이용하면 프로세스들이 특정 순서로 실행되도록 강제할 수 있다.

#### 기본 원리

세마포 변수 S를 0으로 설정하고, 순서를 제어하고 싶은 프로세스에 다음과 같이 적용:

- **먼저 실행할 프로세스** 뒤에 `signal()` 함수
- **나중에 실행할 프로세스** 앞에 `wait()` 함수

#### 동작 과정 분석 (P1 → P2 순서)

```c
// 세마포 변수
int S = 0;  // 초기값이 핵심!

// P1 프로세스 (먼저 실행되어야 할 프로세스)
P1의 코드 실행
signal(S);     // S를 0 → 1로 증가, "P2야 이제 실행해도 돼"

// P2 프로세스 (나중에 실행되어야 할 프로세스)
wait(S);       // S가 0이면 대기, 1이면 통과 후 S를 1 → 0으로 감소
P2의 코드 실행
```

**Case 1: P1이 먼저 실행되는 경우**

1. P1 실행 → signal(S) 호출 → S = 1
2. P2 실행 → wait(S) 호출 → S ≥ 1이므로 즉시 통과, S = 0
3. 결과: P1 → P2 순서

**Case 2: P2가 먼저 실행되는 경우**

1. P2 실행 → wait(S) 호출 → S = 0이므로 **대기 상태**로 전환
2. P1 실행 → signal(S) 호출 → 대기 중인 P2를 깨움
3. P2가 다시 실행
4. 결과: P1 → P2 순서 (P2가 먼저 시작했지만 P1을 기다림!)

**핵심**: 세마포 초기값 0과 wait/signal의 적절한 배치로 실행 순서를 강제할 수 있다.

---

## 모니터 (Monitor)

### 모니터의 개념

매번 임계 구역 앞뒤로 일일이 wait, signal 함수를 명시하는 것은 번거롭다. 모니터는 이를 해결하기 위한 고수준 동기화 기법이다.

### 모니터 vs 세마포 비교

#### 세마포 방식 (번거로운 방법)

```c
// 매번 임계구역 앞뒤로 수동 관리
wait(semaphore);     // 진입 전 필수
// 임계구역 코드
signal(semaphore);   // 나갈 때 필수
```

#### 모니터 방식 (자동화된 방법)

```c
monitor BankAccount {
    int balance = 1000;

    withdraw(int amount) {
        // 상호배제는 모니터가 자동 보장!
        // wait/signal은 불필요
        balance -= amount;
    }
}
```

### 모니터의 특징

- **공유 자원**과 **접근 인터페이스**를 묶어서 관리
- 프로세스는 **반드시 인터페이스를 통해서만** 공유 자원에 접근
- 모니터 안에 **항상 하나의 프로세스만** 들어올 수 있어 **상호배제 자동 보장**

### 모니터의 상호배제 메커니즘

모니터는 언어 차원에서 제공되는 기능으로, 내부적으로 뮤텍스를 사용하여 상호배제를 자동으로 보장한다.

```c
// 모니터가 내부적으로 수행하는 작업 (개발자에게는 숨겨짐)
monitor BankAccount {
    private Lock monitor_lock;        // 자동 생성
    private Queue entry_queue;        // 자동 생성

    withdraw(int amount) {
        monitor_lock.acquire();       // 자동 실행
        try {
            balance -= amount;        // 개발자가 작성한 코드
        } finally {
            monitor_lock.release();   // 자동 실행
        }
    }
}
```

**동작 과정:**

1. 프로세스 A가 withdraw 호출 → 모니터 진입 → 자동으로 락 획득
2. 프로세스 B가 동시에 deposit 호출 시도 → 모니터가 사용 중임을 감지 → 진입 큐에서 대기
3. 프로세스 A 완료 → 자동으로 락 해제 → 프로세스 B를 깨워서 진입 허용

### 조건 변수를 통한 실행 순서 제어

조건 변수는 **프로세스 실행 순서를 제어**하기 위한 특별한 변수다.

#### 은행 계좌 완전한 예시

```c
monitor BankAccount {
    int balance = 1000;
    condition sufficient_funds;  // 조건 변수: "잔액이 충분함"

    withdraw(int amount) {
        // 상호배제는 자동 보장됨
        while (balance < amount) {
            wait(sufficient_funds);  // "입금 후에 나를 실행해줘"
        }
        balance -= amount;
        console.log("출금 완료: " + amount);
    }

    deposit(int amount) {
        balance += amount;
        signal(sufficient_funds);   // "이제 출금 프로세스 실행해도 돼"
        console.log("입금 완료: " + amount);
    }
}
```

**실행 순서 제어 과정:**

1. 출금 프로세스가 withdraw(500) 호출, 잔액 1000원 → 충분하므로 즉시 실행
2. 또 다른 출금 프로세스가 withdraw(800) 호출, 잔액 500원 → 부족하므로 wait() 호출로 대기
3. 입금 프로세스가 deposit(400) 호출 → signal() 호출로 대기 중인 출금 프로세스 깨움
4. 대기 중이던 출금 프로세스가 다시 실행

#### 조건 변수의 연산

**wait 연산**

- "아직 내가 실행될 조건이 안 돼, 기다릴게"
- 프로세스를 조건 변수 대기 큐에 삽입
- 모니터가 비게 되어 다른 프로세스 진입 가능

**signal 연산**

- "이제 기다리던 프로세스 실행 조건이 만족됐어"
- 조건 변수 대기 큐의 프로세스를 깨움
- 깨어난 프로세스가 모니터로 다시 진입

### 모니터의 동기화 제공

1. **상호배제**: 모니터 자체가 자동으로 한 번에 하나의 프로세스만 허용
2. **실행 순서 제어**: 조건 변수를 통해 "누가 먼저, 누가 나중에 실행될지" 결정

---

## 실무에서의 동기화 (Spring Boot 백엔드 관점)

### 프레임워크가 자동으로 보장해주는 경우

#### 1. 데이터베이스 트랜잭션

```java
@Transactional  // Spring이 자동으로 동기화 보장!
public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
    Account from = accountRepository.findById(fromId);
    Account to = accountRepository.findById(toId);

    from.withdraw(amount);  // DB 레벨에서 동기화됨
    to.deposit(amount);     // Race Condition 걱정 없음
}
```

**DB가 자동으로 제공하는 것:**

- ACID 트랜잭션으로 원자성 보장
- 락(Lock) 자동 관리
- 트랜잭션 격리 수준 보장
- 데드락 감지 및 해결

#### 2. HTTP Request별 격리

```java
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        // 각 HTTP 요청마다 별도 스레드 + 별도 지역변수
        // 자동으로 격리되어 동기화 문제 없음
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
}
```

#### 3. Spring Bean의 Stateless 특성

```java
@Service  // 기본적으로 싱글톤이지만 상태가 없으면 안전
public class UserService {

    @Autowired
    private UserRepository userRepository;  // 스레드 안전

    public User findUser(Long id) {
        return userRepository.findById(id).orElse(null);  // 안전함
    }
}
```

### 개발자가 직접 동기화해야 하는 경우

#### ❌ 위험한 코드 (Race Condition 발생)

```java
@Service
public class CounterService {
    private int count = 0;  // 위험! 여러 요청이 공유하는 상태

    public int increment() {
        return ++count;  // Race Condition 발생 가능
    }
}
```

#### ✅ 안전한 코드 1: synchronized 사용

```java
@Service
public class CounterService {
    private int count = 0;

    public synchronized int increment() {  // 모니터 개념 적용
        return ++count;  // 상호배제 자동 보장
    }
}
```

#### ✅ 안전한 코드 2: AtomicInteger 사용

```java
@Service
public class CounterService {
    private final AtomicInteger count = new AtomicInteger(0);

    public int increment() {
        return count.incrementAndGet();  // 원자적 연산으로 안전
    }
}
```

#### ✅ 안전한 코드 3: Lock 명시적 사용

```java
@Service
public class CounterService {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public int increment() {
        lock.lock();    // 세마포/뮤텍스 개념 적용
        try {
            return ++count;
        } finally {
            lock.unlock();
        }
    }
}
```

### 실무 가이드라인

**✅ 자동으로 안전한 경우:**

- **@Transactional** 메서드 내 DB 작업
- **Request Scope** 변수들 (Controller의 지역변수 등)
- **Stateless 서비스** (상태를 갖지 않는 Service)
- **메시지 큐** 처리 (@RabbitListener, @KafkaListener 등)

**⚠️ 주의가 필요한 경우:**

- **Singleton Bean에서 인스턴스 변수 사용**
- **캐시 구현** (HashMap, List 등의 공유 자료구조)
- **카운터, 통계 수집** 등의 상태 관리
- **외부 API 호출 제한** (Rate Limiting)
