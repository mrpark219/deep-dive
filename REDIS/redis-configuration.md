# Redis 환경 설정

## 1. 캐싱 전략 (Cache Aside, Write Through, Write Behind, Read Through)

- 캐싱을 도입할 때는 **데이터 저장소(DB)와 캐시 간의 일관성(Consistency)** 을 반드시 고려해야 한다.
- 캐시 전략을 잘못 수립하면 DB에는 최신 값이 반영되었으나 캐시에는 오래된 값이 남아있는 **데이터 불일치(Stale Data)** 문제가 장기화될 수 있다.

### 1.1. 캐싱이란

- **캐싱(Caching)**: 자주 사용하는 데이터를 메모리(RAM)와 같은 빠른 저장소에 보관하여 응답 성능을 극대화하는 기법이다.
- **주요 목적**:
  - **데이터베이스 부하 감소**: 빈번한 디스크 I/O를 방지하고 DB 단일 장애점(SPOF) 위험을 완화한다.
  - **응답 속도 개선**: 마이크로초($\mu s$) 단위의 빠른 데이터 조회를 제공한다.
- **성능과 일관성의 트레이드오프(Trade-off)**:
  - **강한 일관성 추구**: 데이터 변경 시마다 캐시를 반복 갱신·삭제하므로 시스템 전반의 성능 및 지연 시간이 저하된다.
  - **높은 성능 추구**: 캐시 TTL을 길게 유지하여 Cache Hit을 극대화하는 대신, 원본 데이터와의 일시적인 불일치를 감수해야 한다.

| 패턴              | 특징                                                                                                     |
| :---------------- | :------------------------------------------------------------------------------------------------------- |
| **Cache Aside**   | 가장 보편적인 방식으로, 애플리케이션이 캐시를 직접 제어하며 필요할 때만 데이터를 적재(Lazy Loading)한다. |
| **Write Through** | 쓰기 발생 시 Cache와 DB를 동기식으로 동시에 갱신하여 데이터 일관성을 상시 유지한다.                      |
| **Write Behind**  | 캐시에 우선 기록 후 백그라운드 큐를 통해 DB에 비동기 배치(Batch)로 반영한다.                             |
| **Read Through**  | 캐시 계층이 DB 조회를 전담하여 자동 로드하므로 애플리케이션 코드가 간결해진다.                           |

### 1.2. Cache Aside (Lazy Loading)

- 가장 단순하고 널리 쓰이는 방식으로, **조회 요청이 발생할 때만 캐시에 데이터를 적재**한다.

#### 읽기 및 쓰기 흐름

```mermaid
sequenceDiagram
    participant App as 애플리케이션
    participant Redis
    participant DB as 데이터베이스

    App->>Redis: 1. GET key
    Redis-->>App: Cache Miss
    App->>DB: 2. SELECT *
    DB-->>App: 데이터 반환
    App->>Redis: 3. SET key value
    App->>App: 4. 데이터 반환
    Note over App,DB: 다음 요청은 캐시에서 즉시 반환
```

- **읽기 흐름**: 캐시 조회 $\rightarrow$ Hit 시 즉시 반환 / Miss 시 DB 조회 후 캐시 적재(`SET`) 및 결과 반환
- **쓰기 흐름**: 데이터베이스에 최신 값을 갱신한 뒤 **기존 캐시 키를 삭제(Evict)** 한다.

#### 예시 구현

```typescript
async function getUser(userId: number): Promise<User> {
  // 1. 캐시 확인
  const cachedUser = await cache.get(userId);
  if (cachedUser) {
    return JSON.parse(cachedUser);
  }

  // 2. 캐시 미스 시 DB 조회
  const user = await db.findUserById(userId);

  // 3. DB 조회 결과를 캐시에 저장 (다음 조회를 위한 최적화)
  await cache.set(userId, JSON.stringify(user));

  return user;
}

async function updateUser(user: User): Promise<void> {
  // 1. DB 갱신
  await db.updateUser(user);

  // 2. 캐시 무효화 (삭제)
  await cache.delete(user.id);
}
```

#### 장단점 및 고려사항

