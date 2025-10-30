# HINT: Day 4 과제 가이드

## [힌트]
- `count_uninitialized`는 `for peer in peers` 반복문 안에서 `if let None = peer.last_slot` 형태로도 풀 수 있지만, `if peer.last_slot.is_none()` 같은 메서드를 활용하면 더 간단합니다.
- `fastest_peer`에서는 "현재 가장 빠른 피어"를 `Option<&NodePeer>` 변수로 유지하면서 새 피어와 비교하세요. `match current_fastest` 패턴을 쓰면 공백 없이 비교가 가능합니다.
- `summarize_slots`를 작성할 때는 빈 벡터를 만들고, 반복문 안에서 `push` 하거나 `iter().map(...).collect()`로 한 번에 만들 수도 있습니다.
- 테스트에서 기대 문자열은 하드코딩해도 괜찮습니다. 다만 공백이나 대소문자 실수를 피하려면 `format!`을 테스트에서도 사용하는 것이 안전합니다.

## [참고자료]
- The Rust Book - Enums and Pattern Matching: https://doc.rust-lang.org/book/ch06-02-match.html
- The Rust Book - Common Collections (`Vec<T>`): https://doc.rust-lang.org/book/ch08-01-vectors.html
- Rust By Example - `if let`: https://doc.rust-lang.org/rust-by-example/flow_control/if_let.html

## [참고 키워드]
- `Vec<T>`
- `if let`
- `Option::is_none`
- `Iterator::min_by`
- Solana gossip metrics
