# Day 10: Reth 스타일 포크 선택기 기초 구현

**난이도: MEDIUM (Reth 코어 엔지니어링)**

## [전날 과제 요약]
- Day 9에서는 제네시스부터 시작하는 헤더 버퍼를 만들고 부모/번호/난이도 검증을 통해 잘못된 헤더를 걸러냈습니다.
- canonical 체인과 해시 인덱스를 유지하며 total difficulty를 누적하는 방식을 연습했습니다.
- 다양한 실패 시나리오 테스트로 방어적 프로그래밍 감각을 익혔습니다.

## [전날 과제로 얻은 역량]
- 체인 상태를 Vec과 HashMap으로 모델링하고 제네시스 초기화를 명확히 처리할 수 있게 되었습니다.
- 헤더 유효성 검사를 에러 enum으로 표현하는 습관이 생겼습니다.
- 테스트를 통해 성공/실패 경로를 명확히 드러내는 방법을 학습했습니다.

## [오늘 과제 목표]
- Reth의 HeaderAccumulator가 포크를 추적해 가장 무거운 체인을 선택하는 아이디어를 체험합니다.
- total difficulty가 더 큰 체인이 들어왔을 때 canonical 체인을 재구성(reorg)하는 절차를 구현합니다.
- 포크 정보와 결과를 명확한 타입으로 표현하는 습관을 기릅니다.

## [오늘 과제 설명]
Reth는 네트워크로부터 다양한 포크를 동시에 받으면서도 가장 무거운(total difficulty가 큰) 체인을 선택해야 합니다. 오늘은 어제 만든 버퍼를 확장해 "포크 선택기"를 만들어봅니다. 다수의 헤더를 그래프 형태로 보관하고, 새로운 헤더가 들어왔을 때 어떤 포크가 canonical이 되어야 하는지 결정하세요.

1. **프로젝트 생성**
   - `cargo new day10_header_fork_choice --lib` 명령을 실행합니다.
   - 라이브러리 코드는 `src/lib.rs`, 통합 테스트는 `tests/fork_choice.rs`에 작성합니다.

2. **헤더와 저장 구조 정의 (`src/lib.rs`)**
   - Day 9와 동일한 필드를 가진 `BlockHeader`를 정의하되, 구조체 위에 "Reth Header 구조와의 대응 관계"를 설명하는 한 줄 주석을 남깁니다.
   - `StoredHeader` 구조체를 추가하고 다음 필드를 포함합니다.
     ```rust
     pub struct StoredHeader {
         pub header: BlockHeader,
         pub parent: Option<String>,
         pub total_difficulty: u128,
     }
     ```
   - `StoredHeader` 정의 위에는 왜 parent와 total difficulty를 함께 저장해야 하는지 설명하는 주석을 작성하세요.

3. **포크 선택기 상태 정의 (`src/lib.rs`)**
   - `use std::collections::HashMap;`를 추가합니다.
   - `HeaderForkChoice` 구조체를 선언하고 아래 필드를 포함하세요.
     ```rust
     pub struct HeaderForkChoice {
         genesis_hash: String,
         canonical: Vec<String>,
         nodes: HashMap<String, StoredHeader>,
     }
     ```
   - 각 필드 위에 Reth의 어떤 구성요소를 모사하는지 설명하는 주석을 작성합니다.
   - `impl HeaderForkChoice` 블록을 열고 다음 메서드를 구현하세요.
     1. `pub fn new(genesis: BlockHeader) -> Self`
        - 제네시스를 `nodes`에 저장하고 canonical 벡터에 제네시스 해시 하나만 담습니다.
        - 함수 위에 "제네시스 해시를 canonical의 기준점으로 고정하는 이유"를 설명하는 주석을 작성하세요.
     2. `pub fn head(&self) -> &BlockHeader`
        - canonical의 마지막 해시에 해당하는 헤더 참조를 반환합니다.
     3. `pub fn canonical_hashes(&self) -> impl Iterator<Item = &String>`
        - canonical 체인을 순회하기 위한 이터레이터를 제공합니다.

4. **에러와 결과 타입 설계 (`src/lib.rs`)**
   - `HeaderInsertError` enum을 만들고 아래 변형을 포함합니다.
     - `DuplicateHash { hash: String }`
     - `UnknownParent { parent_hash: String }`
     - `NumberMismatch { expected: u64, got: u64 }`
   - 각 변형 위에 어떤 상황에서 발생하는지 간단한 주석을 작성합니다.
   - `ReorgOutcome` enum을 만들고 아래 변형을 포함합니다.
     - `NoReorg`
     - `Extended { new_head: BlockHeader }`
     - `Reorganized { new_head: BlockHeader, old_head: BlockHeader, depth: usize }`
   - `ReorgOutcome` 위에는 "학습자가 어떤 상태 변화를 관찰해야 하는지"를 설명하는 주석을 작성하세요.

