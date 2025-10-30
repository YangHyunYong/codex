# Day 11: Reth 스타일 트랜잭션 풀 우선순위 큐 구현

**난이도: MEDIUM (트랜잭션 스케줄링)**

## [전날 과제 요약]
- Day 10에서는 포크 선택기를 확장해 total difficulty가 더 큰 체인을 canonical로 재구성하는 로직을 완성했습니다.
- 헤더를 그래프로 저장하고, reorg 깊이를 계산해 체인 롤백 상황을 감지할 수 있게 되었습니다.
- 에러/결과 타입을 명시적으로 설계하며 Reth의 HeaderAccumulator 개념을 모사했습니다.

## [전날 과제로 얻은 역량]
- 블록 헤더 그래프 상태를 HashMap과 Vec으로 모델링하는 감각을 익혔습니다.
- 체인 동기화에서 canonical head를 유지하기 위한 재구성 절차를 이해했습니다.
- 테스트 주도로 포크 확장/유지/재구성 시나리오를 검증하는 습관을 들였습니다.

## [오늘 과제 목표]
- Reth의 TxPool이 트랜잭션을 우선순위 기반으로 정렬하는 핵심 개념을 체험합니다.
- 동일한 sender의 nonce 순서를 보장하면서도 전역 우선순위를 유지하는 자료구조를 설계합니다.
- 트랜잭션 삽입, 대기열 포화 시 축출(eviction), 배치(pop) 로직을 테스트 주도 방식으로 구현합니다.

## [오늘 과제 설명]
Reth TxPool은 각 계정(sender)의 nonce 순서를 깨지 않으면서도, 전역적으로 높은 가스 가격(gas_price)을 우선 처리하도록 설계되어 있습니다. 오늘은 메모리 기반 간단한 트랜잭션 풀을 만들어 이 개념을 연습합니다. 계정별로 정렬된 큐를 유지하고, 전역 우선순위 큐로 head 트랜잭션만 비교하며 풀 크기를 제한합니다.

1. **프로젝트 생성**
   - `cargo new day11_tx_pool --lib` 명령을 실행합니다.
   - 라이브러리 로직은 `src/lib.rs`, 통합 테스트는 `tests/pool.rs`에 작성하세요.

2. **트랜잭션 및 래퍼 정의 (`src/lib.rs`)**
   - `PendingTransaction` 구조체를 정의하고 다음 필드를 포함합니다.
     ```rust
     pub struct PendingTransaction {
         pub hash: String,
         pub sender: String,
         pub nonce: u64,
         pub gas_price: u64,
         pub priority: u128,
     }
     ```
   - 구조체 위에 `priority` 필드를 별도로 두는 이유(가스 가격 외 가중치를 고려) 를 설명하는 주석을 작성합니다.
   - `PendingTransaction`에 `impl` 블록을 추가하고 `fn priority_key(&self) -> (u128, u64)`를 구현하여 `(priority, gas_price)` 튜플을 반환하도록 합니다. 반환 순서에 대한 설명 주석을 함수 위에 남기세요.

3. **풀 설정과 상태 정의 (`src/lib.rs`)**
   - `TxPoolConfig` 구조체를 만들고 다음 필드를 추가하세요.
     ```rust
     pub struct TxPoolConfig {
         pub capacity: usize,
         pub max_account_slots: usize,
     }
     ```
   - 각 필드 위에 Reth TxPool의 어떤 제한을 단순화한 것인지 주석으로 적습니다.
   - `TxPool` 구조체를 선언하고 아래 필드를 포함합니다.
     ```rust
     use std::collections::{BinaryHeap, HashMap, VecDeque};

     pub struct TxPool {
         config: TxPoolConfig,
         per_account: HashMap<String, VecDeque<PendingTransaction>>,
         global_queue: BinaryHeap<QueuedTx>,
         total_txs: usize,
     }
     ```
   - `QueuedTx`는 BinaryHeap에서 사용할 래퍼입니다. `QueuedTx` 구조체와 `Ord`/`PartialOrd` 구현을 작성해 높은 priority_key가 먼저 나오도록 하세요. 구현 부분에 "Reth는 max-heap으로 우선 처리"에 대한 짧은 주석을 남깁니다.

