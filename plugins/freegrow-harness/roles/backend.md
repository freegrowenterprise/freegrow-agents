# Backend 역할 하네스

서버사이드 기술 스택(Java, Kotlin, Go, Python, Node.js 등) 프로젝트에서 로드됩니다.

## 규칙

### Spring Boot / Java
- Java 21 이상을 사용해라. 이전 버전으로 설정하지 마라.
- 환경별 설정 파일(`application-{local,dev,prod}.yml`)을 반드시 분리해라. 하나의 application.yml에 모든 환경을 넣지 마라.
- `application.yml`에 시크릿을 직접 넣지 마라. 환경변수(`${ENV_VAR}`)로 주입해라.
- Entity 클래스에 Lombok `@Getter`, `@Setter`, `@Builder`를 활용해라.
- JPA Entity에 `@Column(nullable = false)` 등 제약조건을 명시해라.

### API 설계
- REST API 응답에 에러 코드와 메시지를 포함하는 공통 응답 형식을 사용해라.
- API 문서화를 위해 springdoc-openapi를 활용해라.

### 데이터베이스
- JPA 쿼리 작성 시 N+1 문제를 반드시 확인해라. `@EntityGraph` 또는 `fetch join`을 사용해라.
- DB 마이그레이션이 필요한 변경은 마이그레이션 스크립트를 함께 작성해라.

### 배포
- Dockerfile을 유지하고, 멀티스테이지 빌드를 사용해라.
- Sentry Spring Boot Starter를 통해 에러 모니터링을 연동해라.
