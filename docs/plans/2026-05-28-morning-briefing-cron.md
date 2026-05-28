# 모닝 브리핑 Cron 스케쥴 등록 구현 플랜

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Hermes Docker 컨테이너에 매주 월~금 KST 오전 9시 모닝 브리핑 cron job을 등록하여 Slack #morning-briefing 채널로 자동 발송한다.

**Architecture:** `~/.hermes/scripts/morning-briefing.sh` 스크립트에 브리핑 프롬프트를 embed한다. `hermes cron create --script` 옵션으로 스크립트 stdout을 에이전트 프롬프트로 주입한다. 에이전트가 WebSearch + slack_send_message 내장 툴로 데이터 수집 및 Slack 발송까지 완료한다.

**Tech Stack:** Hermes Agent (Docker), bash script, hermes cron CLI, Slack Bot

---

## 파일 구조

| 파일 | 동작 |
|------|------|
| `~/.hermes/scripts/morning-briefing.sh` (신규) | 브리핑 프롬프트 출력. cron 실행 시 stdout이 에이전트 프롬프트로 주입됨 |

> 참고: `~/.hermes/` 는 컨테이너 내부 `/opt/data/` 에 볼륨 마운트됨. 호스트에서 파일 생성하면 컨테이너가 즉시 인식.

---

### Task 1: morning-briefing.sh 스크립트 생성

**Files:**
- Create: `~/.hermes/scripts/morning-briefing.sh`

- [ ] **Step 1: scripts 디렉토리 생성**

```bash
mkdir -p ~/.hermes/scripts
```

Expected: 오류 없이 완료. `ls ~/.hermes/scripts/` 로 빈 디렉토리 확인.

- [ ] **Step 2: morning-briefing.sh 작성**

아래 내용을 `~/.hermes/scripts/morning-briefing.sh` 로 저장한다. (`'PROMPT'` 싱글 쿼트 heredoc으로 `$` 등 특수문자 escape 불필요.)

```bash
#!/bin/bash
cat <<'PROMPT'
당신은 PNC 무역회사(재활용 플라스틱 원료 전문)의 아침 브리핑 어시스턴트입니다.
매일 아침 아래 데이터를 수집·정리하여 Slack #morning-briefing 채널(ID: C0B5Z8XUV41)로 발송하세요.

## 수집할 데이터

### 1. HDPE / LDPE 국제 원료가
WebSearch를 사용하여 최신 HDPE, LDPE 국제 시세를 조회하세요.
- 검색어 예시: "HDPE price today 2026", "LDPE resin price this week"
- 참고 소스: ICIS, Plastics News, ChemAnalyst, Chemical Week
- 수집 항목: 현재가(USD/톤), 전주 대비 변동, 시장 동향 한 줄 요약

### 2. USD/KRW 환율
WebSearch를 사용하여 당일 USD/KRW 환율을 조회하세요.
- 검색어 예시: "USD KRW exchange rate today"
- 수집 항목: 현재 환율, 전일 대비 변동(원 단위), 방향(강세/약세)

### 3. 혼합 원료가 계산
HDPE와 LDPE 시세를 바탕으로 PNC 취급 원료의 혼합 기준가를 계산하세요.
- 혼합 비율: HDPE 60% + LDPE 40%
- 계산식: (HDPE가격 × 0.6) + (LDPE가격 × 0.4) = 혼합 기준가 (USD/톤)

## Slack 발송 포맷

아래 형식으로 Slack 메시지를 작성하여 채널 ID C0B5Z8XUV41 로 발송하세요.
Slack 도구(slack_send_message)를 사용하세요.

---
📊 *PNC 모닝 브리핑* | {오늘 날짜 예: 2026년 5월 26일 (화)}

*💱 USD/KRW 환율*
> {환율} 원  |  전일 대비 {+/-X} 원  ({달러 강세 or 원화 강세})

*📦 재활용 플라스틱 원료가 (국제)*
> HDPE: ${가격}/톤  |  전주 대비 {+/-$X}
> LDPE: ${가격}/톤  |  전주 대비 {+/-$X}
> ━━━━━━━━━━━━━━━
> 🔢 *혼합 기준가 (HDPE 60% + LDPE 40%)*
> *$혼합가/톤*  ({계산식: HDPE×0.6 + LDPE×0.4})

*📝 오늘의 시장 포인트*
> {환율·원료가 동향을 바탕으로 PNC 수출 원가에 미치는 영향을 2-3문장으로 간략히 요약. 예: 원화 약세로 수출 채산성 개선 / HDPE 강세로 원가 압박 등}

_Powered by Hermes | 데이터 출처: {검색한 주요 소스명}_
---

## 주의사항
- 데이터를 찾지 못한 경우, 해당 항목에 "데이터 수집 실패 - 수동 확인 필요" 라고 표시하고 발송
- 혼합 기준가는 HDPE와 LDPE 가격을 모두 수집한 경우에만 계산하여 표시
- 시장 포인트는 PNC의 사업 관점(재활용 HDPE 60%/LDPE 40% 혼합 원료 수출)에서 작성
- 반드시 Slack 메시지 발송까지 완료할 것
PROMPT
```

