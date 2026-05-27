# Docker + Hermes Agent 설치 구현 플랜

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 맥미니에 Colima + Docker로 Hermes Agent를 설치하고, Slack Socket Mode로 연동한다.

**Architecture:** Colima VM이 Docker 런타임을 제공하고, `docker-compose.yml`로 Hermes 컨테이너를 관리한다. 데이터는 `~/.hermes`에 볼륨 마운트로 영속 저장된다. Slack Socket Mode로 아웃바운드 연결만 사용해 포트 오픈이 불필요하다.

**Tech Stack:** Homebrew, Colima, Docker CLI, Docker Compose, nousresearch/hermes-agent:latest, Slack Socket Mode, Google Gemini API (무료 → 추후 OpenRouter로 전환 가능)

---

## 파일 구조

생성할 파일:
- `docker-compose.yml` — Hermes 컨테이너 정의 ✅
- `.env.example` — 필요 환경변수 목록 (레포에 커밋) ✅
- `~/.hermes/.env` — 실제 시크릿 (레포 외부, gitignore) ✅
- `~/.hermes/config.yaml` — Hermes 모델/프로바이더 설정 ✅
- `docs/install-guide.md` — 단계별 설치 가이드
- `docs/hermes-cmd.md` — 명령어 레퍼런스 ✅

수정할 파일:
- `.gitignore` — `.env` 추가 확인 ✅

---

### Task 0: 기존 설치 제거 (클린 슬레이트) ✅

**Files:**
- (시스템 정리, 파일 변경 없음)

- [x] **Step 1: Hermes alias 제거 (`~/.zshrc`에서)**

```bash
sed -i '' '/alias hermes=/d' ~/.zshrc
grep 'alias hermes' ~/.zshrc || echo "confirmed gone"
```

- [x] **Step 2: 기존 Hermes 컨테이너 + 이미지 삭제**

```bash
docker rm -f hermes 2>/dev/null || true
docker rmi nousresearch/hermes-agent:latest 2>/dev/null || true
docker images | grep hermes || echo "no hermes images"
```

- [x] **Step 3: `~/.hermes/` 데이터 전체 삭제**

```bash
rm -rf ~/.hermes
ls ~/.hermes 2>/dev/null || echo "deleted"
```

- [x] **Step 4: OpenClaw npm 패키지 제거**

```bash
npm uninstall -g openclaw 2>/dev/null || true
which openclaw 2>/dev/null || echo "openclaw removed"
```

- [x] **Step 5: Docker Desktop 제거 (수동)**

Finder → Applications → `Docker.app` → 휴지통

- [x] **Step 6: 제거 확인**

```bash
ls /Applications/Docker.app 2>/dev/null && echo "STILL EXISTS" || echo "REMOVED"
```

---

### Task 1: Colima + Docker CLI 설치 ✅

**Files:**
- (시스템 설치, 파일 변경 없음)

- [x] **Step 1: Homebrew 설치 확인**

- [x] **Step 2: Colima + Docker 설치**

```bash
brew install colima docker docker-compose
```

- [x] **Step 3: docker-compose 플러그인 경로 등록**

```bash
mkdir -p ~/.docker
cat > ~/.docker/config.json << 'EOF'
{
  "cliPluginsExtraDirs": [
    "/usr/local/lib/docker/cli-plugins"
  ]
}
EOF
```

- [x] **Step 4: 설치 확인**

```
colima version 0.10.1
Docker version 29.5.2
Docker Compose version 5.1.4
```

- [x] **Step 5: Colima VM 최초 시작 (CPU 2코어, 메모리 4GB)**

```bash
colima start --cpu 2 --memory 4 --arch x86_64
```

- [x] **Step 6: Docker 동작 확인**

```bash
docker ps   # 빈 목록 정상 출력
```

---

### Task 2: Colima 자동 시작 등록 ✅

- [x] **Step 1: launchd 서비스로 등록**