- **장점**:
  - 구현이 직관적이며 실제 조회된 데이터만 캐시되므로 **메모리가 효율적**이다.
  - Redis가 다운되어도 DB에서 직접 데이터를 읽어올 수 있어 **서비스 가용성을 유지**할 수 있다.
- **단점**:
  - 캐시 미스 시 DB 조회를 거쳐야 하므로 **첫 번째 요청에서 지연(Latency)** 이 발생한다.
  - DB 갱신 후 캐시 삭제 직전까지 일시적인 데이터 불일치가 발생할 수 있다.
- **동시 트래픽 문제**:
  - 캐시가 비어있는 상태에서 대량의 요청이 몰릴 경우 모든 요청이 DB로 집중되어 **커넥션 풀 고갈 및 서버 과부하**가 발생할 수 있다. (Cache Stampede 현상)

### 1.3. Write Through

- 쓰기 작업 발생 시 **Cache와 Database를 단일 흐름으로 동시에 갱신**하는 패턴이다.

```mermaid
sequenceDiagram
    participant App as 애플리케이션
    participant Redis
    participant DB as 데이터베이스

    App->>Redis: 1. SET key value
    Redis->>DB: 2. INSERT/UPDATE
    DB-->>Redis: 성공 응답
    Redis-->>App: 완료 반환
    Note over App,DB: 캐시와 DB가 상시 동기화됨
```

- **쓰기 흐름**: 쓰기 요청이 캐시 계층으로 전달되면, 캐시가 DB에 쓰기 작업을 전달하고 두 작업이 모두 완료되어야 성공을 응답한다.
- **읽기 흐름**: 항상 캐시에서 우선 조회하며, 캐시 미스 시 DB에서 로드한다.

#### 예시 구현

```typescript
async function updateUser(userId: number, data: User): Promise<void> {
  // 1. 캐시 업데이트
  await redis.set(`user:${userId}`, JSON.stringify(data));

  // 2. 데이터베이스 업데이트 (동기 처리)
  await db.execute("UPDATE users SET ... WHERE id = ?", userId);
}

async function getUser(userId: number): Promise<User> {
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
  await redis.set(`user:${userId}`, JSON.stringify(user));

  return user;
}
```

#### 장단점

- **장점**:
  - 캐시와 DB가 상시 일치하여 **데이터 불일치 구간이 발생하지 않는다**.
  - 방금 작성된 데이터도 이미 캐시에 적재되어 있어 **읽기 지연이 최소화**된다.
  - DB 쓰기 완료가 보장되므로 캐시 장애 시에도 **데이터 유실 위험이 없다**.
- **단점**:
  - 모든 쓰기마다 Cache와 DB에 동시 쓰기를 수행하므로 **쓰기 지연(Latency)이 증가**한다.
  - 한 번 쓰인 후 다시 읽히지 않는 데이터까지 캐시되어 메모리가 낭비될 수 있다.
- **적합한 사용처**: **읽기 비중이 압도적으로 높고 쓰기가 드문 환경**(상품 카탈로그, 공지사항)이나 정합성이 중요한 금융/재고 데이터.

### 1.4. Write Behind (Write Back)

- 쓰기 요청을 **캐시에만 즉시 반영**한 뒤 성공을 반환하고, DB 쓰기는 백그라운드 큐/워커를 통해 **비동기 배치(Batch)로 처리**하는 패턴이다.

```mermaid
sequenceDiagram
    participant App as 애플리케이션
    participant Redis
    participant Queue as 큐
    participant DB as 데이터베이스

    App->>Redis: 1. SET key value
    Redis-->>App: 즉시 완료 반환
    Redis->>Queue: 2. 비동기 전달
    Queue->>DB: 3. 백그라운드 INSERT/UPDATE
    Note over App,DB: 쓰기가 매우 빠르지만 데이터 유실 위험 존재
```

#### 예시 구현

