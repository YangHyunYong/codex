# YOU_MUST_KNOW

## 1. 포크 선택(Fork Choice)의 기본 원리
- 작업증명(PoW) 체인에서는 total difficulty가 가장 높은 체인이 "가장 긴 체인"으로 간주됩니다.
- Reth도 total difficulty를 비교해 canonical을 유지합니다. 더 큰 값이 들어오면 기존 체인을 롤백하고 새로운 체인을 채택합니다.

## 2. 헤더 그래프와 부모 추적
- 헤더는 `parent_hash`를 통해 트리 구조를 형성합니다. 해시를 키로 한 HashMap에 저장하면 빠르게 부모를 찾을 수 있습니다.
- 부모를 저장하지 않으면 reorg 시 역추적이 어려워집니다. 따라서 parent hash는 반드시 함께 보관해야 합니다.

## 3. Reorg(재구성) 깊이 계산
- reorg 깊이는 기존 canonical에서 몇 개의 블록이 롤백되는지를 의미합니다.
- 두 canonical 경로의 공통 prefix를 찾으면 롤백해야 하는 블록 수는 `old_len - prefix_len`으로 계산할 수 있습니다.

## 4. Result 타입을 통한 상태 전달
- 헤더 삽입 결과를 `ReorgOutcome`으로 표현하면 상위 모듈이 어떤 행동을 취해야 하는지 명확해집니다.
- 예를 들어 `Extended`면 새 블록을 실행 Stage에 전달하고, `Reorganized`면 롤백 후 재실행이 필요하다는 신호가 됩니다.

## 5. 테스트로 검증해야 할 포인트
- 동일한 해시가 중복 삽입되지 않는지, total difficulty 비교가 올바르게 작동하는지, canonical 벡터가 재정렬되는지를 모두 확인해야 합니다.
- 테스트에서 다양한 난이도와 parent 조합을 사용하면 실전 환경에서 발생할 수 있는 버그를 미리 잡을 수 있습니다.
