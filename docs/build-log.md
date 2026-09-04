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
- AGENTS.md의 중복 규칙 제거
- 불필요하게 누적된 Pterodactyl 지침 제거
- Workspace 프롬프트 정리

현재 Exec 정책은 다음과 같습니다.

```text
security = allowlist
ask = on-miss
askFallback = deny
```

읽기 전용으로 허용한 명령은 자동 실행할 수 있고, 새로운 명령이나 상태 변경 작업은 사용자 승인을 요청합니다.

### 결과

- OpenClaw 기본 모델을 `ollama/qwen3.5:4b`로 변경
- Context를 8192로 조정하고 Thinking OFF 적용
- System prompt estimate 약 2,898 tokens
- Tracked prompt estimate 약 3,173 tokens
- AGENTS.md 약 316 tokens
- Tool schemas 약 275 tokens
- 사용 도구를 `exec`, `session_status`로 축소
- Ollama runtime에서 `qwen3.5:4b`의 `100% GPU`, Context 8192 확인
- `openclaw infer model run`에서 약 3.14초 만에 정상 한국어 응답 확인

초기 약 13K token 수준과 비교해 작은 로컬 모델이 실제 요청을 처리할 수 있는 크기로 줄였습니다.

## Exec 권한을 승인 기반 정책으로 변경

### 문제

초기 Exec 설정은 `security=full`, `ask=off`로 OpenClaw가 별도 승인 없이 shell 명령을 실행할 수 있는 상태였습니다. 서버 조회와 상태 변경이 같은 권한으로 동작하는 것은 홈서버 운영에 적합하지 않았습니다.

### 해결

Exec Policy를 `cautious`로 바꾸고 다음 정책을 적용했습니다.

```text
security = allowlist
ask = on-miss
askFallback = deny
```

- 안전한 읽기 전용 조회만 allowlist에 등록
- 서버 시작·종료·재시작은 영구 허용하지 않음
- 상태 변경 명령은 실행할 때마다 `Allow once` 승인

### 결과

OpenClaw에 무제한 shell access를 주지 않으면서 상태 조회는 간단하게 유지하고, 실제 서버 제어에는 사용자의 명시적인 승인이 필요하도록 분리했습니다.

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

## 작은 모델의 서버 제어 명령 생성 문제

### 문제

Qwen3.5 4B가 자연어 요청마다 Pterodactyl API 호출 형식과 긴 명령 구문을 직접 조합하도록 하면 안정성이 떨어졌습니다. `팰월드`와 `박현종의 팰월드`처럼 이름 일부가 겹치는 서버를 잘못 선택할 위험도 있었습니다.

### 해결

복잡한 처리는 내부 `ptero-control`에 모으고 OpenClaw에는 작은 전용 인터페이스만 제공했습니다.

- `ptero-status`: 상태 조회
- `ptero-start`: 서버 시작
- `ptero-stop`: 서버 종료
- `ptero-restart`: 서버 재시작

각 wrapper는 정확한 서버 이름을 전달하고 내부적으로 Pterodactyl Client API의 power endpoint를 사용합니다. DRY-RUN에서 `팰월드`와 `박현종의 팰월드`를 별개의 대상으로 정확히 구분하는 것을 먼저 확인했습니다.

### 실동작 검증

1. OpenClaw에 “팰월드 서버 켜줘”라고 자연어로 요청
2. Qwen3.5 4B가 `ptero-start` 선택
3. Exec Approval에서 사용자가 `Allow once` 승인
4. Pterodactyl Client API로 서버 시작
5. `ptero-status`로 상태와 자원 사용량 재조회
6. OpenClaw가 RUNNING 상태, CPU, RAM, Disk를 한국어로 보고

같은 구조로 서버 종료와 재시작도 정상 동작하는 것을 확인했습니다.

### 결과

- 상태 조회, 시작, 종료, 재시작 실동작 검증 완료
- 제어 후 상태 재확인 완료
- 작은 모델이 복잡한 API 구문을 직접 만들지 않고 제한된 wrapper를 선택하도록 단순화
- 조회와 제어 권한을 분리하고 상태 변경에는 매번 사용자 승인 적용

## 통합 Health Check의 잘못된 Docker 판정

### 문제

Ollama, OpenClaw, Open WebUI, Wings와 서버 자원을 한 번에 확인하기 위해 `~/.local/bin/ser8-health`를 만들었습니다. 초기 구현은 내부에서 `sudo docker`를 사용해 터미널에서는 동작했지만, OpenClaw의 비대화형 실행에서는 권한 입력을 처리하지 못해 Docker와 Open WebUI 상태를 잘못 판정했습니다.

### 원인

- 비대화형 실행에는 `sudo` 비밀번호를 입력할 터미널이 없음
- 컨테이너 상태만으로 실제 웹 서비스 응답까지 판단하기 어려움
- 시스템 언어에 따라 `free -h` 출력 형식이 달라질 수 있음
- `ollama ps`의 `UNTIL`을 장애나 종료 예정 시각으로 오해할 수 있음

### 해결

- Health Check 내부에서 `sudo` 제거
- Open WebUI는 HTTP endpoint 응답으로 확인
- daemon은 systemd 상태로 확인
- 메모리 확인에 `LC_ALL=C free -h` 사용
- Ollama의 `UNTIL` 값을 idle unload time으로 명시

`ser8-health`는 현재 다음 항목을 한 번에 확인합니다.

- Ollama와 Ollama API
- OpenClaw와 Open WebUI
- Wings와 Docker
- CPU Load, RAM, Swap, Disk, Uptime
- Ollama에 로딩된 모델
- Pterodactyl 서버 상태

### 결과

