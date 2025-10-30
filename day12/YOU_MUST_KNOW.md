# YOU_MUST_KNOW.md

## 파이프라인 Stage 개념 요약 (만 14세도 이해할 수 있게)
- **Stage는 일꾼**: 블록체인 동기화는 여러 단계(Stage)로 나뉘어요. `Headers` Stage는 블록의 목차를 가져오고, `Bodies` Stage는 그 안의 거래 내용을 받고, `Execution` Stage는 거래를 진짜로 적용해요. 이렇게 역할을 나누면 한 일꾼(Stage)이 실패해도 다른 일꾼에게 영향을 최소화할 수 있어요.
- **State는 일꾼의 메모장**: Stage마다 `StageState`라는 메모장이 있어서 "나는 블록 번호 120까지 끝냈어" 같은 기록을 남깁니다. Driver는 이 메모장을 보고 "그러면 다음엔 121부터 하면 되겠네"라고 판단해요.
- **Driver는 감독관**: `PipelineDriver`는 Stage들을 차례로 불러서 일을 시키는 감독관이에요. `run_to_block`으로 "200까지 달려!"라고 말하면 각 Stage가 순서대로 실행되죠. 만약 문제가 생기면 `unwind_to_block`으로 "150까지 되돌아가!"라고 말해요.
- **execute / unwind / validate는 계약**: Stage마다 세 가지 계약이 있어요.
  - `execute`: "새로운 블록을 처리할게"라는 약속.
  - `unwind`: "필요하면 되돌릴게"라는 약속.
  - `validate`: "내가 처리한 내용이 맞는지 다시 확인해줄게"라는 약속.
  이런 계약이 명확해야 시스템이 커져도 서로 오해 없이 동작합니다.
- **Progress 기록은 안전장치**: Driver는 Stage가 언제 어디까지 갔는지를 `StageResult::Progress` 형태로 기록해요. 나중에 재시작해도 "어제는 여기까지 했네"라고 쉽게 알 수 있어요.

## 실무 관점의 상세 설명
- Reth는 Stage 실행 결과를 데이터베이스에 저장해 노드가 재시작되더라도 정확히 이어서 실행할 수 있도록 설계합니다. `StageState.last_block`은 이때 핵심 키입니다.
- Stage 간 의존성은 엄격합니다. 예를 들어 `Bodies` Stage는 `Headers`가 끝난 이후에만 실행될 수 있습니다. 그래서 Driver가 Stage 순서를 명시적으로 관리해야 합니다.
- `unwind` 설계는 매우 중요합니다. 체인이 재구성(reorg)될 때 빠르게 이전 상태로 돌아갈 수 있어야 하기 때문이죠. Reth는 unwind 시 필요한 데이터를 미리 캐싱하거나, DB에 역순으로 삽입한 로그를 이용합니다.
- `validate`는 Stage가 실제로 target block까지 처리했는지 확인하는 사후 점검입니다. 만약 데이터가 뒤엉켰다면 바로 에러를 띄우고 재동기화를 시도해야 합니다.
- 파이프라인은 보통 배치 처리입니다. 한 번에 너무 많은 블록을 실행하면 메모리가 터질 수 있으니, StageResult로 "얼마나 진행했는지"를 알려주고 Driver가 배치 크기를 조절합니다.

## 용어 정리
- **Checkpoint**: Stage가 언제든 다시 시작할 수 있도록 남겨두는 복구 지점.
- **Unwind**: 잘못된 데이터나 포크로 인해 블록 처리 결과를 되돌리는 과정.
- **Progress History**: Stage가 처리한 범위를 시간순으로 기록한 로그.
- **Contract**: 메서드 시그니처와 주석으로 표현되는 약속. Driver는 이 약속을 믿고 Stage를 호출합니다.
