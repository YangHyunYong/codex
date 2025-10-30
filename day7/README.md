# Day 7: 슬롯 플래너로 천천히 묶음 만들기

**난이도: EASY (입문자 친화형 연습)**

## [전날 과제 요약]
- Day 6에서는 수수료 점수 기반 우선순위 큐를 만들고, 높은 점수의 트랜잭션을 먼저 꺼내는 로직을 연습했습니다.
- `BinaryHeap`과 커스텀 정렬 기준을 직접 다루면서 자료구조를 안전하게 조작하는 법을 익혔습니다.
- 테스트 코드를 활용해 정렬 규칙이 잘 지켜지는지 확인하는 TDD 흐름을 경험했습니다.

## [전날 과제로 얻은 역량]
- 정렬된 데이터를 반복해서 꺼내며 처리 순서를 제어할 수 있습니다.
- 구조체와 열거형을 이용해 명확한 데이터 모델을 설계하는 감각을 키웠습니다.
- 작은 단위의 테스트부터 작성해 요구사항을 안전하게 구현하는 습관을 들였습니다.

## [오늘 과제 목표]
- 블록에 넣을 트랜잭션 묶음을 고르는 **아주 단순한 `SlotPlanner`** 를 만들어 봅니다.
- 합계를 계산하면서 제한을 넘기지 않는 **그리디 선택 패턴**을 연습합니다.
- `BinaryHeap`을 이용해 **수수료가 높은 순으로 정렬**된 입력을 만들어 봅니다.
- 실패 없이 통과하는 행복한 시나리오부터 테스트를 작성해 **테스트 주도 학습**을 이어 갑니다.

## [오늘 과제 설명]
Day 6에서 얻은 정렬된 트랜잭션 목록을 입력으로 받고, 블록 제약을 넘지 않는 범위에서 차례대로 담아 보는 가벼운 연습입니다. 복잡한 에러 타입이나 롤백 없이, "담을 수 있으면 담고 아니면 건너뛴다"라는 단순한 규칙만 지키면 됩니다. 아래 순서를 천천히 따라오세요.

1. **프로젝트 생성**
   - `cargo new day7_slot_planner --lib` 명령으로 새 라이브러리 프로젝트를 만듭니다.
   - 최종 코드 파일은 `src/lib.rs`, 테스트 파일은 `tests/planner.rs`를 사용합니다.

2. **데이터 구조 만들기 (`src/lib.rs`)**
   - Day 6에서 사용했던 필드를 복습한다는 마음으로 `MempoolEntry` 구조체를 정의하세요.
     ```rust
     pub struct MempoolEntry {
         pub compute_units: u32,
         pub fee_micro_lamports: u64,
     }
     ```
   - 블록 제한을 담는 `BlockConstraint` 구조체를 추가합니다.
     ```rust
     pub struct BlockConstraint {
         pub max_compute_units: u32,
         pub max_transactions: usize,
     }
     ```
   - 실제로 블록에 담긴 결과를 표현할 `PlannedBundle` 구조체를 만듭니다.
     ```rust
     pub struct PlannedBundle {
         pub entries: Vec<MempoolEntry>,
         pub total_compute_units: u32,
         pub total_fee_micro_lamports: u64,
     }
     ```
   - 각 구조체 위에는 **"이 구조체는 무엇을 담나요?"** 라는 질문에 답하는 한 문장짜리 한국어 주석을 적어 주세요.

3. **BinaryHeap으로 입력 정렬하기 (`src/lib.rs`)**
   - 수수료가 높은 항목을 먼저 처리하면 블록 수익이 극대화됩니다. `BinaryHeap`을 활용해 정렬된 벡터를 만드는 작은 도우미 함수를 추가하세요.
   - 아래 시그니처를 참고해 `pub fn drain_sorted_by_fee(heap: &mut BinaryHeap<MempoolEntry>) -> Vec<MempoolEntry>` 함수를 만듭니다.
     - `BinaryHeap`에서 `pop`을 반복해 벡터에 push하면 내림차순으로 정렬됩니다.
     - `BinaryHeap`을 사용하려면 `std::collections::BinaryHeap`을 `use`하고, `MempoolEntry`에 `Ord`, `PartialOrd`, `Eq`, `PartialEq` 구현을 추가하세요. (필요한 필드만 비교하면 됩니다.)
   - Day 6에서 만든 항목을 이 도우미 함수에 넣으면 **자연스럽게 정렬된 벡터**를 얻을 수 있다는 한 줄짜리 주석을 함수 위에 남겨 주세요.

