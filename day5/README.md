# Day 5: 멀티 필터링으로 견고한 메인넷 밸리데이터 멤풀 만들기

**난이도: MEDIUM (중반으로 가는 디딤돌)**

## [전날 과제 요약]
- Day 4에서는 여러 피어 상태를 벡터에 담아 반복문으로 요약 리포트를 작성했습니다.
- `Option`을 다루며 슬롯 정보를 안전하게 표현하는 연습을 했고, 최소 지연 시간을 찾는 작은 알고리즘을 구현했습니다.
- 통합 테스트로 원하는 동작을 먼저 서술하고 구현이 맞춰지도록 TDD 리듬을 익혔습니다.

## [전날 과제로 얻은 역량]
- `Vec<T>` 순회와 패턴 매칭을 조합해 데이터 요약 보고서를 작성할 수 있습니다.
- "데이터 수집 → 가공 → 문자열화" 흐름을 직접 만들어 보며 단계별 사고에 익숙해졌습니다.
- `cargo fmt`, `cargo test` 루틴을 반복해 기본 개발 사이클을 몸에 익혔습니다.

## [오늘 과제 목표]
- 구조체와 열거형을 이용해 멤풀 트랜잭션 도메인을 정의하고, 트레이트를 통해 필터링 로직을 추상화합니다.
- 라이프타임과 참조를 활용해 불필요한 복사를 줄이며, 표준 라이브러리 컬렉션(`BTreeMap`, `Vec`)을 실전처럼 엮어 봅니다.
- 통합 테스트로 시나리오를 먼저 정의하고, 요구사항을 만족하는 구현을 채워 넣는 TDD 루틴을 강화합니다.

## [오늘 과제 설명]
오늘은 실제 메인넷 운영자가 가장 먼저 관리하는 영역인 "멤풀"을 모티브로 미니 프로젝트를 진행합니다. 멤풀은 노드가 아직 블록에 포함되지 않은 트랜잭션을 보관하는 공간입니다. 여러 기준으로 트랜잭션을 필터링하고 계정별로 묶어 요약 통계를 만드는 작업을 수행해 봅시다.

1. **새 프로젝트 만들기**  
   - `cargo new day5_mempool_pipeline --lib` 명령으로 라이브러리 프로젝트를 생성합니다.  
   - `src/lib.rs`에 핵심 로직을 작성하고, `tests` 디렉터리에 통합 테스트(`mempool.rs`)를 생성하세요.

2. **도메인 모델 정의하기 (`src/lib.rs`)**  
   - `PendingTx` 구조체를 선언하고 아래 필드를 추가합니다.  
     - `pub id: String` — 트랜잭션 해시를 문자열로 표현합니다.  
     - `pub account: String` — 트랜잭션을 보낸 계정(공개키)입니다.  
     - `pub fee_micro_lamports: u64` — 지불한 수수료(1 lamport = 10^-6 SOL).  
     - `pub payload_size: u32` — 직렬화된 트랜잭션 크기(Byte 단위).  
     - `pub status: TxStatus` — 아래에서 정의할 상태 열거형입니다.  
   - `TxStatus` 열거형을 정의하고 `Pending`, `Simulated { compute_units: u64 }`, `Rejected { reason: String }` 변형을 선언합니다.  
   - 각 구조체/열거형 위에는 "이 데이터가 왜 필요한지" 어린이도 이해할 수 있게 한국어+영어로 짧은 주석을 작성하세요.

3. **필터 추상화 설계 (`src/lib.rs`)**  
   - `pub trait MempoolFilter`를 선언하고 `fn allow(&self, tx: &PendingTx) -> bool` 메서드를 정의합니다.  
   - `ThresholdFilter` 구조체를 만들어 아래 필드를 추가합니다.  
     - `pub min_fee: u64` — 허용할 최소 수수료.  
     - `pub max_payload: u32` — 허용할 최대 페이로드 크기.  
     - `pub reject_simulation_failures: bool` — 시뮬레이션 실패(`TxStatus::Rejected`)를 거부할지 여부.  
   - `MempoolFilter`를 `ThresholdFilter`가 구현하도록 하고, 각 조건을 만족해야 `true`를 반환하도록 작성하세요. 조건 체크 순서를 주석으로 설명해 주세요.

