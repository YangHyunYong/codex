# HINT Day 7

## [힌트]
- `MempoolEntry`에 `Ord`를 구현할 때는 `fee_micro_lamports` 값만 비교하도록 `derive` 대신 직접 구현하거나 `#[derive(Ord, PartialOrd, Eq, PartialEq, Clone)]`를 사용하고 `BinaryHeap`이 최대 힙이라는 점을 기억하세요.
- `BinaryHeap`에서 정렬된 벡터를 뽑을 때는 `while let Some(entry) = heap.pop()` 패턴을 쓰면 안전하고 읽기 좋습니다.
- `try_add`를 구현할 때는 "먼저 계산 → 비교 → push" 순서를 항상 동일하게 유지하세요.
- 현재 번들의 합계를 구조체 안에 저장해 두면, 매번 `entries.iter().sum()`을 호출하지 않아도 됩니다.
- `remaining_capacity`는 `constraint.max_*` 값에서 현재 사용량을 빼는 단순한 계산입니다. `saturating_sub`를 쓰면 0보다 작은 결과를 자동으로 0으로 만들어 줍니다.

## [참고자료]
- Solana Docs - Transaction Processing Overview: https://docs.solanalabs.com/validator/transaction-processing
- Rust Book - Structs: https://doc.rust-lang.org/book/ch05-00-structs.html
- Rust Book - Testing: https://doc.rust-lang.org/book/ch11-01-writing-tests.html
- Rust std - BinaryHeap: https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html

## [참고 키워드]
- greedy selection
- compute budget
- saturating arithmetic
- happy path testing
- max-heap ordering
