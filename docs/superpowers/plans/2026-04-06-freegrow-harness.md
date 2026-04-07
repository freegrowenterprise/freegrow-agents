# Freegrow Harness Framework Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 사내 개발자의 실수 패턴을 자동 감지하여 하네스(규칙/Hook)로 만들어주는 자기 진화형 Claude Code 플러그인 구축

**Architecture:** 단일 플러그인(`freegrow-harness`)으로 배포. session-start hook이 프로젝트 파일을 스캔하여 역할(backend/frontend/hardware)을 자동 판별하고, post-tool hook이 실시간으로 패턴을 감지한다. 개인 하네스는 `~/.claude/harness/personal/`에 자동 생성되며, 스마트 로딩으로 관련된 것만 컨텍스트에 로드한다.

**Tech Stack:** Claude Code Plugin System (plugin.json, hooks, skills, commands, agents), Markdown with YAML frontmatter, JSON/JSONL for data, Bash for hook scripts

**Spec:** `docs/superpowers/specs/2026-04-06-freegrow-harness-design.md`

---

## File Structure

```
plugins/freegrow-harness/
├── plugin.json                          # 플러그인 매니페스트
├── CLAUDE.md                            # 공통 하네스 규칙 (항상 로드)
├── hooks/
│   ├── session-start.md                 # 역할 감지 + 스마트 로딩 (SessionStart hook)
│   └── post-tool.md                     # 실시간 패턴 감지 (PostToolUse hook)
├── roles/
│   ├── backend.md                       # 서버사이드 역할 규칙
│   ├── frontend.md                      # UI/클라이언트 역할 규칙
│   └── hardware.md                      # 임베디드/저수준 역할 규칙
├── skills/
│   └── harness-manage/SKILL.md          # /harness 커맨드의 실행 로직
├── commands/
│   ├── setup.md                         # /harness setup
│   ├── list.md                          # /harness list
│   ├── add.md                           # /harness add
│   ├── promote.md                       # /harness promote
│   └── remove.md                        # /harness remove
└── agents/
    └── harness-detector.md              # 패턴 감지 + 하네스 자동 생성 에이전트
```

**개인 하네스 디렉토리** (플러그인이 자동 생성):
```
~/.claude/harness/personal/
├── meta.json                            # 프로젝트별 역할 캐시 + 하네스 통계
├── pattern-log.jsonl                    # 패턴 감지 로그
├── ignored-patterns.json                # 사용자가 거절한 패턴 목록
├── rules/                               # 텍스트 규칙 하네스
└── hooks/                               # 실행 가능한 Hook 하네스
```

---

### Task 1: 플러그인 스캐폴드

**Files:**
- Create: `plugins/freegrow-harness/plugin.json`
- Create: `plugins/freegrow-harness/CLAUDE.md`

- [ ] **Step 1: plugin.json 생성**

```json
{
  "name": "freegrow-harness",
  "description": "자기 진화형 하네스 프레임워크 - 실수 패턴 자동 감지 및 규칙 생성",
  "version": "0.1.0",
  "author": {
    "name": "Freegrow Enterprise"
  },
  "commands": ["./commands/"],
  "skills": ["./skills/"],
  "hooks": ["./hooks/"]
}
```

- [ ] **Step 2: CLAUDE.md 공통 하네스 규칙 생성**

이 파일은 플러그인이 설치된 모든 프로젝트에서 항상 로드되는 공통 규칙이다. 초기에는 빈 틀만 만들고, 실제 규칙은 팀에서 채운다.

