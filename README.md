# SER8 로컬 LLM 서버

Minisforum SER8과 Ubuntu를 기반으로 직접 구축하고 운영하는 개인 로컬 LLM 서버 프로젝트입니다.

## 프로젝트 목적

기존 홈 서버에서 로컬 AI를 직접 구동하고, 실제 사용성과 자원 부담을 확인하기 위해 시작했습니다. Ollama와 OpenClaw를 연결한 현재 구성을 바탕으로 성능과 안정성을 검증하며 기능을 확장하고 있습니다.

## 서버 환경

| 항목 | 현재 환경 |
| --- | --- |
| 하드웨어 | Minisforum SER8 |
| CPU | AMD Ryzen 7 8745HS |
| 메모리 | DDR5 16GB |
| 운영체제 | Ubuntu Desktop 24.04.4 LTS |
| LLM 실행 환경 | Ollama |
| 주 모델 | `qwen3:8b` |
| AI 에이전트 | OpenClaw `2026.8.2` |

## 현재 구성

현재 실제로 동작을 확인한 구성만 표시합니다.

```mermaid
flowchart LR
    A["OpenClaw"] --> B["Ollama"]
    B --> C["qwen3:8b"]
```

## 현재 상태

### 완료

- [x] Ollama 설치 및 실행
- [x] `qwen3:8b` 실행
- [x] OpenClaw 설치
- [x] OpenClaw 기본 모델을 `ollama/qwen3:8b`로 설정
- [x] OpenClaw → Ollama 직접 추론 1차 성공

### 검증 예정

- [ ] 반복 추론 안정성 확인
- [ ] 재부팅 이후 동일 구성 재현
- [ ] CPU, 메모리, 스왑 사용량 측정
- [ ] 응답 속도와 게임 서버 동시 실행 성능 측정

## 동작 확인 결과

2026-09-02, SER8의 Ubuntu 환경에서 `OpenClaw → Ollama → qwen3:8b` 경로로 실제 추론 요청과 응답에 성공했습니다. 기본 모델 설정뿐 아니라 전체 추론 경로가 동작하는 것을 확인한 1차 검증 결과입니다.

실행 화면은 민감정보를 정리한 뒤 이 위치에 추가할 예정입니다.

## 다음 작업

- `qwen3:8b` 자원 사용량과 응답 속도 측정
- 반복 요청 및 재부팅 후 재현성 확인
- 기존 게임 서버와의 동시 실행 테스트

## 문서

- [구축 기록](docs/build-log.md) — 실제 설치와 연결 과정
- [벤치마크](docs/benchmark.md) — 측정 기준과 결과
- [로드맵](docs/roadmap.md) — 다음 작업과 장기 계획

## AI 활용

개발 과정의 문제 해결과 문서 정리에 ChatGPT를 보조 도구로 활용했습니다. 설치, 설정 및 동작 검증은 실제 SER8 서버 환경에서 직접 수행했습니다.