```typescript
async function updateUser(userId: number, data: User): Promise<boolean> {
  // 1. 캐시 업데이트 (즉시 완료)
  await redis.set(`user:${userId}`, JSON.stringify(data));

  // 2. 비동기 큐에 작업 적재
  await redis.rpush(
    "write_queue",
    JSON.stringify({
      type: "update_user",
      userId: userId,
      data: data,
    }),
  );

  return true;
}

// 백그라운드 워커: 큐에서 대기하다가 DB 반영
async function worker(): Promise<void> {
  while (true) {
    const item = await redis.blpop("write_queue", 1);
    if (!item) {
      continue;
    }

    const task = JSON.parse(item[1]);

    if (task.type === "update_user") {
      await db.execute("UPDATE users SET ... WHERE id = ?", task.userId);
    }
  }
}
```

#### 장단점

- **장점**:
  - DB 디스크 I/O를 대기하지 않고 메모리에만 쓰므로 **쓰기 응답 속도가 극도로 빠르다**.
  - 여러 쓰기 요청을 모아 배치 쿼리로 DB에 반영하므로 **DB 부하를 획기적으로 줄인다**.
  - 쓰기 트래픽 폭증 시 큐가 버퍼 역할을 수행하여 시스템 장애를 방지한다.
- **단점**:
  - DB에 반영되기 전에 캐시나 큐 서버가 다운되면 **데이터가 영구 유실**될 수 있다.
  - 캐시에는 최신 데이터가 있으나 DB에는 아직 반영되지 않은 시차가 존재하여 타 시스템 조회 시 정합성이 깨질 수 있다.
  - 재시도, 순서 보장, 중복 처리 등 인프라 구현 복잡도가 높다.
- **적합한 사용처**: 초당 대량의 쓰기가 발생하며 소량의 데이터 유실을 감수할 수 있는 **조회수 통계, 로그 수집, 실시간 지표 집계** 등.

### 1.5. Read Through

- **애플리케이션은 오직 캐시 레이어만 상대**하며, 캐시 미스 시 캐시 레이어가 직접 DB 조회를 수행하고 데이터를 적재한 후 반환하는 패턴이다.

```mermaid
sequenceDiagram
    participant App as 애플리케이션
    participant CacheLayer as 캐시 레이어
    participant DB as 데이터베이스

    App->>CacheLayer: GET key
    CacheLayer->>CacheLayer: 캐시 확인
    alt 캐시 미스
        CacheLayer->>DB: SELECT *
        DB-->>CacheLayer: 데이터 반환
        CacheLayer->>CacheLayer: 캐시 저장
    end
    CacheLayer-->>App: 최종 데이터 반환
    Note over App,DB: 애플리케이션은 캐시 계층만 호출함
```

#### Cache Aside와의 차이점

- **책임의 분리**: Cache Aside는 애플리케이션 코드가 직접 캐시/DB 조회 흐름을 제어하지만, Read Through는 **캐시 추상화 계층이 데이터 로딩 책임을 캡슐화**한다.

#### 예시 구현

```typescript
function readThroughCache<T>(
  keyFunc: (...args: any[]) => string,
  fetchFunc: (...args: any[]) => Promise<T>,
  ttl: number = 3600,
) {
  return async (...args: any[]): Promise<T> => {
    const key = keyFunc(...args);

    // 1. 캐시 확인
    const cached = await redis.get(key);
    if (cached) {
      return JSON.parse(cached);
    }

    // 2. 캐시 미스 시 원본 DB 조회 함수 실행
    const result = await fetchFunc(...args);

    // 3. 캐시 저장
    await redis.setex(key, ttl, JSON.stringify(result));

    return result;
  };
}

// 사용 예시
async function fetchUserFromDB(userId: number): Promise<User> {
  return db.query("SELECT * FROM users WHERE id = ?", userId);
}

const getUser = readThroughCache<User>(
  (userId: number) => `user:${userId}`,
  fetchUserFromDB,
);
```

#### 장단점

- **장점**: 비즈니스 로직과 캐시 제어 로직이 분리되어 코드가 간결해지며, 일관된 캐싱 정책을 중앙에서 관리할 수 있다. (Spring `@Cacheable` 등)
- **단점**: 별도의 캐시 추상화 레이어를 구축해야 하며, 엔드포인트별 세부 튜닝 유연성이 다소 떨어질 수 있다.