```bash
brew services start colima
```

- [x] **Step 2: 서비스 등록 확인**

```
colima  started  hw-11135  ~/Library/LaunchAgents/homebrew.mxcl.colima.plist
```

---

### Task 3: Hermes 데이터 디렉터리 생성 ✅

- [x] **Step 1: 디렉터리 생성**

```bash
mkdir -p ~/.hermes
```

---

### Task 4: `.env.example` 생성 (레포에 커밋) ✅

**Files:**
- Create: `.env.example`

- [x] **Step 1: `.env.example` 파일 생성**

```bash
cat > .env.example << 'EOF'
# Slack Socket Mode 토큰
SLACK_APP_TOKEN=xapp-1-...

# Slack Bot 토큰
SLACK_BOT_TOKEN=xoxb-...

# LLM API 키 — 현재: Google Gemini (무료)
# Google AI Studio에서 발급: https://aistudio.google.com/apikey
GOOGLE_API_KEY=AIza...

# 추후 OpenRouter로 전환 시 아래로 교체:
# OPENROUTER_API_KEY=sk-or-...

# 기타 프로바이더 (미사용 시 주석 유지)
# ANTHROPIC_API_KEY=sk-ant-...
# NOUS_API_KEY=...
EOF
```

---

### Task 5: `~/.hermes/.env` 생성 (실제 시크릿) ✅

**Files:**
- Create: `~/.hermes/.env`

- [x] **Step 1: 실제 `.env` 파일 생성 및 토큰 입력**

```
SLACK_APP_TOKEN=xapp-1-실제토큰
SLACK_BOT_TOKEN=xoxb-실제토큰
GOOGLE_API_KEY=AIza실제키
GATEWAY_ALLOW_ALL_USERS=true   # 개인 비서용 전체 접근 허용
```

토큰 발급 위치:
- `SLACK_APP_TOKEN`: https://api.slack.com/apps → Basic Information → App-Level Tokens (`connections:write` scope)
- `SLACK_BOT_TOKEN`: OAuth & Permissions → Bot User OAuth Token
- `GOOGLE_API_KEY`: https://aistudio.google.com/apikey

- [x] **Step 2: 파일 권한 보호**

```bash
chmod 600 ~/.hermes/.env
```

---

### Task 5.5: Hermes `config.yaml` 생성 (Gemini 모델 설정) ✅

**Files:**
- Create: `~/.hermes/config.yaml`

- [x] **Step 1: `config.yaml` 작성**

```yaml
provider: gemini
model: gemini-flash-latest
```

- [x] **Step 2: 파일 권한**

```bash
chmod 600 ~/.hermes/config.yaml
```

---

### Task 6: `docker-compose.yml` 생성 ✅

**Files:**
- Create: `docker-compose.yml`

- [x] **Step 1: `docker-compose.yml` 작성**

```yaml
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    platform: linux/amd64
    container_name: hermes
    restart: unless-stopped
    env_file:
      - path: ${HOME}/.hermes/.env
    volumes:
      - ${HOME}/.hermes:/opt/data
    command: gateway run
```

- [x] **Step 2: YAML 문법 확인**

```bash
docker compose config   # 에러 없음 확인
```

---

### Task 6.5: Hermes 관리 alias 등록 ✅ (추가)

**Files:**
- Modify: `~/.zshrc`
- Create: `docs/hermes-cmd.md`

- [x] **Step 1: `~/.zshrc`에 alias 추가**

```bash
alias hermes-up="docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml up -d"
alias hermes-down="docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml stop"
alias hermes-restart="docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml restart"
alias hermes-logs="docker logs hermes -f"
alias hermes-status="docker ps | grep hermes"
alias hermes-update="docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml pull && docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml up -d"
```

- [x] **Step 2: `docs/hermes-cmd.md` 작성** (명령어 레퍼런스)

---