4. **핵심 기능 구현 (`src/lib.rs`)**  
   아래 함수들을 구현하고 함수 위에 "무엇을 반환하며 왜 필요한지" 서술형 주석을 추가하세요.
   1. `pub fn filter_transactions<'a, F: MempoolFilter>(txs: &'a [PendingTx], filter: &F) -> Vec<&'a PendingTx>`  
      - 주어진 필터를 통과한 트랜잭션 참조만 모아 반환합니다.  
      - 라이프타임 매개변수(`'a`)를 명시적으로 선언해 참조가 안전하게 유지되도록 하세요.  
   2. `pub fn group_by_account(txs: &[PendingTx]) -> BTreeMap<String, Vec<PendingTx>>`  
      - 계정별로 트랜잭션을 묶어 사전식 정렬이 보장되는 `BTreeMap`에 저장합니다.  
      - 각 계정의 벡터는 원본 트랜잭션을 복사해 새로운 벡터로 관리합니다.  
   3. `pub struct AccountStats { pub total_fee: u64, pub total_bytes: u32, pub pending: usize }`를 선언하고,  
      `pub fn compute_account_stats(grouped: &BTreeMap<String, Vec<PendingTx>>) -> BTreeMap<String, AccountStats>` 함수를 구현합니다.  
      - 각 계정의 총 수수료, 총 페이로드 크기, `TxStatus::Pending` 개수를 계산합니다.  
      - 루프 내부에서 `match`를 활용해 상태별로 분기하세요.

5. **통합 테스트 작성 (`tests/mempool.rs`)**  
   - 테스트 파일 맨 위에는 "무엇을 검증하는지" 주석을 남기고, 아래 시나리오를 각각 별도의 테스트 함수로 작성하세요.  
     - 임계값 필터가 수수료, 페이로드, 시뮬레이션 실패 조건을 모두 적용하는지 확인합니다.  
     - `filter_transactions`가 참조를 반환하며 원본 벡터와 수명(`lifetime`)이 연동되는지 검증합니다. (테스트 내에서 벡터를 수정하지 말고, 반환된 참조를 사용해 필드 값을 비교하세요.)  
     - `group_by_account`와 `compute_account_stats`가 계정별 묶음과 통계를 정확히 만드는지 확인합니다. (수수료 합계, 바이트 합계, pending 개수 확인)
   - 모든 테스트는 `cargo test`로 통과해야 하며, 적절한 픽스처 데이터(3~5개 트랜잭션)를 직접 구성하세요.

6. **마무리 루틴**  
   - `cargo fmt`, `cargo clippy`, `cargo test` 순서로 실행하며 코드 품질을 확인하세요.  
   - README 맨 아래 "오늘의 TIL" 섹션에 학습자가 느낀 점을 자유롭게 적을 수 있도록 빈 Bullet을 남겨 주세요.

## [이해를 돕기 위한 예시]
다음 예시는 트랜잭션을 라이프타임과 참조로 처리하는 방법을 보여 줍니다.

```rust
fn high_fee_ids<'a>(txs: &'a [PendingTx], min_fee: u64) -> Vec<&'a String> {
    txs.iter()
        .filter(|tx| tx.fee_micro_lamports >= min_fee)
        .map(|tx| &tx.id)
        .collect()
}
```

- 반복자 체이닝을 사용하면 읽기 쉬운 필터링 파이프라인을 만들 수 있습니다.
- 반환 타입에 참조를 담을 때는 라이프타임 매개변수(`'a`)를 명시해야 컴파일러가 안전성을 이해합니다.
- 오늘 구현할 함수들도 이러한 패턴을 토대로 멤풀에서 원하는 데이터만 뽑아내는 능력을 키우는 것이 목표입니다.

---

### 오늘의 TIL (Today I Learned)
- 
