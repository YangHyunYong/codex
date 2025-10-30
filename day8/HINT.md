# HINT Day 8

## [힌트]
- 먼저 도우미 함수(`check_account_conflicts`, `would_exceed_compute`)를 작성한 뒤, 메인 로직에서 조합하면 코드가 깔끔해집니다.
- `HashSet::extend`를 이용하면 여러 계정을 한 번에 추가할 수 있지만, 처음에는 반복문으로 명시적으로 추가해도 괜찮습니다.
- `VecDeque`에서 특정 ID를 제거할 때는 `retain` 메서드를 활용하면 소유권 문제를 피하면서 간단하게 구현할 수 있습니다.

## [참고자료]
- [Rust 공식 문서: std::collections::HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html)
- [Solana Docs: Runtime Overview - Account Locks](https://docs.solana.com/developing/programming-model/runtime#account-locks)
- [Rust Book: Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html)

## [참고 키워드]
- `account locking`
- `hashset conflict detection`
- `transaction scheduler`
- `result error handling`
