# reth_DESIGN.md

## 핵심 설계 의도 요약
- **Stage는 독립적 모듈**: Reth Pipeline의 각 Stage는 DB TX와 Execution Context를 분리해 재사용성과 장애 격리를 얻습니다. Stage는 `execute`/`unwind`/`validate` 세 가지 메서드 계약으로 Driver와 통신하며, Driver는 이 계약을 통해 Stage를 순차적으로 호출합니다.
- **Checkpoint 기반 재시작**: Reth는 Stage 별로 `StageCheckpoint`를 저장해 노드 재시작 시 중단 지점부터 이어갈 수 있게 합니다. 체크포인트에는 마지막 처리 블록, Stage-specific metadata, execution progress가 포함되며, Driver는 이를 읽어 `StageInput`을 구성합니다.
- **Lazy unwind 전략**: 파이프라인은 포크 전환 시 전체 Stage를 즉시 롤백하지 않고, 필요할 때만 `unwind`를 호출해 성능을 유지합니다. StageResult와 History 로그는 어느 Stage가 얼마나 진행됐는지 빠르게 판단하는 근거가 됩니다.
- **Validation pass**: 실행이 끝난 뒤 Driver는 Stage 정합성을 확인하기 위해 `validate`를 호출합니다. 이는 DB와 Stage State가 어긋났을 때 빠르게 감지하도록 하는 fail-fast 메커니즘입니다.

## 학습자가 주목해야 할 코드 포인트 (reth 저장소 기준)
1. `crates/pipeline/src/stage.rs` — `Stage` trait 정의와 `StageId`, `StageCheckpoint` 타입을 살펴보고 메서드 계약을 익히세요.
2. `crates/pipeline/src/driver.rs` — `Pipeline` 및 `Pipeline::run` 구현을 참고해 Stage 실행 순서와 checkpoint 저장이 어떻게 이루어지는지 확인하세요.
3. `crates/pipeline/stages/execution/src/lib.rs` — Execution Stage가 `execute`, `unwind`, `validate`를 어떻게 오버라이드하고, DB 트랜잭션을 어떻게 구성하는지 분석하세요.
4. `crates/pipeline/tests/pipeline.rs` — Stage 실행 및 unwind 시나리오를 어떻게 테스트하는지 살펴보고, 오늘 과제 테스트 설계에 반영하세요.

## 오늘 과제와의 연결 고리
- 오늘 구현할 `PipelineDriver`는 Reth의 `Pipeline::run` 루프를 축약한 형태입니다. StageResult를 history로 남기는 이유는 Reth가 DB에 StageProgress를 기록하는 전략을 그대로 따르기 위함입니다.
- `StageState` 구조체는 Reth의 `StageCheckpoint`를 단순화한 모델입니다. 실제 Reth는 block number 외에도 `Checkpoint BlockHash`, `Execution segment` 등 추가 데이터를 보관합니다.
- Dummy Stage를 통해 `execute`/`unwind`/`validate`가 각각 어떤 계약을 지니는지 체감한 뒤, 학습자가 Reth Stage 소스를 읽을 때 메서드 의도를 빠르게 파악할 수 있도록 설계되었습니다.
- 테스트 시나리오에서 `OutOfSync`를 확인하는 절차는 Reth의 Stage Validation 패턴과 동일합니다. 실제 코드에서도 Stage가 target block과 어긋나면 Driver가 즉시 오류를 발생시키고, 필요한 경우 `unwind`나 재시동을 수행합니다.
