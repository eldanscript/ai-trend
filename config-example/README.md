# Claude Code Harness 설정 가이드

Claude Code에 Harness 방법론을 적용하여 에이전트 팀 기반 개발을 수행하기 위한 **설정 파일 통합 레퍼런스**입니다.

## 목차

1. [Harness란?](#1-harness란)
2. [설정 구조 개요](#2-설정-구조-개요)
3. [Global 설정](#3-global-설정)
4. [Project 설정](#4-project-설정)
5. [Harness 사용 가이드](#5-harness-사용-가이드)
6. [프로젝트별 구성 예제](#6-프로젝트별-구성-예제)
7. [적용 순서](#7-적용-순서)

---

## 1. Harness란?

Harness는 Claude Code의 **메타 스킬(meta-skill)**로, 복잡한 작업을 **전문 에이전트 팀**으로 분해하여 협업시키는 구조화된 프레임워크입니다.

### 핵심 개념

| 개념 | 역할 | 파일 위치 |
|------|------|----------|
| **에이전트(Agent)** | "누가" — 역할, 원칙, 협업 프로토콜 정의 | `.claude/agents/{name}.md` |
| **스킬(Skill)** | "어떻게" — 구체적 작업 방법론 정의 | `.claude/skills/{name}/skill.md` |
| **오케스트레이터** | "언제, 어떤 순서로" — 팀 조율 워크플로우 | `.claude/skills/orchestrate/skill.md` |

### 6가지 아키텍처 패턴

| 패턴 | 구조 | 적합한 경우 |
|------|------|-----------|
| **파이프라인** | A→B→C→D | 순차 의존 작업 |
| **팬아웃/팬인** | 분배→[A,B,C]→통합 | 병렬 독립 작업 |
| **전문가 풀** | 상황→전문가 선택 | 입력에 따라 다른 전문가 필요 |
| **생성-검증** | 생성→검수→(재생성) | 품질 보증 중요 |
| **감독자** | 중앙이 동적 분배 | 작업량이 가변적 |
| **계층적 위임** | 상위→하위 재귀적 | 대규모 복잡 작업 |

### 효과

연구 결과 (revfactory/claude-code-harness):
- 평균 품질 점수: 49.5 → 79.3 (**+60% 향상**)
- 승률: **100%** (15/15 태스크)
- 작업 복잡도가 높을수록 개선 폭 증가 (Expert 난이도: +36.2점)

---

## 2. 설정 구조 개요

```
~/.claude/                              ← Global (모든 프로젝트 공통)
├── settings.json                       ← 환경변수, 모델, 에이전트 팀 활성화
├── CLAUDE.md                           ← 글로벌 작업 원칙
└── skills/harness/                     ← harness 메타 스킬
    ├── SKILL.md                        ← 6-Phase 워크플로우 정의
    └── references/                     ← 패턴, 템플릿, 가이드 6개

프로젝트/                                ← Project (프로젝트별 커스텀)
├── CLAUDE.md                           ← 프로젝트 컨텍스트, 기술 스택
└── .claude/
    ├── settings.local.json             ← 프로젝트별 권한 설정
    ├── agents/                         ← 에이전트 정의 파일들
    │   ├── researcher.md
    │   ├── architect.md
    │   ├── developer.md
    │   └── qa.md
    └── skills/                         ← 스킬 정의 파일들
        ├── research/skill.md
        ├── build/skill.md
        └── orchestrate/skill.md
```

### Global vs Project 역할 분담

| 항목 | Global | Project |
|------|--------|---------|
| 에이전트 팀 활성화 | `settings.json` (env) | - |
| 모델 설정 | `settings.json` (env) | - |
| 언어/코딩 원칙 | `CLAUDE.md` | - |
| harness 메타 스킬 | `skills/harness/` | - |
| 프로젝트 맥락 | - | `CLAUDE.md` |
| 에이전트 정의 | - | `.claude/agents/` |
| 도메인 스킬 | - | `.claude/skills/` |
| 권한 설정 | - | `.claude/settings.local.json` |

---

## 3. Global 설정

### 3-1. settings.json

> 파일: `~/.claude/settings.json`
> 예시: [global/settings.json](global/settings.json)

**필수 설정:**

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

| 환경변수 | 설명 | 필수 |
|---------|------|------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 에이전트 팀 기능 활성화 | **필수** |
| `ANTHROPIC_MODEL` | 기본 모델 지정 | 권장 |
| `CLAUDE_CODE_USE_BEDROCK` | AWS Bedrock 사용 시 | 선택 |

### 3-2. CLAUDE.md (Global)

> 파일: `~/.claude/CLAUDE.md`
> 예시: [global/CLAUDE.md](global/CLAUDE.md)

모든 프로젝트에 적용되는 공통 원칙:
- 언어 설정 (대화 언어, 코드 언어)
- Harness 방법론 핵심 원칙
- 코드 품질 기준
- Git 컨벤션

### 3-3. Harness 메타 스킬

> 파일: `~/.claude/skills/harness/SKILL.md` + `references/`
> 설치: 아래 [적용 순서](#7-적용-순서) 참조

"하네스 구성해줘" 한 마디로 6-Phase 워크플로우를 실행하여 프로젝트에 맞는 에이전트 팀과 스킬을 자동 생성합니다.

---

## 4. Project 설정

### 4-1. CLAUDE.md (Project)

> 파일: `프로젝트/CLAUDE.md`
> 예시: [project/CLAUDE.md](project/CLAUDE.md)

프로젝트별 컨텍스트 정보:
- 프로젝트 개요 및 목적
- 기술 스택 상세
- 디렉토리 구조
- 하네스 팀 구성 및 작업 흐름
- 코딩 컨벤션
- 콘텐츠/도메인 카테고리

### 4-2. 에이전트 정의

> 파일: `프로젝트/.claude/agents/{name}.md`
> 예시: [project/agents/](project/agents/)

각 에이전트 정의 파일의 **필수 섹션:**

```markdown
# {이름} — {역할} 전문 에이전트

## 핵심 역할
{에이전트가 담당하는 핵심 책임}

## 작업 원칙
{품질 기준, 우선순위, 제약 조건}

## 입력/출력 프로토콜
- **입력**: {어디서 무엇을 받는지}
- **출력**: {산출물 경로 및 형식}

## 에러 핸들링
{실패 시 대응 방법}

## 팀 통신 프로토콜
- **수신**: {누구로부터 무엇을}
- **발신**: {누구에게 무엇을}
```

**핵심 규칙:**
- 빌트인 타입(general-purpose, Explore, Plan)을 사용하더라도 반드시 파일 생성
- 모든 Agent 호출 시 `model: "opus"` 명시
- QA 에이전트는 `general-purpose` 타입 사용 (Explore는 읽기 전용)

### 4-3. 스킬 정의

> 파일: `프로젝트/.claude/skills/{name}/skill.md`
> 예시: [project/skills/](project/skills/)

스킬 파일 구조:

```
skill-name/
├── skill.md (필수)
│   ├── YAML frontmatter (name, description 필수)
│   └── Markdown 본문 (<500줄)
└── references/ (선택, 상세 내용 분리)
```

**description 작성 핵심 — 적극적(pushy) 트리거:**

```yaml
# 나쁜 예
description: "리서치를 수행하는 스킬"

# 좋은 예
description: "AI 기술 트렌드 리서치 및 deep dive 분석을 수행한다.
NPU, TPU, LPU, GPU, AI 컴파일러, 추론 최적화, 모델 아키텍처 등
AI 하드웨어/소프트웨어 주제에 대해 기술적으로 깊이 있는 분석 문서를
작성할 때 사용. '리서치해줘', '분석해줘', '조사해줘' 등의 키워드가
포함되면 이 스킬을 트리거한다."
```

### 4-4. settings.local.json

> 파일: `프로젝트/.claude/settings.local.json`

프로젝트별로 자주 사용하는 도구의 권한을 미리 허용:

```json
{
  "permissions": {
    "allow": [
      "Bash(git:*)",
      "Bash(npm:*)",
      "Bash(npx:*)"
    ]
  }
}
```

---

## 5. Harness 사용 가이드

### 5-1. 새 프로젝트에 하네스 구성하기

프로젝트 디렉토리에서 Claude Code를 실행하고:

```
하네스 구성해줘
```

또는 더 구체적으로:

```
풀스택 웹 개발 하네스를 구성해줘. Next.js + TypeScript + PostgreSQL 스택으로,
디자인, 프론트엔드, 백엔드, QA를 파이프라인으로 조율하는 팀.
```

Harness 메타 스킬이 자동 트리거되어 6-Phase를 실행합니다:

| Phase | 수행 내용 |
|-------|----------|
| 1. 도메인 분석 | 프로젝트 특성, 기술 스택, 작업 유형 파악 |
| 2. 팀 아키텍처 설계 | 패턴 선택, 실행 모드 결정, 에이전트 분리 |
| 3. 에이전트 정의 생성 | `.claude/agents/` 파일 자동 생성 |
| 4. 스킬 생성 | `.claude/skills/` 파일 자동 생성 |
| 5. 오케스트레이션 | 데이터 전달, 에러 핸들링 프로토콜 설정 |
| 6. 검증 | 구조 검증, 트리거 테스트, 드라이런 |

### 5-2. 에이전트 팀으로 작업 수행하기

하네스가 구성된 후, 오케스트레이터 스킬에 정의된 워크플로우를 트리거합니다:

```
# 위키 문서 생성 (워크플로우 A)
NPU Systolic Array에 대한 위키 문서를 작성해줘

# 기능 개발 (워크플로우 B)
검색 기능을 추가해줘

# 개별 스킬 트리거
Groq LPU의 최신 벤치마크를 조사해줘    → research 스킬
메인 페이지 레이아웃을 구현해줘          → build 스킬
```

### 5-3. 팀 통신 흐름

에이전트 팀 모드에서의 통신 패턴:

```
[오케스트레이터/리더]
    ├── TeamCreate(team_name, members)      ← 팀 구성
    ├── TaskCreate(tasks with dependencies) ← 작업 할당
    │
    ├── [researcher] ──SendMessage──→ [architect]   ← 팀원 간 직접 통신
    ├── [architect]  ──SendMessage──→ [developer]
    ├── [developer]  ──SendMessage──→ [qa]
    ├── [qa]         ──SendMessage──→ [developer]   ← 버그 리포트
    │
    ├── 파일 기반 전달: _workspace/{phase}_{agent}_{artifact}.md
    └── 최종 결과 종합 및 보고
```

### 5-4. 중간 산출물 관리

```
_workspace/                          ← git ignore, 에이전트 팀 작업 공간
├── 01_researcher_systolic-array.md  ← Phase 1 리서치 결과
├── 02_architect_search-feature.md   ← Phase 2 설계 문서
├── 03_developer_notes.md            ← Phase 3 구현 메모
└── 04_qa_search-feature_report.md   ← Phase 4 검증 리포트
```

파일명 컨벤션: `{phase번호}_{에이전트}_{산출물명}.{ext}`

### 5-5. Progressive Disclosure (단계적 정보 공개)

컨텍스트 윈도우를 효율적으로 관리하기 위한 3단계 로딩:

| 단계 | 로딩 시점 | 크기 목표 |
|------|----------|----------|
| **Metadata** (name + description) | 항상 존재 | ~100단어 |
| **skill.md 본문** | 스킬 트리거 시 | <500줄 |
| **references/** | 필요할 때만 Read | 무제한 |

### 5-6. 에러 핸들링 원칙

- **1회 재시도 → 스킵**: 재실패 시 해당 결과 없이 진행 (보고서에 누락 명시)
- **상충 데이터**: 삭제하지 않고 출처 병기
- **critical 이슈**: 다음 Phase 진행 전 반드시 해결
- **QA는 점진적**: 전체 완성 후 1회가 아닌, 각 모듈 완성 직후 실행

---

## 6. 프로젝트별 구성 예제

### 예제 1: AI 기술 Wiki (본 프로젝트)

**패턴**: 파이프라인 + 생성-검증 하이브리드

```
에이전트 4명: researcher, architect, developer, qa
스킬 3개:    ai-research, wiki-build, wiki-orchestrate
기술 스택:   Next.js 15, TypeScript, Tailwind, MDX, PostgreSQL, pgvector
```

워크플로우:
```
[researcher] → [architect] → [developer] ⇄ [qa]
   리서치        설계          구현      점진적 검증
```

### 예제 2: SaaS 대시보드

**패턴**: 감독자 (Supervisor)

```
에이전트 5명: pm, designer, frontend-dev, backend-dev, qa
스킬 4개:    design, frontend-build, backend-build, dashboard-orchestrate
기술 스택:   React, Node.js, GraphQL, PostgreSQL
```

```
        [pm/감독자]
       ↙    ↓    ↘
[designer] [frontend] [backend]
       ↘    ↓    ↙
         [qa]
```

### 예제 3: 데이터 파이프라인

**패턴**: 팬아웃/팬인

```
에이전트 4명: schema-designer, etl-builder, validator, monitor-setup
스킬 3개:    schema-design, etl-build, pipeline-orchestrate
기술 스택:   Python, Apache Airflow, dbt, PostgreSQL
```

```
             [분배]
          ↙    ↓    ↘
[schema] [etl-build] [monitor]
          ↘    ↓    ↙
           [validator]
```

### 예제 4: 모바일 앱 (React Native)

**패턴**: 파이프라인

```
에이전트 4명: ux-researcher, designer, rn-developer, qa
스킬 3개:    ux-research, rn-build, mobile-orchestrate
기술 스택:   React Native, Expo, TypeScript, Zustand
```

### 예제 5: 기술 블로그 / 문서 사이트

**패턴**: 생성-검증

```
에이전트 3명: writer, reviewer, publisher
스킬 3개:    content-write, content-review, blog-orchestrate
기술 스택:   Astro, MDX, Tailwind
```

```
[writer] → [reviewer] → 수정필요? → [writer] (반복)
                       → 통과 → [publisher]
```

---

## 7. 적용 순서

### Step 1: Global 설정 (1회만)

```bash
# 1-1. settings.json에 에이전트 팀 활성화
# ~/.claude/settings.json의 env에 추가:
# "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"

# 1-2. 글로벌 CLAUDE.md 생성
cp config-example/global/CLAUDE.md ~/.claude/CLAUDE.md

# 1-3. Harness 메타 스킬 설치 (소스가 있는 경우)
cp -r /path/to/harness/skills/harness ~/.claude/skills/harness

# 또는 직접 다운로드
# git clone https://github.com/revfactory/harness
# cp -r harness/skills/harness ~/.claude/skills/harness
```

### Step 2: Project 설정 (프로젝트마다)

```bash
# 2-1. 프로젝트 디렉토리에서 Claude Code 실행 후:
하네스 구성해줘

# 이 한 마디로 Phase 1~6이 자동 실행되어
# .claude/agents/와 .claude/skills/가 생성됩니다.
```

또는 수동으로 구성:

```bash
# 2-1. CLAUDE.md 생성 (프로젝트 컨텍스트 작성)
cp config-example/project/CLAUDE.md ./CLAUDE.md
# → 프로젝트에 맞게 수정

# 2-2. 에이전트 정의 생성
mkdir -p .claude/agents
cp config-example/project/agents/*.md .claude/agents/
# → 프로젝트에 맞게 수정

# 2-3. 스킬 정의 생성
mkdir -p .claude/skills/{research,build,orchestrate}
cp config-example/project/skills/*.md .claude/skills/
# → 프로젝트에 맞게 수정

# 2-4. _workspace를 .gitignore에 추가
echo "_workspace/" >> .gitignore
```

### Step 3: 검증

```bash
# Claude Code에서 실행:
하네스 검증해줘

# 또는 수동 확인:
# - .claude/agents/ 파일 존재 확인
# - .claude/skills/ frontmatter 유효성 확인
# - 테스트 프롬프트로 트리거 확인
```

---

## 참고 자료

- [revfactory/harness](https://github.com/revfactory/harness) — Harness 플러그인 원본
- [revfactory/harness-100](https://github.com/revfactory/harness-100) — 100개 프로덕션 레디 하네스 예제
- [revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness) — A/B 테스트 연구 결과
