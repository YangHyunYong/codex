# HINT

- **참고자료**
  - Reth Header Accumulator 소스: https://github.com/paradigmxyz/reth/blob/main/crates/stages/src/stages/header/accumulator.rs
  - Geth 포크 선택 설명: https://geth.ethereum.org/docs/fundamentals/consensus
  - Rust HashMap 문서: https://doc.rust-lang.org/std/collections/struct.HashMap.html
- **참고 키워드**: reorg, total difficulty comparison, canonical chain, fork choice, parent tracking
- **힌트**
  1. canonical을 재구성할 때 기존 벡터를 직접 수정하기보다 새 경로를 계산한 뒤 교체하면 borrow checker가 편해집니다.
  2. reorg 깊이를 계산하려면 두 canonical 경로의 공통 prefix를 찾으면 됩니다. `Iterator::zip`을 활용해도 좋습니다.
  3. 테스트에서 난이도를 다르게 설정해 포크를 만들 때, 각 헤더의 parent_hash와 number가 일관되도록 helper 함수를 작성하면 중복을 줄일 수 있습니다.
