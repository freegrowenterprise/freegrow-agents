# Freegrow Harness 플러그인

사내 개발자의 실수 패턴을 자동 감지하여 하네스(규칙/Hook)로 만들어주는 자기 진화형 Claude Code 플러그인입니다.

## 하네스 엔지니어링이란?

> Agent = Model + **Harness**

AI 모델을 감싸는 **규칙, 가드레일, 피드백 루프** 등 전체 운영 환경을 설계하는 분야입니다. 모델을 더 똑똑하게 만드는 것이 아니라, **실수해도 괜찮은 환경**을 만드는 것이 핵심입니다.

## 설치

```bash
claude plugin add github:freegrowenterprise/freegrow-agents/plugins/freegrow-harness
```

## 시작하기

```bash
# 1. 초기 설정 (최초 1회)
/harness setup

# 2. 끝. 평소대로 작업하면 됩니다.
#    → 실수 패턴 자동 감지 → 하네스 자동 제안
```

## 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/harness setup` | 초기 설정 (최초 1회) |
| `/harness list` | 현재 활성 하네스 목록 확인 |
| `/harness add "<규칙>"` | 수동으로 하네스 등록 |
| `/harness promote <id>` | 개인 → 공통/역할 승격 PR 생성 |
| `/harness remove <id>` | 하네스 삭제 |

## 동작 원리

### 계층 구조

```
공통 하네스 (CLAUDE.md)      ← 전사 공통. "헌법". 거의 안 바뀜.
    ↓
역할별 하네스 (roles/)       ← backend / frontend / hardware
    ↓
개인 하네스 (로컬)           ← 자동 감지로 쌓임. 자유롭게 관리.
```

### 역할 자동 감지

세션 시작 시 프로젝트 파일을 스캔하여 역할을 자동 판별합니다.

| 카테고리 | 판별 기준 |
|----------|----------|
| **backend** | 서버사이드 언어/프레임워크 (Java, Spring Boot, Go 등) |
| **frontend** | UI 프레임워크/모바일 (Flutter, React, Vue 등) |
| **hardware** | 임베디드/저수준 (C, Zephyr, CMake 등) |

- 특정 확장자를 하드코딩하지 않고 **유연하게 판별**합니다.
- 모노레포에서 여러 역할이 감지되면 **전부 로드**합니다.

### 실시간 패턴 감지

작업 중 `PostToolUse` hook이 실시간으로 패턴을 모니터링합니다.

```
[같은 실수 3회 반복]
    ↓
⚡ 패턴 감지: [설명]을 3회 반복했습니다.
   → 개인 하네스로 등록할까요? (y/n)
    ↓
[승인 시] ~/.claude/harness/personal/rules/backend-001.md 자동 생성
```

**감지 유형:**
- 반복 수정 (같은 에러를 3번 이상 수정)
- 위험 패턴 (시크릿 하드코딩, rm -rf 등 → 즉시 경고)
- 컨벤션 위반 (네이밍, 코드 스타일 불일치)
- 비효율 패턴 (같은 파일 5번 이상 반복 수정)

### 스마트 로딩

모든 하네스를 보관하되, **관련된 것만 로드**하여 컨텍스트 낭비를 최소화합니다.

1. **역할 필터**: 감지된 역할 + common만 로드
2. **사용 빈도 필터**: 30일 이내 사용된 것만 로드

## 하네스 관리

### 개인 하네스

- 자동 감지로 생성되거나 `/harness add`로 수동 등록
- `~/.claude/harness/personal/rules/`에 저장
- 별도의 관리 프로세스 없이 자유롭게 사용

### 공통/역할 하네스 수정

공통 규칙이나 역할별 규칙은 **팀 전체에 영향**을 주므로 PR 리뷰를 통해 관리합니다.

```
1. freegrow-agents 레포에서 규칙 수정
2. PR 작성 → 팀 리뷰 → 머지
3. 팀원: claude plugin update freegrow-harness
```

### 개인 → 공통 승격

```bash
/harness promote backend-001

# → freegrow-agents 레포에 PR 자동 생성
# → 팀 리뷰 후 머지되면 모두에게 적용
```

## 하네스 파일 형식

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

