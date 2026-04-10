---
name: jira-elaborate
description: Jira 이슈 제목과 코드베이스를 분석하여 구체적인 이슈 설명을 생성합니다.
argument-hint: <ISSUE-KEY> [배경 힌트]
allowed-tools: Bash, Read, Grep, Glob, Agent, AskUserQuestion, mcp__atlassian__*
---

# Jira 이슈 설명 생성기

Jira 이슈 제목, 사용자가 제공한 배경 정보, 부모/형제 이슈 분석, 코드베이스 탐색을 결합하여 구체적이고 구조화된 이슈 설명을 생성합니다.

## 사용법

```
/jira-elaborate PROJ-123
/jira-elaborate PROJ-123 mask 목록을 컨텍스트 메뉴로 이동하여 패널 공간 확보
```

- 첫 번째 인자: Jira 이슈 키 (필수)
- 이후 텍스트: 배경/목적에 대한 힌트 (선택, 배경 섹션에 반영)

## 관련 파일

| 파일 | 설명 |
|------|------|
| `templates/description.md` | 이슈 설명 템플릿. 팀 컨벤션에 맞게 커스터마이즈 가능. |

## 사전 요구사항

- **Atlassian MCP server** 연결 필요 (`mcp__atlassian__*` 도구 사용 가능 상태)
- 대상 Jira 이슈가 존재하고 접근 가능해야 함

## 워크플로우

### Step 1: Jira 컨텍스트 수집

1. `mcp__atlassian__getJiraIssue`로 대상 이슈의 제목, 현재 설명, 이슈 타입을 조회한다.
2. 이슈가 **Subtask**인 경우 부모 이슈도 조회하여 상위 맥락을 파악한다.
3. `mcp__atlassian__searchJiraIssuesUsingJql`로 `parent = <PARENT-KEY>` 쿼리하여 형제 서브태스크 목록과 전체 작업 범위를 파악한다.

**Atlassian MCP 공통 파라미터:**
- `cloudId`: Jira 이슈 URL에 표시된 사이트 URL 사용 (예: `myteam.atlassian.net`)
- `responseContentFormat`: `markdown`

### Step 2: 코드베이스 분석

단계적 탐색 전략을 사용한다. 넓은 범위에서 시작하여 점차 좁혀간다.

**Layer 1 — 범위 식별**:
- 이슈 제목에서 모듈/컴포넌트 태그 확인 (예: `[Auth]`, `[UI]`, `[API]`). 태그가 있으면 해당 영역으로 탐색 범위를 좁힌다.
- 태그가 없으면 부모 이슈나 사용자 힌트에서 범위를 추론한다.
- 프로젝트 최상위 디렉토리 구조(`ls`, CLAUDE.md, README)를 확인하여 코드베이스 레이아웃을 파악한다.

**Layer 2 — 현재 구현 상태 파악**:
- `Grep`/`Glob`으로 이슈 키워드와 관련된 파일, 인터페이스, 클래스를 탐색한다.
- 핵심 파일을 읽어 현재 동작 방식과 데이터 흐름을 이해한다.
- 이미 존재하는 것과 새로 추가/변경해야 할 것을 구분한다.

**Layer 3 — 참조 패턴 탐색**:
- 코드베이스 내 다른 곳에 이미 구현된 유사 기능을 찾는다 (예: 모듈 A에 컨텍스트 메뉴를 추가하는 이슈라면, 모듈 B에 이미 구현된 컨텍스트 메뉴를 확인).
- 기존 패턴의 구조를 분석한다: 인터페이스, 이벤트 흐름, UI 컴포넌트, 데이터 모델.
- 재사용 가능한 부분과 새로 생성해야 할 부분을 구분한다.

**Layer 4 — 영향 범위 분석**:
- 변경에 영향 받는 파일/모듈을 식별한다.
- 엣지 케이스를 확인한다: 데이터가 없거나, 비어있거나, 에러 상태일 때 어떻게 되는지.

복잡한 탐색이 필요한 경우 `Agent` (subagent_type: Explore)를 사용한다.

### Step 3: 초안 작성

수집한 정보를 바탕으로 `templates/description.md` 템플릿의 각 섹션을 작성한다.

**작성 가이드라인:**

- **배경 / 목적**: 사용자가 힌트를 제공했다면 그것을 중심으로 작성. 없으면 부모 이슈, 형제 태스크, 코드 분석에서 유추.
- **요구사항**: 번호를 매기고 **굵은 소제목** 포함. UI 동작, 데이터 흐름, 기존 코드와의 관계를 구체적으로 명시.
- **완료 조건**: 검증 가능한 체크리스트(`- [ ]`) 형식. 각 요구사항에 대응하는 검증 항목 + 엣지 케이스 포함.
- **테스트 시나리오**: 번호를 매기고 **굵은 시나리오명**: 구체적 단계 → 기대 결과 형식.

### Step 4: 사용자 확인

작성한 초안을 사용자에게 보여주고 피드백을 받는다.

- 수정 요청이 있으면 반영 후 다시 보여준다.
- **사용자의 명시적 승인 후에만 Step 5로 진행한다.**

### Step 5: Jira 업데이트

`mcp__atlassian__editJiraIssue`로 이슈 설명을 업데이트한다.

```
mcp__atlassian__editJiraIssue
  cloudId: <site-url>
  issueIdOrKey: <ISSUE-KEY>
  contentFormat: markdown
  fields: { "description": "<작성된 설명>" }
```

제목에 오타나 태그 수정이 필요한 경우 `summary` 필드도 함께 업데이트한다.

완료 후 이슈 URL을 사용자에게 안내한다.

## 주의사항

- **사용자 확인 없이 Jira를 업데이트하지 않는다.**
- 기존 설명이 이미 구체적으로 작성되어 있으면, 덮어쓰기 전에 사용자에게 알린다.
- 코드베이스에서 파악할 수 없는 비즈니스 요구사항은 추측하지 않고 `AskUserQuestion`으로 질문한다.
