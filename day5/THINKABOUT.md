# Day 5 THINKABOUT & 직접 리서치

## [메타인지 체크]
1. `filter_transactions`가 참조(`&PendingTx`)를 반환할 때 소유권을 옮기지 않아도 되는 이유를 설명할 수 있나요?
2. `ThresholdFilter`와 같은 추상화가 없으면 테스트가 얼마나 복잡해질지, 코드 유지보수 관점에서 비교해 보세요.
3. `group_by_account`에서 `BTreeMap` 대신 `HashMap`을 사용하면 어떤 장단점이 있는지 서술해 보세요.

## [직접 리서치해 볼 문제]
1. Solana 노드에서 멤풀(실제로는 "ingress shaper"와 gossip queue)이 어떻게 동작하는지 공식 문서를 찾아 개략적으로 정리해 보세요.
2. Ethereum의 멤풀 우선순위 정책(예: EIP-1559 이후 base fee + tip 구조)이 Rust 코드로 어떻게 모델링될 수 있을지 생각해 보세요.
3. Rust에서 동시성 채널(`std::sync::mpsc`)을 이용해 멤풀 필터 결과를 다른 스레드로 전달하려면 어떤 추가 고려 사항이 필요한지 조사해 보세요.
