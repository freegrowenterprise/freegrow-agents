# Freegrow Enterprise Claude Code Plugins

Freegrow 내부 개발자용 Claude Code 플러그인 마켓플레이스입니다.

## 설치

### 마켓플레이스 등록

```bash
/plugin marketplace add freegrowenterprise/freegrow-claude-plugins
```

### 개별 플러그인 설치

```bash
/plugin install <plugin-name>
```

예시:
```bash
/plugin install backend-development
/plugin install cloud-infrastructure
/plugin install embedded-development
```

### 개발 중인 코드로 설치 (플러그인 개발자용)

안정 버전(main)이 아닌 개발 브랜치의 최신 코드를 테스트하려면 브랜치를 지정하여 설치합니다.

#### develop 브랜치 (개발 최신)

```bash
/plugin marketplace add https://github.com/freegrowenterprise/freegrow-claude-plugins.git#develop
```

#### 특정 feature 브랜치

```bash
/plugin marketplace add https://github.com/freegrowenterprise/freegrow-claude-plugins.git#feature/2-freegrow-git-plugin-update
```

#### 안정 버전으로 복귀

개발 브랜치 마켓플레이스를 삭제하고 안정 버전으로 재등록합니다:

```bash
/plugin marketplace remove freegrow-claude-plugins
/plugin marketplace add freegrowenterprise/freegrow-claude-plugins
```

> **참고**: 브랜치 미지정 시 기본 브랜치(main)의 안정 코드가 사용됩니다.

### 슬래시 명령어

마켓플레이스 설치 후 사용 가능한 명령어:

| 명령어 | 설명 |
|--------|------|
| `/plugin-install` | 플러그인 설치 가이드 (역할별 추천 포함) |
| `/plugin-list` | 설치 가능한 플러그인 목록 및 상세 정보 |
| `/plugin-setup` | 마켓플레이스 초기 설정 및 팀 구성 가이드 |

## 플러그인 출처

