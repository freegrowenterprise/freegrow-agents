# Freegrow Harness Framework Design Spec

## 개요

사내 모든 개발자(백엔드, 프론트, 하드웨어)가 Claude Code를 사용할 때, 자주 하는 실수나 이상한 패턴을 자동으로 감지하여 하네스(규칙/Hook)로 만들어주는 자기 진화형 하네스 프레임워크.

### 핵심 원칙

- **자동 감지 우선**: 대부분의 개발자는 하네스가 뭔지 모르는 상태. 시스템이 알아서 감지하고 제안.
- **수동 등록 가능**: 하네스를 이해한 사용자는 직접 등록 가능.
- **계층 구조**: 회사 공통 > 역할별 > 개인. 공통은 "헌법"처럼 한번 잡으면 거의 안 바뀜.
- **스마트 로딩**: 전부 보관하되, 현재 작업과 관련된 하네스만 로드하여 컨텍스트 낭비 최소화.

---

## 1. 전체 아키텍처

### 배포 방식

단일 Claude Code 플러그인(`freegrow-harness`)으로 배포. `claude plugin add`로 설치하면 모든 프로젝트에서 자동 적용.

### 플러그인 구조

```
freegrow-harness/
├── plugin.json
├── CLAUDE.md                    # 공통 하네스 규칙 (항상 로드)
├── hooks/
│   ├── session-start.md         # 역할 자동 감지 + 스마트 로딩
│   └── post-tool.md             # 실시간 패턴 감지
├── roles/
│   ├── backend.md               # 서버사이드 기술 스택 감지 시 로드
│   ├── frontend.md              # UI/클라이언트 기술 스택 감지 시 로드
│   └── hardware.md              # 임베디드/저수준 기술 스택 감지 시 로드
├── skills/
│   ├── harness-manage/SKILL.md  # /harness list, add, promote, remove
│   └── harness-setup/SKILL.md   # /harness setup
├── commands/
│   └── harness.md               # /harness 진입점
└── agents/
    └── harness-detector.md      # 패턴 감지 + 하네스 자동 생성
```

### 개인 하네스 저장 위치

```
~/.claude/harness/personal/
├── meta.json                    # 프로젝트별 역할 캐시 + 하네스 사용 통계
├── pattern-log.jsonl            # 패턴 감지 로그
├── rules/
│   ├── backend-001.md
│   ├── frontend-003.md
│   └── common-002.md
└── hooks/
    ├── backend-lint-check.sh
    └── secret-scan.sh
```

### 동작 흐름

1. Claude Code 세션 시작
2. `session-start` hook이 프로젝트 파일 스캔 → 역할 판별
3. 공통 CLAUDE.md + 해당 역할 규칙 + 관련 개인 하네스만 로드
4. 작업 중 `post-tool` hook이 실시간 패턴 감지
5. 문제 패턴 발견 시 → 하네스 등록 제안

---

## 2. 역할 자동 감지

### 감지 방식

특정 파일 확장자를 하드코딩하지 않는다. 대신 **카테고리 기반 유연한 판별**을 사용한다.

| 카테고리 | 판별 기준 | 예시 (고정 아님) |
|----------|----------|-----------------|
| backend | 서버사이드 언어/프레임워크, 빌드 도구, API 정의 파일 | Java, Kotlin, Go, Python(Django/FastAPI), Spring, Node.js(Express) |
| frontend | UI 프레임워크, 클라이언트 번들러, 모바일 프레임워크 | Flutter, React, Vue, Angular, SwiftUI, Dart, TypeScript(프론트) |
| hardware | 저수준 언어, 임베디드 빌드 시스템, MCU/RTOS 관련 | C, C++(임베디드), Assembly, Makefile, CMake, RTOS 설정 |

**판별 로직**: session-start hook이 프로젝트 파일을 스캔한 뒤, LLM에게 "이 프로젝트의 기술 스택은 backend/frontend/hardware 중 어디에 해당하는가?"를 질의하여 유연하게 판별한다. 새로운 언어/프레임워크가 등장해도 플러그인 수정 없이 대응 가능.

### 복수 역할

모노레포 등에서 여러 역할이 감지되면 **전부 로드**.

### 캐싱

감지 결과를 `meta.json`에 캐싱. 프로젝트 파일 구조의 해시를 저장하고, 변경 시에만 재스캔.

```json
{
  "projects": {
    "/Users/dev/my-app": {
      "roles": ["frontend", "backend"],
      "detected_at": "2026-04-06",
      "files_hash": "a3f2c1..."
    }
  }
}
```

---

## 3. 실시간 패턴 감지

### 트리거

`PostToolUse` hook으로 Claude Code의 모든 도구 사용을 모니터링.

### 감지 유형

