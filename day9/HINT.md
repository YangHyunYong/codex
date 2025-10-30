# HINT

- **참고자료**
  - Reth 공식 문서: https://paradigmxyz.github.io/reth/
  - Ethereum Yellow Paper (헤더 구조 참고): https://ethereum.github.io/yellowpaper/paper.pdf
  - Rust 표준 라이브러리 HashMap 문서: https://doc.rust-lang.org/std/collections/struct.HashMap.html
- **참고 키워드**: reth header accumulator, total difficulty, canonical chain, hash index, genesis block
- **힌트**
  1. 제네시스 헤더는 부모가 없으므로 `parent_hash`를 빈 문자열이나 0 해시로 고정해 두고, 테스트에서 일관되게 사용하세요.
  2. total difficulty는 `u128`으로 관리하면 `u64` 난이도 값이 많이 누적되어도 안전합니다.
  3. 해시 인덱스는 `HashMap<String, usize>` 형태로 canonical 벡터 인덱스를 저장하면 `parent_hash` 찾기가 쉬워집니다.
