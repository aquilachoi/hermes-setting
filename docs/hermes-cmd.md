# Hermes Agent 명령어 레퍼런스

## 컨테이너 관리 (어디서든 실행 가능)

| 명령어 | 동작 |
|--------|------|
| `hermes-up` | Hermes 컨테이너 시작 |
| `hermes-down` | Hermes 컨테이너 중지 |
| `hermes-restart` | Hermes 컨테이너 재시작 |
| `hermes-logs` | 실시간 로그 스트림 (Ctrl+C로 종료) |
| `hermes-status` | 컨테이너 실행 상태 확인 |
| `hermes-update` | 최신 이미지 pull 후 재시작 |

> alias는 `~/.zshrc`에 등록됨. 새 터미널에서 바로 사용 가능.

---

## Docker 직접 명령어

```bash
# 컨테이너 시작
docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml up -d

# 컨테이너 중지
docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml stop

# 컨테이너 재시작
docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml restart

# 로그 확인 (최근 50줄)
docker logs hermes --tail 50

# 실시간 로그
docker logs hermes -f

# 컨테이너 상태
docker ps | grep hermes

# 이미지 업데이트 후 재시작
docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml pull
docker compose -f ~/Desktop/workspace/hermes-setting/docker-compose.yml up -d
```

---

## Colima 관리

```bash
# 상태 확인
colima status

# 시작 (수동)
colima start

# 중지
colima stop

# 자동 시작 등록 확인
brew services list | grep colima
```

---

## 환경설정 파일 위치

| 파일 | 경로 | 용도 |
|------|------|------|
| 환경변수 (시크릿) | `~/.hermes/.env` | Slack 토큰, API 키 |
| 모델 설정 | `~/.hermes/config.yaml` | LLM 프로바이더/모델 |
| 컨테이너 정의 | `~/Desktop/workspace/hermes-setting/docker-compose.yml` | Docker 설정 |

---

## LLM 전환 방법

### 현재: Google Gemini (무료)

`~/.hermes/config.yaml`:
```yaml
provider: gemini
model: gemini-flash-latest
```

`~/.hermes/.env`:
```
GOOGLE_API_KEY=AIza...
```

### OpenRouter로 전환 시

`~/.hermes/config.yaml`:
```yaml
provider: openrouter
model: google/gemini-2.5-flash
```

`~/.hermes/.env`:
```
# GOOGLE_API_KEY 주석 처리
OPENROUTER_API_KEY=sk-or-...
```

전환 후: `hermes-restart`

---

## 트러블슈팅

```bash
# 로그에서 에러만 필터
docker logs hermes 2>&1 | grep -i "error\|warning\|failed"

# 컨테이너 내부 진입
docker exec -it hermes sh

# 컨테이너 완전 재생성
hermes-down
docker rm hermes
hermes-up
```