```markdown
# Freegrow 공통 하네스

이 규칙은 모든 프로젝트, 모든 역할에 공통으로 적용됩니다.

## 공통 규칙

<!-- 팀에서 합의된 공통 규칙을 여기에 추가합니다. -->
<!-- 예시: -->
<!-- - 시크릿 키, API 키, 비밀번호를 코드에 하드코딩하지 마라 -->
<!-- - .env 파일을 커밋하지 마라 -->

## 개인 하네스 연동

이 플러그인은 `~/.claude/harness/personal/` 디렉토리에서 개인 하네스를 자동 로드합니다.
개인 하네스는 세션 시작 시 역할과 사용 빈도에 따라 스마트 로딩됩니다.

### 사용 가능한 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/harness setup` | 초기 설정 (최초 1회) |
| `/harness list` | 현재 활성 하네스 목록 |
| `/harness add "<규칙>"` | 수동 하네스 등록 |
| `/harness promote <id>` | 개인 → 공통/역할 승격 PR 생성 |
| `/harness remove <id>` | 하네스 삭제 |
```

- [ ] **Step 3: 플러그인 디렉토리 구조 생성**

```bash
cd /Users/mingwanchoi/Desktop/freegrow_enterprise/freegrow-agents
mkdir -p plugins/freegrow-harness/{hooks,roles,skills/harness-manage,commands,agents}
```

- [ ] **Step 4: 커밋**

```bash
git add plugins/freegrow-harness/plugin.json plugins/freegrow-harness/CLAUDE.md
git commit -m "feat: freegrow-harness 플러그인 스캐폴드 생성"
```

---

### Task 2: 역할별 규칙 파일

**Files:**
- Create: `plugins/freegrow-harness/roles/backend.md`
- Create: `plugins/freegrow-harness/roles/frontend.md`
- Create: `plugins/freegrow-harness/roles/hardware.md`

- [ ] **Step 1: backend.md 생성**

```markdown
# Backend 역할 하네스

서버사이드 기술 스택(Java, Kotlin, Go, Python, Node.js 등) 프로젝트에서 로드됩니다.

## 규칙

<!-- 백엔드 팀에서 합의된 규칙을 여기에 추가합니다. -->
<!-- 예시: -->
<!-- - API 응답에 항상 error code를 포함해라 -->
<!-- - DB 쿼리 작성 시 N+1 문제를 확인해라 -->
<!-- - 환경변수는 직접 참조하지 말고 설정 파일을 통해 접근해라 -->
```

- [ ] **Step 2: frontend.md 생성**

```markdown
# Frontend 역할 하네스

UI/클라이언트 기술 스택(Flutter, React, Vue, Angular 등) 프로젝트에서 로드됩니다.

## 규칙

<!-- 프론트엔드 팀에서 합의된 규칙을 여기에 추가합니다. -->
<!-- 예시: -->
<!-- - 상태 관리는 팀 표준 라이브러리를 사용해라 -->
<!-- - 하드코딩된 문자열 대신 i18n 키를 사용해라 -->
<!-- - 접근성(a11y) 속성을 빠뜨리지 마라 -->
```

- [ ] **Step 3: hardware.md 생성**

```markdown
# Hardware 역할 하네스

임베디드/저수준 기술 스택(C, C++, Assembly, RTOS 등) 프로젝트에서 로드됩니다.

## 규칙

<!-- 하드웨어 팀에서 합의된 규칙을 여기에 추가합니다. -->
<!-- 예시: -->
<!-- - 동적 메모리 할당(malloc) 후 반드시 해제(free)를 확인해라 -->
<!-- - 인터럽트 핸들러에서 블로킹 함수를 호출하지 마라 -->
<!-- - 레지스터 접근 시 volatile 키워드를 사용해라 -->
```

- [ ] **Step 4: 커밋**

```bash
git add plugins/freegrow-harness/roles/
git commit -m "feat: 역할별 하네스 규칙 템플릿 생성 (backend, frontend, hardware)"
```

---

### Task 3: session-start hook (역할 감지 + 스마트 로딩)

**Files:**
- Create: `plugins/freegrow-harness/hooks/session-start.md`

- [ ] **Step 1: session-start.md 생성**

이 hook은 Claude Code 세션이 시작될 때마다 실행된다. 프로젝트의 기술 스택을 판별하고, 해당 역할의 규칙 + 관련 개인 하네스를 로드한다.

```markdown
---
name: session-start
description: 세션 시작 시 프로젝트 역할 자동 감지 및 하네스 스마트 로딩
hooks:
  - event: SessionStart
---

# 하네스 스마트 로더

세션 시작 시 아래 절차를 **자동으로** 수행하세요. 사용자에게 결과를 보여주지 않고 조용히 처리합니다.

## 1단계: 개인 하네스 디렉토리 확인

`~/.claude/harness/personal/` 디렉토리가 존재하는지 확인합니다.
- 존재하지 않으면: 아무것도 하지 않음 (사용자가 `/harness setup`을 실행해야 함)
- 존재하면: 2단계로 진행

## 2단계: 역할 감지

`~/.claude/harness/personal/meta.json` 파일을 읽어 현재 프로젝트의 캐시된 역할 정보를 확인합니다.

### 캐시 히트 (meta.json에 현재 프로젝트 경로가 있고, files_hash가 일치)
- 캐시된 roles 배열을 사용

