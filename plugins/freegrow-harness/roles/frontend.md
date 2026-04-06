# Frontend 역할 하네스

UI/클라이언트 기술 스택(Flutter, React, Vue, Angular 등) 프로젝트에서 로드됩니다.

## 규칙

### Flutter / Dart
- `pubspec.yaml`에 Flutter SDK 버전 제약을 반드시 명시해라.
- `analysis_options.yaml`을 유지하고 strict lint 규칙을 활성화해라.
- 상태 관리는 Riverpod 또는 Provider를 사용해라. `setState`만으로 복잡한 상태를 관리하지 마라.
- 라우팅은 `go_router`를 사용해라.

### 코드 품질
- API 엔드포인트 URL을 코드에 하드코딩하지 마라. 환경별 설정으로 분리해라.
- UI에 표시되는 문자열을 하드코딩하지 마라. 상수 파일 또는 i18n을 사용해라.
- `Equatable`을 활용하여 모델 클래스의 동등성 비교를 구현해라.

### 에러 처리
- `sentry_flutter`를 연동하여 프로덕션 에러를 추적해라.
- 네트워크 요청 실패 시 사용자에게 적절한 에러 메시지를 보여줘라. 빈 화면으로 방치하지 마라.

### 모노레포
- 모노레포 구조에서는 `packages/` 디렉토리에 공유 모듈을 분리해라.
- 공유 모듈 간 의존성을 명확히 하고, 순환 의존을 만들지 마라.