4. **슬롯 플래너 기본 뼈대 (`src/lib.rs`)**
   - `SlotPlanner` 구조체를 선언하고, 아래와 같이 두 필드를 준비합니다.
     ```rust
     pub struct SlotPlanner {
         constraint: BlockConstraint,
         current_bundle: PlannedBundle,
     }
     ```
   - `impl SlotPlanner` 안에 다음 생성 함수를 구현합니다.
     - `pub fn new(constraint: BlockConstraint) -> Self`
       - 비어 있는 번들을 만들고, 제한 정보만 저장합니다.
   - "왜 생성자에서 번들을 비워 두는가?"를 설명하는 짧은 주석을 함수 위에 작성하세요.

5. **핵심 메서드 구현 (`src/lib.rs`)**
   - `SlotPlanner`에 아래 메서드를 채워 넣습니다.
     1. `pub fn try_add(&mut self, entry: MempoolEntry) -> bool`
        - 새 항목을 추가했을 때 **컴퓨트 합**과 **트랜잭션 수**가 제한을 넘지 않으면 번들에 push하고 `true`를 반환합니다.
        - 제한을 넘으면 아무 것도 하지 않고 `false`를 반환합니다.
     2. `pub fn finalize(self) -> PlannedBundle`
        - 지금까지 쌓아 둔 번들을 그대로 돌려줍니다.
   - 로직을 단순하게 유지하기 위해 **도우미 함수 `fn can_add(&self, entry: &MempoolEntry) -> bool`** 를 추가해도 좋습니다. 이 함수는 합계를 계산하고 비교하는 일을 담당합니다.

6. **잔여 용량 계산 (`src/lib.rs`)**
   - `impl PlannedBundle` 블록 안에 `pub fn remaining_capacity(&self, constraint: &BlockConstraint) -> (u32, usize)` 함수를 작성합니다.
   - 현재 사용량을 제한에서 빼서 남은 컴퓨트 유닛과 남은 트랜잭션 수를 `(남은_컴퓨트, 남은_트랜잭션)` 형태로 반환합니다.
   - 빼기 연산에서 음수가 나오지 않도록 `saturating_sub`를 활용하면 안전합니다.

7. **테스트 작성 (`tests/planner.rs`)**
   - 파일 맨 위에 "이 테스트는 무엇을 검증하는가"를 한 문장으로 정리한 주석을 남겨 주세요.
   - 두 가지 시나리오를 각각 독립된 테스트 함수로 작성합니다.
     1. 제한을 넘는 항목이 들어왔을 때 `try_add`가 `false`를 돌려주고, 번들 상태가 그대로 유지되는지 확인합니다.
     2. `BinaryHeap`으로 만든 정렬된 항목을 순서대로 추가했을 때 `finalize`와 `remaining_capacity`가 정확한 합계를 반환하는지 확인합니다.
        - 테스트 안에서 작은 `BinaryHeap`을 만들어 `drain_sorted_by_fee` 도우미 함수가 원하는 순서를 주는지 검증해 보세요.
   - 테스트 실행 명령은 프로젝트 루트(`day7_slot_planner`)에서 `cargo test`입니다.

8. **마무리 루틴 안내**
   - 학습자가 스스로 점검하도록 README 맨 아래에 아래 순서를 안내하는 문장을 남겨 주세요.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
아래 코드는 제한을 체크하는 가장 단순한 형태의 계산입니다. 그대로 복사하기보다는 "어떤 계산이 필요한가"를 이해하는 데 활용하세요.

```rust
fn can_insert(current_compute: u32, current_count: usize, new_entry: &MempoolEntry, constraint: &BlockConstraint) -> bool {
    let next_compute = current_compute + new_entry.compute_units;
    let next_count = current_count + 1;
    next_compute <= constraint.max_compute_units && next_count <= constraint.max_transactions
}
```

- `next_compute`와 `next_count`를 미리 계산한 다음 제한과 비교하면 됩니다.
- 제약을 넘지 않으면 항목을 추가하고, 넘으면 건너뛰면 됩니다.
- 실제 블록체인에서도 이런 간단한 합계 검사를 반복해서 수행합니다.

---

### 오늘의 TIL (Today I Learned)
- 

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
