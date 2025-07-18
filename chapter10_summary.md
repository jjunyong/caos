# Chapter 10: 프로세스와 스레드 요약

## 10-1: 프로세스 개요

### 프로세스란?

- **프로그램**: 보조기억장치에 있는 데이터 덩어리
- **프로세스**: 메모리에 적재되어 실행되는 프로그램

**프로그램 vs 프로세스 (클래스 vs 객체 비유)**

```java
// 프로그램 = 클래스 (정적, 파일로 저장)
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}

// 프로세스 = 객체 (동적, 메모리에서 실행)
Calculator calc1 = new Calculator(); // 프로세스1
Calculator calc2 = new Calculator(); // 프로세스2 (같은 프로그램이지만 별도 프로세스)
```

### 프로세스 확인

- `ps` 명령어로 확인 가능 (주로 `ps -ef` 사용)
- **포그라운드 프로세스**: 사용자가 볼 수 있는 공간에서 실행
- **백그라운드 프로세스**: 사용자가 볼 수 없는 공간에서 실행
  - 사용자와 상호작용 가능한 프로세스
  - 데몬(Linux) / 서비스(Windows): 사용자와 상호작용하지 않음

### 프로세스 제어 블록(PCB)

- 프로세스가 정해진 시간만큼 CPU 사용 후 타이머 인터럽트로 차례를 넘김
- PCB는 프로세스의 태그 역할, 커널 영역에 생성
- 프로세스 생성 시 만들어지고 종료 시 폐기

**PCB에 담기는 정보:**

- **프로세스 ID (PID)**
- **레지스터 값**: 이전 작업 이어서 실행하기 위한 PC 및 레지스터 값
- **프로세스 상태**: CPU 대기, 입출력 대기, CPU 사용 중 등
- **CPU 스케줄링 정보**: CPU 할당 순서와 시점
- **메모리 관리 정보**: 베이스 레지스터, 한계 레지스터, 페이지 테이블 등
- **사용한 파일과 입출력장치 목록**

### 스레드 제어 블록(TCB)

스레드도 PCB와 유사하게 **TCB(Thread Control Block)**를 가짐

**TCB에 담기는 정보:**

- **스레드 ID (TID)**
- **레지스터 값**: PC, 스택 포인터 등
- **스케줄링 정보**: 스레드 상태, 우선순위

**PCB vs TCB 차이점:**

- PCB: 프로세스 전체의 메모리, 파일, 자원 정보 저장
- TCB: 스레드의 실행 흐름 정보만 저장 (스택 위치, 레지스터 등)

### 문맥 교환(Context Switching)

- **Context**: 프로세스 수행 재개를 위해 기억해야 할 정보 (PCB에 저장)
- **Context Switching**: 기존 프로세스 Context를 백업하고 새 프로세스 Context를 복구하여 실행
- 과도한 문맥 교환 시 오버헤드 발생

### 프로세스의 메모리 영역

**정적 할당 영역:**

- **코드 영역(텍스트 영역)**: 기계어 명령어 저장 (읽기 전용)
- **데이터 영역**: 전역 변수, static 변수 등 프로그램 실행 중 유지할 데이터

**동적 할당 영역:**

- **힙 영역**: 프로그래머가 직접 할당하는 공간 (낮은→높은 주소), 메모리 누수 주의
- **스택 영역**: 함수 매개변수, 지역변수 등 일시적 데이터 (높은→낮은 주소)

#### 자바에서의 메모리 영역 사용 예시

**데이터 영역:**

```java
public class AppConfig {
    public static final String DB_URL = "jdbc:mysql://localhost:3306/db";  // static 변수
    private static int connectionCount = 0;                                 // static 변수
}
```

**힙 영역:**

```java
// 모든 객체와 동적 데이터
User user = new User();           // 객체
String name = "홍길동";           // 문자열
List<User> users = new ArrayList<>();  // 컬렉션
int[] array = {1, 2, 3};         // 배열
```

**스택 영역:**

```java
public void processUser() {
    int age = 25;                 // 지역변수
    String name = "김철수";       // 실제 문자열 객체는 힙에 저장, 그에 대한 참조값은 스택에 저장
    calculateAge(age);            // 메서드 호출 스택
}  // 메서드 종료시 모든 지역변수 제거
```

## 10-2: 프로세스 상태와 계층 구조

### 프로세스 상태

운영체제는 PCB를 통해 프로세스 상태를 관리

- **생성 상태(new)**: 메모리 적재 후 PCB 할당받은 상태
- **준비 상태(ready)**: CPU 할당 대기 상태
- **실행 상태(running)**: CPU 할당받아 실행 중
- **대기 상태(blocked)**: 입출력 작업 완료 대기
- **종료 상태(terminated)**: 프로세스 종료, PCB와 메모리 정리

