# Local LLM Server

기존 Ubuntu 홈 게임 서버에 로컬 LLM 환경을 함께 구축하고, 제한된 하드웨어에서 실제 활용 가능성을 검증하는 프로젝트입니다.

## 현재 상태

마지막 업데이트: 2026-09-03

> **Phase 1 — Ollama 구동 및 OpenClaw 기본 연동 완료, 벤치마크 진행 중**

- Ollama 설치 및 로컬 API 동작 확인
- `qwen3:4b` 초기 구동 확인
- `qwen3:8b` 다운로드 및 한국어 추론 확인
- OpenClaw `2026.8.2` 설치
- OpenClaw 기본 모델을 `ollama/qwen3:8b`로 설정
- 정량 벤치마크와 게임 서버 동시 실행 테스트는 진행 예정
- Open WebUI와 RAG는 아직 미구현

## 서버 환경

| 구분 | 사양 |
| --- | --- |
| 기기 | Minisforum SER8 |
| CPU | AMD Ryzen 7 8745HS (8코어 16스레드) |
| GPU | Radeon 780M 내장 그래픽 |
| 메모리 | Micron DDR5 16GB |
| 저장장치 | Samsung SSD 512GB |
| 운영체제 | Ubuntu Desktop 24.04.4 LTS |
| 기존 서비스 | Docker, Pterodactyl Wings |
| 운영 방식 | 24시간 홈 서버 |
| 게임 서버 | Minecraft, Palworld, Project Zomboid |

## 진행 순서

1. Ollama 모델 구동 및 벤치마크
2. 게임 서버와의 동시 실행 안정성 확인
3. Open WebUI 구축
4. RAG 구성 및 개인 문서 검색 실험
5. OpenClaw 도구·자동화 범위 확장
6. Pterodactyl 및 Discord 연동

## 문서

- [현재 상황](docs/current-status.md)
- [목표 아키텍처](docs/architecture.md)
- [Ollama 벤치마크 계획](docs/benchmark-plan.md)
- [로드맵](docs/roadmap.md)

## 원칙

- 게임 서버의 안정성을 우선한다.
- 16GB 메모리에서 무리하게 큰 모델을 운영하지 않는다.
- 실제 측정 결과를 기준으로 모델과 구성을 결정한다.
- 설치 과정, 설정 변경, 실패 사례까지 기록한다.

## AI 사용 고지

이 프로젝트의 설계와 문서 작성에는 ChatGPT의 도움을 사용했습니다. 최종 구성과 검증은 실제 서버 환경을 기준으로 진행합니다.
