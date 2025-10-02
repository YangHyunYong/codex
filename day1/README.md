# Day 1 과제 (난이도: EASY)

## [전날 과제 요약]
- 첫 날이라 전날 과제가 없습니다.

## [전날 과제로 얻은 역량]
- 해당 사항 없음.

## [오늘 과제 목표]
- Rust 개발 환경과 도구 체인을 설치하고 점검한다.
- Cargo 프로젝트 구조와 패키지 관리 방식을 이해한다.
- 테스트 주도 개발(TDD) 흐름으로 간단한 CLI 프로그램을 설계하고 구현한다.
- 기본적인 러스트 문법(변수, 함수, 소유권, 모듈, Result 타입)을 익힌다.

## [오늘 과제 설명]
1. **개발 환경 준비**
   - Rustup을 사용해 최신 stable 툴체인을 설치한다. 이미 설치되어 있다면 `rustup update`로 최신 상태로 맞춘다.
   - `cargo --version`, `rustc --version` 명령으로 환경 구성을 확인하고 출력 결과를 캡처한다.
   - VSCode 또는 선호 IDE에 Rust Analyzer 확장을 설치하고, `rust-analyzer`가 정상 작동하는지 간단한 프로젝트에서 확인한다.
2. **프로젝트 생성 및 구조 파악**
   - `cargo new temperature_cli` 명령으로 새 패키지를 만든다.
   - 생성된 디렉터리 구조(`src/main.rs`, `Cargo.toml`, `src/lib.rs` 생성 여부 등)를 README에 정리한다.
   - Git으로 버전 관리를 시작하고, `.gitignore`에 `target/`과 IDE 캐시 폴더를 추가한다.
3. **요구사항 분석 및 TDD 설계**
   - 섭씨(Celsius)를 화씨(Fahrenheit)로 변환하는 CLI 프로그램을 만든다.
   - 프로그램은 `temperature_cli <value> --to <unit>` 형태로 실행하며, `<unit>`은 `c` 또는 `f` 중 하나이다.
   - 핵심 로직은 라이브러리 크레이트(`src/lib.rs`)에 작성하고, 변환 함수를 단위 테스트로 먼저 정의한 뒤 구현한다.
   - 잘못된 입력(숫자가 아닌 값, 지원하지 않는 단위)에 대해서는 친절한 오류 메시지를 반환한다.
4. **문서화 및 리포트**
   - README(학습자용)에는 실행 방법, 지원하는 옵션, 예제 입출력과 테스트 실행 방법을 정리한다.
   - 오늘 학습 과정에서 배운 점, 문제 해결 전략, 향후 개선 아이디어를 `RESULT.md`에 작성하도록 안내한다.

## [이해를 돕기 위한 예시]
```bash
# 프로젝트 생성
cargo new temperature_cli
cd temperature_cli

# 변환 함수에 대한 예시 테스트 (src/lib.rs)
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn converts_celsius_to_fahrenheit() {
        assert!((celsius_to_fahrenheit(100.0) - 212.0).abs() < f64::EPSILON);
    }
}

# CLI 실행 예시
cargo run -- 36.5 --to f
# 출력 예시: 36.5°C is 97.7°F
```
- 위 예시는 방향성을 제시하기 위한 것으로, 실제 구현에서는 입력 파싱과 에러 처리를 스스로 설계해야 한다.
- 테스트는 먼저 작성하고, 테스트가 통과하도록 최소한의 구현을 반복적으로 추가하는 TDD 사이클을 반드시 경험하자.
