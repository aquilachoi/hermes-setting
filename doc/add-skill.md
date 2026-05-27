# 스킬 추가 절차

사용자가 "X 스킬 추가해 줘" 요청 시 아래 순서대로 실행.

## 1. 스킬 소스 확인

- `npx skills` 패키지 (GitHub repo): `npx skills add <owner/repo>`
- gstack처럼 별도 installer 있는 경우: 개별 처리 (아래 예외 참고)

## 2. 설치

```bash
# 모든 에이전트 대상 설치 (자동 감지)
npx skills add <owner/repo> -a "*" -y
```

설치 후 에이전트별 symlink 확인:

```bash
# .claude/skills/, .hermes/skills/에 누락된 symlink 생성
AGENTS_DIR=".agents/skills"
for skill in "$AGENTS_DIR"/*/; do
  name=$(basename "$skill")
  [ ! -e ".claude/skills/$name" ] && ln -s "../../.agents/skills/$name" ".claude/skills/$name"
  [ ! -e ".hermes/skills/$name" ] && ln -s "../../.agents/skills/$name" ".hermes/skills/$name"
done
```

> Codex CLI / Gemini CLI는 `.agents/skills/`를 직접 읽으므로 자동 지원.

## 3. doc/my-skills.md 업데이트

아래 형식으로 섹션 추가:

```markdown
## N. <owner/repo>

**소스:** `github.com/<owner/repo>`

### 설치 방법

\`\`\`bash
npx skills add <owner/repo> -a "*" -y
\`\`\`

### 설치되는 스킬 목록

| 스킬 | 설명 |
|------|------|
| `skill-name` | 설명 |
```

## 4. 커밋

```bash
git add .agents/skills/ .claude/skills/ .hermes/skills/ doc/my-skills.md
git commit -m "Add <owner/repo> skill suite"
git push origin main
```

---

## 예외: gstack 계열 (절대경로 symlink 사용)

gstack은 설치 시 절대경로 symlink 생성 → 프로젝트 repo에 커밋 불가.

```bash
# 글로벌 설치
git clone --single-branch --depth 1 <repo-url> ~/.claude/skills/<name>
cd ~/.claude/skills/<name> && ./setup
```

`doc/my-skills.md`에 글로벌 설치임을 명시. 커밋하지 않음.
