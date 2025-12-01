---
description: 사용 가능한 설계 아티팩트를 기반으로 기존 작업을 실행 가능하고 의존성이 정렬된 GitHub 이슈로 변환합니다.
tools: ['github/github-mcp-server/issue_write']
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## 사용자 입력

```text
$ARGUMENTS
```

진행하기 전에 사용자 입력을 **반드시** 고려해야 합니다(비어있지 않은 경우).

## 개요

1. 저장소 루트에서 `{SCRIPT}`를 실행하고 FEATURE_DIR 및 AVAILABLE_DOCS 목록을 파싱합니다. 모든 경로는 절대 경로여야 합니다. "I'm Groot"와 같이 인수에 작은따옴표가 있는 경우 이스케이프 구문 사용: 예: 'I'\''m Groot' (또는 가능한 경우 큰따옴표: "I'm Groot").
1. 실행된 스크립트에서 **tasks** 경로를 추출합니다.
1. 다음을 실행하여 Git 원격 저장소를 가져옵니다:

```bash
git config --get remote.origin.url
```

**원격 저장소가 GITHUB URL인 경우에만 다음 단계로 진행하세요**

1. 목록의 각 작업에 대해 GitHub MCP 서버를 사용하여 Git 원격 저장소를 대표하는 저장소에 새 이슈를 생성합니다.

**어떤 경우에도 원격 URL과 일치하지 않는 저장소에 이슈를 생성하지 마세요**
