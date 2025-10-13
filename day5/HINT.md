# Day 5 힌트 & 참고자료

## [힌트]
- `MempoolFilter` 트레이트 구현 시, 조건을 자연스러운 순서(예: 수수료 → 페이로드 → 상태)로 배치하면 테스트 실패 시 어떤 조건이 문제인지 빠르게 파악할 수 있습니다.
- `filter_transactions`는 `iter()` 대신 `iter().filter(...)` 체인을 사용해도 되지만, 중간에 디버그 로그를 찍고 싶다면 `for` 루프와 `push` 조합이 가독성이 좋습니다.
- `group_by_account`는 `BTreeMap::entry` API를 사용하면 기존 벡터를 가져와 수정하거나 새로 생성하는 코드를 간결하게 만들 수 있습니다.
- `compute_account_stats`에서 `match tx.status`를 사용하면 `Pending`, `Simulated`, `Rejected` 각각을 명확히 구분할 수 있습니다.

## [참고자료]
- Rust Book: [Structs](https://doc.rust-lang.org/book/ch05-00-structs.html)
- Rust Book: [Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html)
- Rust Book: [Traits](https://doc.rust-lang.org/book/ch10-02-traits.html)
- Rust Book: [Collections - BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html)
- Rust By Example: [Iterators](https://doc.rust-lang.org/rust-by-example/trait/iter.html)

## [참고 키워드]
- Rust trait object vs. generic trait bound
- Rust lifetime annotations
- BTreeMap entry API
- Ownership vs. borrowing in collections
- Pending transaction prioritization
