# Day 8: 슬롯 실행 대기열에서 계정 잠금 시뮬레이션

**난이도: MEDIUM (기본기에서 한 단계 더 응용)**

## [전날 과제 요약]
- Day 7에서는 `SlotPlanner`를 만들어 수수료가 높은 트랜잭션을 우선적으로 선택하고, 블록 제한을 넘기지 않는 안전한 선택 로직을 구성했습니다.
- `BinaryHeap`을 이용해 정렬된 입력을 다루고, 제한을 초과하는 항목은 건너뛰는 단순한 그리디 전략을 구현했습니다.
- 테스트 코드를 통해 성공/실패 시나리오를 검증하고, 잔여 용량 계산까지 마무리했습니다.

## [전날 과제로 얻은 역량]
- 제한 조건을 명확히 정의하고 구조체로 모델링할 수 있습니다.
- 우선순위 큐와 순차 선택 로직을 연결해 실용적인 번들 생성기를 만들었습니다.
- 테스트 주도 흐름에서 간단한 헬퍼 함수와 주석을 활용해 의도를 설명하는 습관을 익혔습니다.

## [오늘 과제 목표]
- **슬롯 실행 대기열**에 새 트랜잭션이 도착했을 때, 이미 실행 중인 트랜잭션과 충돌하지 않는지 검증하는 로직을 만듭니다.
- 계정 잠금(Account Locking) 개념을 모사하여 **중복 계정 접근을 방지**하는 연습을 합니다.
- `Result`와 커스텀 에러 타입을 도입해 **명확한 실패 이유**를 표현하고, 단위 테스트로 각 경우를 입증합니다.

## [오늘 과제 설명]
오늘은 Solana 실행 엔진에서 자주 등장하는 "계정 잠금" 개념을 축소판으로 체험합니다. 동일한 계정을 동시에 두 트랜잭션이 사용하면 충돌이 발생하므로, 대기열에 들어가기 전에 검증해야 합니다. 아래 순서를 따라 프로젝트를 구성하세요.

1. **프로젝트 생성**
   - `cargo new day8_account_locking --lib` 명령으로 새 라이브러리 프로젝트를 생성합니다.
   - 핵심 로직은 `src/lib.rs`, 테스트는 `tests/lock_queue.rs`에 작성합니다.

2. **데이터 구조 정의 (`src/lib.rs`)**
   - `TransactionMeta` 구조체를 만듭니다. 다음 필드를 포함하세요.
     ```rust
     pub struct TransactionMeta {
         pub id: String,
         pub writable_accounts: Vec<String>,
         pub readonly_accounts: Vec<String>,
         pub compute_units: u32,
     }
     ```
   - "이 구조체가 담는 정보"를 설명하는 한 문장짜리 한국어 주석을 구조체 위에 적어 주세요.
   - 슬롯에 이미 실행 중인 정보를 추적하기 위한 `SlotExecutionState` 구조체를 만듭니다. 내부에는 최소한 아래 필드를 포함하세요.
     ```rust
     pub struct SlotExecutionState {
         pub locked_writable: HashSet<String>,
         pub locked_readonly: HashSet<String>,
         pub consumed_compute_units: u32,
     }
     ```
   - `HashSet` 사용을 위해 `std::collections::HashSet`을 `use`하고, 구조체 위에 "왜 HashSet이 필요한가"를 설명하는 주석을 작성합니다.
   - 블록 제한을 재사용하기 위해 `BlockConstraint` 구조체를 Day 7과 동일한 필드로 정의합니다.

3. **에러 타입 정의 (`src/lib.rs`)**
   - `AccountLockError`라는 `enum`을 만들고, 아래 변형을 포함하세요.
     - `Conflict { account: String }`
     - `ComputeLimitExceeded { requested: u32, limit: u32 }`
   - 각 변형 위에 "어떤 상황에서 발생하는가"를 간단히 설명하는 주석을 남깁니다.

4. **핵심 구조체 설계 (`src/lib.rs`)**
   - `ExecutionQueue` 구조체를 선언하고 다음 필드를 갖도록 합니다.
     ```rust
     pub struct ExecutionQueue {
         constraint: BlockConstraint,
         state: SlotExecutionState,
         pending: VecDeque<TransactionMeta>,
     }
     ```
   - `VecDeque`를 사용하려면 `std::collections::VecDeque`를 임포트하고, "왜 일반 Vec 대신 VecDeque를 사용하는가"를 설명하는 주석을 필드 위에 적습니다.
   - `impl ExecutionQueue` 블록에서 다음 메서드를 구현합니다.
     1. `pub fn new(constraint: BlockConstraint) -> Self`
        - 비어 있는 상태와 대기열을 준비하고, 제한을 저장합니다.
        - 생성자 위에 "초기 상태에서 어떤 잠금이 적용되는가"를 설명하는 주석을 남기세요.
     2. `pub fn try_enqueue(&mut self, tx: TransactionMeta) -> Result<(), AccountLockError>`
        - 계정 충돌 검사를 수행한 뒤, 성공하면 대기열과 상태를 업데이트합니다.
        - 실패하면 `AccountLockError`를 돌려주고 상태를 변경하지 않습니다.
     3. `pub fn release(&mut self, tx_id: &str)`
        - 실행이 끝난 트랜잭션을 가정하고, 해당 ID에 해당하는 계정 잠금과 컴퓨트 사용량을 해제합니다.
        - 대기열에서도 동일한 ID의 항목이 있으면 제거합니다.
        - ID가 없을 경우 조용히 아무 일도 일어나지 않도록 처리합니다.

