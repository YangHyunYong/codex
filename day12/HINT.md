# HINT.md

## 힌트
- 각 Stage 메서드 위에 의도를 설명하는 주석을 적을 때, "이 메서드는 Driver에게 어떤 계약을 제공하는가?"를 질문한 뒤 답변 형식으로 작성하세요.
- `StageResult`를 기록하는 `history`는 단순 Vec으로도 충분합니다. Reth도 Stage Progress를 DB에 append-only로 기록한 뒤, 필요할 때만 최신 값으로 압축합니다.
- `unwind_to_block` 구현 시 `history`를 역순으로 순회하지 않으면 롤백 순서가 꼬입니다. `Vec::iter().rev()` 패턴을 활용하세요.

## 참고자료
- Reth Book: [https://paradigmxyz.github.io/reth/pipeline/stages/overview.html](https://paradigmxyz.github.io/reth/pipeline/stages/overview.html)
- Stage 설계 인사이트: [https://www.paradigm.xyz/2023/08/reth-architecture](https://www.paradigm.xyz/2023/08/reth-architecture)
- Rust Trait 객체 패턴 복습: [https://doc.rust-lang.org/book/ch17-02-trait-objects.html](https://doc.rust-lang.org/book/ch17-02-trait-objects.html)

## 참고 키워드
- stage pipeline
- checkpoint tagging
- unwind contract
- execution driver
- progress history
