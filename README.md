# jira-elaborate

Jira 이슈 제목과 코드베이스를 분석하여 구체적인 이슈 설명을 자동 생성하는 Claude Code 플러그인.

## 기능

- Jira 이슈 제목에서 키워드를 추출하고 코드베이스를 탐색하여 맥락을 파악
- 부모/형제 이슈를 분석하여 상위 작업 범위를 이해
- 4단계 코드베이스 탐색: 범위 식별 → 현재 상태 → 참조 패턴 → 영향 범위
- 구조화된 설명 초안 생성: 배경, 요구사항, 완료 조건, 테스트 시나리오
- 사용자 확인 후 Jira에 자동 업데이트

## 사전 요구사항

- [Claude Code](https://claude.com/claude-code)
- [Atlassian MCP server](https://github.com/sooperset/mcp-atlassian) 연결 (`mcp__atlassian__*` 도구 사용 가능 상태)

## 설치

### 마켓플레이스에서 설치

```bash
claude plugin marketplace add <owner>/claude-jira-elaborate-plugin
claude plugin install jira-elaborate
```

### 로컬에서 테스트

```bash
claude --plugin-dir ./path/to/claude-jira-elaborate-plugin
```

## 사용법

```
/jira-elaborate PROJ-123
/jira-elaborate PROJ-123 mask 목록을 컨텍스트 메뉴로 이동하여 패널 공간 확보
```

- 첫 번째 인자: Jira 이슈 키 (필수)
- 이후 텍스트: 배경/목적 힌트 (선택)

## 워크플로우

```
Step 1: Jira 이슈 + 부모/형제 이슈 조회
Step 2: 코드베이스 4단계 탐색
Step 3: 템플릿 기반 초안 작성
Step 4: 사용자 확인
Step 5: Jira 업데이트
```

## 커스터마이즈

`skills/jira-elaborate/templates/description.md`를 수정하여 팀 컨벤션에 맞게 템플릿을 변경할 수 있습니다.

## 라이선스

MIT