### 1.6. Cache Stampede (Thundering Herd) 현상과 해결 방안

```mermaid
sequenceDiagram
    participant C1 as 클라이언트 1
    participant C2 as 클라이언트 2
    participant C3 as 클라이언트 3
    participant Redis
    participant DB as 데이터베이스

    Note over C1,DB: 캐시 만료 직후
    C1->>Redis: GET key (Miss)
    C2->>Redis: GET key (Miss)
    C3->>Redis: GET key (Miss)
    C1->>DB: SELECT *
    C2->>DB: SELECT * (중복!)
    C3->>DB: SELECT * (중복!)
    Note over C1,DB: DB 과부하 및 커넥션 풀 고갈 발생
```

- 특정 핫키(Hot Key)의 캐시가 만료되는 순간, **동시에 수백~수천 개의 요청이 한꺼번에 DB로 몰려 커넥션 풀이 고갈되고 서버가 다운되는 현상**이다.

#### 해결 방안 1: PER (Probabilistic Early Recomputation) 알고리즘

```typescript
async function getUserWithEarlyRecompute(userId: number): Promise<User> {
  const cacheKey = `user:${userId}`;

  const cached = await redis.get(cacheKey);
  const ttl = await redis.ttl(cacheKey);

  if (cached) {
    const beta = 1; // 튜닝 계수 (1~10)
    const delta = (3600 - ttl) / 3600; // 현재 경과 시간 비율

    // 확률적 조기 갱신 조건 판별
    if (Math.random() < delta * beta) {
      const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
      await redis.setex(cacheKey, 3600, JSON.stringify(user));
      return user;
    }

    return JSON.parse(cached);
  }

  // 캐시 미스 처리
  const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
  await redis.setex(cacheKey, 3600, JSON.stringify(user));
  return user;
}
```

- 캐시가 완전히 만료되기 전, **남은 TTL과 연산 비용, 난수를 기반으로 확률적인 조기 갱신을 수행**하는 방식이다.
- 만료 시점에 가까워질수록 갱신 확률이 점진적으로 증가하여, 단 1개의 요청만 선제적으로 DB를 조회해 캐시를 갱신하도록 유도한다.

#### 해결 방안 2: 분산 락(Lock) 기반 제어 (`SETNX`)

```typescript
async function getUserWithLock(userId: number): Promise<User> {
  const cacheKey = `user:${userId}`;
  const lockKey = `lock:${cacheKey}`;

  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // 락 획득 시도 (10초 만료)
  const acquired = await redis.set(lockKey, "1", "NX", "EX", 10);

  if (acquired) {
    try {
      // 락을 획득한 1개의 요청만 DB 조회 및 캐시 적재
      const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
      await redis.setex(cacheKey, 3600, JSON.stringify(user));
      return user;
    } finally {
      // 락 해제
      await redis.del(lockKey);
    }
  } else {
    // 락 획득 실패 시 잠시 대기 후 재시도
    await new Promise((resolve) => setTimeout(resolve, 100));
    return getUserWithLock(userId);
  }
}
```

- 캐시 미스 발생 시 Redis의 `SET key value NX EX ttl` 또는 분산 락을 활용하여 **오직 1개의 요청만 락을 획득해 DB를 조회**하도록 제한한다.
- 락을 획득하지 못한 나머지 요청들은 잠시 대기한 뒤 캐시가 갱신되면 캐시에서 값을 읽어가도록 유도한다.

#### 해결 방안 3: 백그라운드 주기적 갱신 (Scheduler / Cron)

```typescript
async function backgroundRefresh(): Promise<void> {
  while (true) {
    const popularUsers = await getPopularUserIds();

    for (const userId of popularUsers) {
      const user = await db.query("SELECT * FROM users WHERE id = ?", userId);
      await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
    }

    // 30분 주기로 갱신
    await new Promise((resolve) => setTimeout(resolve, 1800000));
  }
}
```

