# Local LLM Server

기존 홈 게임 서버에 로컬 LLM 환경을 함께 구축하고, 제한된 하드웨어에서 실제 활용 가능성을 검증하는 프로젝트입니다.

## 현재 상태

> **Phase 0 — 설계 및 벤치마크 준비**

- GitHub 저장소 및 문서 구조 구성
- 현재 서버 사양과 운영 제약 정리
- Ollama 모델별 성능 측정 계획 수립
- 아직 Ollama, Open WebUI, RAG, OpenClaw는 설치하지 않음

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

1. 현재 16GB 환경에서 Ollama 모델 벤치마크
2. 사용할 모델과 자원 제한 기준 결정
3. Open WebUI 구축
4. RAG 구성 및 개인 문서 검색 실험
5. OpenClaw 연동 실험
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

