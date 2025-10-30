# HINT: Day 11

## 힌트
- BinaryHeap은 기본적으로 최대 힙이며 `Ord` 구현이 중요합니다. priority_key 비교 시 tuple 비교를 활용하면 코드가 깔끔해집니다.
- per_account 큐는 `VecDeque`를 사용하면 front pop이 O(1)이라 pop_batch 구현이 쉬워집니다.
- lazy eviction은 pop할 때 "stale" 트랜잭션을 건너뛰는 방식입니다. BinaryHeap top이 per_account 실제 front와 다르면 반복해서 pop하세요.

## 참고자료
- [Reth TxPool RFC](https://github.com/paradigmxyz/reth/blob/main/docs/design/txpool.md)
- [Rust BinaryHeap 문서](https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html)
- [Priority Queue Patterns](https://www.cs.princeton.edu/courses/archive/fall03/cs126/assignments/priority.html)

## 참고 키워드
- `reth::transaction_pool`
- `max heap`, `lazy eviction`
- `sender queue`, `nonce ordering`
