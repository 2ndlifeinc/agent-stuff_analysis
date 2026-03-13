# 주요 스킬 상세 분석

## 1. commit — 커밋 자동화

**구조**: SKILL.md만 (스크립트 없음)

순수 프롬프트 기반 스킬. 에이전트에게 Conventional Commits 규칙을 주입하고, 단계별 절차를 명시한다.

**주목할 점**:
- `git log -n 50 --pretty=format:%s`로 기존 스코프 패턴 학습
- 인자 해석 규칙이 상세함: 파일 경로, 글롭, 자유 텍스트를 구분
- "push하지 말 것", "애매하면 사용자에게 물어볼 것" 같은 안전장치

**vs 우리 commit 스킬**: OpenClaw의 commit 스킬도 유사하지만, 인자 해석 규칙이 덜 명시적. 파일 경로 + 지시사항 조합 처리가 참고할 만함.

## 2. sentry — 에러 트래킹 연동

**구조**: SKILL.md + 5개 Node.js 스크립트

```
scripts/
├── fetch-issue.js       ← 이슈 상세 (스택트레이스 포함)
├── fetch-event.js       ← 이벤트 상세 (breadcrumbs, spans)
├── search-events.js     ← Discover 기반 이벤트 검색
├── list-issues.js       ← 이슈 목록/필터
└── search-logs.js       ← 로그 검색
```

**주목할 점**:
- SKILL.md에 **디버깅 워크플로우**를 명시 ("What went wrong at this time?" → 이 명령 → 저 명령)
- 에이전트가 단순 API 래퍼가 아니라, 디버깅 **사고 과정**을 따라가도록 설계
- Sentry URL 파싱 지원: 사용자가 URL을 던져도 처리 가능

**vs 우리**: `sentry-cli`를 직접 사용 중. 스크립트화하면 에이전트가 더 효과적으로 사용할 수 있을 것.

## 3. web-browser — CDP 브라우저 제어

**구조**: SKILL.md + 8개 스크립트

```
scripts/
├── start.js             ← Chrome 실행 (프로필 복사 옵션)
├── nav.js               ← 페이지 이동
├── eval.js              ← JS 실행
├── screenshot.js        ← 스크린샷
├── pick.js              ← 인터랙티브 요소 선택
├── dismiss-cookies.js   ← 쿠키 동의 자동 처리
├── watch.js             ← 콘솔/네트워크 로그 기록
├── logs-tail.js         ← 로그 조회
└── net-summary.js       ← 네트워크 응답 요약
```

**주목할 점**:
- MCP 서버 대신 **원자적 CLI 스크립트**로 구현. 에이전트가 bash로 조합하여 사용.
- `--profile` 옵션: 사용자의 쿠키/로그인을 복사하여 인증된 상태로 브라우징
- 백그라운드 로깅이 자동 시작 → 에이전트가 나중에 "아까 무슨 요청 갔지?" 확인 가능
- `dismiss-cookies.js`: EU 쿠키 동의를 자동 처리 — 웹 자동화의 현실적 문제 해결

**vs Playwright MCP**: Playwright MCP는 MCP 프로토콜 기반. 여기선 더 단순하게 CDP 직접 연결 + bash 스크립트. 복잡도 트레이드오프가 다름.

## 4. tmux — 대화형 CLI 제어

**구조**: SKILL.md + 2개 스크립트

에이전트가 python REPL, gdb, psql 등 대화형 프로그램을 tmux를 통해 원격 제어.

**주목할 점**:
- **소켓 격리**: 에이전트 전용 tmux 소켓 → 사용자의 tmux 세션과 분리
- **동기화 패턴**: `wait-for-text.sh`로 프롬프트 대기 후 명령 전송 (레이스 컨디션 방지)
- **사용자 모니터링 강제**: "세션 시작 후 반드시 attach 명령을 사용자에게 알릴 것"
- `PYTHON_BASIC_REPL=1` 강제: 비기본 콘솔이 send-keys와 간섭하는 문제 해결

**실전 레시피가 풍부**: Python REPL, gdb, psql 등 도구별 시작/대기/종료 패턴이 명시되어 있음.

## 5. librarian — 원격 레포 캐시

**구조**: SKILL.md + checkout.sh

`~/.cache/checkouts/<host>/<org>/<repo>`에 원격 레포를 캐싱. partial clone + 300초 throttled refresh.

**주목할 점**:
- 에이전트가 참조용 레포를 반복 clone하는 낭비를 방지
- `--filter=blob:none`으로 히스토리 없이 최신 코드만
- "편집이 필요하면 worktree를 만들어라" — 캐시와 작업 공간 분리

**vs 우리**: analysis-nexus submodule 방식과 다른 접근. librarian은 일시적 참조용, 우리는 영구 분석용.

## 6. frontend-design — 디자인 가이드

**구조**: SKILL.md만 (스크립트 없음)

프롬프트만으로 에이전트의 디자인 품질을 끌어올리는 스킬.

**주목할 점**:
- **"Generic AI aesthetics 금지"** — 기본 폰트, 뻔한 그라데이션, 3카드 레이아웃 명시적 차단
- **디자인 시스템 체크리스트**: 방향 → 차별점 → 타이포 → 컬러 → 레이아웃 → 모션
- 코드 전에 디자인 결정을 먼저 내리도록 강제
- **셀프 검증 체크리스트** 포함

## 7. intercepted-commands — 명령 인터셉트

에이전트가 실행하는 명령을 가로채는 독특한 패턴.

```
python3 -m pip install X  →  에러 + "uv add X 쓰세요"
pip install X              →  에러 + "uv 쓰세요"
python3 script.py          →  uv run python script.py로 리디렉트
```

**설계 원리**:
1. **차단만 하면 에이전트가 막힘** → 에러 메시지에 올바른 대안 포함
2. **python은 완전 차단 아닌 리디렉트** → `exec uv run python "$@"`
3. **py_compile도 차단** → `__pycache__` 파일 생성 방지, AST 체크 대안 제시

이 패턴은 에이전트의 "나쁜 습관"을 교정하는 가장 확실한 방법. 프롬프트로 "uv 써라"고 100번 말하는 것보다 명령 자체를 가로채는 게 확실하다.
