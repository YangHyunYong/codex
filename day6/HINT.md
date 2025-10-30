# Day 6 Hint

1. `BinaryHeap`은 기본적으로 최대 힙(max-heap)입니다. 높은 점수가 먼저 나오게 하려면 `Ord` 구현에서 `score`를 비교하고, 동점 시에는 `id`를 비교하는 보조 조건을 추가하세요.
2. `thiserror` 크레이트를 사용하려면 `Cargo.toml`에 `thiserror = "1"`을 추가하는 것을 잊지 마세요.
3. "작은 벤치마크" 테스트는 진짜 벤치마크가 아니라 반복문을 통해 `push`/`pop`이 기대대로 작동하는지만 빠르게 확인하면 충분합니다.

## 참고자료
- Rust Book: [BinaryHeap](https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html)
- `thiserror` 문서: <https://docs.rs/thiserror/latest/thiserror/>
- Rust Book: [Derive 매크로](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)

## 참고 키워드
`BinaryHeap`, `Ord`, `Result`, `thiserror`, `Priority Queue`