OpenClaw에 “서버 전체 상태 점검해줘”라고 요청하면 `ser8-health`를 실행하고 결과를 정상적으로 요약합니다.

## Heartbeat `no-route` 문제

### 문제

OpenClaw Heartbeat를 30분 간격 자동 Health Check로 설정했지만 다음 상태로 건너뛰었습니다.

```text
heartbeat skipped: no-route
```

### 원인

Heartbeat 대상이 `target=last`였지만 Control UI만 사용하고 있어 결과를 전달할 외부 route가 없었습니다.

### 해결

외부 전송이 아닌 내부 점검이 목적이므로 `target=none`으로 변경했습니다. 현재 설정의 핵심은 다음과 같습니다.

- 실행 간격: 30분
- 모델: `ollama/qwen3.5:4b`
- `lightContext=true`
- `isolatedSession=true`
- `timeoutSeconds=90`
- `target=none`
- 점검 도구는 `ser8-health`로 제한

모든 주요 서비스가 정상이고 FAIL 또는 WARN이 없으면 `NO_REPLY`로 종료합니다. Ollama의 idle unload time과 의도적으로 꺼둔 Pterodactyl 서버는 오류로 취급하지 않습니다.

### 결과

- 수동 Heartbeat 실행 상태 `ok`
- completion status `succeeded`
- 실행 시간 약 9~17초
- 30분 내부 자동 Health Check 동작 확인

현재 `target=none`이므로 장애 발생 시 외부로 능동 알림을 보내지는 않습니다. Discord 알림 연동은 의도적으로 보류했습니다.

## 재부팅 후 자동 복구 검증

### 검증 목적

설정 파일만 확인하는 대신 실제 Ubuntu 재부팅 후 서비스가 사람의 개입 없이 복구되는지 점검했습니다.

### 확인 결과

- Ollama, OpenClaw, Wings: enabled 및 active
- OpenClaw user service와 `Linger=yes` 확인
- Open WebUI: Docker `restart=unless-stopped`로 자동 실행
- Ollama API와 Open WebUI HTTP 200 응답 확인
- OpenClaw `18789`, Ollama `11434`, Open WebUI `3000`, Wings `8080` 포트 확인
- 부팅 직후 로딩된 Ollama 모델이 없는 정상 유휴 상태 확인
- Ollama 상태 API가 약 0.03초 만에 응답
- 사용 가능 메모리 약 12GiB, Swap 0B 확인

재부팅 후 Ollama, OpenClaw, Wings 로그에서 error, failed, timeout, OOM, DeviceLost 계열 문제를 발견하지 못했습니다.

## Local AI 백업 범위 설계

### 문제

복구에 필요한 Local AI 데이터와 대용량 모델·게임 서버 데이터를 구분하지 않으면 백업이 불필요하게 커지고 프로젝트 범위도 모호해집니다.

### 해결

`~/ser8-backup`에 Local AI 환경 복구에 필요한 항목만 모았습니다.

- OpenClaw 설정, Workspace, Memory, SQLite DB와 user service 참고본
- `ptero-status`, `ptero-control`, start·stop·restart wrapper, `ser8-health`
- Ollama service 참고본과 `OLLAMA_IGPU_ENABLE=1` override
- Open WebUI의 `/app/backend/data` Docker volume 데이터
- `~/llm-knowledge` RAG 원본 문서
- 별도 `secrets/` 영역의 Pterodactyl 인증 파일

Ollama 모델 파일은 백업하지 않고 복구 시 `qwen3.5:4b`, `qwen3.5:9b`, `bge-m3`를 다시 내려받도록 했습니다. Pterodactyl Panel DB, Wings 전체 설정, 게임 서버 월드와 세이브는 이 Local AI 프로젝트의 백업 범위에서 제외했습니다.

### 검증

- Local AI 백업 크기 약 979MB
- Open WebUI, RAG 원본, OpenClaw Memory tar archive 확인
- `openclaw.sqlite`, `openclaw-agent.sqlite`의 `PRAGMA quick_check` 결과 `ok`
- 내부 `SHA256SUMS` 전체 검증 성공

## 암호화 외부 백업과 복구 절차

### 보안과 외부 복사

Local AI 백업 전체를 tar.gz로 묶고 GPG symmetric AES256 방식으로 암호화했습니다. 암호는 명령 인자가 아닌 입력 방식으로 처리해 shell history에 남기지 않았습니다.

암호화 후 실제 복호화와 tar 목록 확인을 수행했으며, 검증이 끝난 평문 archive는 삭제했습니다. 약 967MB의 암호화 파일은 SCP로 Windows PC에 복사해 서버와 외부 PC 두 위치에 보관했습니다.

### 무결성 검증

SER8에서 계산한 암호화 파일의 SHA256과 Windows의 `certutil -hashfile` 결과가 완전히 일치하는 것을 확인했습니다. 이로써 암호화, 외부 전송, 전송 후 무결성까지 검증했습니다.

### 복구 절차

백업 내부에 `RECOVERY.md`를 작성해 다음 흐름을 기록했습니다.

1. 새 Ubuntu에 Docker, Ollama, Node, OpenClaw 설치
2. Radeon 780M override 복원
3. Ollama 모델 재다운로드
4. OpenClaw 설정, DB, Workspace와 관리 wrapper 복원
5. Pterodactyl API Secret 복원
6. Open WebUI volume과 RAG 원본 복원
7. `ser8-health`로 최종 검증

버전 변경 가능성을 고려해 기존 systemd service 전체를 무조건 덮어쓰지 않고, 새 설치본을 우선한 뒤 custom override를 복원하는 방향으로 정리했습니다. `RECOVERY.md`와 실제 백업 파일은 운영용 백업 내부에만 보관하며 GitHub 저장소에는 추가하지 않습니다.
