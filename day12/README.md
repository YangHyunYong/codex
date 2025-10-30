# Day 12: Reth 파이프라인 Stage 드라이버 설계

**난이도: MEDIUM (동기화 파이프라인 추상화)**

## [전날 과제 요약]
- Day 11에서는 계정별/전역 우선순위 큐를 결합한 트랜잭션 풀을 구현했습니다.
- lazy eviction으로 BinaryHeap을 관리하며, sender 별 nonce 순서를 깨지 않는 삽입/배치 pop 로직을 완성했습니다.
- 우선순위 기반 스케줄링 테스트를 통해 Reth TxPool의 핵심 개념을 체득했습니다.

## [전날 과제로 얻은 역량]
- 계정 단위 상태와 전역 스케줄러를 동시에 다루는 자료구조 감각을 길렀습니다.
- 트랜잭션 큐의 일관성 검증 테스트를 직접 설계하고 실행하는 TDD 루틴을 연습했습니다.
- Reth TxPool의 우선순위 키와 lazy eviction 철학을 이해했습니다.

## [오늘 과제 목표]
- Reth 파이프라인(Stage Pipeline)의 실행/롤백/검증 흐름을 단순화해 체험합니다.
- Stage 추상화가 왜 명확한 의도를 가진 메서드 집합으로 구성되는지 이해합니다.
- PipelineDriver가 Stage 간 의존성을 관리하며 상태를 전파하는 방식을 모사합니다.

## [오늘 과제 설명]
Day 12의 목표는 Reth의 동기화 파이프라인이 Stage 추상화를 통해 블록 데이터를 단계적으로 처리하는 방식을 연습하는 것입니다. 각 Stage는 `execute`/`unwind`/`validate` 같은 명시적 메서드로 구성되어 있고, Driver는 Stage 상태를 추적하며 반복적으로 호출합니다. 오늘은 간소화된 Stage 트레이트와 Driver를 Rust로 작성하고, 테스트를 통해 단계적 실행 및 롤백 시나리오를 검증하세요. 메서드를 설계할 때는 "이 메서드가 어떤 상태 전이를 담당하는지", "Driver와 어떤 계약을 맺는지"를 README와 주석에서 명시적으로 설명해야 합니다.

1. **프로젝트 생성**
   - `cargo new day12_pipeline_stage --lib` 명령을 실행합니다.
   - 주요 로직은 `src/lib.rs`, 통합 테스트는 `tests/pipeline.rs`에 작성합니다.

2. **Stage 식별자와 상태 모델 (`src/lib.rs`)**
   - `StageId` enum을 정의하고 `Headers`, `Bodies`, `Execution` 변형을 포함합니다.
     - 각 변형 위에 "Reth Stage에서 무엇을 담당하는지"를 요약하는 문장을 주석으로 작성합니다.
   - `StageState` 구조체를 정의하고 아래 필드를 추가합니다.
     ```rust
     pub struct StageState {
         pub last_block: u64,
         pub checkpoint_tag: String,
     }
     ```
   - 구조체 주석에는 `last_block`이 Stage가 어디까지 처리했는지 추적하고 `checkpoint_tag`가 Driver 재시작 시 참조하는 이유를 서술하세요.

3. **Stage 트레이트 설계 (`src/lib.rs`)**
   - `pub trait Stage`를 정의하고 다음 메서드를 선언합니다.
     1. `fn id(&self) -> StageId` — Driver가 Stage를 식별하고 의존성 순서를 구성하는 데 사용됨을 주석으로 설명합니다.
     2. `fn execute(&mut self, input: StageInput) -> StageResult` — 상태를 전진시키는 메서드로, 주석에 "블록 데이터 적용" 의도를 설명합니다.
     3. `fn unwind(&mut self, input: UnwindInput) -> StageResult` — 롤백 시 어떤 책임을 지는지 주석을 덧붙입니다.
     4. `fn validate(&self, target_block: u64) -> StageValidation` — 실행 후 Driver가 Stage의 정합성을 확인하기 위해 호출한다는 의도를 적습니다.
   - 메서드 시그니처에서 사용하는 `StageInput`, `UnwindInput`, `StageResult`, `StageValidation` 타입을 정의하세요.
     - `StageInput`은 `target_block`과 `state: StageState` 필드를 가지는 구조체입니다.
     - `UnwindInput`은 `target_block`과 `checkpoint: StageState` 필드를 포함합니다.
     - `StageResult`는 `Progress { stage: StageId, done_block: u64 }`와 `Idle { stage: StageId }` 변형을 갖는 enum으로 만듭니다. 각 변형 주석에 Driver와의 계약을 설명하세요.
     - `StageValidation`은 `Ok`와 `OutOfSync { expected: u64, actual: u64 }` 변형을 갖는 enum으로 정의하고, 불일치 시 Driver가 어떤 조치를 취해야 하는지 주석을 남깁니다.
   - 각 구조체/enum/메서드 위주석을 통해 메서드의 의도를 구체적으로 기술하세요. 예: `execute` 앞에는 "새 블록 데이터를 적용해 StageState.last_block을 target_block까지 끌어올린다"와 같은 설명을 남깁니다.

