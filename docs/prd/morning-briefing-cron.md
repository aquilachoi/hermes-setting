# PRD: Hermes 모닝 브리핑 자동 스케쥴

## 문제 정의

PNC 무역회사(재활용 플라스틱 원료 전문)의 운영진은 매일 아침 HDPE/LDPE 국제 원료가, USD/KRW 환율, 혼합 기준가를 수동으로 확인하고 정리해야 한다. 이 작업은 반복적이고 시간 소모적이며, 데이터 수집을 잊거나 늦어지면 당일 의사결정에 지장을 준다.

## 해결책

로컬에서 실행 중인 Hermes 에이전트에 cron 스케쥴을 등록하여, 매주 월~금 오전 9시(KST)에 자동으로 데이터를 수집·계산하고 Slack `#morning-briefing` 채널(ID: C0B5Z8XUV41)로 포맷된 브리핑 메시지를 발송한다.

## 사용자 스토리

1. PNC 운영진으로서, 매일 오전 9시(KST)에 모닝 브리핑을 자동 수신하고 싶다. 수동 조회 없이 최신 시장 정보로 업무를 시작하기 위해.
2. PNC 운영진으로서, 현재 HDPE 국제 시세(USD/톤)를 확인하고 싶다. 원료 조달 비용을 파악하기 위해.
3. PNC 운영진으로서, 현재 LDPE 국제 시세(USD/톤)를 확인하고 싶다. 혼합 원료 비용을 정확히 산출하기 위해.
4. PNC 운영진으로서, HDPE·LDPE 전주 대비 가격 변동을 확인하고 싶다. 한눈에 시장 추세를 파악하기 위해.
5. PNC 운영진으로서, 당일 USD/KRW 환율을 확인하고 싶다. 환율 변동이 수출 채산성에 미치는 영향을 파악하기 위해.
6. PNC 운영진으로서, USD/KRW 전일 대비 변동(원 단위)과 방향(강세/약세)을 확인하고 싶다. 달러 강세·원화 강세를 빠르게 판단하기 위해.
7. PNC 운영진으로서, 혼합 기준가(HDPE 60% + LDPE 40%)를 자동 계산한 결과를 확인하고 싶다. PNC 제품 구성에 맞는 가중 투입 원가를 즉시 파악하기 위해.
8. PNC 운영진으로서, PNC 수출 관점에서 작성된 2~3문장 시장 포인트를 확인하고 싶다. 별도 분석 없이 비즈니스 시사점을 이해하기 위해.
9. PNC 운영진으로서, 브리핑을 Slack `#morning-briefing` 채널로 수신하고 싶다. 팀 전체가 기존 커뮤니케이션 채널에서 동시에 확인하기 위해.
10. PNC 운영진으로서, 브리핑을 월~금에만 자동 실행하고 싶다. 시장이 닫혀 데이터가 부실한 주말에 불필요한 알림을 받지 않기 위해.
11. PNC 운영진으로서, 데이터 수집 실패 시 해당 항목에 명확한 안내 문구가 표시되길 원한다. 누락 데이터를 수동으로 확인해야 함을 즉시 알기 위해.
12. 시스템 관리자로서, 브리핑 프롬프트를 버전 관리되는 단일 파일(`morning-briefing.md`)로 유지하고 싶다. cron job을 재생성하지 않고 브리핑 형식이나 데이터 소스를 수정하기 위해.
13. 시스템 관리자로서, cron job이 기존 Hermes Docker 컨테이너 안에서 실행되길 원한다. 이미 구성된 환경·자격증명·툴 연동을 그대로 활용하기 위해.

## 구현 결정사항

### Cron Job 등록
- 실행 중인 Hermes 컨테이너 내부에서 `hermes cron create` 내장 커맨드 사용.
- 스케쥴 표현식: `0 0 * * 1-5` — UTC 자정 = KST 09:00, 월~금.
- Job 이름: `morning-briefing`.