- 캐시 만료 시간에 의존하지 않고, 별도의 백그라운드 스케줄러가 **주기적으로 DB에서 인기 데이터를 조회하여 캐시를 사전에 덮어쓰기(Warm-up)** 하는 방식이다.
- **특징**: Cache Miss 발생률을 0에 가깝게 유지할 수 있으나, 스케줄링 주기만큼의 데이터 변경 지연이 발생할 수 있어 실시간성이 덜 중요한 데이터에 적합하다.

## 2. Redis Cluster 심화 (Hash Slot, Sharding, 재구성)

- 운영 환경(Production)에서 Redis를 운용할 때는 **고가용성(HA)**, **자동 장애 조치(Failover)**, **데이터 분산**을 위해 **Redis 클러스터**를 구성하는 것이 표준이다.

### 2.1. Redis Cluster 개요 및 도입 이유

- **Redis 클러스터**: 여러 Redis 인스턴스를 하나의 논리적 데이터베이스로 묶어 데이터를 자동으로 분산·관리하는 분산형 아키텍처이다.
- **수평 확장(Horizontal Scaling)**: 노드를 증설함에 따라 저장 용량과 처리량이 선형적으로 증가한다.
- **도입 목적**:
  - **메모리 한계 극복**: 단일 서버의 물리적 RAM 한계를 넘어 대규모 데이터를 분산 저장한다.
  - **처리량 극대화**: 읽기 및 쓰기 부하를 여러 노드로 분산하여 초당 요청 처리량(TPS)을 대폭 향상한다.
  - **고가용성 보장**: 특정 마스터 노드 장애 시 레플리카가 마스터로 자동 승격(Failover)되어 중단 없는 서비스를 유지한다.

### 2.2. 클러스터 아키텍처

- **최소 구성 요건**: 클러스터는 **최소 3개의 마스터 노드**가 필요하며, 운영 환경에서는 각 마스터마다 1개 이상의 레플리카를 두는 **3 마스터 + 3 레플리카(총 6개 노드)** 구조가 기본 권장 사양이다.
- **완전 분산형 구조**: 중앙 코디네이터(ZooKeeper, Consul 등) 없이 모든 노드가 **가십(Gossip) 프로토콜**을 통해 클러스터 상태와 토폴로지를 상호 공유한다.

```mermaid
graph TD
    Client["클라이언트"]
    Client -- "리다이렉션" --> M1
    Client -- "리다이렉션" --> M2
    Client -- "리다이렉션" --> M3

    subgraph Cluster["클러스터"]
        M1["마스터 1<br/>슬롯 0-5460"]
        M2["마스터 2<br/>슬롯 5461-10922"]
        M3["마스터 3<br/>슬롯 10923-16383"]
        R1["레플리카 1-1"]
        R2["레플리카 2-1"]
        R3["레플리카 3-1"]

        M1 -. "복제" .-> R1
        M2 -. "복제" .-> R2
        M3 -. "복제" .-> R3

        M1 <-- "가십" --> M2
        M2 <-- "가십" --> M3
    end
```

#### 가십(Gossip) 프로토콜과 클러스터 버스

- **동작 방식**: 각 노드는 무작위로 선택된 노드들과 `PING`/`PONG` 메시지를 주고받으며 슬롯 매핑, 노드 헬스체크 정보를 수초 내로 동기화한다.
- **장애 감지**: 다수의 마스터 노드가 특정 노드의 무응답(실패)에 합의하면 자동 장애 조치(Failover)를 시작한다.
- **클러스터 버스 포트**: 클라이언트 통신 포트에 **10000을 더한 포트**를 사용한다. (예: 서비스 포트 `6379` $\rightarrow$ 클러스터 버스 `16379`)
- **방화벽 주의**: 노드 간 바이너리 통신을 위해 서비스 포트뿐만 아니라 **클러스터 버스 포트도 반드시 개방**해야 한다.

### 2.3. Hash Slot

```mermaid
flowchart LR
    A["키: user:123"] --> B["CRC16 해시"]
    B --> C["해시값 % 16384"]
    C --> D["슬롯 번호: 8452"]
    D --> E["마스터 2<br/>슬롯 5461-10922"]

    style E fill:#d4f4dd
```

