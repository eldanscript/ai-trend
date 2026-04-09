# Harness 방법론을 Claude Code에 적용하는 완전 가이드

## 1. Harness 방법론이란?

Harness(하네스)는 Anthropic이 공개한 AI 에이전트 아키텍처 설계 패턴으로, LLM을 단순한 텍스트 생성기에서 신뢰할 수 있는 코딩 에이전트로 변환하는 "모든 인프라"를 의미합니다.

핵심 개념은 다음과 같습니다:

- **모델은 지능을 제공하고, 하네스는 신뢰성을 제공한다** — 아무리 좋은 모델이라도 하네스 없이는 장기 작업에서 일관된 결과를 내지 못합니다.
- **컨텍스트 윈도우 사이의 간극을 해결한다** — 각 세션이 이전 세션의 기억 없이 시작하는 문제를 구조화된 아티팩트(progress 파일, git 히스토리, feature 리스트)로 해결합니다.
- **생성과 평가를 분리한다** — GAN에서 영감을 받아, 코드를 작성하는 에이전트와 평가하는 에이전트를 분리하여 품질을 높입니다.

### 1.1 Anthropic이 밝힌 핵심 실패 모드와 해결책

Anthropic의 연구에서 발견한 에이전트의 주요 실패 패턴:

| 실패 모드 | 원인 | Harness 해결책 |
|-----------|------|----------------|
| One-shotting | 에이전트가 전체 앱을 한 번에 구현 시도 | Feature 리스트로 분해 → 한 번에 하나씩 |
| 조기 완료 선언 | 진행 상황을 보고 "완료"라 판단 | Feature별 pass/fail 체크리스트 |
| 컨텍스트 불안 | 컨텍스트 한계 접근 시 작업 급히 마무리 | 컨텍스트 리셋 + 구조화된 핸드오프 |
| 자기 과대평가 | 자신의 작업을 항상 높이 평가 | 생성-평가 에이전트 분리 |
| 테스트 부실 | 코드 변경 후 E2E 검증 생략 | 브라우저 자동화 기반 강제 테스트 |

### 1.2 Harness의 3-Agent 아키텍처

Anthropic의 최신 연구(2026.03)에서 확립된 패턴:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Planner    │────▶│  Generator   │────▶│  Evaluator   │
│              │     │              │     │              │
│ 1줄 프롬프트 │     │ 스프린트 단위 │     │ E2E 테스트   │
│ → 전체 스펙  │     │ 코드 구현     │     │ 품질 평가    │
│ → 디자인 랭귀지│    │ → Git 커밋   │     │ → 버그 리포트│
└──────────────┘     └──────────────┘     └──────────────┘
        │                    ▲                    │
        │                    │    피드백 루프      │
        │                    └────────────────────┘
```

## 2. Claude Code에서의 Harness 적용 구조

Claude Code는 하네스 패턴을 네이티브로 지원하는 설정 체계를 갖고 있습니다:

### 2.1 설정 파일 계층 구조

```
우선순위 (높은 순서):
┌─────────────────────────────────────────────────┐
│ Managed (조직 정책) — 최우선, 변경 불가           │
├─────────────────────────────────────────────────┤
│ CLI Arguments — 세션 레벨 임시 설정               │
├─────────────────────────────────────────────────┤
│ Local (.claude/settings.local.json) — 개인 프로젝트│
├─────────────────────────────────────────────────┤
│ Project (.claude/settings.json) — 팀 공유, Git 커밋│
├─────────────────────────────────────────────────┤
│ User (~/.claude/settings.json) — 개인 전역 설정    │
└─────────────────────────────────────────────────┘
```

### 2.2 CLAUDE.md vs settings.json 역할 분리

- **CLAUDE.md** = Advisory(조언적). Claude가 약 80% 따릅니다. 코딩 스타일, 아키텍처 결정, 워크플로우 가이드에 적합합니다.
- **settings.json** = Deterministic(결정적). 100% 강제됩니다. 퍼미션, 훅, 환경변수, 모델 설정에 사용합니다.
- **Hooks** = 코드 포매팅, 보안 스캔 등 "반드시 매번 실행되어야 하는 것"에 사용합니다.

원칙: 행동 강제는 settings.json/hooks로, 코딩 지침은 CLAUDE.md로 분리합니다.

## 3. 제공 파일 구성과 적용 방법

### 3.1 Global 설정 (모든 프로젝트에 적용)

다음 파일들을 홈 디렉토리에 복사합니다:

```bash
# CLAUDE.md → 전역 코딩 지침
cp global-settings/CLAUDE.md ~/.claude/CLAUDE.md

