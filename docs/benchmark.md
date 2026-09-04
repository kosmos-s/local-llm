# 벤치마크

## 목적

SER8의 16GB 메모리와 Radeon 780M 환경에서 실행 가능한 모델 크기와 용도별 응답성을 확인합니다. 아래에는 실제로 확인한 값만 기록합니다.

## 테스트 환경

| 항목 | 환경 |
| --- | --- |
| 하드웨어 | Minisforum SER8 |
| CPU | AMD Ryzen 7 8745HS |
| 메모리 | 16GB |
| GPU | Radeon 780M 내장 그래픽 |
| GPU 실행 환경 | Vulkan · RADV PHOENIX |
| 운영체제 | Ubuntu 24.04 LTS |
| LLM 실행 환경 | Ollama |

## 현재 유지 모델

| 모델 | 역할 | Context | Thinking | GPU |
| --- | --- | ---: | --- | --- |
| `qwen3.5:4b` | OpenClaw Agent | 8192 | OFF | 100% GPU |
| `qwen3.5:9b` | Open WebUI 메인 LLM | 8192 | OFF | 100% GPU |
| `bge-m3` | RAG 임베딩 | 해당 없음 | 해당 없음 | 별도 속도 측정 없음 |

## Qwen3 초기 실험

| 모델 | 측정 결과 | GPU | 확인 내용 |
| --- | --- | --- | --- |
| `qwen3:4b` | 약 15.6 tok/s | 100% GPU | 빠르지만 OpenClaw에서 구형 thinking 출력이 불필요하게 나타남 |
| `qwen3:8b` | 약 8.3 tok/s | 100% GPU | 대화 품질은 나았지만 큰 Agent Context에서 응답성이 좋지 않음 |
| `qwen3:14b` | 실행 실패 | 측정 불가 | Vulkan 메모리 부족과 `ErrorDeviceLost` 발생 |

14B에서 확인한 핵심 오류:

```text
radv/amdgpu: Not enough memory for command submission
vk::Queue::submit: ErrorDeviceLost
```

세 모델은 현재 Ollama에서 삭제한 상태입니다.

## Qwen3.5 응답 시간

| 모델 | 테스트 경로 | 조건 | 확인 결과 |
| --- | --- | --- | --- |
| `qwen3.5:4b` | Ollama API 직접 호출 | Context 8192, Thinking OFF, 100% GPU | 약 4초 |
| `qwen3.5:4b` | `openclaw infer model run` | Context 8192, Thinking OFF, 100% GPU | 약 3.14초, 정상 한국어 응답 |
| `qwen3.5:9b` | Ollama 테스트 | Context 8192, Thinking OFF, 100% GPU, 첫 로딩 포함 | 약 5.34초, 짧은 한국어 응답 |

## 운영 점검 시간

| 항목 | 조건 | 확인 결과 |
| --- | --- | --- |
| OpenClaw Heartbeat | `qwen3.5:4b`, `ser8-health`, 경량 Context, 격리 세션 | 약 9~17초, status `ok`, completion status `succeeded` |

## 측정값 해석 시 주의사항

- Qwen3의 `tok/s`는 생성 속도이고, Qwen3.5의 초 단위 값은 요청부터 응답까지의 경과 시간입니다.
- 프롬프트, 출력 길이, 모델 로딩 여부와 호출 경로가 서로 달라 두 수치를 같은 성능 지표처럼 직접 비교할 수 없습니다.
- `qwen3.5:9b`의 약 5.34초에는 첫 모델 로딩 시간이 포함되어 있습니다.
- Heartbeat 시간은 여러 서비스와 자원을 확인하는 전체 점검 시간으로, 모델 응답 속도와 직접 비교할 수 없습니다.
- 아직 동일 프롬프트를 사용한 반복 측정이나 게임 서버 동시 실행 비교는 진행하지 않았습니다.
- 측정하지 않은 처리량, 메모리 사용량, 온도 값은 추측해 기록하지 않습니다.
