---
description: 수동으로 개인 하네스 등록
---

# Harness Add

사용자가 직접 하네스 규칙을 등록합니다.

## 인자

`$ARGUMENTS` - 등록할 규칙 내용

## 전제 조건

`~/.claude/harness/personal/` 디렉토리가 존재하지 않으면:
```
❌ 하네스가 설정되지 않았습니다. /harness setup 을 먼저 실행하세요.
```

## 실행 절차

### 1. 규칙 내용 파싱

`$ARGUMENTS`에서 규칙 내용을 추출합니다.

인자가 없으면 사용자에게 질문합니다:
```
어떤 규칙을 등록할까요?
```

### 2. 역할 판별

규칙 내용을 분석하여 가장 적합한 역할을 판별합니다:
- 특정 역할에만 해당하면 → 해당 역할 (backend, frontend, hardware)
- 범용적이면 → common

사용자에게 확인합니다:
```
이 규칙의 역할을 [{판별된 역할}]로 설정합니다. 맞나요? (y/n)
```

### 3. 하네스 형태 결정

- 자동 검증이 가능한 규칙 → `type: hook`
- 텍스트 가이드 수준 → `type: rule`

### 4. 하네스 파일 생성

기존 하네스 파일의 ID를 스캔하여 다음 순번을 결정합니다.

`~/.claude/harness/personal/rules/{role}-{순번}.md` 파일 생성:

```markdown
---
id: {role}-{순번}
type: rule
role: {역할}
created: {오늘 날짜}
trigger_count: 0
last_triggered: null
source: manual
---

{규칙 내용}
```

### 5. 승격 제안

```
✅ {id} 등록 완료 (rule, {역할})
→ 이 규칙을 {역할}별 공통으로 승격 제안할까요? (y/n)
```

승격을 원하면 `/harness promote {id}` 안내.

### 6. 결과 출력

```
✅ 하네스 등록 완료!

ID: {id}
타입: {type}
역할: {role}
내용: "{규칙 내용}"
```