| 출처 | 플러그인 |
|------|----------|
| Freegrow 자체 개발 | `embedded-development`, `flutter-development`, `freegrow-git` `freegrow-harness` |
| [wshobson/agents](https://github.com/wshobson/agents) 기반 + 확장 | `multi-platform-apps` |
| [wshobson/agents](https://github.com/wshobson/agents) 포크 | 나머지 8개 플러그인 |

## 플러그인 목록

| # | 플러그인 | 설명 | 에이전트 | Skills | Commands |
|---|----------|------|----------|--------|----------|
| 1 | [arm-cortex-microcontrollers](#1-arm-cortex-microcontrollers) | ARM Cortex MCU 펌웨어 개발 | 1 | - | - |
| 2 | [backend-development](#2-backend-development) | 백엔드 아키텍처, Event Sourcing, GraphQL | 5 | 9 | 1 |
| 3 | [cloud-infrastructure](#3-cloud-infrastructure) | 클라우드, Kubernetes, Terraform | 7 | 8 | - |
| 4 | [code-documentation](#4-code-documentation) | 기술 문서화 및 코드 분석 | 3 | - | 2 |
| 5 | [code-refactoring](#5-code-refactoring) | 코드 품질 개선, 레거시 현대화 | 2 | - | 3 |
| 6 | [code-review-ai](#6-code-review-ai) | AI 기반 코드 리뷰 | 1 | - | 1 |
| 7 | [embedded-development](#7-embedded-development) | 임베디드, RTOS, ESP32, BLE | 5 | 5 | 5 |
| 8 | [flutter-development](#8-flutter-development) | Flutter 클린 아키텍처 + Riverpod 3.0 | 1 | 4 | - |
| 9 | [freegrow-git](#9-freegrow-git) | **Freegrow 전용** Git 워크플로우 자동화 | - | 1 | 3 |
| 10 | [frontend-mobile-development](#10-frontend-mobile-development) | React, Next.js, React Native | 2 | 4 | 1 |
| 11 | [git-pr-workflows](#11-git-pr-workflows) | Git 워크플로우, PR 관리 (범용) | 1 | - | 3 |
| 12 | [multi-platform-apps](#12-multi-platform-apps) | iOS, React Native 크로스플랫폼 | 5 | - | 1 |
| 13 | [freegrow-harness](#13-freegrow-harness) | **Freegrow 전용** 자기 진화형 하네스 프레임워크 | 1 | 1 | 5 |

---

### 1. arm-cortex-microcontrollers

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**ARM Cortex-M 마이크로컨트롤러 펌웨어/드라이버 개발 전문**

| 구분 | 내용 |
|------|------|
| 지원 플랫폼 | Teensy 4.x, STM32 (F4/F7/H7), nRF52, SAMD |
| 주요 기능 | HAL/베어메탈 드라이버, 인터럽트, DMA, 캐시 동기화, 메모리 배리어 |
| 포함 에이전트 | `arm-cortex-expert` |

**추천 대상**
- 펌웨어 엔지니어
- 임베디드 C/C++ 개발자
- MCU 드라이버 개발자
- 하드웨어 인터페이스 담당자

---

### 2. backend-development

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**백엔드 시스템 아키텍처 및 분산 시스템 개발 전문**

| 구분 | 내용 |
|------|------|
| API 설계 | REST, GraphQL, gRPC, WebSocket |
| 아키텍처 패턴 | Event Sourcing, CQRS, Saga, Clean Architecture, DDD |
| 워크플로우 | Temporal Python SDK |
| 개발 방법론 | TDD, BDD |
| 포함 에이전트 | `backend-architect`, `event-sourcing-architect`, `graphql-architect`, `tdd-orchestrator`, `temporal-python-pro` |
| Skills | API 설계 원칙, 마이크로서비스 패턴, CQRS 구현, Saga 오케스트레이션, Temporal 테스팅 |

**추천 대상**
- 백엔드 개발자
- 시스템 아키텍트
- API 설계 담당자
- 분산 시스템 개발자
- Temporal 워크플로우 개발자

---

### 3. cloud-infrastructure

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**클라우드 인프라 및 DevOps 전문**

| 구분 | 내용 |
|------|------|
| 클라우드 | AWS, Azure, GCP, 멀티클라우드, 하이브리드 |
| 컨테이너 | Kubernetes (EKS/AKS/GKE), Docker |
| Service Mesh | Istio, Linkerd |
| IaC | Terraform, OpenTofu, CDK, Pulumi |
| CI/CD | ArgoCD, Flux, GitHub Actions, GitLab CI |
| 포함 에이전트 | `cloud-architect`, `kubernetes-architect`, `terraform-specialist`, `deployment-engineer`, `network-engineer`, `hybrid-cloud-architect`, `service-mesh-expert` |
| Skills | Terraform 모듈, 비용 최적화, mTLS, Service Mesh 옵저버빌리티, 하이브리드 네트워킹 |

**추천 대상**
- DevOps 엔지니어
- SRE (Site Reliability Engineer)
- 클라우드 아키텍트
- 인프라 엔지니어
- 플랫폼 엔지니어

---

### 4. code-documentation

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**기술 문서화 및 코드 분석 전문**

| 구분 | 내용 |
|------|------|
| 문서 유형 | API 문서, 아키텍처 가이드, 튜토리얼, 온보딩 문서 |
| 분석 기능 | AI 기반 코드 분석, 복잡도 분석, 패턴 식별 |
| 포함 에이전트 | `code-reviewer`, `docs-architect`, `tutorial-engineer` |
| Commands | `code-explain`, `doc-generate` |

**추천 대상**
- 테크니컬 라이터
- 시니어 개발자 (문서화 담당)
- 팀 리드
- 온보딩 담당자
- 오픈소스 메인테이너

---

### 5. code-refactoring

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**코드 품질 개선 및 레거시 현대화 전문**

| 구분 | 내용 |
|------|------|
| 리팩토링 | 클린 코드, SOLID 원칙, 디자인 패턴 적용 |
| 레거시 | 기술 부채 분석, 모더나이제이션 전략 |
| 컨텍스트 | 장기 프로젝트 컨텍스트 복원 |
| 포함 에이전트 | `code-reviewer`, `legacy-modernizer` |
| Commands | `refactor-clean`, `tech-debt`, `context-restore` |

**추천 대상**
- 시니어 개발자
- 리팩토링 전담 개발자
- 레거시 시스템 담당자
- 기술 부채 해결 담당자

---

### 6. code-review-ai

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**AI 기반 코드 리뷰 및 품질 분석 전문**

| 구분 | 내용 |
|------|------|
| 분석 영역 | 보안 취약점, 성능 최적화, 아키텍처 패턴, 테스트 커버리지 |
| 도구 연동 | SonarQube, CodeQL, Semgrep, Snyk |
| 자동화 | PR 자동 분석, CI/CD 파이프라인 연동 |
| 포함 에이전트 | `architect-review` |
| Commands | `ai-review` |

**추천 대상**
- 코드 리뷰어
- 팀 리드
- QA 엔지니어
- 보안 담당자
- 품질 관리 담당자

---

### 7. embedded-development

> 출처: **Freegrow 자체 개발**

**임베디드 시스템 및 IoT 개발 전문**

| 구분 | 내용 |
|------|------|
| RTOS | FreeRTOS, Zephyr, ThreadX |
| IoT 플랫폼 | ESP32, nRF52 (BLE) |
| 임베디드 Linux | 드라이버, 빌드 시스템 |
| 프로토콜 | CAN, Modbus, I2C, SPI, UART, MQTT, CoAP |
| 저전력 설계 | 슬립 모드, 전력 도메인, 배터리 최적화 |
| 포함 에이전트 | `embedded-architect`, `rtos-expert`, `esp32-iot-expert`, `nrf-ble-expert`, `embedded-linux-expert` |
| Skills | RTOS 패턴, BLE 개발, 저전력 설계, 임베디드 빌드 시스템, 통신 프로토콜 |

**추천 대상**
- 임베디드 개발자
- IoT 개발자
- RTOS 개발자
- BLE 애플리케이션 개발자
- 하드웨어-소프트웨어 통합 담당자

---

### 8. flutter-development

> 출처: **Freegrow 자체 개발** (multi-platform-apps에서 분리)

**Flutter 클린 아키텍처 + Riverpod 3.0 기반 앱 개발 전문**

| 구분 | 내용 |
|------|------|
| 상태 관리 | Riverpod 3.0+ (flutter_riverpod, riverpod_annotation, hooks_riverpod) |
| 라우팅 | go_router |
| HTTP | http 패키지 |
| 에러 처리 | dartz (Either 타입), sentry_flutter |
| 아키텍처 | Clean Architecture (Domain → Data → Presentation) |
| 포함 에이전트 | `flutter-expert` |
| Skills | `flutter-architecture`, `flutter-platform-channels`, `flutter-state-management`, `flutter-testing-patterns` |

**주요 패턴**
- Entity/Model 분리 (Model extends Entity)
- Repository 패턴 (Interface → Implementation)
- UseCase 패턴 (UseCaseWithParams, UseCaseWithoutParams)
- HookConsumerWidget 기반 UI
- ResultFuture<T> = Future<Either<Failure, T>>

**추천 대상**
- Flutter 개발자
- Riverpod 사용자
- 클린 아키텍처 적용 프로젝트
- 모바일 앱 아키텍트

---

### 9. freegrow-git

> 출처: **Freegrow 자체 개발** | [📖 상세 문서](./plugins/freegrow-git/README.md)

**Freegrow 팀 전용 Git 워크플로우 자동화**

| 구분 | 내용 |
|------|------|
| 이슈/브랜치 | 이슈 생성 + 브랜치 생성 통합 (`/start`) |
| 커밋 | Freegrow 형식 강제 (`type: message (#이슈)`) |
| PR | 템플릿 기반 PR 생성 + Development Log 자동 정리 |
| Skills | `freegrow-conventions` |
| Commands | `start`, `com`, `pr` |

**커맨드 요약**

| 커맨드 | 기능 | 예시 |
|--------|------|------|
| `/freegrow-git:start` | 이슈 생성 + 브랜치 생성 | `/start [FEAT] 로그인 기능` |
| `/freegrow-git:com` | Freegrow 형식 커밋 | `/com feat: 구현 완료` |
| `/freegrow-git:pr` | PR 생성 (Development Log 자동) | `/pr` |

**워크플로우**
```
/start "[FEAT] 기능명" → 코드 작업 → /com → /pr
```

**추천 대상**
- Freegrow 내부 개발자 (필수)
- Freegrow Git 컨벤션을 따르는 프로젝트

> **Note**: 범용 Git 워크플로우는 [git-pr-workflows](#11-git-pr-workflows)를 사용하세요.

---

### 10. frontend-mobile-development

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**프론트엔드 및 React Native 모바일 개발 전문**

| 구분 | 내용 |
|------|------|
| 프레임워크 | React, Next.js 14+ (App Router), React Native |
| 스타일링 | Tailwind CSS, 디자인 시스템 |
| 상태 관리 | Redux Toolkit, Zustand, Jotai, React Query |
| 포함 에이전트 | `frontend-developer`, `mobile-developer` |
| Skills | Next.js App Router 패턴, React 상태 관리, React Native 아키텍처, Tailwind 디자인 시스템 |
| Commands | `component-scaffold` |

**추천 대상**
- 프론트엔드 개발자
- React Native 개발자
- UI/UX 구현 담당자
- 웹 애플리케이션 개발자

---

### 11. git-pr-workflows

> 출처: [wshobson/agents](https://github.com/wshobson/agents)

**Git 워크플로우 및 협업 프로세스 전문**

| 구분 | 내용 |
|------|------|
| Git | Conventional Commits, 브랜치 전략 (trunk-based, feature-branch) |
| PR | PR 최적화, 자동 리뷰, 템플릿 |
| 온보딩 | 신규 팀원 온보딩 프로세스 |
| 포함 에이전트 | `code-reviewer` |
| Commands | `git-workflow`, `pr-enhance`, `onboard` |

**추천 대상**
- 모든 개발자 (필수 권장)
- 팀 리드
- 프로젝트 매니저
- 신규 입사자 온보딩 담당자

---

### 12. multi-platform-apps

> 출처: [wshobson/agents](https://github.com/wshobson/agents) 기반 + **Freegrow 확장**

**iOS 및 React Native 크로스플랫폼 앱 개발 전문**

| 구분 | 내용 |
|------|------|
| iOS | Swift/SwiftUI, UIKit |
| React Native | Expo, React Navigation |
| 백엔드 | 모바일 BFF 아키텍처 |
| UI/UX | 모바일 디자인 시스템 |
| 포함 에이전트 | `ios-developer`, `mobile-developer`, `frontend-developer`, `backend-architect`, `ui-ux-designer` |
| Commands | `multi-platform` |

**추천 대상**
- iOS 네이티브 개발자
- React Native 개발자
- 크로스플랫폼 앱 개발자
- 모바일 앱 아키텍트

> **Note**: Flutter 개발은 [flutter-development](#8-flutter-development) 플러그인을 사용하세요.

---

### 13. freegrow-harness

> 출처: **Freegrow 자체 개발** | [📖 상세 문서](./plugins/freegrow-harness/README.md)

**자기 진화형 하네스 프레임워크 - 실수 패턴 자동 감지 및 규칙 생성**

| 구분 | 내용 |
|------|------|
| 하네스 엔지니어링 | AI 에이전트의 실수를 줄이는 규칙/가드레일 자동 생성 |
| 계층 구조 | 공통 → 역할별(backend/frontend/hardware) → 개인 |
| 역할 감지 | 프로젝트 파일 기반 자동 판별 (하드코딩 없음) |
| 패턴 감지 | PostToolUse hook으로 실시간 감지, 3회 반복 시 제안 |
| 스마트 로딩 | 역할 + 30일 사용 여부로 이중 필터링 |
| 포함 에이전트 | `harness-detector` |
| Skills | `harness-manage` |
| Commands | `setup`, `list`, `add`, `promote`, `remove` |

**사용법**

```bash
# 설치 + 초기 설정 (1번만)
/plugin install freegrow-harness
/harness setup

# 이후 평소대로 작업하면 자동으로 패턴 감지 → 하네스 생성
```

**실전 예시**

```
같은 실수 3번 반복
  → ⚡ "이 패턴을 하네스로 등록할까요?" (y/n)
  → 다음부터 Claude가 알아서 방지

개인 하네스가 유용하면
  → /harness promote → PR 자동 생성 → 팀 전체 적용
```

**추천 대상**
- Freegrow 내부 모든 개발자 (필수 권장)
- Claude Code를 팀 단위로 사용하는 조직

---

## 요구사항

- Claude Code 1.0.33+

## 로컬 테스트

```bash
claude --plugin-dir ./plugins/<plugin-name>
```

## Credits

- 기반 플러그인: [wshobson/agents](https://github.com/wshobson/agents)
- Freegrow Enterprise 확장 및 커스터마이징

## 라이선스

PROPRIETARY - Freegrow Enterprise 내부 사용 전용