### 캐시 미스 (프로젝트 경로가 없거나 files_hash 불일치)
- 프로젝트 루트의 파일 목록을 스캔 (1단계 깊이만, `ls` 수준)
- 스캔 결과를 바탕으로 프로젝트의 기술 스택을 판별:
  - **backend**: 서버사이드 언어/프레임워크가 주요 구성인 경우
  - **frontend**: UI 프레임워크, 클라이언트 앱이 주요 구성인 경우
  - **hardware**: 임베디드, 저수준 시스템 프로그래밍이 주요 구성인 경우
  - 복수 역할 가능 (모노레포 등)
- 판별 결과를 meta.json에 캐싱

### 역할 판별 기준 (유연한 카테고리 기반)

특정 확장자를 하드코딩하지 않는다. 프로젝트 파일 구성을 보고 다음 카테고리 중 해당하는 것을 판별한다:

| 카테고리 | 판별 기준 |
|----------|----------|
| backend | 서버사이드 언어/프레임워크, 빌드 도구, API 정의 파일 |
| frontend | UI 프레임워크, 클라이언트 번들러, 모바일 프레임워크 |
| hardware | 저수준 언어, 임베디드 빌드 시스템, MCU/RTOS 관련 |

## 3단계: 역할별 규칙 로드

감지된 역할에 해당하는 규칙 파일의 내용을 컨텍스트로 인식합니다:
- `common` → 항상 (CLAUDE.md에서 처리됨)
- 감지된 각 역할 → 해당 `roles/{role}.md` 파일 내용을 참조

## 4단계: 개인 하네스 스마트 로딩

`~/.claude/harness/personal/rules/` 디렉토리의 하네스 파일들을 스캔합니다.

### 로드 조건 (이중 필터링)
1. **역할 필터**: 하네스의 `role` 필드가 `common`이거나 감지된 역할과 일치
2. **사용 빈도 필터**: `last_triggered`가 30일 이내

### 로드 방법
- 조건에 맞는 하네스 파일의 규칙 내용을 읽어 현재 세션의 컨텍스트로 인식
- 비활성(30일 초과) 하네스는 무시하되, 보관은 유지

## 중요

- 이 과정은 **조용히** 수행합니다. 사용자에게 로딩 과정을 출력하지 않습니다.
- 오류 발생 시에도 세션을 중단하지 않습니다. 하네스 로딩 실패는 무시합니다.
```

- [ ] **Step 2: 검증 — hook frontmatter 형식 확인**

hook의 YAML frontmatter가 기존 freegrow-git 플러그인의 `commit-rules.md`와 동일한 패턴인지 확인한다. `event: SessionStart`가 올바른 Claude Code hook 이벤트인지 확인.

- [ ] **Step 3: 커밋**

```bash
git add plugins/freegrow-harness/hooks/session-start.md
git commit -m "feat: session-start hook 구현 (역할 감지 + 스마트 로딩)"
```

---

### Task 4: post-tool hook (실시간 패턴 감지)

**Files:**
- Create: `plugins/freegrow-harness/hooks/post-tool.md`

- [ ] **Step 1: post-tool.md 생성**

```markdown
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
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/hooks/post-tool.md
git commit -m "feat: post-tool hook 구현 (실시간 패턴 감지 및 하네스 등록 제안)"
```

---

### Task 5: /harness setup 커맨드

**Files:**
- Create: `plugins/freegrow-harness/commands/setup.md`

- [ ] **Step 1: setup.md 생성**

```markdown
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
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/commands/setup.md
git commit -m "feat: /harness setup 커맨드 구현"
```

---

### Task 6: /harness list 커맨드

**Files:**
- Create: `plugins/freegrow-harness/commands/list.md`

- [ ] **Step 1: list.md 생성**

```markdown
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

[역할: backend]
  - roles/backend.md의 규칙

[개인] (활성)
  backend-001  "규칙 내용 요약"          트리거 12회  (auto)
  common-002   "규칙 내용 요약"          트리거 5회   (manual)

[개인] (비활성 - 30일 미사용)
  frontend-003 "규칙 내용 요약"          마지막 사용: 2026-03-01
```

각 개인 하네스는 frontmatter의 `id`, `trigger_count`, `last_triggered`, `source`를 표시합니다.
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/commands/list.md
git commit -m "feat: /harness list 커맨드 구현"
```

