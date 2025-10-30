# THINKABOUT.md

## 메타인지 체크
1. PipelineDriver가 Stage의 `execute`를 호출할 때 어떤 상태 스냅샷을 넘겨야 race condition 없이 재시도를 설계할 수 있을까요?
2. `StageResult::Idle`을 반환하는 조건을 바꾸면 Driver의 루프 구조는 어떻게 달라져야 할지 스스로 설명해 보세요.
3. `validate`가 `OutOfSync`를 반환했을 때 재동기화 전략으로 어떤 추가 메타데이터를 수집해야 할까요?

## 직접 리서치해 볼 문제
1. Reth Pipeline에서 `StageId::Execution`이 실제로 다루는 데이터 구조와 외부 의존성은 무엇인지 공식 문서나 소스코드에서 찾아보세요.
2. Reth는 Stage마다 `checkpoint block`을 어떻게 기록하고 복원하는지(예: DB column family, metadata table)를 조사해 정리해 보세요.
3. Stage Driver가 병렬 실행을 지원하려면 어떤 Lock-Free 또는 메시지 기반 디자인을 고려해야 하는지 비교 연구해 보세요.
