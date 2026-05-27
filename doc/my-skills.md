# 설치된 Skills

## 에이전트별 지원 현황

모든 스킬 원본은 `.agents/skills/`에 저장. Codex/Gemini CLI는 이 디렉터리를 직접 읽음. Claude Code/Hermes는 별도 symlink 디렉터리 사용.

| 에이전트 | Project skillsDir | 방식 |
|---------|------------------|------|
| Claude Code | `.claude/skills/` | symlink → `.agents/skills/` |
| Hermes Agent | `.hermes/skills/` | symlink → `.agents/skills/` |
| Codex CLI | `.agents/skills/` | 직접 읽기 (별도 디렉터리 불필요) |
| Gemini CLI | `.agents/skills/` | 직접 읽기 (별도 디렉터리 불필요) |
| GitHub Copilot | `.agents/skills/` | 직접 읽기 |

---

## 설치 방법

### 사전 조건

```bash
# Node.js 필요 (npx 사용)
node --version   # v18 이상 권장
```

### 기본 설치 명령

```bash
# 설치된 에이전트 자동 감지 후 모두에 설치
npx skills add <owner/repo> -a "*" -y

# 기존 프로젝트 복원 (skills-lock.json 있을 때)
npx skills experimental_install
```

---

## 에이전트별 설정 방법

#### Claude Code

```bash
npx skills add <owner/repo> -a claude
```

`.claude/skills/`에 symlink 생성. Claude Code에서 `/skill-name` 형태로 호출.

#### Gemini CLI

Gemini CLI는 `.agents/skills/`를 직접 읽으므로 별도 설정 불필요.

```bash
npx skills add <owner/repo> -a gemini-cli
```

#### Codex CLI

Codex CLI도 `.agents/skills/`를 직접 읽으므로 별도 설정 불필요.

```bash
npx skills add <owner/repo> -a codex
```

#### Hermes Agent

```bash
npx skills add <owner/repo> -a hermes-agent
```

`.hermes/skills/` → `.agents/skills/` symlink 생성.

---

## 스킬 목록

## 1. mattpocock/skills

**소스:** `github.com/mattpocock/skills`

### 설치 방법

```bash
npx skills add mattpocock/skills -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `diagnose` | 버그/성능 이슈 단계별 진단 루프 |
| `grill-me` | 플랜/디자인 인터뷰 방식 스트레스 테스트 |
| `grill-with-docs` | CONTEXT.md, ADR 기반 플랜 검증 |
| `handoff` | 대화 내용을 다른 에이전트용 handoff 문서로 압축 |
| `improve-codebase-architecture` | 코드베이스 아키텍처 개선 기회 탐색 |
| `prototype` | 터미널 앱 또는 UI 변형 프로토타입 생성 |
| `setup-matt-pocock-skills` | 이 스킬 모음 초기 설정 (이슈 트래커, 라벨 등) |
| `tdd` | TDD 방식 구현 |
| `to-issues` | 플랜/스펙을 GitHub 이슈로 분해 |
| `to-prd` | 요구사항을 PRD로 변환 |
| `triage` | 이슈 트리아지 state machine |
| `write-a-skill` | 새 skill 작성 |
| `zoom-out` | 현재 작업의 더 큰 맥락 파악 |

### 초기 설정 (최초 1회)

```
/setup-matt-pocock-skills
```

이슈 트래커(GitHub/GitLab/로컬 markdown), 트리아지 라벨, 도메인 문서 경로 설정.

---

## 2. JuliusBrussee/caveman

**소스:** `github.com/JuliusBrussee/caveman`

### 설치 방법

```bash
npx skills add JuliusBrussee/caveman -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `caveman` | 토큰 ~75% 절감 압축 통신 모드. 6단계 intensity 지원 |
| `cavecrew` | 전문화된 서브에이전트 팀 오케스트레이션 |
| `caveman-commit` | 압축 스타일 git 커밋 메시지 생성 |
| `caveman-compress` | CLAUDE.md 등 메모리 파일을 caveman 포맷으로 압축 |
| `caveman-help` | caveman 모드/커맨드 퀵 레퍼런스 |
| `caveman-review` | 초압축 코드 리뷰 코멘트 생성 |
| `caveman-stats` | 세션 토큰 절감 통계 확인 |

### 사용 방법