4. **Dummy Stage 구현 (`src/lib.rs`)**
   - `struct DummyStage`를 정의하고 내부에 `id: StageId`와 `state: StageState`를 보관하세요.
   - `DummyStage`의 `execute`는 `StageInput`의 `target_block`까지 `last_block`을 증가시키는 방식으로 동작시키고, `unwind`는 `target_block` 아래로 `last_block`을 되돌립니다.
     - 각 메서드 내부에 "왜 이렇게 상태를 조작하는지"를 설명하는 주석을 남깁니다.
   - `validate`는 내부 `state.last_block`이 `target_block`과 다르면 `StageValidation::OutOfSync`를 반환하게 하세요.

5. **PipelineDriver 설계 (`src/lib.rs`)**
   - `pub struct PipelineDriver<S: Stage> { stages: Vec<S>, history: Vec<StageResult> }`를 정의합니다.
   - 다음 메서드를 구현하세요. 메서드별로 Driver가 Stage와 어떤 계약을 맺는지 상세 주석을 작성합니다.
     1. `pub fn new(stages: Vec<S>) -> Self` — Driver가 Stage 순서를 고정하는 이유를 설명합니다.
     2. `pub fn run_to_block(&mut self, target: u64)` — 각 Stage의 `execute`를 호출하고, `StageResult::Progress`가 나오면 `history`에 기록합니다. `Idle`이 나오면 해당 Stage는 target에 이미 도달했다고 간주한다는 설명을 남깁니다.
     3. `pub fn unwind_to_block(&mut self, target: u64)` — 역순으로 Stage를 순회하며 `unwind`를 호출합니다. `history`에서 관련된 `Progress` 기록을 찾아 `done_block`이 target보다 크다면 되돌린다고 주석을 작성하세요.
     4. `pub fn validate(&self, target: u64) -> Vec<(StageId, StageValidation)>` — Driver가 전체 Stage 정합성을 점검하는 이유와 반환값 형식을 설명합니다.

6. **테스트 작성 (`tests/pipeline.rs`)**
   - 파일 첫 줄에 "Stage execute/unwind/validate 계약 검증"이라는 문장을 주석으로 남깁니다.
   - 다음 시나리오를 테스트합니다.
     1. **전진 실행**: `run_to_block` 호출 후 모든 Stage의 `last_block`이 target에 도달하는지 검증합니다.
     2. **부분 롤백**: 일부 Stage의 `last_block`이 target보다 크면 `unwind_to_block`이 올바르게 state를 되돌리는지 확인합니다.
     3. **검증 실패 감지**: 한 Stage의 내부 상태를 인위적으로 어긋나게 만든 후 `validate`가 `OutOfSync`를 반환하는지 테스트합니다.
   - 각 테스트에서 Driver가 기록한 `history`를 확인하거나 Stage 내부 state를 직접 검사해 계약이 잘 지켜졌는지 검증하세요.

7. **마무리 루틴 안내**
   - README 마지막에 학습자가 실행해야 할 명령을 아래 순서대로 안내합니다.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
아래는 Driver가 Stage에 입력을 전달할 때 상태와 목표 블록을 분리하는 이유를 보여주는 예시 코드입니다. 실제 구현 시 주석을 추가해 설계 의도를 명확히 하세요.

```rust
fn plan_execute<S: Stage>(stage: &S, target: u64, state: StageState) -> StageInput {
    StageInput {
        target_block: target,
        state,
    }
}
```

- Reth Pipeline은 Stage 실행 전후로 상태 체크포인트를 저장해 재시작 시 손쉽게 복원합니다.
- 메서드 시그니처로 의도를 표현하면 테스트가 Stage 간 계약을 검증할 수 있고, Driver 변경에도 안정적으로 작동합니다.
- Stage별 `execute`/`unwind`/`validate` 구분은 재동기화나 포크 전환 시 매우 중요한 계약입니다.

---

### 오늘의 TIL (Today I Learned)
- 파이프라인 Stage가 명확한 계약을 가진 메서드 집합으로 구성되어야 Reth 같은 시스템이 안정적으로 동작한다는 점을 학습합니다.

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