# settings.json → 전역 퍼미션, 환경변수, Bedrock 연결
cp global-settings/settings.json ~/.claude/settings.json
```

**`~/.claude/CLAUDE.md`** 주요 내용:
- 개발 원칙: 점진적 변경, 클린 상태 유지, 테스트 우선
- 코드 품질: TypeScript strict, 에러 핸들링 필수
- 컨텍스트 관리: 컴팩션 시 보존할 정보 명시
- 커뮤니케이션: 직접적이고 기술적인 응답 스타일
- 클라우드: 서버리스 우선, CDK/CloudFormation 사용

**`~/.claude/settings.json`** 주요 내용:
- Bedrock 연결 설정 (CLAUDE_CODE_USE_BEDROCK, 모델 지정)
- 자동 컴팩션 임계값 70%
- 기본 퍼미션 allow/deny 규칙
- Git attribution 설정

### 3.2 Project 설정 (AI Deep Dive Wiki 프로젝트)

프로젝트 루트에 다음 구조를 생성합니다:

```bash
# 프로젝트 디렉토리로 이동
cd ~/projects/ai-deep-dive-wiki

# 설정 파일 복사
cp -r project-settings/CLAUDE.md ./
cp -r project-settings/.claude ./
cp -r project-settings/.mcp.json ./
cp -r project-settings/features.json ./
cp -r project-settings/PROGRESS.md ./
cp -r project-settings/init.sh ./
chmod +x init.sh
```

**프로젝트 파일 구성:**

| 파일 | 역할 | Git 커밋 여부 |
|------|------|:------------:|
| `CLAUDE.md` | 프로젝트 아키텍처, 기술 스택, 코딩 컨벤션 | ✅ |
| `.claude/settings.json` | 프로젝트 퍼미션, 훅 (포매터, 세션 종료 알림) | ✅ |
| `.claude/settings.local.json` | 개인 API 키, 로컬 설정 | ❌ |
| `.claude/agents/planner.md` | Planner 에이전트 정의 | ✅ |
| `.claude/agents/qa-evaluator.md` | QA 평가 에이전트 정의 | ✅ |
| `.claude/commands/harness-plan.md` | `/harness-plan` 명령어 | ✅ |
| `.claude/commands/harness-work.md` | `/harness-work` 명령어 | ✅ |
| `.claude/commands/harness-review.md` | `/harness-review` 명령어 | ✅ |
| `.mcp.json` | MCP 서버 설정 (Playwright, Context7) | ✅ |
| `features.json` | 구현할 기능 목록 (pass/fail 추적) | ✅ |
| `PROGRESS.md` | 세션별 진행 기록 | ✅ |
| `init.sh` | 세션 시작 스크립트 | ✅ |

## 4. 실전 개발 워크플로우

### Phase 0: 프로젝트 초기화

```bash
# 1. 프로젝트 디렉토리 생성 및 Git 초기화
mkdir ai-deep-dive-wiki && cd ai-deep-dive-wiki
git init

# 2. 설정 파일 배치 (위 3.2 참조)

# 3. Claude Code 시작
claude

# 4. Planner 에이전트로 스펙 생성
> Use the planner agent to expand this into a full spec:
> "Build a technical wiki webapp for AI/ML deep-dive articles
>  with search, categorization, and AI-powered features"
```

### Phase 1: Plan → Work → Review 루프

```
세션 N에서의 워크플로우:

