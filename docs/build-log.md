# 구축 기록

SER8 서버에서 실제로 수행한 설치, 설정, 검증 과정을 날짜별로 기록합니다. 긴 원본 로그 대신 재현에 필요한 명령과 확인된 결과만 남깁니다.

## 2026-09-02 — Ollama와 OpenClaw 기본 연동

### 작업

- Ollama 실행 환경 확인
- `qwen3:8b` 로컬 추론 확인
- OpenClaw 설치 및 모델 상태 확인
- OpenClaw 기본 모델을 `ollama/qwen3:8b`로 설정
- OpenClaw에서 Ollama로 직접 추론 요청

### 주요 명령

```bash
ollama run qwen3:8b "안녕하세요. 한 문장으로 대답해줘."
openclaw models status
openclaw infer model run \
  --model ollama/qwen3:8b \
  --prompt "한국어로 정확히 '정상'이라고만 답해."
```

### 결과

- Ollama에서 `qwen3:8b` 한국어 응답 확인
- OpenClaw 기본 모델이 `ollama/qwen3:8b`로 설정된 것을 확인
- `OpenClaw → Ollama → qwen3:8b` 직접 추론 성공

### 문제 및 해결

이 단계에서 별도로 기록한 오류나 해결 과정은 없습니다.

### 다음 작업

- 동일 요청을 반복해 추론 안정성 확인
- 서버 재부팅 후 설정과 실행 상태 재확인
- CPU, 메모리, 스왑 및 응답 속도 측정
- 게임 서버와 동시에 실행했을 때의 영향 확인
