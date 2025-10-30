# HINT

## [힌트]
- `annotate_borrowing` 함수에서는 입력 문자열을 그대로 사용하기만 하고 소유권을 이동시키지 않으므로, `format!` 매크로를 활용해 설명 문자열을 구성하면 편리합니다.
- `mutate_wallet`에서 가변 참조가 동시에 두 개 이상 존재하지 않음을 보여 주기 위해, 중간에 다른 가변 참조를 만들지 않도록 `scope`를 주석으로 설명하거나, 추가 변수를 두지 않는 것이 좋습니다.
- 음수 잔액을 방지할 때는 먼저 임시 합계를 계산한 뒤, 조건에 따라 `Ok(())` 또는 `Err("...".to_string())`을 반환하면 테스트가 쉽게 작성됩니다.
- `summarize_slice`는 슬라이스를 다시 슬라이스 하는 연습입니다. `blocks.split_at()` 또는 범위 연산자를 이용하면 간결하게 표현할 수 있습니다.

## [참고자료]
- The Rust Programming Language 4장: 참조와 빌림 - https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html
- Rust By Example: Slices - https://doc.rust-lang.org/rust-by-example/primitives/array.html#slices
- Solana Validator Docs: Accounts and State - https://docs.solana.com/validator/accounts (상태 데이터를 읽기/쓰기 할 때 불변/가변 참조 개념이 어떻게 적용되는지 참고해 보세요.)

## [참고 키워드]
- borrowing rules
- mutable reference conflict
- slice projection
- Result-based error handling
- blockchain state safety
