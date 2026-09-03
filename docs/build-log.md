# 구축 기록

SER8 서버에서 실제로 겪은 문제를 중심으로 원인, 해결 과정, 확인 결과를 기록합니다. 긴 원본 로그와 인증 정보는 남기지 않고 재현에 필요한 핵심만 정리합니다.

## Radeon 780M이 Ollama 추론 장치에서 제외됨

### 문제

Ollama를 Ubuntu 호스트에 설치했지만 처음에는 Radeon 780M을 추론 장치로 사용하지 못했습니다.

### 원인 확인

- 로그에서 `AMD Radeon 780M Graphics (RADV PHOENIX)` Vulkan 장치 자체는 감지됨
- 통합 GPU라는 이유로 기본 탐색 과정에서 제외됨
- ROCm 경로에서는 `gfx1103` 지원 문제가 발생함

### 해결

Ollama의 systemd 서비스 환경에 다음 값을 적용했습니다.

```ini
[Service]
Environment="OLLAMA_IGPU_ENABLE=1"
```

### 결과

- Radeon 780M이 Vulkan inference compute 장치로 정상 감지됨
- `ollama ps`에서 실행 중인 모델이 `100% GPU`로 표시되는 것을 확인함

## 모델 크기와 실행 환경 재선정

### 문제

Qwen3 계열을 비교하는 과정에서 모델마다 속도, 대화 품질, OpenClaw 적합성에 차이가 있었습니다.

- `qwen3:4b`: 약 15.6 tok/s로 빨랐지만 구형 thinking 동작이 OpenClaw에서 불필요한 내부 추론 출력을 만들었음
- `qwen3:8b`: 약 8.3 tok/s로 대화 품질은 나았지만 Agent의 큰 시스템 프롬프트와 함께 사용할 때 응답성이 좋지 않았음
- `qwen3:14b`: Radeon 780M Vulkan 환경에서 메모리 부족으로 실행에 실패함

14B 실행 시 확인한 핵심 오류는 다음과 같습니다.

```text
radv/amdgpu: Not enough memory for command submission
vk::Queue::submit: ErrorDeviceLost
```

### 해결

Qwen3.5 계열로 전환하고 역할을 나눴습니다.

- `qwen3.5:4b`: OpenClaw Agent 전용
- `qwen3.5:9b`: Open WebUI의 대화·코딩·RAG 답변 생성
- `bge-m3`: RAG 임베딩 전용

기존 `qwen3:4b`, `qwen3:8b`, `qwen3:14b`는 비교 후 삭제했습니다.

### 결과

현재 Ollama에는 `qwen3.5:4b`, `qwen3.5:9b`, `bge-m3`만 유지하며 세 모델 모두 역할이 구분되어 있습니다.

## Open WebUI와 RAG 구축

### 구성

Docker로 Open WebUI를 구축하고 Ollama와 연결했습니다. 일반 채팅에는 `qwen3.5:9b`, 임베딩에는 `bge-m3`를 사용합니다.

### 검증

초기 `SER8 Server Test` Knowledge Base에서 다음을 확인했습니다.

1. 문서에 있는 CPU와 메인 LLM 정보를 정확히 검색
2. 모델 벤치마크 데이터를 찾아 비교
3. 문서에 없는 SSD 모델을 질문했을 때 모른다고 응답
4. 답변에 Source가 정상 표시됨

이후 `Home Server Knowledge`를 만들고 `server-spec.md`, `services.md`, `local-llm.md`를 추가했습니다. 여러 문서를 함께 검색해 홈서버 구성과 로컬 LLM 구축 내역을 답하는 것까지 확인했습니다.

외부 공개, 도메인, HTTPS, Reverse Proxy는 아직 구성하지 않았습니다.

## OpenClaw Context 과다 사용과 응답 지연

### 문제

Ollama API를 직접 호출하면 응답했지만 OpenClaw Agent에서는 매우 느리거나 실패하는 경우가 있었습니다. `/context detail` 확인 결과 Context가 약 13K token까지 증가해 설정한 12K context보다 시스템 입력이 큰 상황도 발생했습니다.

### 원인

- OpenClaw 기본 시스템 프롬프트
- Workspace 파일
- Skills
- Tool schemas

위 항목이 모델 추론 전에 큰 Context를 사용하고 있었습니다.

### 해결

- `localModelLean` 활성화
- Tool Profile을 `minimal`로 축소
- 도구를 `exec`와 `session_status` 중심으로 제한
- Tool Search 비활성화
- Thinking OFF
- Dreaming 비활성화
- Memory Search를 Ollama와 BGE-M3 조합으로 변경
- Exec Policy를 `cautious`로 변경

현재 Exec 정책은 다음과 같습니다.

```text
security = allowlist
ask = on-miss
```

읽기 전용으로 허용한 명령은 자동 실행할 수 있고, 새로운 명령이나 상태 변경 작업은 사용자 승인을 요청합니다.

### 결과

- OpenClaw 기본 모델을 `ollama/qwen3.5:4b`로 변경
- Context를 8192로 조정하고 Thinking OFF 적용
- `openclaw infer model run`에서 약 3.14초 만에 정상 한국어 응답 확인
- 추가적인 Workspace 및 Context 최적화는 계속 진행 중

## Pterodactyl API 응답의 민감정보 노출 방지

### 문제

Pterodactyl Client API를 직접 테스트하는 과정에서 원본 JSON에 서버 상태 외의 민감한 변수 값이 포함될 수 있음을 확인했습니다. 원본 응답을 그대로 Agent나 로그에 전달하기에는 위험했습니다.

### 해결

- API 인증 정보는 `~/.config/ptero-openclaw/env`에 저장
- 인증 파일 권한을 `600`으로 제한
- `~/.local/bin/ptero-status` 읽기 전용 도구 구현
- 전체 서버 또는 특정 서버를 조회하되 필요한 필드만 출력

출력 항목은 다음으로 제한했습니다.

- 서버 이름과 상태
- CPU 사용량
- RAM 사용량
- Disk 사용량

API Token, 환경 변수, 관리자 비밀번호 및 원본 JSON은 출력하거나 저장소에 포함하지 않습니다.

### 결과

- OpenClaw의 자연어 요청으로 실제 Pterodactyl 서버 목록과 실행 상태 조회 성공
- 특정 서버 조회 시 정확히 일치하는 이름을 우선 선택하도록 구현
- 조회 기능은 읽기 전용 명령으로 allowlist 적용 가능

## Pterodactyl 서버 제어 기능

서버 시작·종료·재시작용 `ptero-start`, `ptero-stop`, `ptero-restart`를 개발하고 있습니다. Pterodactyl Client API의 power endpoint를 사용하며 상태 변경 전 사용자 승인을 필수로 하는 방향입니다.

현재 DRY-RUN에서는 `팰월드`와 `박현종의 팰월드`를 정확히 구분해 START 작업을 준비하는 것까지 확인했습니다. 자연어 요청에서 제어 명령을 안정적으로 선택하는 과정은 아직 조정 중이므로 완료 기능으로 기록하지 않습니다.
