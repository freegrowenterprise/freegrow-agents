# Hardware 역할 하네스

임베디드/저수준 기술 스택(C, C++, Assembly, RTOS 등) 프로젝트에서 로드됩니다.

## 규칙

### Zephyr / nRF52840
- `CMakeLists.txt`에 Zephyr HINTS 경로를 반드시 지정해라.
- `prj.conf`에 BLE, GPIO, SPI/I2C 등 사용하는 주변장치를 명시적으로 설정해라.
- Zephyr 버전 업그레이드 시 `west update`를 실행하고 빌드 테스트를 반드시 해라.

### C 코드 품질
- 동적 메모리 할당(`malloc`) 후 반드시 해제(`free`)를 확인해라. 메모리 누수를 만들지 마라.
- 인터럽트 핸들러(ISR)에서 블로킹 함수를 호출하지 마라.
- 하드웨어 레지스터 접근 시 `volatile` 키워드를 사용해라.
- 모든 `.h` 헤더 파일에 include guard(`#ifndef`/`#define`/`#endif`)를 사용해라.

### 프로젝트 구조
- `src/`와 `include/` 디렉토리를 분리하고, 기능별 모듈(BLE, UWB, core 등)로 나눠라.
- BLE advertising 설정 변경 시 Extended Advertising 호환성을 확인해라.

### 디버깅
- UART 로깅을 활성화하고, `printk` 또는 Zephyr logging API를 사용해라.
- Watchdog 타이머를 설정하여 펌웨어 hang 시 자동 리셋되도록 해라.