```
/caveman          # full 모드 활성화 (기본)
/caveman lite     # 라이트 모드 (문장 유지, filler만 제거)
/caveman ultra    # 최대 압축 (약어, 화살표 표기)
/caveman wenyan-full  # 문언문 스타일 최대 압축
stop caveman      # 일반 모드 복귀
```

### Intensity 레벨

| 레벨 | 토큰 절감 | 특징 |
|------|---------|------|
| lite | ~20-30% | filler 제거, 문장 구조 유지 |
| full | ~50-60% | 관사 제거, 단편 OK (기본값) |
| ultra | ~70-75% | 약어, 화살표 인과관계 표기 |
| wenyan-full | ~80-90% | 문언문 최대 압축 |

---

## 3. imxv/pretty-mermaid-skills

**소스:** `github.com/imxv/pretty-mermaid-skills`

### 설치 방법

```bash
npx skills add imxv/pretty-mermaid-skills -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `pretty-mermaid` | Mermaid 다이어그램을 시각적으로 보기 좋게 생성/개선 |

---

## 4. obra/superpowers

**소스:** `github.com/obra/superpowers`

### 설치 방법

```bash
npx skills add obra/superpowers -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `brainstorming` | 아이디어 브레인스토밍 |
| `using-superpowers` | superpowers 스킬 활용 가이드 |
| `systematic-debugging` | 체계적 디버깅 방법론 |
| `writing-plans` | 구현 플랜 작성 |
| `requesting-code-review` | 코드 리뷰 요청 방법 |
| `receiving-code-review` | 코드 리뷰 수신 및 처리 |
| `test-driven-development` | TDD 방법론 |
| `dispatching-parallel-agents` | 병렬 에이전트 디스패치 |
| `executing-plans` | 플랜 실행 |
| `finishing-a-development-branch` | 개발 브랜치 완료 절차 |
| `subagent-driven-development` | 서브에이전트 기반 개발 |
| `using-git-worktrees` | git worktree 활용 |
| `verification-before-completion` | 완료 전 검증 절차 |
| `writing-skills` | 문서/코드 작성 스킬 |

---

## 5. safishamsi/graphify

**소스:** `github.com/safishamsi/graphify`

### 설치 방법

```bash
npx skills add safishamsi/graphify -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `graphify` | 데이터/개념을 그래프/다이어그램으로 시각화 |

---

## 6. openai/codex-plugin-cc

**소스:** `github.com/openai/codex-plugin-cc`

### 설치 방법

```bash
npx skills add openai/codex-plugin-cc -a "*" -y
```

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `codex-cli-runtime` | Codex CLI 런타임 환경 설정 및 실행 |
| `codex-result-handling` | Codex 결과 처리 및 후속 작업 |
| `gpt-5-4-prompting` | GPT-4.5 프롬프팅 최적화 |

---

## 7. garrytan/gstack

**소스:** `github.com/garrytan/gstack`

### 사전 요건

```bash
brew install oven-sh/bun/bun   # bun 런타임 필요
```

### 설치 방법

gstack은 절대경로 symlink를 사용하므로 **글로벌(user-level)** 에 설치. 프로젝트 repo에 커밋하지 않음.

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git \
  ~/.claude/skills/gstack

cd ~/.claude/skills/gstack && ./setup
```

> 설치 위치: `~/.claude/skills/gstack` → 모든 프로젝트에서 자동 사용 가능.  
> Codex CLI / Gemini CLI / Hermes Agent는 별도 설정 불필요 (Claude Code global skill 공유).

### 업그레이드

```bash
/gstack-upgrade
```

---

### 스킬 전체 목록

#### 핵심 & 설정

| 스킬 | 설명 |
|------|------|
| `gstack` | gstack 메인 진입점. 전체 도구 스택의 오케스트레이터 역할 |
| `gstack-upgrade` | gstack 업데이트 관리. 글로벌/로컬 설치 자동 감지 후 업그레이드, 변경 내역 표시 |
| `setup-deploy` | `/land-and-deploy` 워크플로에 필요한 배포 설정 자동 구성. Fly.io, Vercel, Netlify 등 플랫폼 자동 감지 후 CLAUDE.md에 저장 |
| `setup-gbrain` | gbrain(코드 시맨틱 검색 엔진)을 로컬 Mac에 설정. Supabase, PGLite, Remote MCP 중 선택 가능 |
| `find-skills` | 원하는 기능에 맞는 스킬 검색 및 설치 방법 안내 |