1. /harness-plan
   └─ PROGRESS.md + git log 읽기
   └─ features.json에서 다음 작업 선택
   └─ 스프린트 계획 제시 → 사용자 승인 대기

2. /harness-work  (승인 후)
   └─ init.sh 실행 (서버 시작, 스모크 테스트)
   └─ 기능 구현 (DB → API → Component → Page → Test)
   └─ 셀프 검증 (lint, test, 브라우저)
   └─ features.json 업데이트
   └─ Git 커밋 + PROGRESS.md 업데이트

3. /harness-review  (주기적으로)
   └─ QA 에이전트가 모든 "passes: true" 기능 테스트
   └─ 실패한 기능은 다시 "passes: false"로 전환
   └─ QA-REPORT.md 생성
   └─ 다음 세션에서 수정 작업 우선 수행
```

### Phase 2: 세션 간 연속성 유지

세션이 끝날 때마다 다음이 자동으로 유지됩니다:
- `PROGRESS.md` — 무엇을 했고, 다음에 무엇을 해야 하는지
- `features.json` — 각 기능의 pass/fail 상태
- Git 히스토리 — 코드 변경 추적, 롤백 가능
- `QA-REPORT.md` — 마지막 QA 결과

새 세션 시작 시 Claude는 이 파일들을 읽고 이전 작업을 이어갑니다.

## 5. 고급 패턴

### 5.1 Subagent 활용

```bash
# 단일 세션 내에서 전문화된 서브에이전트 호출
> Use the qa-evaluator agent to test the search feature

# 병렬 리뷰
> Use 3 subagents to review: security, performance, accessibility
```

### 5.2 Agent Teams (실험적 기능)

```bash
# 여러 Claude 인스턴스가 Git worktree로 병렬 작업
> Create an agent team:
> - Frontend: implement search UI components
> - Backend: implement search API with FTS5
> - Test: write Playwright tests for search
```

### 5.3 Hooks로 품질 강제

settings.json의 hooks 설정으로 다음을 자동화합니다:
- **PostToolUse**: 파일 편집 후 Prettier 자동 실행
- **Stop**: 세션 종료 시 PROGRESS.md 업데이트 알림
- **PreToolUse**: Git commit 전 lint 검사 (추가 가능)

### 5.4 Bedrock 환경에서의 비용 최적화

- Generator/Work 에이전트: Sonnet 4.6 사용 (비용 효율)
- Planner/Evaluator: Opus 모델 사용 (높은 판단력 필요)
- Subagent: Haiku 사용 (빠른 리서치/검증)
- 자동 컴팩션 70%로 설정하여 토큰 효율 유지

## 6. Bedrock 사용 시 추가 설정

기존 `awsup` 워크플로우와 연동:

```bash
# 1. Bedrock 모델 액세스 활성화 (AWS Console)
# Amazon Bedrock → Model access → Claude 모델 요청

# 2. IAM 권한 확인
# bedrock:InvokeModel, bedrock:InvokeModelWithResponseStream 필요

# 3. AWS 자격증명 설정 후 Claude Code 시작
awsup  # 커스텀 credential 스크립트
claude  # Bedrock 모드로 자동 연결 (settings.json의 env 참조)
```

## 7. 핵심 원칙 요약

1. **한 번에 하나씩**: 전체 앱 one-shot 금지. Feature 단위로 점진적 구현.
2. **클린 상태 유지**: 매 세션 종료 시 커밋 가능한 상태로 남기기.
3. **생성과 평가 분리**: 같은 에이전트가 자기 코드를 평가하지 않기.
4. **구조화된 핸드오프**: PROGRESS.md + features.json + Git으로 세션 간 연속성.
5. **행동 강제는 코드로**: CLAUDE.md는 조언, hooks/settings는 강제.
6. **모델 개선에 맞춰 단순화**: 모델이 좋아지면 불필요한 scaffold 제거하고 더 복잡한 작업에 투자.