- **해시 슬롯(Hash Slot)**: 클러스터 내부에서 데이터를 분할·배치하는 논리적 최소 단위이다.
- Redis 클러스터는 **총 16,384개(0 ~ 16383)** 의 슬롯을 지원하며, 모든 키는 하나의 슬롯에 매핑되고 각 슬롯은 특정 마스터에 할당된다.
  - 예: 3개 마스터 노드 기준 각 노드가 약 5,461개의 슬롯을 균등 분할 소유한다.
- **슬롯 계산식**: $\text{Slot} = \text{CRC16}(\text{key}) \pmod{16384}$
- **결정적(Deterministic) 연산**: 동일한 키는 상시 동일한 슬롯 번호로 계산된다.

#### 해시태그 (Hash Tag)

- 다중 키 연산(`MGET`, `MSET`, 트랜잭션 등)은 연관된 모든 키가 **동일한 슬롯(동일 노드)** 에 위치해야만 실행 가능하다.
- **문법**: 키 이름 내부의 **중괄호(`{...}`) 영역만 해싱**에 참여하도록 강제한다.
  - 예: `{user:1}:profile`, `{user:1}:orders` $\rightarrow$ 두 키 모두 `user:1` 문자열만 해싱되어 동일한 슬롯에 할당된다.
- **주의사항**: 해시태그를 남용하면 특정 슬롯/노드로 데이터와 트래픽이 쏠려 **데이터 불균형(Hotspot)** 이 발생할 수 있다.

### 2.4. Sharding과 클라이언트 리다이렉션

```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant M1 as 마스터 1
    participant M2 as 마스터 2

    Client->>M1: 1. SET user:123
    M1->>M1: 2. 슬롯 8452 계산
    M1-->>Client: 3. MOVED 8452 127.0.0.1:7001
    Client->>M2: 4. SET user:123
    M2-->>Client: 5. OK
```

- 클라이언트가 보낸 요청의 키가 해당 노드의 담당 슬롯이 아닌 경우, 노드는 올바른 대상 노드의 주소와 함께 **`MOVED`** 응답을 반환한다.
- **클라이언트 캐싱**: 스마트 클라이언트 라이브러리는 `MOVED` 수신 시 내부의 **슬롯-노드 매핑 테이블을 갱신**하여 이후 요청은 올바른 노드로 직접 전송한다.
- **`CROSSSLOT` 에러**: 서로 다른 슬롯에 속한 키들을 단일 다중 키 명령어로 호출할 때 발생하며, **해시태그**를 적용하여 단일 슬롯으로 통합해야 한다.

### 2.5. 클러스터 재구성과 리밸런싱

- **클러스터 재구성**: 서비스 중단 없이 노드를 추가(Scale-Out)하거나 제거(Scale-In)하며 해시 슬롯을 재분배하는 작업이다.

#### 노드 추가 및 슬롯 마이그레이션 5단계

```mermaid
flowchart TD
    A["1. 상태 표시<br/>소스: MIGRATING / 타겟: IMPORTING"] --> B["2. 키 점진적 복사<br/>MIGRATE 명령 (원자적)"]
    B --> C{"마이그레이션 중<br/>요청 처리"}
    C -->|"키가 소스에 있음"| D["소스에서 그대로 처리"]
    C -->|"키가 타겟으로 이동됨"| E["ASK 리다이렉션 반환"]
    D --> F["3. 모든 키 이동 완료"]
    E --> F
    F --> G["4. 소유권 변경<br/>SETSLOT 명령으로 슬롯 테이블 갱신"]
    G --> H["5. 가십 프로토콜로 전체 전파<br/>모든 노드가 새 매핑 학습"]
```

1. **상태 표시**: 소스 노드는 해당 슬롯을 **`MIGRATING`**, 타겟 노드는 **`IMPORTING`** 상태로 설정한다. (두 노드 간에만 인지)
2. **키 점진적 복사**: **`MIGRATE`** 명령을 통해 키 단위로 원자적 이동을 수행한다.
   - 마이그레이션 도중 소스 노드로 요청 인입 시: 키가 소스에 남아있으면 **직접 처리**, 이미 타겟으로 옮겨졌다면 **`ASK` 리다이렉션**을 응답한다.