5. **도우미 함수 작성 (`src/lib.rs`)**
   - `fn check_account_conflicts(state: &SlotExecutionState, tx: &TransactionMeta) -> Option<String>` 함수를 추가합니다.
     - 충돌이 있으면 해당 계정 이름을 `Some(account_name)`으로 반환하고, 없으면 `None`을 반환합니다.
     - 충돌 규칙은 다음과 같습니다.
       1. 새 트랜잭션의 `writable_accounts`가 기존 `locked_writable` 또는 `locked_readonly`와 겹치면 충돌입니다.
       2. 새 트랜잭션의 `readonly_accounts`가 기존 `locked_writable`과 겹치면 충돌입니다.
   - `fn would_exceed_compute(state: &SlotExecutionState, tx: &TransactionMeta, constraint: &BlockConstraint) -> bool` 함수도 추가합니다.
     - 현재 사용량에 새 트랜잭션의 컴퓨트 유닛을 더했을 때 제한을 초과하면 `true`를 반환합니다.
   - 두 함수 모두 위에 "이 함수가 왜 필요한가"를 설명하는 짧은 주석을 적습니다.

6. **상태 업데이트 로직 (`src/lib.rs`)**
   - `try_enqueue` 내부에서 도우미 함수를 활용해 다음 순서대로 처리하세요.
     1. `would_exceed_compute`가 `true`면 `AccountLockError::ComputeLimitExceeded`를 반환합니다. `requested`에는 현재 소비량과 요청량의 합계를, `limit`에는 제한을 넣으세요.
     2. `check_account_conflicts`가 `Some(account)`를 돌려주면 `AccountLockError::Conflict { account }`를 반환합니다.
     3. 둘 다 문제가 없으면, `state`의 집합과 `consumed_compute_units`를 업데이트하고, `pending` 큐에 트랜잭션을 push합니다.
   - `release`에서는 잠금 해제 시 HashSet에서 계정을 제거하고, `consumed_compute_units`를 `saturating_sub`로 줄이세요.

7. **테스트 작성 (`tests/lock_queue.rs`)**
   - 파일 상단에 "이 테스트 모음이 무엇을 확인하는가"를 한 문장으로 설명하는 주석을 작성합니다.
   - 최소 세 가지 테스트를 작성하세요.
     1. **성공 시나리오**: 충돌이 없고 제한을 넘지 않는 두 개의 트랜잭션을 순서대로 enqueue하고, 상태가 올바르게 업데이트되었는지 검증합니다.
     2. **계정 충돌**: 같은 계정을 쓰려는 트랜잭션을 enqueue할 때 `AccountLockError::Conflict`가 반환되는지 확인합니다.
     3. **컴퓨트 초과**: 제한을 초과하는 컴퓨트 사용량을 가진 트랜잭션이 들어오면 `AccountLockError::ComputeLimitExceeded`가 반환되는지 검증합니다.
   - 각 테스트는 `release` 호출이 상태를 제대로 초기화하는지도 함께 확인하세요.
   - 테스트 실행 명령은 `cargo test`입니다.

8. **마무리 루틴 안내**
   - README 마지막에 학습자가 실행해야 할 명령을 아래와 같은 문장으로 안내하세요.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
다음은 충돌 검사를 간략히 표현한 의사 코드입니다. 실제 구현에서는 `HashSet`과 러스트 소유권 규칙을 활용해야 합니다.

```rust
fn check_conflict_example(state: &SlotExecutionState, tx: &TransactionMeta) -> Option<String> {
    for account in &tx.writable_accounts {
        if state.locked_writable.contains(account) || state.locked_readonly.contains(account) {
            return Some(account.clone());
        }
    }
    for account in &tx.readonly_accounts {
        if state.locked_writable.contains(account) {
            return Some(account.clone());
        }
    }
    None
}
```

- 위처럼 읽기/쓰기 계정을 나누어 검사하면 충돌 여부를 명확하게 판단할 수 있습니다.
- `HashSet`은 `contains`가 평균 O(1)이므로, 대기열이 커져도 빠르게 충돌 여부를 확인할 수 있습니다.
- 실제 Solana 러untime도 비슷한 전략으로 동시에 실행 가능한 트랜잭션을 골라냅니다.

---

### 오늘의 TIL (Today I Learned)
- 계정 잠금 상태를 추적하면서 트랜잭션을 안전하게 대기열에 넣는 방법을 설계했습니다.

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
