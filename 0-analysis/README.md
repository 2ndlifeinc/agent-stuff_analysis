# agent-stuff 분석

## 분석 목적

Armin Ronacher(mitsuhiko, Flask/Sentry 창시자)가 **Pi Coding Agent**를 커스터마이징하여 사용하는 방식을 분석한다.
AI 코딩 에이전트를 실전에서 어떻게 확장하고, 어떤 패턴으로 스킬/확장을 설계하는지 참고하여 우리의 OpenClaw 스킬 및 Claude Code 확장에 적용한다.

### 핵심 관심사

1. **스킬 설계 패턴** — SKILL.md 구조, 스크립트 구성, 에이전트에게 주는 지시 방식
2. **명령 인터셉트 패턴** — pip/python 등을 가로채서 uv로 강제하는 방식 (우리도 유사 니즈 있음)
3. **Pi Extension 아키텍처** — TUI 기반 확장 시스템 (review, session-breakdown 등)
4. **도구 통합 패턴** — Sentry, GitHub, tmux, CDP 브라우저 등 외부 도구를 에이전트에 연결하는 방식
5. **워크플로우 자동화** — commit, review, changelog 등 반복 작업의 자동화 설계

## 문서 목록

| 문서 | 내용 |
|------|------|
| [architecture.md](architecture.md) | 전체 구조, 모듈별 역할, 설계 패턴 |
| [skills-deep-dive.md](skills-deep-dive.md) | 주요 스킬 상세 분석 및 OpenClaw 적용 아이디어 |
| [openclaw-adaptation.md](openclaw-adaptation.md) | OpenClaw/Claude Code에 적용 가능한 패턴 정리 |

## 분석 대상

- **레포**: [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff)
- **버전**: v1.4.0 (npm: `mitsupi`)
- **플랫폼**: [Pi Coding Agent](https://buildwithpi.ai/) (mariozechner 제작)
- **라이선스**: 명시 없음 (LICENSE 파일 존재)
