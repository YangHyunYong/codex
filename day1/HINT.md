# HINT

## [힌트]
- `cargo new`로 프로젝트를 만들 때 `--lib` 옵션 없이 생성하면 기본적으로 `src/main.rs`가 생깁니다. 이번 과제에서는 `lib.rs`에 핵심 로직을 작성하고, 필요하다면 `main.rs`에서 재사용해도 됩니다.
- 문자열 안에 코드 블록 형태의 예시를 넣을 때는 역슬래시(`\n`)로 줄바꿈을 처리하면 테스트에서 비교하기 수월합니다.
- 소유권 이동을 설명할 때 `String` 타입을 사용하면 명시적인 move가 일어나므로 케이스를 만들기 좋습니다.
- 테스트에서는 `assert_eq!` 외에도 `assert!(value.contains("ownership"))`처럼 문자열에 포함된 키워드를 검증해 보세요.

## [참고자료]
- The Rust Programming Language (TRPL) 3장: 변수와 가변성 - https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html
- Rust By Example: Ownership - https://doc.rust-lang.org/rust-by-example/ownership.html
- Rustlings 연습 문제(Variables, Shadowing) - https://github.com/rust-lang/rustlings/tree/main/exercises/variables

## [참고 키워드]
- immutable variable
- mutable variable
- shadowing
- ownership move
- Rust unit test