| 유형 | 설명 | 하네스 형태 |
|------|------|------------|
| 반복 수정 | 같은 유형의 에러를 3번 이상 수정 | Hook (자동 검증) |
| 위험 패턴 | `rm -rf`, `force push`, 시크릿 하드코딩 | Hook (차단) |
| 컨벤션 위반 | 커밋 메시지, 네이밍 규칙 불일치 | 규칙 (텍스트) |
| 비효율 패턴 | 같은 파일 5번 이상 반복 수정 | 규칙 (가이드) |
| 언어별 안티패턴 | null 체크 누락, 메모리 해제 누락 등 | 규칙 + Hook |

### 감지 흐름

```
[Claude가 도구 사용]
    │
    ▼
[post-tool hook 실행]
    │
    ├── 패턴 로그에 기록 (pattern-log.jsonl)
    │
    ├── 같은 패턴 3회 이상?
    │      ├── YES → "이 패턴을 하네스로 등록할까요?"
    │      │          ├── 승인 → 하네스 생성
    │      │          └── 거절 → 무시 목록에 추가
    │      └── NO → 계속 카운팅
    │
    └── 위험 패턴? (시크릿, rm -rf 등)
           └── 즉시 경고 (횟수 무관)
```

### 하네스 등록 제안 UI

```
⚡ 패턴 감지: [패턴 설명]을 [N]회 반복했습니다.

제안 하네스:
  "[규칙 내용]"

  → 개인 하네스로 등록할까요? (y/n)
  → [역할]별 공통으로 승격 제안할까요? (y/n)
```

### 패턴 로그 형식

```jsonl
{"ts":"2026-04-06T10:30:00","tool":"Edit","pattern":"lint_error_repeat","file":"src/Main.java","count":3}
{"ts":"2026-04-06T10:32:00","tool":"Bash","pattern":"dangerous_cmd","cmd":"rm -rf /tmp/*","count":1}
```

---

## 4. 하네스 저장 및 스마트 로딩

### 하네스 파일 형식

```markdown
---
id: backend-001
type: rule              # rule | hook
role: backend           # common | backend | frontend | hardware
created: 2026-04-06
trigger_count: 12
last_triggered: 2026-04-05
source: auto            # auto | manual
---

[하네스 규칙 내용]
```

### 스마트 로딩 로직

이중 필터링으로 컨텍스트 낭비 최소화:

1. **역할 필터**: 감지된 역할과 `common`만 로드
2. **사용 빈도 필터**: `last_triggered`가 30일 이내인 것만 로드. 30일 초과는 비활성 처리.

```
세션 시작
    │
    ├── 역할 감지: backend
    │
    ├── 1차 필터 (역할):
    │   ├── role == "common" → 후보
    │   ├── role == "backend" → 후보
    │   ├── role == "frontend" → 스킵
    │   └── role == "hardware" → 스킵
    │
    └── 2차 필터 (사용 빈도):
        ├── last_triggered 30일 이내 → 로드
        └── 30일 초과 → 비활성 (필요 시 온디맨드)
```

---

## 5. 커맨드 인터페이스

### `/harness setup`

최초 1회 실행. 개인 하네스 디렉토리(`~/.claude/harness/personal/`) 생성 및 초기화.

### `/harness list`

현재 로드된 하네스 목록 확인. 공통/역할/개인 구분 표시, 트리거 횟수, 비활성 개수 표시.

### `/harness add "<규칙>"`

수동으로 개인 하네스 등록. 등록 후 역할별/공통 승격 여부 질문.

### `/harness promote <id>`

개인 하네스를 역할별/공통으로 승격. `freegrow-harness` Git 레포에 PR 자동 생성.

### `/harness remove <id>`

하네스 삭제.

---

## 6. 하네스 계층 및 관리

### 계층 구조

```
공통 하네스    →  플러그인 CLAUDE.md에 포함. Git PR로 관리. 거의 안 바뀜.
역할별 하네스  →  플러그인 roles/ 디렉토리. Git PR로 관리.
개인 하네스    →  ~/.claude/harness/personal/. 로컬 자유 관리. 자동 생성.
```

### 공통/역할 하네스 수정 프로세스

1. 개발자가 `freegrow-harness` 레포에서 브랜치 생성
2. 규칙 수정 → PR 작성 → 리뷰 → 머지
3. 팀원들이 `claude plugin update freegrow-harness`로 동기화

### 개인 → 공통 승격

`/harness promote` 커맨드로 PR 자동 생성. 팀 리뷰 후 머지되면 모두에게 적용.

---

## 7. 향후 확장 가능성

- 팀 단위 패턴 분석: 여러 개발자의 패턴 로그를 집계하여 팀 공통 문제 식별
- 하네스 효과 측정: 하네스 적용 전후 실수 빈도 비교
- 하네스 마켓플레이스: 다른 팀/회사의 하네스 공유