---

### Task 7: /harness add 커맨드

**Files:**
- Create: `plugins/freegrow-harness/commands/add.md`

- [ ] **Step 1: add.md 생성**

```markdown
---
description: 수동으로 개인 하네스 등록
---

# Harness Add

사용자가 직접 하네스 규칙을 등록합니다.

## 인자

`$ARGUMENTS` - 등록할 규칙 내용 (따옴표로 감싸기)

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
이 규칙의 역할을 [판별된 역할]로 설정합니다. 맞나요? (y/n)
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
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/commands/add.md
git commit -m "feat: /harness add 커맨드 구현 (수동 하네스 등록)"
```

---

### Task 8: /harness promote 커맨드

**Files:**
- Create: `plugins/freegrow-harness/commands/promote.md`

- [ ] **Step 1: promote.md 생성**

```markdown
---
description: 개인 하네스를 역할별/공통으로 승격 (PR 생성)
---

# Harness Promote

개인 하네스를 역할별 또는 공통 하네스로 승격합니다. freegrow-harness 플러그인 레포에 PR을 자동 생성합니다.

## 인자

`$ARGUMENTS` - 승격할 하네스 ID (예: `backend-001`)

## 전제 조건

- `~/.claude/harness/personal/` 디렉토리가 존재해야 함
- 해당 ID의 하네스 파일이 `~/.claude/harness/personal/rules/`에 존재해야 함

## 실행 절차

### 1. 하네스 파일 읽기

`$ARGUMENTS`에서 하네스 ID를 추출하고, `~/.claude/harness/personal/rules/{id}.md` 파일을 읽습니다.

### 2. 승격 대상 확인

```
🚀 하네스 승격

ID: {id}
내용: "{규칙 내용}"
현재 역할: {role}

어디로 승격할까요?
1. {role}별 공통 → roles/{role}.md에 추가
2. 전사 공통 → CLAUDE.md에 추가
```

### 3. PR 생성 안내

Claude Code 플러그인은 별도의 Git 레포로 관리되므로, 승격은 직접 PR을 생성하는 방식으로 진행합니다.

```
📋 승격 내용을 준비했습니다.

아래 규칙을 freegrow-harness 플러그인 레포의 {대상 파일}에 추가하는 PR을 생성해주세요:

---
{규칙 내용}
---

레포: freegrowenterprise/freegrow-agents
대상 파일: plugins/freegrow-harness/{대상 파일 경로}

gh를 사용하여 PR을 생성할까요? (y/n)
```

### 4. PR 자동 생성 (사용자 승인 시)

```bash
# freegrow-agents 레포 경로에서 실행
cd {freegrow-agents 레포 경로}
git checkout -b harness/promote-{id}
# 대상 파일에 규칙 추가
git add {대상 파일}
git commit -m "feat: 하네스 승격 - {id} ({규칙 요약})"
git push origin harness/promote-{id}
gh pr create --title "[FEAT] 하네스 승격: {규칙 요약}" --body "## 📋 Summary
개인 하네스를 {대상} 하네스로 승격합니다.

## 하네스 내용
{규칙 내용}

## 출처
- 원본 ID: {id}
- 감지 방식: {source}
- 트리거 횟수: {trigger_count}회" --assignee @me
```

### 5. 결과 출력

```
✅ PR 생성 완료!

PR: #{PR번호} [FEAT] 하네스 승격: {규칙 요약}
URL: {PR URL}

팀 리뷰 후 머지되면 모두에게 적용됩니다.
```
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/commands/promote.md
git commit -m "feat: /harness promote 커맨드 구현 (승격 PR 자동 생성)"
```

---

### Task 9: /harness remove 커맨드

**Files:**
- Create: `plugins/freegrow-harness/commands/remove.md`

- [ ] **Step 1: remove.md 생성**

```markdown
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
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/commands/remove.md
git commit -m "feat: /harness remove 커맨드 구현"
```

---

### Task 10: harness-manage 스킬

**Files:**
- Create: `plugins/freegrow-harness/skills/harness-manage/SKILL.md`

- [ ] **Step 1: SKILL.md 생성**

이 스킬은 `alwaysApply: true`로 설정되어, 하네스 관련 키워드가 나오면 자동으로 컨텍스트에 로드된다.

```markdown
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
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/skills/harness-manage/SKILL.md
git commit -m "feat: harness-manage 스킬 구현 (하네스 관리 컨벤션)"
```

