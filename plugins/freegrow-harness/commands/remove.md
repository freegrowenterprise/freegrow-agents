---
description: 개인 하네스 삭제
---

# Harness Remove

개인 하네스를 삭제합니다.

## 인자

`$ARGUMENTS` - 삭제할 하네스 ID (예: `backend-001`)

## 전제 조건

- `~/.claude/harness/personal/` 디렉토리가 존재해야 함
- 해당 ID의 하네스 파일이 존재해야 함

## 실행 절차

### 1. 하네스 파일 확인

`~/.claude/harness/personal/rules/{id}.md` 파일을 읽어 내용을 확인합니다.

인자가 없으면:
```
삭제할 하네스 ID를 입력하세요. /harness list 로 목록을 확인할 수 있습니다.
```

### 2. 삭제 확인

```
🗑️ 하네스 삭제 확인

ID: {id}
내용: "{규칙 내용}"
트리거 횟수: {trigger_count}회

정말 삭제할까요? (y/n)
```

### 3. 파일 삭제

```bash
rm ~/.claude/harness/personal/rules/{id}.md
```

hook 타입이면 관련 스크립트도 삭제:
```bash
rm ~/.claude/harness/personal/hooks/{id}.sh 2>/dev/null
```

### 4. 결과 출력

```
✅ {id} "{규칙 요약}" 삭제 완료
```
