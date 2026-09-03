# Benchmark

## Purpose

SER8의 16GB 메모리 환경에서 로컬 모델의 응답 성능과 자원 사용량을 확인하고, 기존 게임 서버와 함께 운영할 수 있는 기준을 찾습니다.

## Test Environment

| 항목 | 환경 |
| --- | --- |
| Hardware | Minisforum SER8 |
| CPU | AMD Ryzen 7 8745HS |
| Memory | DDR5 16GB |
| OS | Ubuntu Desktop 24.04.4 LTS |
| Runtime | Ollama |

## Test Models

- `qwen3:8b` — 현재 메인 모델, 기본 추론 성공
- `qwen3:4b` — 초기 구동 시험에 사용, 정량 측정 기록 없음

## Metrics

- 첫 응답까지 걸린 시간
- 생성 속도
- RAM 및 스왑 사용량
- CPU 사용률과 온도
- 반복 요청 안정성
- 게임 서버 동시 실행 시 영향

## Results

| Model | Basic Inference | First Response | Generation Speed | Memory | Game Server Test |
| --- | --- | --- | --- | --- | --- |
| `qwen3:8b` | Success | TBD | TBD | TBD | Not tested |
| `qwen3:4b` | Initial test only | TBD | TBD | TBD | Not tested |

## Notes

- 메인 운영 후보는 `qwen3:8b`입니다.
- 모델마다 같은 프롬프트를 사용합니다.
- 첫 실행과 워밍업 이후 실행을 구분합니다.
- 단독 실행과 게임 서버 동시 실행 결과를 따로 기록합니다.
- 측정하지 않은 값은 추측하지 않고 `TBD` 또는 `Not tested`로 남깁니다.