### Task 7: `.gitignore` 확인 및 업데이트 ✅

- [x] **Step 1: `.env` 패턴 추가**

```
.env
*.env
```

---

### Task 8: Hermes 컨테이너 최초 실행 및 검증 (진행 중)

- [x] **Step 1: 이미지 pull**

```bash
docker compose pull   # 완료
```

- [x] **Step 2: 컨테이너 시작**

```bash
docker compose up -d   # Container hermes Started
```

- [x] **Step 3: 컨테이너 상태 확인**

```bash
docker ps   # Up 상태 확인
```

- [x] **Step 4: 로그 확인**

Hermes Gateway 정상 시작 확인. 단, Slack scope 부족 경고 발생:

```
missing_scope: channels:read, groups:read, mpim:read, im:read
```

- [ ] **Step 5: Slack scope 추가 및 재설치**

Slack App → OAuth & Permissions → Bot Token Scopes에 추가:
- `channels:read`
- `groups:read`
- `mpim:read`
- `im:read`

"Reinstall to Workspace" → 새 Bot Token → `~/.hermes/.env` 업데이트 → `hermes-restart`

- [ ] **Step 6: Slack 연결 확인**

Slack에서 Hermes 봇에게 DM 전송 → 응답 확인

---

### Task 9: `unless-stopped` 동작 검증

- [ ] **Step 1: 수동 중지 테스트**

```bash
docker stop hermes
docker ps   # 목록에 없음 확인
```

- [ ] **Step 2: Colima 재시작 후 미시작 확인**

```bash
colima stop && colima start
docker ps   # hermes 자동으로 뜨지 않음 확인
```

- [ ] **Step 3: 수동 시작 후 재부팅 자동복구 확인**

```bash
hermes-up
colima stop && colima start
docker ps   # hermes 자동 Up 확인
```

---

### Task 10: 업데이트 절차 검증

- [ ] **Step 1~3:** `hermes-update` 실행 → `~/.hermes` 데이터 유지 확인

---

### Task 11: 설치 가이드 문서 작성

- [ ] **Step 1: `docs/install-guide.md` 작성**

---

### Task 12: 최종 커밋

- [ ] **Step 1: 모든 파일 커밋 후 push**

```bash
git add docker-compose.yml .env.example .gitignore docs/
git commit -m "feat: Docker+Colima+Hermes 설치 완료"
git push origin main
```

---

## 현재 상태 요약

| Task | 상태 |
|------|------|
| Task 0: 기존 설치 제거 | ✅ 완료 |
| Task 1: Colima + Docker 설치 | ✅ 완료 |
| Task 2: Colima 자동시작 | ✅ 완료 |
| Task 3: ~/.hermes 생성 | ✅ 완료 |
| Task 4: .env.example | ✅ 완료 |
| Task 5: ~/.hermes/.env | ✅ 완료 |
| Task 5.5: config.yaml | ✅ 완료 |
| Task 6: docker-compose.yml | ✅ 완료 |
| Task 6.5: alias + hermes-cmd.md | ✅ 완료 |
| Task 7: .gitignore | ✅ 완료 |
| Task 8: 컨테이너 실행 | 🔄 Slack scope 추가 후 완료 |
| Task 9: unless-stopped 검증 | ⏳ 대기 |
| Task 10: 업데이트 검증 | ⏳ 대기 |
| Task 11: install-guide.md | ⏳ 대기 |
| Task 12: 최종 커밋 | ⏳ 대기 |

## 검증 체크리스트

- [x] `colima status` → Running
- [x] `brew services list | grep colima` → started
- [x] `docker compose up -d` 성공
- [x] `docker ps` → hermes Up
- [ ] Slack DM → Hermes 응답 (scope 추가 후)
- [ ] `docker stop hermes` → 재부팅 후 자동 미시작
- [ ] `hermes-update` → 데이터 유지
- [ ] `git log` → 커밋 확인
