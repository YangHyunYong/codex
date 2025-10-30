# Day 3 Hint

## [힌트]
- `match`는 조건이 많아 보이더라도 "위에서 아래로 하나씩" 읽으면 됩니다. 우선 가장 구체적인 경우(예: `Some(value)`)를 먼저 적고, 마지막에 전체를 포괄하는 패턴(`None`, 범위 밖 등)을 배치하세요.
- `Option`과 `Result`를 다룰 때는 `unwrap` 대신 `match`를 쓰는 습관을 들이세요. 이렇게 하면 실패 가능성이 눈에 보이고, 나중에 동료가 코드를 읽을 때도 왜 안전한지 바로 이해할 수 있습니다.
- 범위 패턴은 `0..=150` 처럼 끝값을 포함하도록 `=`를 붙이는지 확인하세요. 숫자 범위를 다룰 때는 경계(150, 151 등)를 먼저 테스트 코드에 적어 두면 실수를 바로 발견할 수 있습니다.

## [참고자료]
- [The Rust Programming Language - Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html)
- [Rust by Example - Option and Result](https://doc.rust-lang.org/rust-by-example/std/option.html)
- [Solana Validator Monitoring Guide](https://docs.solana.com/validator/monitoring)

## [참고 키워드]
- `match expression`
- `Option<T>`
- `Result<T, E>`
- Range pattern (`0..=150`)
