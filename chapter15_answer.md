# Chapter 15: 파일 시스템 퀴즈 정답

## 단답형 정답

### 1. 파티셔닝(Partitioning)과 포맷팅(Formatting)

### 2. inode (index node)

### 3. 2번

### 4. 블록을 표현하는 비트 수

### 5. 직접 블록 (Direct Block)

---

## 서술형 정답

### 1. 파티셔닝과 포맷팅의 차이점

**답:**
**파티셔닝(Partitioning)**은 저장장치의 논리적인 영역을 구획하는 작업이다. 서랍에 칸막이를 설치하는 것과 같은 개념으로, 서랍 안에 물건을 그냥 마구잡이로 넣어버리면 나중에 필요한 물건을 찾기도 어렵고 정돈하기도 쉽지 않다. 하지만 칸막이를 설치하고 목적에 따라 물건들을 정돈해주면 나중에 필요한 물건들을 찾기도 수월해지고 정돈하기도 쉬워진다.

**포맷팅(Formatting)**은 단순히 저장장치를 삭제하는 것이 아니라, 파일 시스템을 설정해서 어떤 방식으로 파일을 저장하고 관리할 것인지를 결정하고 새로운 데이터를 쓸 준비를 하는 작업이다. 포맷 작업을 통해서 파일 시스템이 결정된다.

### 2. 유닉스 파일 시스템의 inode 구조

**답:**
유닉스 파일 시스템의 inode는 파일의 속성 정보와 더불어 15개의 블록 주소를 저장할 수 있다.

**15개 블록 주소의 구성:**

- **직접 블록 (1~12번)**: 파일의 데이터가 저장된 블록을 직접 가리키는 주소. 작은 파일의 경우 직접 블록 12개만으로도 충분하다.
- **1차 간접 블록 (13번)**: 데이터 블록의 주소들을 저장하는 블록을 가리킨다. 파일의 데이터를 저장한 블록들의 주소를 담고 있다.
- **2차 간접 블록 (14번)**: 1차 간접 블록들의 주소를 저장하는 블록을 가리킨다.
- **3차 간접 블록 (15번)**: 2차 간접 블록들의 주소를 저장하는 블록을 가리킨다.

이러한 계층적 구조를 통해 작은 파일은 직접 블록만으로, 큰 파일은 간접 블록을 활용하여 매우 효율적으로 관리할 수 있다. 예를 들어 블록 크기가 4KB이고 블록 주소가 4바이트라면, 1차 간접 블록은 1024개의 블록 주소를 저장할 수 있다.

### 3. FAT 파일 시스템의 문제점 해결과 사용 이유

**답:**
FAT 파일 시스템은 기존 연결 할당 방식의 단점을 보완하기 위해 만들어졌다. 연결 할당에서는 각 블록마다 다음 블록의 주소를 저장해야 했기 때문에 순차 접근만 가능했고, 중간 블록이 손상되면 그 이후 데이터에 접근할 수 없었다.

**FAT가 널리 사용되는 이유:**

- **테이블 기반 관리**: 모든 블록의 연결 정보를 별도의 FAT 테이블로 관리하여 임의 접근이 가능
- **성능 향상**: FAT 테이블을 메모리에 캐시하여 접근 속도 대폭 개선
- **높은 호환성**: 옛날 MS-DOS부터 현재의 USB 메모리, SD 카드까지 널리 사용되는 높은 호환성
- **안정성**: 블록 손상 시에도 FAT 테이블을 통해 복구 가능
- **다양한 버전**: FAT-12, FAT-16, FAT-32로 다양한 용량의 저장장치에 적합

### 4. 연속 할당과 불연속 할당의 차이점

**답:**
**연속 할당(Contiguous Allocation)**은 파일을 연속된 블록들에 순차적으로 저장하는 방식이다. 파일의 첫 번째 블록 주소와 블록 수만 알면 전체 파일에 접근할 수 있어 빠른 접근이 가능하지만, 외부 단편화 문제가 발생할 수 있다.

**불연속 할당(Non-contiguous Allocation)**은 하나의 파일이 여러 개의 블록에 걸쳐 저장되지만, 반드시 연속적일 필요는 없는 방식이다.

**불연속 할당의 세 가지 방법:**

1. **연결 할당(Linked Allocation)**: 각 블록의 일부에 다음 블록의 주소를 저장하여 다음 블록을 가리키는 형태로 할당하는 방법이다. 순차 접근만 가능하고, 중간 블록 손상 시 이후 데이터 접근 불가능하다는 단점이 있다.

2. **색인 할당(Indexed Allocation)**: 모든 블록의 주소를 하나의 색인 블록에 모아서 관리하는 방식이다. 임의 접근이 가능하고 외부 단편화 문제를 해결할 수 있다. 유닉스 파일 시스템의 inode가 이 방식을 사용한다.

3. **FAT 방식**: 각 블록이 포함된 다음 블록 주소를 한데 모아 표(FAT 테이블)로서 관리하는 방식이다. 연결 할당의 단점을 보완하여 임의 접근이 가능하고, FAT 테이블을 메모리에 캐시하여 접근 속도를 향상시킬 수 있다.

### 5. `/home/cs-study/a.shell` 파일 접근 과정

**답:**
유닉스 파일 시스템에서 `/home/cs-study/a.shell` 파일에 접근하는 과정:

1. **루트 디렉터리 접근**: 루트 디렉터리의 inode(보통 2번)에 접근하여 루트 디렉터리의 데이터 블록 위치를 파악한다.

2. **home 디렉터리 탐색**: 루트 디렉터리 데이터에서 'home' 엔트리를 찾아 해당 inode 번호(예: 121번)를 확인하고 접근한다.

3. **cs-study 디렉터리 탐색**: 121번 inode에 접근하여 home 디렉터리의 데이터 블록에서 'cs-study' 엔트리를 찾아 해당 inode 번호를 확인한다.

4. **파일 엔트리 찾기**: cs-study 디렉터리의 데이터에서 'a.shell' 파일 엔트리를 찾아 해당 inode 번호를 확인한다.

5. **실제 데이터 접근**: a.shell 파일의 inode에 접근하여 저장된 블록 주소들(직접 블록 또는 간접 블록)을 읽어와서 실제 파일 데이터가 저장된 블록들에 접근한다.

각 단계에서 **inode 번호 → inode 내용 → 데이터 블록**의 과정을 거쳐 최종 목표에 도달한다.
