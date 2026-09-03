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
- [x] Ubuntu 기본 상태 조회 Agent 동작 확인
- [x] Pterodactyl 읽기 전용 상태 조회 구현 및 자연어 호출 확인

## 진행 중

- [ ] Pterodactyl `start` / `stop` / `restart` 제어 명령 안정화
- [ ] 상태 변경 작업의 사용자 승인 흐름 검증
- [ ] OpenClaw Workspace 파일 축소와 Context 추가 최적화

서버 제어 기능은 정확한 대상 선택을 DRY-RUN으로 확인한 단계입니다. 자연어 요청에서 명령을 안정적으로 선택하고 승인 후 실행하는 전체 흐름은 아직 완료되지 않았습니다.

## 예정

- [ ] Open WebUI와 OpenClaw의 접근 범위 및 운영 방식 정리
- [ ] 외부 접속이 필요할 경우 HTTPS 적용
- [ ] Reverse Proxy 구성
- [ ] 도메인 연결
- [ ] 외부 접속 인증과 보안 정책 검토
- [ ] OpenClaw Agent 기능 확장
- [ ] 필요 시 모니터링 및 로그 분석 자동화

완료 여부는 실제 SER8 서버에서 직접 동작을 확인한 결과를 기준으로 갱신합니다.
