---
description: 개인 하네스 초기 설정 (최초 1회)
---

# Harness Setup

개인 하네스 디렉토리를 생성하고 초기화합니다.

## 실행 절차

### 1. 디렉토리 존재 확인

```bash
ls ~/.claude/harness/personal/ 2>/dev/null
```

이미 존재하면:
```
✅ 이미 설정되어 있습니다.
현재 하네스: /harness list 로 확인하세요.
```

### 2. 디렉토리 생성

```bash
mkdir -p ~/.claude/harness/personal/{rules,hooks}
```

### 3. meta.json 초기화

```bash
echo '{"projects":{}}' > ~/.claude/harness/personal/meta.json
```

### 4. pattern-log.jsonl 초기화

```bash
touch ~/.claude/harness/personal/pattern-log.jsonl
```

### 5. ignored-patterns.json 초기화

```bash
echo '{"patterns":[]}' > ~/.claude/harness/personal/ignored-patterns.json
```

### 6. 현재 프로젝트 역할 감지

현재 프로젝트의 파일 구성을 스캔하여 역할을 판별하고 meta.json에 기록합니다.

### 7. 결과 출력

```
✅ 하네스 초기 설정 완료!

📁 개인 하네스 위치: ~/.claude/harness/personal/
🔍 현재 프로젝트 역할: [감지된 역할들]

다음 단계:
- 작업하면서 패턴이 자동 감지됩니다.
- /harness add "규칙" 으로 직접 등록할 수도 있습니다.
- /harness list 로 현재 하네스를 확인하세요.
```
