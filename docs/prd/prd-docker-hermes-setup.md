# PRD: Mac Mini에 Docker + Hermes Agent 설치

## Problem Statement

맥미니를 AI 에이전트 서버로 활용하고 싶으나, Hermes Agent를 직접 설치하면 시스템 환경이 오염되고, 다른 맥에서 동일 환경을 재현하기 어렵다. 또한 업그레이드 시 롤백이 복잡하고, 버전 관리가 안 된다.

## Solution

Colima + Docker CLI 기반으로 Hermes Agent를 컨테이너로 실행한다. `docker-compose.yml`을 이 레포에 커밋해 다른 맥에서 `docker compose up -d` 하나로 동일 환경을 재현할 수 있게 한다. Slack Socket Mode로 통신하므로 포트 오픈 없이 동작한다.

## User Stories

1. 맥미니 운영자로서, Homebrew로 Colima와 Docker CLI를 설치하고 싶다. Docker Desktop의 높은 CPU 사용 없이 가벼운 Docker 런타임을 갖추기 위해.
2. 맥미니 운영자로서, 재부팅 시 Colima가 자동으로 시작되길 원한다. 수동 개입 없이 에이전트 서버가 복구되게 하기 위해.
3. 맥미니 운영자로서, Hermes Agent를 Docker 컨테이너로 실행하고 싶다. 호스트 시스템 환경을 오염시키지 않기 위해.
4. 맥미니 운영자로서, 재부팅 후 컨테이너가 자동으로 재시작되길 원한다. 전원 사이클 후에도 Hermes가 항상 가용하게 하기 위해.
5. 맥미니 운영자로서, `docker stop`으로 Hermes를 수동 중지하고 싶다. 자동 재시작 설정을 해제하지 않고도 가동 시간을 제어하기 위해.
6. 맥미니 운영자로서, Hermes 데이터(메모리, 스킬, 설정)를 호스트의 `~/.hermes`에 영속 저장하고 싶다. 컨테이너 업그레이드 시 데이터가 삭제되지 않게 하기 위해.
7. 맥미니 운영자로서, 시크릿(Slack 토큰, LLM API 키)을 `.env` 파일로 로드하고 싶다. `docker-compose.yml`에 자격증명이 하드코딩되지 않게 하기 위해.
8. 맥미니 운영자로서, Slack을 통해 Hermes와 소통하고 싶다. 어떤 기기에서도 에이전트와 대화하기 위해.
9. 맥미니 운영자로서, 인바운드 포트를 열지 않고 Slack 연동이 동작하길 원한다. 방화벽 뒤에서도 서버가 안전하게 유지되게 하기 위해.
10. 맥미니 운영자로서, 최신 이미지를 수동으로 pull해서 Hermes를 업데이트하고 싶다. 업그레이드 전 변경 내역을 검토하기 위해.
11. 여러 맥을 사용하는 개발자로서, 이 레포를 클론하고 Intel 맥 어디서든 `docker compose up -d`를 실행하고 싶다. 설치 환경이 완전히 재현되게 하기 위해.
12. 개발자로서, 단계별 설치 가이드를 원한다. 분산된 문서를 찾아다니지 않고 새 머신을 셋업하기 위해.

## Implementation Decisions

### Docker Runtime
- **Colima** 사용 (Docker Desktop 대신). CPU 유휴 시 ~0.2% vs Docker Desktop ~300%.
- 설치: `brew install colima docker docker-compose`
- 자동 시작: `brew services start colima`
- VM 사양: `colima start --cpu 2 --memory 4` (맥미니 사양에 따라 조정)

### Hermes Agent 이미지
- 공식 이미지: `nousresearch/hermes-agent:latest`
- 아키텍처: `linux/amd64` (Intel 맥미니)
- 업데이트 전략: 수동 (`docker compose pull && docker compose up -d`)

### docker-compose.yml 구조
- 서비스: `hermes` 단일 서비스
- 재시작 정책: `unless-stopped` (수동 stop 후 재부팅해도 안 뜸)
- 볼륨: `~/.hermes:/opt/data` (데이터 영속성)
- 포트 매핑: 없음 (Slack Socket Mode 사용)
- 환경변수: `env_file: .env`

### 환경변수 (.env)
- 위치: `~/.hermes/.env` (레포 외부, gitignore)
- 포함 항목: `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`, LLM API 키 등
- `.env.example` 파일을 레포에 커밋해 필요 변수 목록 문서화

### Slack 연동
- Socket Mode 사용. 인바운드 포트 불필요.
- 방화벽/공인 IP 설정 없이 동작.
- 멀티 워크스페이스: `SLACK_BOT_TOKEN` 쉼표로 구분해 복수 등록 가능.

## Testing Decisions

인프라 설치 PRD이므로 유닛 테스트 대신 수동 검증 체크리스트로 대체한다.

### 검증 항목
- [ ] `colima status` → Running 확인
- [ ] `docker compose up -d` 성공
- [ ] `docker ps` → hermes 컨테이너 Up 상태
- [ ] 맥미니 재부팅 후 `docker ps` → 자동 복구 확인
- [ ] `docker stop hermes` → 재부팅 후 컨테이너 안 뜨는 것 확인 (unless-stopped 동작 검증)
- [ ] Slack에서 Hermes에 메시지 전송 → 응답 확인
- [ ] `docker compose pull && docker compose up -d` → 데이터 유지 확인

## Out of Scope

- HTTPS/TLS 설정 (Slack Socket Mode라 불필요)
- Watchtower 자동 업데이트
- Docker Swarm / Kubernetes
- Apple Silicon(ARM) 지원 (현재 Intel 맥미니 대상)
- Hermes skills 설치 자동화 (별도 PRD)
- 모니터링/알림 설정 (Grafana, Prometheus 등)

## Further Notes

- `docker-compose.yml`은 이 레포에 커밋. `.env`는 절대 커밋하지 않음.
- `.env.example`을 레포에 포함해 온보딩 가이드 역할 겸용.
- 다른 맥(Intel)에서 재현 시: 레포 클론 → `brew install colima docker docker-compose` → `~/.hermes/.env` 작성 → `colima start` → `docker compose up -d`.
- Hermes 공식 Docker 문서: https://hermes-agent.nousresearch.com/docs/user-guide/docker