**상태 전환:**

- 준비 → 실행: **디스패치(dispatch)**
- 실행 → 대기: 입출력 요청
- 대기 → 준비: 입출력 완료
- 실행 → 준비: 타이머 인터럽트

#### 프로세스 상태 다이어그램

(아래 별도 아티팩트 참조)

### 프로세스 계층 구조

- 프로세스는 시스템콜을 통해 다른 프로세스 생성 가능
- **부모/자식 프로세스** 관계, PCB에 PID와 PPID 저장
- 최초 프로세스(Linux: systemd, PID=1)가 모든 프로세스의 조상
- `pstree` 명령으로 계층 구조 확인 가능

#### 프로세스 계층 구조 다이어그램

(아래 별도 아티팩트 참조)

### 프로세스 생성 기법

- **fork**: 부모 프로세스의 복사본을 자식 프로세스로 생성
- **exec**: 자식 프로세스가 자신의 메모리 공간을 다른 프로그램으로 교체

## 10-3: 스레드

### 스레드란?

- 프로세스를 구성하는 **실행의 흐름 단위**
- 하나의 프로세스는 여러 개의 스레드를 가질 수 있음

### 프로세스와 스레드

- **단일 스레드 프로세스**: 실행 흐름이 1개
- **멀티스레드 프로세스**: 실행 흐름이 여러 개

**스레드 구성 요소:**

- 스레드 ID
- PC값을 비롯한 레지스터 값
- 스택

스레드는 **프로세스 자원을 공유**하며 실행 (핵심 개념)

### 멀티프로세스 vs 멀티스레드

**가장 큰 차이점: 자원 공유**

- **멀티프로세스**: 자원을 공유하지 않음, 새로운 자원 필요
- **멀티스레드**: 자원을 공유하여 더 효율적

**스레드가 공유하는 자원:**

- 코드, 데이터, 힙, 파일

**스레드마다 개별적으로 가지는 자원:**

- 레지스터, 스택, PC

### 실제 예시: 웹서버의 요청 처리 방식

웹서버가 동시에 들어오는 여러 사용자 요청을 처리하는 방식을 통해 멀티프로세스와 멀티스레드의 차이를 이해할 수 있습니다.

#### Apache 웹서버의 처리 방식

**현재 Apache의 기본값 (2024년 기준):**

- Apache 2.4에서는 기본적으로 **Event MPM**을 사용합니다
- Prefork MPM은 Apache 2.4 이전 버전의 기본값이었습니다

**1. Event MPM (현재 기본값) - 하이브리드 멀티프로세스 + 멀티스레드**

- Worker MPM을 기반으로 한 개선된 버전
- 여러 프로세스, 각 프로세스 내에 여러 스레드
- **핵심 개선점**: Keep-alive 연결을 전용 리스너 스레드가 처리
- 일반 워커 스레드는 실제 요청 처리에만 집중
- 높은 동시 접속자 수를 적은 메모리로 처리 가능

**2. Worker MPM - 하이브리드 멀티프로세스 + 멀티스레드**

- 여러 프로세스, 각 프로세스 내에 여러 스레드
- 각 스레드가 하나의 연결을 담당
- Keep-alive 중에도 스레드가 계속 점유됨 (Event와의 차이점)

**3. Prefork MPM (구버전 기본값) - 멀티프로세스 (단일 스레드)**

- 사용자 A, B, C의 요청이 각각 **별도의 프로세스**로 처리
- 각 프로세스는 독립적인 메모리 공간 사용
- 하나의 프로세스가 크래시되어도 다른 요청에 영향 없음
- 메모리 사용량이 많음
- PHP가 thread-safe하지 않기 때문에 PHP 사용 시 주로 Prefork를 사용

**실제 상황 예시:**

```
사용자 A: "index.html 요청"
사용자 B: "login.php 요청"
사용자 C: "image.jpg 요청"

Prefork MPM:
프로세스1 → A의 요청 처리
프로세스2 → B의 요청 처리
프로세스3 → C의 요청 처리

Worker MPM:
Apache 프로세스
├── 스레드1 → A의 요청 처리
├── 스레드2 → B의 요청 처리
└── 스레드3 → C의 요청 처리
```

이처럼 멀티프로세스는 **안정성**을, 멀티스레드는 **효율성**을 중시하는 방식입니다.

#### 멀티프로세스 vs 멀티스레드 자원 공유 다이어그램

(아래 별도 아티팩트 참조)