5. **헤더 삽입 로직 (`src/lib.rs`)**
   - `impl HeaderForkChoice`에 `pub fn try_insert(&mut self, header: BlockHeader) -> Result<ReorgOutcome, HeaderInsertError>`를 구현합니다.
     - 이미 존재하는 해시라면 `DuplicateHash`를 반환합니다.
     - 부모 해시가 없거나 `nodes`에 없는 경우 `UnknownParent`를 반환합니다.
     - 헤더 번호가 부모 번호 + 1이 아니라면 `NumberMismatch`를 반환합니다.
     - 누적 난이도를 `parent.total_difficulty + header.difficulty as u128`으로 계산하고 `nodes`에 저장합니다.
     - `rebuild_canonical_if_needed`라는 비공개 헬퍼 함수를 호출해 canonical 체인을 재구성하세요.
       - 이 함수는 새 헤더의 total difficulty가 현재 canonical head보다 큰 경우에만 재구성을 수행합니다.
       - 재구성 시 부모 해시를 따라 제네시스까지 역추적한 뒤, 순서를 뒤집어 canonical 벡터를 갱신합니다.
       - 재구성 결과에 따라 `ReorgOutcome`을 결정합니다 (`Extended`는 기존 head의 자식으로 붙은 경우, `Reorganized`는 다른 포크로 이동한 경우, 그렇지 않으면 `NoReorg`).
     - `rebuild_canonical_if_needed` 내부에서는 기존 canonical 벡터와 새 벡터의 공통 prefix 길이를 이용해 reorg 깊이를 계산하세요.

6. **테스트 작성 (`tests/fork_choice.rs`)**
   - 파일 첫 줄에 "테스트가 보장하려는 시나리오"를 설명하는 한 문장 주석을 작성하세요.
   - 다음 세 가지 테스트를 작성합니다.
     1. **단순 확장**: 제네시스 뒤로 두 개의 연속된 헤더를 추가했을 때 `ReorgOutcome::Extended`가 반환되고 canonical 해시 순서가 올바른지 검증합니다.
     2. **포크 삽입과 유지**: 총 난이도가 낮은 포크 헤더를 추가해도 canonical이 바뀌지 않는 상황을 만들고 `ReorgOutcome::NoReorg`임을 확인합니다.
     3. **높은 난이도 포크로 인한 재구성**: 더 높은 난이도의 포크 헤더를 추가해 canonical이 다른 경로로 이동하는 것을 확인하고, reorg 깊이가 기대값과 같은지 검사합니다.
   - 각 테스트는 canonical 길이, head의 hash/number, `nodes`에 저장된 total difficulty 중 하나 이상을 반드시 검증하세요.

7. **마무리 루틴 안내**
   - README 마지막에 학습자가 실행해야 할 명령을 아래 순서대로 안내합니다.
     - `cargo fmt`
     - `cargo clippy`
     - `cargo test`

## [이해를 돕기 위한 예시]
아래는 canonical을 재구성하는 핵심 로직 예시입니다. 각 단계에 대한 설명을 주석으로 남겨 학습자가 쉽게 따라오도록 해 주세요.

```rust
fn rebuild_canonical_if_needed(
    nodes: &HashMap<String, StoredHeader>,
    canonical: &mut Vec<String>,
    new_hash: &str,
) -> (Vec<String>, usize) {
    // 1. 현재 head의 누적 난이도를 가져옵니다.
    let current_head_hash = canonical.last().expect("canonical should never be empty");
    let current_head = nodes.get(current_head_hash).expect("head must exist");

    // 2. 새 헤더의 누적 난이도가 더 작거나 같다면 canonical을 유지합니다.
    let candidate = nodes.get(new_hash).expect("new header must exist");
    if candidate.total_difficulty <= current_head.total_difficulty {
        return (canonical.clone(), 0);
    }

    // 3. 부모 링크를 따라 제네시스까지 역추적합니다.
    let mut path = Vec::new();
    let mut cursor = Some(new_hash.to_string());
    while let Some(hash) = cursor {
        path.push(hash.clone());
        cursor = nodes
            .get(&hash)
            .and_then(|node| node.parent.clone());
    }

    // 4. 역추적 결과를 뒤집어 canonical 순서로 정렬합니다.
    path.reverse();

    // 5. 기존 canonical과의 공통 prefix 길이를 계산해 reorg 깊이를 알 수 있습니다.
    let mut prefix = 0;
    while prefix < canonical.len()
        && prefix < path.len()
        && canonical[prefix] == path[prefix]
    {
        prefix += 1;
    }
    let reorg_depth = canonical.len().saturating_sub(prefix);

    // 6. canonical을 새 경로로 교체하고 reorg 깊이를 반환합니다.
    *canonical = path;
    (canonical.clone(), reorg_depth)
}
```

- 이 함수는 Reth의 `HeaderAccumulator::reorg_canonical` 흐름을 단순화한 것입니다.
- 실제 Reth는 DB 트랜잭션과 stage pipeline을 통해 canonical을 업데이트하지만, 우리는 메모리 구조만으로 개념을 체험합니다.
- reorg 깊이를 계산하면 얼마나 많은 블록이 롤백되었는지 추적할 수 있어 Stage 이후 구성요소와의 인터페이스를 설계하기 쉬워집니다.

---

### 오늘의 TIL (Today I Learned)
- 포크 선택기의 기본 개념과 canonical 재구성 절차를 구현해 보았습니다.

> 마무리 전: `cargo fmt` → `cargo clippy` → `cargo test`
