# Design Tools

## @google/design.md

디자인 시스템과 코드를 연결하는 CLI 도구. DESIGN.md 포맷 lint/diff/export 지원.

**소스:** `github.com/google-labs-code/design.md`

### 설치 방법

```bash
npm install @google/design.md
```

> `package.json`에 의존성 등록됨. 재설치: `npm install`

### 주요 명령

| 명령 | 설명 |
|------|------|
| `npx design.md lint` | DESIGN.md 파일 구조 검증 |
| `npx design.md diff` | 두 DESIGN.md 파일 변경사항 비교 |
| `npx design.md export` | 디자인 토큰 export (CSS/Tailwind/DTCG) |
| `npx design.md spec` | DESIGN.md 포맷 스펙 출력 |

### Export 포맷

| 포맷 | 명령 |
|------|------|
| Tailwind v4 CSS `@theme` | `npx design.md export css-tailwind` |
| Tailwind v3 `theme.extend` JSON | `npx design.md export json-tailwind` |
| W3C Design Tokens (DTCG) | `npx design.md export dtcg` |