#### 브라우저 & QA

| 스킬 | 설명 |
|------|------|
| `browse` | 헤드리스 Chromium으로 웹 페이지 탐색, 스크린샷 촬영, 요소 클릭/입력 등 자동화. 쿠키·세션 상태 유지 |
| `open-gstack-browser` | 화면에 보이는 Chromium 브라우저 실행. AI가 브라우저를 조작하는 과정을 사이드바에서 실시간 확인 가능 |
| `qa` | 실제 사용자처럼 웹 앱 전체를 테스트. 버그 발견 → 스크린샷 첨부 → 소스 코드 수정 → 재검증까지 자동 수행 |
| `qa-only` | QA 리포트만 생성 (코드 수정 없음). 버그 목록·헬스 스코어·재현 방법을 구조화된 보고서로 출력 |
| `scrape` | 웹 데이터 추출. match 모드(~200ms, 기존 패턴 재사용)와 prototype 모드(~30s, 직접 탐색) 두 가지 동작 방식 |
| `pair-agent` | 다른 AI 에이전트와 브라우저를 안전하게 공유. 설정 키 하나로 연결, 탭 단위 접근 권한 제어 |

#### 디자인

| 스킬 | 설명 |
|------|------|
| `design-html` | 디자인을 프로덕션급 HTML/CSS로 변환. Pretext 타이포그래피 엔진 사용, 반응형, 외부 의존성 없음. React/Vue/Svelte 지원 |
| `design-shotgun` | 여러 UI 변형을 동시에 생성해 비교 보드에 표시. 피드백 수집 → 반복 수정으로 최적 디자인 도출 |
| `design-review` | 실제 운영 웹사이트 디자인 감사. 타이포그래피·간격·색상 등 10개 항목 평가 후 CSS 수정 적용 |
| `design-consultation` | 제품 맥락 분석 후 일관된 디자인 시스템(DESIGN.md) 생성. 색상·폰트·레이아웃·모션을 하나의 패키지로 제안 |

#### iOS 개발

| 스킬 | 설명 |
|------|------|
| `ios-qa` | USB로 연결된 실제 iPhone에서 SwiftUI 앱 자동 테스트. 스크린샷 분석 → 조작 → 결과 검증 루프 반복 |
| `ios-fix` | iOS 버그 자율 수정. 재현 스냅샷 캡처 → 원인 파악 → 수정 적용 → 회귀 테스트 작성까지 자동화 |
| `ios-clean` | 개발 완료 후 DebugBridge SPM 패키지와 `#if DEBUG` 계측 코드 제거. Release 빌드 성공 검증 포함 |
| `ios-design-review` | 실제 기기에서 iOS 앱 디자인 감사. Apple HIG 기준 10개 항목 평가 및 "10점 만점을 위해 무엇이 필요한가" 가이드 제공 |
| `ios-sync` | gstack 업그레이드 또는 새 `@Observable` 클래스 추가 시 iOS 디버그 브릿지 아티팩트 재생성 |

#### 플랜 리뷰 & 전략

| 스킬 | 설명 |
|------|------|
| `autoplan` | CEO·디자인·엔지니어링·DX 리뷰를 자동으로 순차 실행. 기계적 질문은 자율 결정, 취향 관련 결정만 사용자에게 전달 |
| `plan-ceo-review` | 전략적 관점 플랜 리뷰. 범위 확장/축소 4가지 모드 제공, 아키텍처·보안·배포까지 11개 항목 심층 검토 |
| `plan-eng-review` | 코딩 전 아키텍처 및 구현 계획 철저 검토. 아키텍처·코드 품질·테스트·성능에 대한 구체적 권고 제공 |
| `plan-design-review` | 구현 전 플랜의 디자인 완성도 리뷰. 7개 항목 평가 후 시각적 목업 생성, 설계 개선 |
| `plan-devex-review` | 개발자 경험(DX) 관점 플랜 감사. 타깃 개발자 여정 추적 후 8개 DX 항목 점수화, 우선순위 개선 사항 도출 |
| `plan-tune` | 질문 응답 패턴을 학습하는 대화형 코치. 선호 결정 방식을 파악해 이후 플랜 리뷰 방향 자동 조정 |
| `office-hours` | YC 스타일 브레인스토밍 파트너. 스타트업 모드(6가지 날카로운 질문)와 빌더 모드(열정적 설계 파트너) 지원 |