---

### Task 11: harness-detector 에이전트

**Files:**
- Create: `plugins/freegrow-harness/agents/harness-detector.md`

- [ ] **Step 1: harness-detector.md 생성**

```markdown
---
name: harness-detector
description: 패턴 감지 및 하네스 자동 생성 전문 에이전트. 반복 실수, 위험 패턴, 컨벤션 위반을 분석하고 적절한 하네스를 제안합니다.
---

# 하네스 디텍터 에이전트

당신은 Freegrow 하네스 프레임워크의 패턴 감지 전문가입니다.

## 역할

1. 세션 중 발생한 패턴 로그를 분석하여 하네스 후보를 식별
2. 식별된 패턴에 대해 적절한 하네스 규칙 또는 Hook을 생성
3. 하네스의 역할(role)을 판별하고 적절한 형태(type)를 결정

## 패턴 분석 기준

### 반복 수정 패턴
- 같은 유형의 에러를 3회 이상 수정
- 같은 파일을 5회 이상 반복 편집
- 같은 린트 에러를 반복 수정

### 위험 패턴
- 시크릿/API 키 하드코딩
- 위험한 셸 명령어 (rm -rf, force push 등)
- 보안 취약점 패턴

### 컨벤션 위반
- 네이밍 규칙 불일치
- 코드 스타일 불일치
- 아키텍처 패턴 위반

## 하네스 생성 규칙

### 규칙(rule) 생성 시
- 명확하고 구체적인 행동 지침으로 작성
- "~하지 마라" 또는 "~할 때는 ~해라" 형식
- 왜 이 규칙이 필요한지 한 줄 설명 포함

### Hook 생성 시
- 자동으로 검증 가능한 패턴에 대해서만
- 실행 가능한 bash 스크립트 형태
- 실패 시 사용자에게 명확한 메시지 출력

## 역할 판별 기준

하네스의 역할은 패턴이 발생한 컨텍스트에서 판별합니다:
- 특정 언어/프레임워크에만 해당 → 해당 역할
- 범용적인 개발 습관 → common

## 중요

- 너무 사소한 패턴은 하네스로 만들지 않습니다.
- 사용자의 의도적인 선택을 존중합니다 (한 번 거절된 패턴은 다시 제안하지 않음).
- 하네스 규칙은 간결하게 유지합니다 (3줄 이내 권장).
```

- [ ] **Step 2: 커밋**

```bash
git add plugins/freegrow-harness/agents/harness-detector.md
git commit -m "feat: harness-detector 에이전트 구현 (패턴 감지 + 하네스 생성)"
```

---

### Task 12: 통합 검증 및 최종 커밋

**Files:**
- Verify: 전체 `plugins/freegrow-harness/` 디렉토리 구조

- [ ] **Step 1: 디렉토리 구조 검증**

```bash
find plugins/freegrow-harness/ -type f | sort
```

예상 결과:
```
plugins/freegrow-harness/CLAUDE.md
plugins/freegrow-harness/agents/harness-detector.md
plugins/freegrow-harness/commands/add.md
plugins/freegrow-harness/commands/list.md
plugins/freegrow-harness/commands/promote.md
plugins/freegrow-harness/commands/remove.md
plugins/freegrow-harness/commands/setup.md
plugins/freegrow-harness/hooks/post-tool.md
plugins/freegrow-harness/hooks/session-start.md
plugins/freegrow-harness/plugin.json
plugins/freegrow-harness/roles/backend.md
plugins/freegrow-harness/roles/frontend.md
plugins/freegrow-harness/roles/hardware.md
plugins/freegrow-harness/skills/harness-manage/SKILL.md
```

- [ ] **Step 2: plugin.json 검증**

plugin.json의 `commands`, `skills`, `hooks` 경로가 실제 파일과 일치하는지 확인.

- [ ] **Step 3: 각 파일의 YAML frontmatter 형식 검증**

모든 hook, command, skill 파일의 frontmatter가 올바른 형식인지 확인:
- hooks: `name`, `description`, `hooks[].event` 필드 존재
- commands: `description` 필드 존재
- skills: `name`, `description` 필드 존재

- [ ] **Step 4: 최종 커밋 (필요 시)**

검증 중 수정사항이 있으면 커밋:

```bash
git add plugins/freegrow-harness/
git commit -m "fix: freegrow-harness 플러그인 통합 검증 및 수정"
```
