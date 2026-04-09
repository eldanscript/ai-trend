# Global Claude Code Guidelines

## Language
- 사용자와의 대화는 한국어로 진행한다.
- 코드 주석, 커밋 메시지, PR은 영어로 작성한다.
- 에이전트/스킬 정의 파일은 한국어로 작성한다.

## Harness 방법론 원칙
- 복잡한 작업(3개 이상 전문 영역)은 반드시 에이전트 팀으로 분해하여 수행한다.
- 모든 에이전트는 `.claude/agents/{name}.md` 파일로 정의한다 — prompt에 직접 역할을 넣지 않는다.
- 모든 Agent 호출 시 `model: "opus"` 파라미터를 명시한다.
- 스킬의 description은 적극적(pushy)으로 작성하여 트리거 확률을 높인다.
- skill.md 본문은 500줄 이내, 초과 시 references/로 분리한다.

## 코드 품질
- 테스트 없이 코드를 완성하지 않는다.
- TypeScript 프로젝트는 strict mode를 사용한다.
- 보안 취약점(XSS, SQL injection, OWASP top 10)에 주의한다.

## Git
- 커밋 메시지는 conventional commits 형식을 따른다 (feat:, fix:, docs:, refactor: 등).
- 작업 단위로 커밋하며, 하나의 커밋에 여러 관심사를 섞지 않는다.