- [ ] **Step 3: 실행 권한 부여**

```bash
chmod +x ~/.hermes/scripts/morning-briefing.sh
```

- [ ] **Step 4: 스크립트 동작 확인**

```bash
~/.hermes/scripts/morning-briefing.sh | head -3
```

Expected 출력:
```
당신은 PNC 무역회사(재활용 플라스틱 원료 전문)의 아침 브리핑 어시스턴트입니다.
매일 아침 아래 데이터를 수집·정리하여 Slack #morning-briefing 채널(ID: C0B5Z8XUV41)로 발송하세요.

```

---

### Task 2: Hermes Cron Job 등록

**Files:**
- 변경 없음 (hermes 내부 상태 변경)

- [ ] **Step 1: cron job 등록**

```bash
docker exec hermes hermes cron create "0 0 * * 1-5" \
  --name morning-briefing \
  --script morning-briefing.sh
```

Expected: 오류 없이 완료. job ID 또는 성공 메시지 출력.

- [ ] **Step 2: 등록 확인**

```bash
docker exec hermes hermes cron list
```

Expected 출력 (예시):
```
┏━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┓
┃ Name              ┃ Schedule      ┃ Status  ┃ Next Run            ┃
┡━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━┩
│ morning-briefing  │ 0 0 * * 1-5   │ active  │ ...                 │
└───────────────────┴───────────────┴─────────┴─────────────────────┘
```

`morning-briefing` job이 `0 0 * * 1-5` 스케쥴로 `active` 상태인지 확인.

---

### Task 3: 수동 트리거 테스트

**Files:**
- 변경 없음

- [ ] **Step 1: 즉시 실행 트리거**

```bash
docker exec hermes hermes cron run morning-briefing
```

Expected: 에이전트 실행 시작 메시지 출력. 완료까지 30초~2분 소요.

- [ ] **Step 2: Slack 채널 확인**

Slack `#morning-briefing` 채널(C0B5Z8XUV41) 접속.

아래 항목 모두 존재하는지 확인:
- `📊 *PNC 모닝 브리핑*` 헤더
- `*💱 USD/KRW 환율*` 섹션
- `*📦 재활용 플라스틱 원료가 (국제)*` 섹션 (HDPE, LDPE 가격)
- `🔢 *혼합 기준가*` 섹션
- `*📝 오늘의 시장 포인트*` 섹션

데이터 수집 실패 시: `"데이터 수집 실패 - 수동 확인 필요"` 텍스트가 해당 항목에 표시되면 정상 동작.

---

### Task 4: 커밋

**Files:**
- Commit: `docs/plans/2026-05-28-morning-briefing-cron.md`

> 주의: `~/.hermes/scripts/morning-briefing.sh` 는 홈 디렉토리 하위이므로 이 repo에 포함되지 않음. 플랜 파일만 커밋.

- [ ] **Step 1: 커밋**

```bash
git -C ~/Desktop/workspace/hermes-setting add docs/plans/2026-05-28-morning-briefing-cron.md
git -C ~/Desktop/workspace/hermes-setting commit -m "docs: add morning-briefing cron implementation plan"
```

---

## 셀프 리뷰

**스펙 커버리지 체크:**
- ✅ `0 0 * * 1-5` 스케쥴 (UTC 0시 = KST 9시, 월~금)
- ✅ `morning-briefing.sh` 스크립트 생성 및 embed
- ✅ `--name morning-briefing` 지정
- ✅ `--deliver` 없음 (프롬프트 내 slack_send_message 사용)
- ✅ `--skill` 없음 (내장 툴)
- ✅ `docker-compose.yml` 변경 없음
- ✅ Slack 메시지 수신 확인 테스트 포함

**플레이스홀더 없음 확인:** 모든 커맨드와 예상 출력 명시됨.
