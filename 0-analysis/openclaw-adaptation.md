# OpenClaw / Claude Code 적용 아이디어

agent-stuff에서 발견한 패턴 중 우리 환경에 적용할 수 있는 것들.

## 1. 명령 인터셉트 → Claude Code hooks

Claude Code에는 `intercepted-commands`가 없지만, **hooks** (`PreToolUse`, `PostToolUse`)로 유사한 효과를 낼 수 있다.

**적용 예시**:
- `Bash` hook에서 `pip install` 감지 → 경고 + `uv add` 제안
- `python -m venv` 감지 → `uv venv` 제안
- 이미 `accumulated-knowledge.md`에 "venv 없이 Python 사용 금지" 규칙이 있지만, hook으로 강제하면 더 확실

**구현 방향**: `.claude/hooks/preToolUse.sh`에서 Bash 명령 파싱

## 2. Sentry 스킬 스크립트화

현재 우리는 `sentry-cli`를 직접 사용하지만, agent-stuff처럼 **워크플로우별 스크립트**를 만들면:

- 에이전트가 더 적은 토큰으로 정확한 명령 생성
- SKILL.md에 "이 상황이면 이 스크립트" 워크플로우 명시 가능
- 이미 `debugging-tools.md`에 패턴이 있으므로 스크립트로 승격

**우선 대상**: `fetch-issue`, `search-events`, `list-issues` 3개면 대부분 커버.

## 3. librarian 패턴 → 분석 레포 참조

현재 analysis-nexus는 submodule 방식이지만, **일시적 참조**가 필요할 때 librarian 패턴이 유용:

```bash
# OpenClaw 스킬로 구현
bash checkout.sh mitsuhiko/minijinja --path-only
# → ~/.cache/checkouts/github.com/mitsuhiko/minijinja
```

분석이 아닌 단순 참조(API 구조 확인, 패턴 참고 등)에 적합.

## 4. tmux 스킬의 "사용자 모니터링 명령 강제 출력"

tmux 스킬에서 **세션 시작 후 반드시 attach 명령을 사용자에게 알리는** 패턴:

> After starting a session ALWAYS tell the user how to monitor the session

이 원칙을 우리 teammate/team 작업에도 적용:
- teammate spawn 후 tmux attach 명령 출력 (이미 규칙에 있지만 스킬 레벨에서 강제)
- 장시간 작업 시작 시 모니터링 방법 안내

## 5. frontend-design의 "Anti-pattern 명시" 패턴

"하지 말 것"을 구체적으로 나열하는 방식:

> - Cookie-cutter hero + 3 card layouts
> - Generic gradients and default font choices
> - Unmotivated decorative elements

이 패턴을 다른 스킬에도 적용 가능:
- 코드 리뷰 스킬: "사소한 스타일 지적 금지", "추측성 버그 리포트 금지"
- PR 작성 스킬: "변경된 파일 나열만으로 설명 대체 금지"

## 6. commit 스킬의 인자 해석 규칙

```markdown
- Freeform instructions should influence scope, summary, and body.
- File paths or globs should limit which files to commit.
- If arguments combine files and instructions, honor both.
```

OpenClaw 스킬에서도 인자를 명시적으로 분류하면 에이전트의 해석 정확도가 올라간다.

## 7. go-to-bed — 안전장치 패턴

심야(자정 이후) 작업 시 명시적 확인을 요구하는 확장.

우리 환경에서의 변형:
- **토큰 사용량 임계치 초과 시 경고** (이미 token-usage 스킬이 있음)
- **프로덕션 영향 작업 전 이중 확인** (accumulated-knowledge에 규칙 있지만 자동화 가능)

## 요약: 적용 우선순위

| 우선순위 | 아이디어 | 난이도 | 효과 |
|---------|---------|--------|------|
| 1 | 명령 인터셉트 (hooks) | 낮음 | 높음 — 에이전트 실수 원천 차단 |
| 2 | Sentry 스크립트화 | 중간 | 중간 — 디버깅 효율 향상 |
| 3 | Anti-pattern 명시 패턴 | 낮음 | 중간 — 스킬 품질 향상 |
| 4 | librarian 패턴 | 중간 | 낮음 — 현재 submodule로 충분 |
| 5 | 인자 해석 규칙 명시 | 낮음 | 중간 — 스킬 정확도 향상 |
