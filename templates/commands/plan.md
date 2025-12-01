---
description: 계획 템플릿을 사용하여 설계 아티팩트를 생성하는 구현 계획 워크플로우를 실행합니다.
handoffs:
  - label: 작업 생성
    agent: speckit.tasks
    prompt: 계획을 작업으로 분해합니다
    send: true
  - label: 체크리스트 생성
    agent: speckit.checklist
    prompt: 다음 도메인에 대한 체크리스트를 생성합니다...
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## 사용자 입력

```text
$ARGUMENTS
```

진행하기 전에 사용자 입력을 **반드시** 고려해야 합니다(비어있지 않은 경우).

## 개요

1. **설정**: 저장소 루트에서 `{SCRIPT}`를 실행하고 FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH에 대한 JSON을 파싱합니다. "I'm Groot"와 같이 인수에 작은따옴표가 있는 경우 이스케이프 구문 사용: 예: 'I'\''m Groot' (또는 가능한 경우 큰따옴표: "I'm Groot").

2. **컨텍스트 로드**: FEATURE_SPEC 및 `/memory/constitution.md`를 읽습니다. IMPL_PLAN 템플릿을 로드합니다 (이미 복사됨).

3. **계획 워크플로우 실행**: IMPL_PLAN 템플릿의 구조를 따릅니다:
   - 기술 컨텍스트 채우기 (알 수 없는 것은 "NEEDS CLARIFICATION"으로 표시)
   - 헌장에서 헌장 확인 섹션 채우기
   - 게이트 평가 (위반이 정당화되지 않으면 ERROR)
   - Phase 0: research.md 생성 (모든 NEEDS CLARIFICATION 해결)
   - Phase 1: data-model.md, contracts/, quickstart.md 생성
   - Phase 1: 에이전트 스크립트를 실행하여 에이전트 컨텍스트 업데이트
   - 설계 후 헌장 확인 재평가

4. **중지 및 보고**: Phase 2 계획 후 명령이 종료됩니다. 브랜치, IMPL_PLAN 경로 및 생성된 아티팩트를 보고합니다.

## 단계

### Phase 0: 개요 및 연구

1. **위의 기술 컨텍스트에서 알 수 없는 것 추출**:
   - 각 NEEDS CLARIFICATION → 연구 작업
   - 각 의존성 → 모범 사례 작업
   - 각 통합 → 패턴 작업

2. **연구 에이전트 생성 및 전달**:

   ```text
   기술 컨텍스트의 각 알 수 없는 것에 대해:
     작업: "{기능 컨텍스트}에 대한 {알 수 없는 것} 연구"
   각 기술 선택에 대해:
     작업: "{도메인}에서 {기술}에 대한 모범 사례 찾기"
   ```

3. **결과를 `research.md`에 통합**하고 다음 형식을 사용합니다:
   - 결정: [선택된 것]
   - 근거: [선택된 이유]
   - 고려된 대안: [평가된 다른 것]

**출력**: 모든 NEEDS CLARIFICATION이 해결된 research.md

### Phase 1: 설계 및 계약

**전제조건:** `research.md` 완료

1. **기능 명세에서 엔티티 추출** → `data-model.md`:
   - 엔티티 이름, 필드, 관계
   - 요구사항의 검증 규칙
   - 해당되는 경우 상태 전환

2. **기능 요구사항에서 API 계약 생성**:
   - 각 사용자 작업 → 엔드포인트
   - 표준 REST/GraphQL 패턴 사용
   - OpenAPI/GraphQL 스키마를 `/contracts/`에 출력

3. **에이전트 컨텍스트 업데이트**:
   - `{AGENT_SCRIPT}` 실행
   - 이 스크립트는 사용 중인 AI 에이전트를 감지합니다
   - 적절한 에이전트별 컨텍스트 파일 업데이트
   - 현재 계획의 새 기술만 추가
   - 마커 사이의 수동 추가 보존

**출력**: data-model.md, /contracts/*, quickstart.md, 에이전트별 파일

## 주요 규칙

- 절대 경로 사용
- 게이트 실패 또는 해결되지 않은 명확화에 대해 ERROR