#### 배포 & 모니터링

| 스킬 | 설명 |
|------|------|
| `ship` | PR 생성부터 CI까지 ~20단계 자동화. 사전 점검·테스트·커버리지·리뷰·버전 범프·체인지로그·PR 생성을 한 번에 처리 |
| `land-and-deploy` | GitHub PR 머지 → CI/CD 대기 → 카나리 검증까지 자동화. `/ship` 이후 단계를 안전 게이트와 모니터링으로 처리 |
| `canary` | 배포 후 ~10분간 운영 앱 모니터링. 페이지 오류·콘솔 에러·성능 저하·깨진 링크를 스크린샷 비교로 자동 감지 |

#### 문서화 & 지식 관리

| 스킬 | 설명 |
|------|------|
| `document-generate` | Diataxis 프레임워크(튜토리얼·how-to·레퍼런스·설명) 기반 완전한 문서 자동 생성 |
| `document-release` | 배포 후 README·ARCHITECTURE·CONTRIBUTING·CHANGELOG 등이 코드 변경과 동기화되었는지 감사 |
| `learn` | 세션 간 프로젝트 학습 내용(패턴·함정·인사이트) 관리. 과거 수정 내역 검색, 오래된 항목 정리 |
| `retro` | 주간 엔지니어링 회고. 커밋 히스토리·작업 패턴·코드 품질 메트릭 분석, 팀별 세분화 및 추세 분석 |
| `make-pdf` | 마크다운을 전문적인 PDF로 변환. 여백·폰트·목차·워터마크 설정 가능 |

#### 컨텍스트 & 메모리

| 스킬 | 설명 |
|------|------|
| `context-save` | 현재 세션 상태(브랜치·변경사항·대화 흐름)를 마크다운 체크포인트로 저장 |
| `context-restore` | 가장 최근 저장된 컨텍스트를 불러와 이전 작업을 이어서 진행 |
| `sync-gbrain` | gbrain 코드 검색 엔진을 최신 코드와 동기화. 증분/전체 동기화 지원, CLAUDE.md 에이전트 가이드 갱신 |

#### 보안 & 안전

| 스킬 | 설명 |
|------|------|
| `cso` | 최고보안책임자(CSO) 역할 감사. git 시크릿 누출·취약 의존성·CI/CD 보안·OWASP Top 10·LLM 특화 위험 스캔 |
| `careful` | 위험한 Bash 명령(rm -rf, DROP TABLE, git push -f 등) 실행 전 차단 및 경고. 사용자가 명시 승인 시 실행 허용 |
| `freeze` | 세션 중 파일 편집을 지정 디렉토리로 제한. 지정 범위 밖 파일 수정 시도 차단 |
| `unfreeze` | `/freeze`로 설정된 편집 제한 해제. 모든 디렉토리 수정 권한 복원 |
| `guard` | `/careful`과 `/freeze` 기능 통합. 프로덕션 환경처럼 위험도가 높은 상황에서 편집 범위를 특정 경계로 제한 |

#### 코드 리뷰 & 분석

| 스킬 | 설명 |
|------|------|
| `review` | PR 코드 변경 분석. SQL 안전성·LLM 신뢰 위반·경쟁 조건 등 구조적 문제를 자동 탐지 후 전문가 디스패치 |
| `codex` | 다른 AI 시스템(OpenAI Codex)에게 독립적 코드 리뷰 요청. review·challenge·consult 3가지 모드 제공 |
| `investigate` | 버그·이슈 근본 원인 조사. 재현 → 분석 → 원인 파악 단계적 접근 |

#### 성능 & 벤치마크

| 스킬 | 설명 |
|------|------|
| `benchmark` | 웹 페이지 성능 측정 (로드 시간·Core Web Vitals·리소스 크기). 기준선 대비 회귀 감지 |
| `benchmark-models` | Claude·GPT·Gemini 동일 프롬프트 비교 실행. 응답 속도·토큰 사용량·비용·품질 측면별 평가 |
| `devex-review` | 개발자 경험 현장 감사. 문서·CLI 도움말·에러 메시지·시작 흐름을 직접 체험하며 8개 DX 항목 점수 산출 |

#### 유틸리티

| 스킬 | 설명 |
|------|------|
| `skillify` | 성공한 `/scrape` 프로토타입을 재사용 가능한 영구 스킬로 변환. 이후 동일 작업을 ~200ms에 처리 |