### 스크립트를 통한 프롬프트 전달
- 호스트의 `~/.hermes/scripts/morning-briefing.sh` 생성 (컨테이너 내부에서 `/opt/data/scripts/morning-briefing.sh`로 접근).
- 스크립트는 `morning-briefing.md` 내용을 정적으로 embed하여 stdout으로 출력.
- 선택 이유: Docker 볼륨 추가 없이 구현 가능. 트레이드오프: 프롬프트 변경 시 스크립트 수동 재생성 필요.
- 스크립트 stdout이 매 실행 시 Hermes 에이전트 프롬프트로 주입됨 (`--script` 모드, 에이전트 실행).

### 툴 사용 (`--skill` 플래그 불필요)
- `WebSearch` — HDPE/LDPE 시세 및 USD/KRW 환율 조회에 사용. Hermes 내장 툴이므로 skill 플래그 불필요.
- `slack_send_message` — 에이전트가 포맷된 브리핑 발송에 사용. 설정된 봇 토큰 기반 Hermes 내장 Slack 연동이므로 skill 플래그 불필요.
- `--deliver` 플래그 없음 — 프롬프트 내부에서 `slack_send_message` 직접 호출. `--deliver` 추가 시 중복 발송 발생.

### 데이터 수집 전략
- 검색어는 `morning-briefing.md`에 정의: `"HDPE price today 2026"`, `"LDPE resin price this week"`, `"USD KRW exchange rate today"`.
- 우선 참고 소스: ICIS, Plastics News, ChemAnalyst, Chemical Week.
- Fallback: 데이터 수집 실패 시 해당 항목에 `"데이터 수집 실패 - 수동 확인 필요"` 표시 후 발송 진행.

### 혼합 기준가 계산
- 계산식: `(HDPE 가격 × 0.6) + (LDPE 가격 × 0.4)` — 두 가격 모두 수집된 경우에만 계산.

### Docker 인프라
- `docker-compose.yml` 변경 없음. 컨테이너 재시작 불필요.
- Hermes 컨테이너(`hermes`)는 `restart: unless-stopped` 유지. cron 스케쥴러는 컨테이너 프로세스 내부에서 실행.

## 테스트 결정사항

### 좋은 테스트 기준
- 관찰 가능한 출력만 테스트: Slack 메시지 발송 여부, 메시지 내 필수 섹션 존재 여부, 데이터 수집 실패 시 fallback 텍스트 표시 여부.
- 내부 검색 쿼리 문자열이나 중간 계산 단계는 테스트하지 않는다.

### 테스트 대상
- **Cron 등록 확인**: `hermes cron list`로 올바른 스케쥴·이름으로 등록됐는지 확인.
- **수동 트리거**: `docker exec hermes hermes cron run morning-briefing`으로 에이전트 실행 후 Slack `#morning-briefing` 채널에 메시지 도착 확인.
- **Fallback 동작**: 검색 실패 상황을 유도하여 Slack 메시지에 fallback 텍스트가 표시되는지 확인.

### 선례
- 이 repo에 자동화 테스트 스위트 없음. 모든 테스트는 Slack 채널 출력을 통한 수동·관찰적 방식.

## 범위 외

- cron 실행 시 LLM 모델 변경 (`config.yaml`의 `gemini-flash-latest` 기본값 사용).
- 주말 실행 또는 공휴일 인식 로직.
- `morning-briefing.md` 변경 시 cron 자동 반영 (수동 스크립트 재생성 필요).
- Slack 외 이메일 등 다른 발송 채널.
- 과거 시세 데이터 저장 및 차트 생성.
- hermes-setting 워크스페이스를 위한 Docker 볼륨 추가.

## 참고 사항

- Hermes 컨테이너 시간대: UTC. 모든 cron 표현식은 UTC 기준으로 작성 (KST = UTC+9).
- `morning-briefing.md`는 `docs/schedule/morning-briefing.md`에 위치하며 브리핑 프롬프트의 단일 소스. 프롬프트 변경 시 `morning-briefing.sh`를 수동으로 재생성해야 한다.