4. **에러 및 결과 타입 설계 (`src/lib.rs`)**
   - `TxInsertError` enum을 만들고 아래 변형을 포함합니다.
     - `DuplicateNonce { sender: String, nonce: u64 }`
     - `AccountLimitReached { sender: String }`
     - `PoolFull`
   - 각 변형 위에 어떤 상황에서 발생하는지 간단한 주석을 추가합니다.
   - `PopResult` enum을 선언하고 `Batch { drained: Vec<PendingTransaction> }`와 `Empty` 변형을 포함하세요. enum 위에 "배치 스케줄링 단위"라는 주석을 남기세요.

5. **주요 메서드 구현 (`src/lib.rs`)**
   - `impl TxPool` 블록에 다음 메서드를 구현합니다.
     1. `pub fn new(config: TxPoolConfig) -> Self` — 내부 컬렉션을 초기화합니다.
     2. `pub fn insert(&mut self, tx: PendingTransaction) -> Result<(), TxInsertError>`
        - sender에 해당하는 큐가 없다면 생성합니다.
        - 동일한 nonce가 이미 존재하면 `DuplicateNonce`를 반환합니다.
        - 계정별 큐 길이가 `max_account_slots`를 초과하면 `AccountLimitReached`를 반환합니다.
        - 풀 전체 용량이 가득 차면 `evict_lowest_priority()` 헬퍼를 호출해 가장 낮은 우선순위를 제거하고 `PoolFull`을 반환합니다.
        - 유효한 경우, 계정 큐에 nonce 순서대로 삽입하고, 해당 계정의 프런트 트랜잭션을 `global_queue`에 반영하세요.
     3. `pub fn pop_batch(&mut self, limit: usize) -> PopResult`
        - `limit` 개수까지 global_queue에서 트랜잭션을 꺼내고, 동일 sender의 다음 nonce를 global_queue에 재삽입합니다.
        - pop 과정에서 제거된 트랜잭션은 per_account에서도 제거되어야 합니다.
        - 반환된 벡터는 priority_key 순서를 보장하도록 정렬합니다.
   - `evict_lowest_priority()`는 `BinaryHeap`을 순회할 수 없으므로, 별도의 `BTreeMap` 또는 `Vec`를 사용하지 않고 "lazy eviction" 전략으로 구현합니다. 함수 구현 위에 이 전략의 의미를 설명하는 주석을 작성하세요.

6. **테스트 작성 (`tests/pool.rs`)**
   - 파일 첫 줄에 "전역/계정별 우선순위 동시 검증"을 설명하는 한 문장 주석을 작성합니다.
   - 다음 테스트를 구현하세요.
     1. **nonce 정렬 보장**: 동일 sender의 nonce가 순서대로 유지되는지 검증합니다.
     2. **글로벌 우선순위 선출**: 서로 다른 sender의 트랜잭션이 priority_key 기준으로 pop되는지 확인합니다.
     3. **풀 포화 및 축출**: capacity를 초과할 때 가장 낮은 priority가 축출되고 `PoolFull` 에러가 발생하는지 검증합니다.
   - 각 테스트는 `per_account` 내부 상태 또는 `global_queue` 크기를 직접 확인해 구조가 기대와 일치하는지 검증하세요.

7. **마무리 루틴 안내**
   - README 마지막에 학습자가 실행해야 할 명령을 아래 순서대로 안내합니다.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
아래는 sender별 큐의 프런트만 전역 큐에 노출하는 로직 예시입니다. 실제 구현 시 주석을 적절히 보강해 주세요.

```rust
fn refresh_global_queue(
    sender: &str,
    per_account: &HashMap<String, VecDeque<PendingTransaction>>,
    global_queue: &mut BinaryHeap<QueuedTx>,
) {
    if let Some(queue) = per_account.get(sender) {
        if let Some(front) = queue.front() {
            // BinaryHeap은 max-heap이므로 높은 priority_key가 먼저 나옵니다.
            global_queue.push(QueuedTx::new(front.clone()));
        }
    }
}
```

- 이 패턴은 Reth TxPool이 계정별로 정렬된 큐를 유지하면서 전역적으로 헤드 트랜잭션만 비교하는 방식을 단순화한 것입니다.
- lazy eviction은 Reth가 내부적으로 사용하는 전략으로, BinaryHeap에서 제거된 항목이 실제 상태와 불일치하면 pop 시점에 무시하여 일관성을 유지합니다.
- priority_key를 튜플로 정의하면 가스 가격이 동일할 때 priority(예: reputation score)로 추가 정렬이 가능합니다.

---

### 오늘의 TIL (Today I Learned)
- 트랜잭션 풀에서 계정별 순서와 전역 우선순위를 동시에 유지하는 패턴을 구현했습니다.

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
