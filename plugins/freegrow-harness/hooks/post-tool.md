---
name: post-tool
description: 도구 사용 후 실시간 패턴 감지 및 하네스 등록 제안
hooks:
  - event: PostToolUse
---

# 실시간 패턴 감지

Claude Code의 도구 사용 후 패턴을 분석하여, 반복되는 실수나 이상 패턴을 감지합니다.

## 전제 조건

- `~/.claude/harness/personal/` 디렉토리가 존재해야 함 (없으면 아무것도 하지 않음)
- `~/.claude/harness/personal/pattern-log.jsonl` 파일이 존재해야 함

## 감지 절차

### 1. 현재 도구 사용 분석

방금 실행된 도구(Edit, Bash, Write 등)의 결과를 분석합니다:

- **Edit/Write 후**: 같은 파일에 유사한 수정이 반복되는지 확인
- **Bash 후**: 위험한 명령어(rm -rf, force push, 시크릿 노출 등)인지 확인
- **전체**: 같은 유형의 에러를 반복 수정하고 있는지 확인

### 2. 패턴 로그 기록

감지된 패턴을 `~/.claude/harness/personal/pattern-log.jsonl`에 기록합니다:

```jsonl
{"ts":"ISO8601","tool":"도구명","pattern":"패턴ID","detail":"상세설명","file":"파일경로","count":N}
```

### 3. 패턴 반복 횟수 확인

같은 `pattern` ID가 현재 세션에서 3회 이상 발생했는지 확인합니다.

### 4. 하네스 등록 제안

#### 3회 이상 반복된 패턴

`~/.claude/harness/personal/ignored-patterns.json`에 이미 거절된 패턴인지 확인한 후, 아니라면 사용자에게 제안합니다:

```
⚡ 패턴 감지: [패턴 설명]을 [N]회 반복했습니다.

제안 하네스:
  "[규칙 내용]"

  → 개인 하네스로 등록할까요? (y/n)
```

- **승인 시**: `~/.claude/harness/personal/rules/`에 하네스 파일 생성. frontmatter 포함.
  - `id`: `{role}-{순번}` 형식 (예: `backend-001`)
  - `type`: 간단한 규칙이면 `rule`, 자동화 가능하면 `hook`
  - `role`: 현재 프로젝트의 감지된 역할
  - `source`: `auto`
  - 이후 추가 질문: "이 규칙을 [역할]별 공통으로 승격 제안할까요? (y/n)"
- **거절 시**: `ignored-patterns.json`에 패턴 추가하여 다시 제안하지 않음

#### 위험 패턴 (횟수 무관)

시크릿 하드코딩, `rm -rf`, `force push` 등 위험 패턴은 즉시 경고합니다:

```
⚠️ 위험 패턴 감지: [패턴 설명]
이 작업은 위험할 수 있습니다. 계속 진행하시겠습니까?
```

## 중요

- 패턴 감지는 **세션 내** 카운팅입니다. 세션이 끝나면 리셋.
- 패턴 로그(`pattern-log.jsonl`)는 세션 간 누적됩니다 (통계 목적).
- 감지 로직이 사용자의 작업 흐름을 **방해하지 않도록** 주의합니다.
- 하네스 등록 제안은 **최소한**으로 합니다. 같은 패턴에 대해 한 세션에서 한 번만 제안.
