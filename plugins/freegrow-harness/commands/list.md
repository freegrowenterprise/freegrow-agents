---
description: 현재 활성 하네스 목록 확인
---

# Harness List

현재 로드된 하네스 목록을 보여줍니다.

## 전제 조건

`~/.claude/harness/personal/` 디렉토리가 존재하지 않으면:
```
❌ 하네스가 설정되지 않았습니다. /harness setup 을 먼저 실행하세요.
```

## 실행 절차

### 1. 공통 하네스 수집

이 플러그인의 `CLAUDE.md`에서 공통 규칙을 수집합니다.

### 2. 역할별 하네스 수집

`~/.claude/harness/personal/meta.json`에서 현재 프로젝트의 역할을 읽고, 해당 `roles/{role}.md` 파일에서 규칙을 수집합니다.

### 3. 개인 하네스 수집

`~/.claude/harness/personal/rules/` 디렉토리의 모든 `.md` 파일을 읽고, YAML frontmatter를 파싱합니다.

### 4. 분류 및 출력

```
📋 활성 하네스

[공통]
  - 플러그인 CLAUDE.md의 공통 규칙

[역할: {감지된 역할}]
  - roles/{role}.md의 규칙

[개인] (활성)
  {id}  "{규칙 내용 요약}"          트리거 {N}회  ({source})
  {id}  "{규칙 내용 요약}"          트리거 {N}회  ({source})

[개인] (비활성 - 30일 미사용)
  {id}  "{규칙 내용 요약}"          마지막 사용: {날짜}
```

각 개인 하네스는 frontmatter의 `id`, `trigger_count`, `last_triggered`, `source`를 표시합니다.