3. **모든 키 이동 완료**: 해당 슬롯에 포함된 모든 데이터의 복사를 마친다.
4. **소유권 변경**: **`SETSLOT`** 명령으로 공식 슬롯 소유권을 타겟 노드로 이관하고 마이그레이션 상태를 해제한다.
5. **가십 전파**: 변경된 슬롯 테이블을 클러스터 전체 노드에 전파한다. 이후 구 노드로 인입되는 요청에는 **`MOVED`** 를 응답한다.

#### MOVED vs ASK 리다이렉션 비교

| 구분                | **MOVED**                                          | **ASK**                                                  |
| :------------------ | :------------------------------------------------- | :------------------------------------------------------- |
| **의미**            | 슬롯의 소유권 이전이 **완전히 완료**됨             | 해당 슬롯이 현재 **마이그레이션 진행 중**임              |
| **클라이언트 캐싱** | 새로운 노드 매핑 정보를 **캐시에 반영(영구 갱신)** | 매핑 캐시를 **갱신하지 않음(1회성 임시 리다이렉션)**     |
| **발생 시점**       | 정상 운영 상태 또는 재구성 완료 후                 | 슬롯 마이그레이션 진행 도중                              |
| **클라이언트 동작** | 새 노드로 바로 요청 전송                           | 타겟 노드에 **`ASKING`** 명령을 선전송한 후 본 명령 실행 |

#### 리밸런싱 (Rebalancing)

- 노드 추가/제거 외에도 특정 슬롯에 데이터가 과도하게 몰려 **노드 간 메모리/부하 불균형이 발생했을 때 슬롯을 재분배**하는 작업이다.
- 슬롯 이동은 네트워크 대역폭과 CPU 자원을 소모하므로, 최소 이동량을 계산하여 **불필요한 데이터 전송을 최소화**하도록 설계한다.

### 2.6. 클러스터 재구성 프로세스 요약 다이어그램

```mermaid
flowchart TD
    subgraph S1["1. 노드 추가 (Scale Out)"]
        A1["새 노드 시작"] --> A2["클러스터 조인<br/>가십 핸드셰이크"]
        A2 --> A3["빈 마스터 등록<br/>슬롯 0개"]
        A3 --> A4["슬롯 재분배 계획<br/>균등 분산 계산"]
        A4 --> A5["슬롯 마이그레이션<br/>무중단 이동"]
    end

    subgraph S2["2. 슬롯 마이그레이션 상태"]
        B1["타겟: IMPORTING"]
        B2["소스: MIGRATING"]
        B1 --> B3["키 복사<br/>점진적 이동"]
        B2 --> B3
        B3 --> B4{"마이그레이션 중<br/>요청 처리"}
        B4 -->|"키가 타겟에"| B5["ASK 리다이렉션"]
        B4 -->|"키가 소스에"| B6["소스에서 처리"]
        B5 --> B7["이동 완료 후<br/>MOVED 전환"]
        B6 --> B7
    end

    subgraph S3["3. 리다이렉션 타입"]
        C1["MOVED"] --> C2["영구적 이동<br/>클라이언트 캐시 갱신"]
        C3["ASK"] --> C4["임시 리다이렉션<br/>캐시 갱신 없음"]
    end

    subgraph S4["4. 노드 제거 (Scale In)"]
        D1["제거 대상 선정"] --> D2["슬롯을 다른<br/>노드로 이동"]
        D2 --> D3["모든 슬롯<br/>이동 완료"]
        D3 --> D4["노드 제거"]
    end

    subgraph S5["5. 리밸런싱"]
        E1["슬롯 분포 분석<br/>불균형 감지"] --> E2["목표 슬롯 계산<br/>16384 / N"]
        E2 --> E3["슬롯 이동 계획<br/>최소 이동량"]
        E3 --> E4["점진적 재분배<br/>무중단 실행"]
    end

    S1 --> S2
    S4 --> S2
    S2 --> S5
```