Java에서 nullable 값을 반환하는 메서드는 Optional로 감싸라.
```

## 실전 예시

### 예시 1: 백엔드 개발자의 하루

```
09:00  Spring Boot 프로젝트에서 Claude Code 실행
       → session-start hook이 Java/Gradle 감지 → backend 역할 로드
       → 공통 규칙 + 백엔드 규칙 + 개인 하네스 자동 적용

09:30  API 개발 중 application.yml에 DB 비밀번호 직접 입력
       → ⚠️ 위험 패턴 감지: 시크릿 하드코딩
       → "환경변수(${DB_PASSWORD})로 변경하세요"

10:00  같은 Entity에서 @Column(nullable) 빠뜨리는 실수 3번 반복
       → ⚡ 패턴 감지: Entity nullable 제약조건 누락을 3회 반복했습니다.
       → 제안 하네스: "JPA Entity 필드에 @Column 제약조건을 명시해라"
       → 개인 하네스로 등록할까요? (y) → backend-001.md 자동 생성

10:30  다음부터는 Entity 작성할 때 Claude가 자동으로 @Column 제약조건 포함
```

### 예시 2: 프론트 개발자가 처음 설치할 때

```bash
# 설치 + 설정
claude plugin add github:freegrowenterprise/freegrow-agents/plugins/freegrow-harness
/harness setup

# 결과:
# ✅ 하네스 초기 설정 완료!
# 📁 개인 하네스 위치: ~/.claude/harness/personal/
# 🔍 현재 프로젝트 역할: frontend (flutter)
#
# 다음 단계:
# - 작업하면서 패턴이 자동 감지됩니다.
# - /harness add "규칙" 으로 직접 등록할 수도 있습니다.
# - /harness list 로 현재 하네스를 확인하세요.
```

### 예시 3: 개인 하네스가 팀 규칙이 되는 과정

```bash
# 1. 한 달간 작업하며 개인 하네스가 자동으로 쌓임
/harness list
# 📋 활성 하네스
# [개인] (활성)
#   backend-001  "Entity에 @Column 제약조건 명시"     트리거 23회  (auto)
#   backend-002  "API 응답에 에러코드 포함"            트리거 15회  (auto)
#   backend-003  "Service 클래스에 @Transactional"    트리거 8회   (auto)

# 2. 트리거 많은 규칙 → 팀 전체에도 유용하겠다고 판단
/harness promote backend-001
# 🚀 "Entity에 @Column 제약조건 명시" → 백엔드 공통으로 승격
#    PR #15 생성 완료: "[FEAT] 하네스 승격: Entity @Column 제약조건 규칙"

# 3. 팀 리뷰 → 머지 → 다른 개발자들도 적용
# 다른 개발자: claude plugin update freegrow-harness
```

### 예시 4: 하드웨어 개발자가 수동으로 규칙 추가

```bash
# "우리 팀은 항상 이렇게 하는데, 자동 감지가 안 되네"
/harness add "nRF52840 프로젝트에서 BLE advertising interval은 100ms 이상으로 설정해라"

# ✅ hardware-001 등록 완료 (rule, hardware)
# → 이 규칙을 hardware 공통으로 승격 제안할까요? (y/n)
```

## 현재 포함된 규칙

| 카테고리 | 규칙 수 | 주요 내용 |
|----------|---------|----------|
| 공통 | 6개 | 시크릿 금지, Sentry 필수, CLAUDE.md/README 유지 |
| Backend | 11개 | Spring Boot/Java 21, 환경별 설정, JPA N+1, Dockerfile |
| Frontend | 11개 | Flutter/Riverpod, go_router, sentry_flutter, 모노레포 |
| Hardware | 10개 | Zephyr/nRF52840, C 메모리관리, ISR, 모듈 분리, Watchdog |

> 규칙 상세 내용은 `CLAUDE.md`, `roles/backend.md`, `roles/frontend.md`, `roles/hardware.md` 참조

## 관련 링크

- [디자인 스펙](../../docs/superpowers/specs/2026-04-06-freegrow-harness-design.md)
- [구현 계획](../../docs/superpowers/plans/2026-04-06-freegrow-harness.md)
- [Freegrow Git 플러그인](../freegrow-git/)
