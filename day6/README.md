# Day 6: 수수료 우선순위 스케줄러로 트랜잭션 대기열 다듬기

**난이도: EASY (부드러운 속도 조절)**

## [전날 과제 요약]
- Day 5에서는 멤풀 도메인을 구조체와 열거형으로 모델링하고, 필터링과 계정별 통계를 계산했습니다.
- 라이프타임이 붙은 참조를 반환하는 함수와 `BTreeMap`을 다루며 데이터 흐름을 연습했습니다.
- 통합 테스트를 작성해 TDD 루틴을 유지하며 요구사항을 검증했습니다.

## [전날 과제로 얻은 역량]
- 트랜잭션 도메인을 Rust 타입으로 탄탄하게 표현하고, 필터 트레이트를 활용해 조건을 추상화할 수 있습니다.
- 표준 컬렉션을 조합해 계정별 그룹화 및 요약 통계를 계산하는 사고방식을 익혔습니다.
- 테스트부터 작성하는 TDD 사이클을 반복하며 Rust 개발 루틴의 자신감을 얻었습니다.

## [오늘 과제 목표]
- `BinaryHeap`과 커스텀 정렬을 사용해 수수료 우선순위 큐를 구축합니다.
- `Result`와 에러 타입을 정의해 잘못된 입력을 정중하게 처리하는 연습을 합니다.
- 간단한 벤치마크 스타일의 테스트로, 작은 반복문을 통해 시간 복잡도를 체감합니다.

## [오늘 과제 설명]
오늘은 Day 5에서 만든 멤풀 데이터를 더 잘 다듬기 위해 "수수료 우선순위 스케줄러"를 만듭니다. 높은 수수료 트랜잭션을 먼저 꺼내는 큐를 구현하고, 의도치 않은 입력은 에러로 막아 봅시다. 현실 세계에서는 이런 스케줄러가 블록 생성 전 대기열을 깔끔하게 정렬해 줍니다.

1. **프로젝트 생성**
   - `cargo new day6_fee_scheduler --lib` 명령으로 라이브러리 프로젝트를 만듭니다.
   - `src/lib.rs`에 핵심 로직을 작성하고, `tests/scheduler.rs` 파일을 생성해 통합 테스트를 작성하세요.

2. **도메인 타입 설계 (`src/lib.rs`)**
   - `TxClass` 열거형을 정의하고 `HighPriority`, `Standard`, `LowPriority` 변형을 추가합니다. 각 변형 위에는 "왜 존재하는지" 두 줄짜리 주석(한국어+영어)을 작성하세요.
   - `MempoolEntry` 구조체를 만들어 다음 필드를 추가합니다.
     - `pub id: String`
     - `pub fee_micro_lamports: u64`
     - `pub compute_units: u32`
     - `pub class: TxClass`
   - 구조체 위에도 어린이도 이해할 수 있는 짧은 주석을 남겨 주세요.

3. **에러 타입과 스케줄러 구조 (`src/lib.rs`)**
   - `#[derive(Debug, thiserror::Error)]`를 이용해 `SchedulerError` 열거형을 정의하고 아래 변형을 추가합니다.
     - `#[error("fee must be greater than zero")] FeeTooLow`
     - `#[error("compute units must be within 200_000")] ComputeUnitsOutOfRange`
   - `PriorityScheduler` 구조체를 선언하고 내부에 `BinaryHeap<ScheduledTx>`를 보관하세요. `ScheduledTx`는 다음 필드를 가진 새 구조체입니다.
     - `pub entry: MempoolEntry`
     - `pub score: u128`
   - `ScheduledTx`에 `Ord`, `PartialOrd`, `Eq`, `PartialEq`를 구현해 `score`가 높은 항목이 먼저 나오도록 하세요. 동점일 때는 `entry.id`의 사전식 역순(큰 값 우선)으로 정렬합니다. 구현 전략을 주석으로 남기세요.

4. **핵심 메서드 구현 (`src/lib.rs`)**
   - `impl PriorityScheduler` 블록을 만들고 아래 메서드를 구현하세요. 각 메서드 위에는 왜 필요한지 설명하는 주석을 작성합니다.
     1. `pub fn new() -> Self`
     2. `pub fn push(&mut self, entry: MempoolEntry) -> Result<(), SchedulerError>`
        - `fee_micro_lamports == 0`이면 `SchedulerError::FeeTooLow`를 반환합니다.
        - `compute_units > 200_000`이면 `SchedulerError::ComputeUnitsOutOfRange`를 반환합니다.
        - `score`는 `fee_micro_lamports as u128 * 1_000 + (200_000 - compute_units as u128)`로 계산합니다.
     3. `pub fn pop(&mut self) -> Option<MempoolEntry>`
     4. `pub fn len(&self) -> usize`
     5. `pub fn is_empty(&self) -> bool`

5. **테스트 작성 (`tests/scheduler.rs`)**
   - 파일 상단에 테스트 목적을 소개하는 주석을 작성합니다.
   - 아래 시나리오를 각각 별도의 테스트 함수로 작성하세요.
     - `push`가 유효성 검사를 수행하고 에러를 반환하는지 확인합니다.
     - 여러 엔트리를 넣은 뒤 `pop`을 반복해 수수료 높은 순서→동점 시 `id` 역순으로 나오는지 검증합니다.
     - 1,000개의 임의 엔트리를 빠르게 넣고 빼며 큐가 비워지는지 검사하는 "작은 벤치마크" 테스트를 작성하세요. (`for` 루프와 단정문만 사용)
   - 모든 테스트는 `cargo test`로 통과해야 합니다.

6. **마무리 루틴**
   - `cargo fmt`, `cargo clippy`, `cargo test` 순서로 실행하도록 안내문을 남기세요.
   - README 맨 아래 "오늘의 TIL" 섹션에 학습자가 느낀 점을 자유롭게 적을 수 있도록 빈 Bullet을 남겨 주세요.

## [이해를 돕기 위한 예시]
`BinaryHeap`에서 높은 점수를 먼저 뽑는 방법 예시는 다음과 같습니다.

```rust
use std::collections::BinaryHeap;

#[derive(Eq, PartialEq)]
struct Item {
    score: u64,
}

impl Ord for Item {
    fn cmp(&self, other: &Self) -> std::cmp::Ordering {
        self.score.cmp(&other.score)
    }
}

impl PartialOrd for Item {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        Some(self.cmp(other))
    }
}

fn main() {
    let mut heap = BinaryHeap::new();
    heap.push(Item { score: 10 });
    heap.push(Item { score: 30 });
    assert_eq!(heap.pop().unwrap().score, 30);
}
```

- `BinaryHeap`은 기본적으로 가장 큰 값이 먼저 나옵니다.
- `Ord` 구현에서 `score`를 비교하면 점수가 높은 요소가 먼저 꺼내집니다.
- 오늘 과제에서는 점수에 수수료와 계산 단위를 합쳐 현실감을 높입니다.

---

### 오늘의 TIL (Today I Learned)
-
