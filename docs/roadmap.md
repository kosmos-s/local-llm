# 로드맵

## 완료

- [x] Ubuntu 호스트에 Ollama 설치
- [x] Radeon 780M Vulkan GPU 추론 활성화 및 `100% GPU` 확인
- [x] Qwen3 계열 비교 후 Qwen3.5 중심으로 모델 전환
- [x] `qwen3.5:4b`, `qwen3.5:9b`, `bge-m3` 역할 분리
- [x] Docker 기반 Open WebUI 구축 및 채팅 확인
- [x] BGE-M3 기반 Knowledge / RAG 구축 및 출처 검증
- [x] OpenClaw 설치와 loopback Gateway 구성
- [x] OpenClaw 기본 모델을 `ollama/qwen3.5:4b`로 연결
- [x] OpenClaw Context와 Workspace 프롬프트 최적화
- [x] Exec 권한을 `allowlist`, `on-miss`, `askFallback=deny` 정책으로 변경
- [x] Ubuntu 기본 상태 조회 Agent 동작 확인
- [x] Pterodactyl Client API 연동
- [x] Pterodactyl 읽기 전용 상태 조회 구현 및 자연어 호출 확인
- [x] Pterodactyl 서버 시작·종료·재시작 실동작 확인
- [x] 상태 변경 전 사용자 승인과 제어 후 상태 재검증 확인
- [x] `ser8-health` 통합 상태 점검 구현
- [x] OpenClaw 30분 내부 Heartbeat 실행 확인
- [x] Ubuntu 재부팅 후 주요 서비스 자동 복구 검증
- [x] Local AI 설정·OpenClaw DB·관리 도구 백업
- [x] Open WebUI volume과 Knowledge / RAG 원본 백업
- [x] OpenClaw SQLite DB consistency 검사
- [x] GPG AES256 암호화 백업과 복호화 검증
- [x] Windows PC 외부 백업과 SHA256 무결성 검증
- [x] 백업 내부 `RECOVERY.md` 복구 절차 작성

## 다음 단계

- [ ] OpenClaw 응답 속도 추가 최적화
- [ ] Agent 로그와 장애 대응 방식 개선
- [ ] 장기 운영 안정성 확인

## 보류

- [ ] Discord 장애 알림 연동

Heartbeat는 현재 내부 점검만 수행합니다. 외부 알림은 필요성과 전달 경로를 정한 뒤 추가할 예정입니다.

## 프로젝트 범위에서 제외

- Pterodactyl Panel DB 전체 백업
- Wings 전체 설정 백업
- 게임 서버 월드 및 세이브 전체 백업

위 항목은 기존 게임 서버 운영 영역으로 분리하고, 이 저장소에서는 Local LLM·RAG·OpenClaw Agent 환경의 운영과 복구만 다룹니다.

## 예정

- [ ] Open WebUI와 OpenClaw의 접근 범위 및 운영 방식 정리
- [ ] 외부 접속이 필요할 경우 HTTPS 적용
- [ ] Reverse Proxy 구성
- [ ] 도메인 연결
- [ ] 외부 접속 인증과 보안 정책 검토
- [ ] OpenClaw Agent 기능 확장
- [ ] 필요 시 모니터링 및 로그 분석 자동화
- [ ] 필요 시 기존 Discord/Pterodactyl 프로젝트와 연계

완료 여부는 실제 SER8 서버에서 직접 동작을 확인한 결과를 기준으로 갱신합니다.
