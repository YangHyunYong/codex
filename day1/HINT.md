# Day 1 HINT (난이도: EASY)

## [힌트]
- TDD 순서를 명확히 지키고 싶다면 `cargo test`를 실패시키는 작은 테스트부터 작성한 후, 해당 테스트를 통과시키는 최소 구현만 추가하세요.
- CLI 인자 파싱은 표준 라이브러리의 `std::env::args`로 직접 처리해도 되지만, 로직을 깔끔하게 유지하려면 파싱 책임을 별도의 함수로 분리해 보세요.
- 오류 메시지는 `Result<T, String>` 대신 커스텀 에러 타입을 만들어 `Display`를 구현하면 추후 확장에 유리합니다.

## [참고자료]
- Rust 공식 문서: https://doc.rust-lang.org/book/
- Rustup 설치 가이드: https://rustup.rs/
- TDD 기초(한국어 블로그): https://evan-moon.github.io/2020/05/13/tdd-basic/
- CLI 프로젝트 예제: https://rust-cli.github.io/book/index.html

## [참고 키워드]
- Rust ownership, borrowing
- Cargo workspace, package
- Unit test, integration test
- Command line arguments parsing
- Error handling with Result
