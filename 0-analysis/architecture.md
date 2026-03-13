# 아키텍처

## 디렉토리 구조

```
agent-stuff/
├── package.json              ← npm 패키지 (mitsupi), Pi 로더 설정
├── AGENTS.md                 ← 에이전트용 릴리즈/확장 가이드
├── CHANGELOG.md              ← 버전별 변경 이력
│
├── skills/                   ← 스킬 (= 에이전트가 수행하는 작업 단위)
│   ├── commit/               ← git 커밋 자동화
│   ├── sentry/               ← Sentry 에러 조회/분석
│   ├── github/               ← GitHub CLI 래퍼
│   ├── web-browser/          ← CDP 기반 브라우저 제어
│   ├── tmux/                 ← tmux 세션 원격 제어
│   ├── frontend-design/      ← 프론트엔드 디자인 가이드
│   ├── librarian/            ← 원격 repo 로컬 캐시 관리
│   ├── uv/                   ← Python uv 패키지 관리
│   ├── summarize/            ← 파일/URL → Markdown 변환
│   ├── ghidra/               ← 리버스 엔지니어링
│   ├── mermaid/              ← 다이어그램 생성
│   ├── google-workspace/     ← Google API 연동
│   ├── apple-mail/           ← Apple Mail 로컬 검색
│   ├── native-web-search/    ← 네이티브 웹 검색
│   ├── pi-share/             ← Pi 세션 트랜스크립트 파싱
│   ├── openscad/             ← 3D 모델링
│   ├── anachb/               ← 오스트리아 대중교통 API
│   ├── oebb-scotty/          ← 오스트리아 철도 API
│   └── update-changelog/     ← 체인지로그 업데이트
│
├── pi-extensions/            ← Pi TUI 확장 (TypeScript)
│   ├── review.ts             ← 코드 리뷰 (PR, diff, uncommitted 등)
│   ├── session-breakdown.ts  ← 세션/비용 분석 대시보드
│   ├── files.ts              ← 파일 브라우저 + git status
│   ├── todos.ts              ← TODO 관리
│   ├── loop.ts               ← 반복 실행 루프
│   ├── prompt-editor.ts      ← 프롬프트 모드 관리
│   ├── answer.ts             ← 질문 대화형 응답
│   ├── context.ts            ← 컨텍스트 정보 표시
│   ├── control.ts            ← 세션 제어
│   ├── notify.ts             ← 데스크톱 알림
│   ├── go-to-bed.ts          ← 심야 안전장치
│   ├── whimsical.ts          ← 재치있는 thinking 메시지
│   ├── uv.ts                 ← uv 헬퍼
│   └── multi-edit.ts         ← 다중 편집
│
├── pi-themes/                ← 테마
│   └── nightowl.json
│
├── commands/                 ← 사용자 프롬프트 명령 (.gitignored)
│
├── plumbing-commands/        ← 커스터마이징 필요한 명령
│   └── make-release.md
│
└── intercepted-commands/     ← 명령 인터셉터 (pip→uv 강제)
    ├── pip                   ← "pip 금지, uv 쓰세요" 에러
    ├── pip3                  ← 위와 동일
    ├── poetry                ← poetry도 차단
    ├── python                ← uv run python으로 리디렉트
    └── python3               ← 위와 동일 + pip/venv/py_compile 차단
```

## 핵심 설계 패턴

### 1. 스킬 = SKILL.md + scripts/

스킬의 기본 구조:

```
skills/<name>/
├── SKILL.md          ← 에이전트가 읽는 지시서 (frontmatter + 마크다운)
└── scripts/          ← 실행 가능한 도구 (Node.js, bash)
```

**SKILL.md frontmatter**:
```yaml
---
name: sentry
description: "Fetch and analyze Sentry issues..."
---
```

- `name`: 스킬 이름 (슬래시 명령으로 호출 가능)
- `description`: 에이전트가 스킬을 언제 사용할지 판단하는 기준

**핵심 원칙**: SKILL.md는 에이전트에게 주는 **프롬프트**다. 사람이 읽는 문서가 아니라, 에이전트가 따라야 할 절차를 기술한다.

### 2. 명령 인터셉트 (Intercepted Commands)

Pi의 `intercepted-commands` 기능을 활용하여 에이전트가 특정 명령을 실행하려 할 때 가로챈다:

| 원래 명령 | 인터셉트 동작 |
|----------|-------------|
| `pip`, `pip3` | 에러 + "uv 쓰세요" 안내 |
| `poetry` | 에러 + uv 대안 안내 |
| `python`, `python3` | `uv run python`으로 리디렉트 + pip/venv/py_compile 차단 |

**설계 의도**: AI 에이전트가 습관적으로 pip install을 시도하는 것을 원천 차단. 에러 메시지에 올바른 대안을 포함시켜 에이전트가 즉시 수정할 수 있게 한다.

### 3. Pi Extension 아키텍처

Pi 확장은 TypeScript로 작성되며 TUI(Terminal UI) 컴포넌트를 포함한다:

```typescript
import type { ExtensionAPI, ExtensionContext } from "@mariozechner/pi-coding-agent";
import { Container, SelectList, Text } from "@mariozechner/pi-tui";
```

주요 패턴:
- **TUI 기반 인터랙션**: SelectList, Input 등으로 사용자 선택 UI 제공
- **세션 데이터 분석**: JSONL 로그 파싱 → 통계 → 시각화 (session-breakdown)
- **Git 통합**: diff, PR checkout, branch 비교 등 (review, files)
- **상태 지속**: 파일 기반 설정 저장 (prompt-editor)

### 4. 외부 도구 통합 패턴

| 도구 | 통합 방식 | 핵심 |
|------|----------|------|
| **Sentry** | Node.js 스크립트로 REST API 호출 | `~/.sentryclirc`에서 토큰 읽기, 워크플로우별 명령 분리 |
| **GitHub** | `gh` CLI 래퍼 | 이미 강력한 CLI가 있으면 래퍼만 |
| **Chrome** | CDP(DevTools Protocol) 직접 연결 | nav/eval/screenshot/pick 등 원자적 명령 |
| **tmux** | send-keys + capture-pane | 대화형 CLI 도구(python, gdb)를 에이전트가 제어 |

### 5. 패키지 배포 구조

```json
{
  "name": "mitsupi",
  "pi": {
    "extensions": ["./pi-extensions"],
    "skills": ["./skills"],
    "themes": ["./pi-themes"],
    "prompts": ["./commands"]
  }
}
```

npm 패키지로 배포 → Pi의 패키지 로더가 자동으로 스킬/확장/테마를 로드. 하나의 패키지에 모든 커스터마이징을 번들.

## 설계 철학

1. **에이전트를 가드레일 안에 가두기**: intercepted-commands로 잘못된 습관 차단, SKILL.md로 올바른 절차 제시
2. **스크립트는 원자적으로**: 하나의 스크립트가 하나의 일만 (nav.js, eval.js, screenshot.js)
3. **사람의 개입 지점 명시**: tmux 스킬에서 "반드시 사용자에게 모니터링 명령을 알려줄 것" 등
4. **개인 워크플로우 최적화**: 오스트리아 교통 API, Apple Mail 등 본인에 특화된 스킬도 포함 — 범용성보다 실용성
