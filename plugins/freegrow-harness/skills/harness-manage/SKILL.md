---
name: harness-manage
description: Freegrow 하네스 관리 컨벤션. "하네스", "harness", "패턴 등록" 등의 요청 시 자동 적용됨.
globs:
  - "**/*"
alwaysApply: true
---

# Freegrow 하네스 관리

이 스킬은 하네스 관련 작업 시 자동으로 적용됩니다.

## 하네스 파일 형식

개인 하네스는 `~/.claude/harness/personal/rules/` 디렉토리에 저장됩니다.

### frontmatter 필수 필드

```yaml
---
id: {role}-{순번}        # 고유 식별자
type: rule | hook         # rule: 텍스트 규칙, hook: 실행 가능한 스크립트
role: common | backend | frontend | hardware
created: YYYY-MM-DD
trigger_count: 0          # 트리거 횟수 (자동 업데이트)
last_triggered: null      # 마지막 트리거 일시 (자동 업데이트)
source: auto | manual     # 자동 감지 또는 수동 등록
---
```

### ID 순번 규칙

같은 role 내에서 순번은 자동 증가합니다:
- `backend-001`, `backend-002`, `backend-003`...
- `common-001`, `common-002`...

기존 파일을 스캔하여 가장 큰 순번 + 1로 설정합니다.

## meta.json 형식

```json
{
  "projects": {
    "/absolute/path/to/project": {
      "roles": ["backend", "frontend"],
      "detected_at": "2026-04-06",
      "files_hash": "sha256_of_file_listing"
    }
  }
}
```

## ignored-patterns.json 형식

```json
{
  "patterns": [
    {
      "pattern": "패턴ID",
      "ignored_at": "2026-04-06T10:30:00",
      "reason": "사용자 거절"
    }
  ]
}
```

## 커맨드 요약

| 커맨드 | 기능 |
|--------|------|
| `/harness setup` | 초기 설정 (최초 1회) |
| `/harness list` | 현재 활성 하네스 목록 |
| `/harness add "<규칙>"` | 수동 하네스 등록 |
| `/harness promote <id>` | 개인 → 공통/역할 승격 PR 생성 |
| `/harness remove <id>` | 하네스 삭제 |
