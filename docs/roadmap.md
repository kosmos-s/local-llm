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
- [x] Exec 권한을 `allowlist`와 `on-miss` 승인 정책으로 변경
- [x] Ubuntu 기본 상태 조회 Agent 동작 확인
- [x] Pterodactyl Client API 연동
- [x] Pterodactyl 읽기 전용 상태 조회 구현 및 자연어 호출 확인
- [x] Pterodactyl 서버 시작·종료·재시작 실동작 확인
- [x] 상태 변경 전 사용자 승인과 제어 후 상태 재검증 확인

## 다음 단계

- [ ] OpenClaw 응답 속도 추가 최적화
- [ ] Agent 로그와 장애 대응 방식 개선
- [ ] 장기 운영 안정성 확인

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
