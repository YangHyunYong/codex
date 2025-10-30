# Day 9: Reth 스타일 헤더 버퍼 기초 구현

**난이도: EASY (Reth 코어 입문)**

## [전날 과제 요약]
- Day 8에서는 슬롯 실행 대기열을 만들고, 계정 잠금 규칙을 이용해 충돌 없는 트랜잭션만 큐에 넣었습니다.
- HashSet과 VecDeque를 활용해 상태를 추적했고, 컴퓨트 한도 초과와 계정 충돌을 명확한 에러 타입으로 표현했습니다.
- release 로직과 테스트를 통해 상태 업데이트가 안정적으로 작동함을 확인했습니다.

## [전날 과제로 얻은 역량]
- 실행 큐 상태를 구조체로 모델링하고, 보조 함수로 로직을 분리하는 습관을 익혔습니다.
- 테스트 우선으로 성공/실패 경로를 설계하는 연습을 이어 갔습니다.
- 리소스 제한과 동시성 충돌을 명확한 에러로 드러내는 법을 배웠습니다.

## [오늘 과제 목표]
- Reth의 "HeaderAccumulator" 아이디어를 축소해 헤더를 순차적으로 적재하는 기초 로직을 이해합니다.
- 부모 해시와 블록 번호를 검증해 잘못된 헤더가 체인에 들어오는 것을 막습니다.
- 난이도(total difficulty)를 누적해 가장 긴 체인을 비교하는 기반 개념을 체험합니다.

## [오늘 과제 설명]
Reth는 실행 클라이언트지만, 헤더 동기화와 Stage 파이프라인을 통해 올바른 체인을 선택합니다. 오늘은 그중 가장 초보적인 구성요소인 "헤더 버퍼"를 직접 만들어봅니다. 실제 Reth에서는 수많은 Stage와 DB, Task가 얽혀 있지만 우리는 매우 단순화된 버전을 구현합니다.

1. **프로젝트 생성**
   - `cargo new day9_reth_header_buffer --lib` 명령을 실행합니다.
   - 모든 코드는 `src/lib.rs`에, 테스트는 `tests/header_buffer.rs`에 작성합니다.

2. **데이터 구조 정의 (`src/lib.rs`)**
   - `#[derive(Clone, Debug, PartialEq, Eq)]`를 붙인 `BlockHeader` 구조체를 선언하고 아래 필드를 포함하세요.
     ```rust
     pub struct BlockHeader {
         pub number: u64,
         pub hash: String,
         pub parent_hash: String,
         pub difficulty: u64,
     }
     ```
   - 구조체 위에는 "이 구조체가 Reth에서 어떤 정보를 흉내 내는지"를 설명하는 한 줄짜리 한국어 주석을 작성하세요.

3. **버퍼 상태 정의 (`src/lib.rs`)**
   - `use std::collections::HashMap;`을 추가합니다.
   - `HeaderBuffer` 구조체를 만들고 아래 필드를 갖도록 합니다.
     ```rust
     pub struct HeaderBuffer {
         canonical: Vec<BlockHeader>,
         index_by_hash: HashMap<String, usize>,
         total_difficulty: u128,
     }
     ```
   - 각 필드 위에 "Reth에서 비슷한 역할을 하는 구성요소"를 설명하는 주석을 적어 주세요.
   - `impl HeaderBuffer` 블록을 열고 다음 메서드를 구현합니다.
     1. `pub fn new(genesis: BlockHeader) -> Self`
        - 제네시스를 canonical 체인에 첫 항목으로 넣고, 해시 인덱스를 채우며, total difficulty를 초기화합니다.
        - 함수 위에 "제네시스가 왜 특별 취급되는가"를 설명하는 주석을 적습니다.
     2. `pub fn head(&self) -> Option<&BlockHeader>`: canonical 체인의 마지막 헤더를 반환합니다.
     3. `pub fn total_difficulty(&self) -> u128`: 누적된 total difficulty를 돌려줍니다.

4. **헤더 추가 로직 (`src/lib.rs`)**
   - `HeaderInsertError`라는 `enum`을 만들고 아래 변형을 포함합니다.
     - `ParentNotFound { parent_hash: String }`
     - `NumberMismatch { expected: u64, got: u64 }`
     - `DuplicateHash { hash: String }`
   - 각 변형 위에 "어떤 상황에서 발생하는가"를 설명하는 간단한 주석을 작성하세요.
   - `impl HeaderBuffer`에 `pub fn try_append(&mut self, header: BlockHeader) -> Result<(), HeaderInsertError>`를 구현합니다.
     - 이미 존재하는 해시라면 `DuplicateHash`를 반환합니다.
     - 부모 해시가 `index_by_hash`에 없다면 `ParentNotFound`를 반환합니다.
     - 헤더 번호가 현재 head 번호 + 1이 아니라면 `NumberMismatch`를 반환합니다.
     - 모든 검증을 통과하면 canonical 벡터와 인덱스를 갱신하고, total difficulty에 새 헤더의 difficulty를 더합니다.

5. **테스트 작성 (`tests/header_buffer.rs`)**
   - 파일 첫 줄에 "이 테스트가 무엇을 보장하는지" 한 문장짜리 주석을 작성합니다.
   - 세 가지 시나리오를 검증하는 테스트를 작성하세요.
     1. **정상 연결**: 제네시스 후 연속된 두 개의 헤더를 추가하고 head/total difficulty가 올바른지 확인합니다.
     2. **부모 없음**: 부모 해시가 등록되지 않은 헤더를 추가할 때 `HeaderInsertError::ParentNotFound`가 나는지 확인합니다.
     3. **번호 불일치**: 부모는 맞지만 번호가 건너뛰어진 헤더에 대해 `HeaderInsertError::NumberMismatch`가 반환되는지 확인합니다.
   - 각 테스트에서 `try_append`의 결과뿐 아니라 canonical 길이나 인덱스 상태도 적어도 한 가지씩 검증해 주세요.

6. **마무리 루틴 안내**
   - README 마지막에 학습자가 실행해야 할 명령을 아래처럼 안내하세요.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
다음은 헤더를 추가하기 전 검증 순서를 요약한 의사 코드입니다.

```rust
fn validate(header: &BlockHeader, head: &BlockHeader, index: &HashMap<String, usize>) -> Result<(), HeaderInsertError> {
    if index.contains_key(&header.hash) {
        return Err(HeaderInsertError::DuplicateHash { hash: header.hash.clone() });
    }
    if !index.contains_key(&header.parent_hash) {
        return Err(HeaderInsertError::ParentNotFound { parent_hash: header.parent_hash.clone() });
    }
    if header.number != head.number + 1 {
        return Err(HeaderInsertError::NumberMismatch { expected: head.number + 1, got: header.number });
    }
    Ok(())
}
```

- Reth의 헤더 Stage 역시 부모 해시 검증, 번호 검증, total difficulty 계산을 통해 체인을 유지합니다.
- 우리는 데이터베이스 대신 메모리 벡터/맵을 사용하지만, 로직의 핵심은 동일합니다.
- 이 버퍼가 쌓여야 나중에 본격적인 Stage 파이프라인과 포크 선택을 구현할 수 있습니다.

---

### 오늘의 TIL (Today I Learned)
- 헤더 동기화의 기본 규칙(부모/번호/난이도)을 손으로 구현해 보았습니다.

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
