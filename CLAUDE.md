# AI Trend Wiki — Project Guidelines

## 프로젝트 개요
AI 도메인의 최신 기술 트렌드를 심층 분석(deep dive)하여 전달하는 wiki 형태의 웹 애플리케이션.
NPU, TPU, LPU, GPGPU 등 AI 하드웨어 가속기부터 AI 모델, 추론 최적화, 컴파일러 스택까지 폭넓게 다룬다.

## 기술 스택
- **Frontend**: Next.js 15 (App Router), TypeScript, Tailwind CSS, MDX
- **Backend**: Next.js API Routes, Prisma ORM
- **Database**: PostgreSQL (Supabase)
- **Search**: Full-text search + vector embedding (pgvector)
- **Deploy**: Vercel
- **Content**: MDX 기반 위키 문서, 자동 목차 생성, 코드 하이라이팅

## 디렉토리 구조
```
ai-trend/
├── src/
│   ├── app/              # Next.js App Router pages
│   ├── components/       # React components
│   ├── lib/              # Utility functions
│   ├── content/          # MDX wiki articles
│   └── types/            # TypeScript type definitions
├── prisma/               # Database schema
├── public/               # Static assets
└── _workspace/           # 에이전트 팀 중간 산출물 (git ignore)
```

## Harness 팀 구성
이 프로젝트는 **파이프라인 + 생성-검증** 하이브리드 패턴을 사용한다.

### 에이전트 역할
- **researcher**: AI 도메인 최신 기술 리서치 및 분석
- **architect**: 시스템 설계 및 기술 아키텍처 결정
- **developer**: 프론트엔드/백엔드 구현
- **qa**: 코드 품질 검증, 통합 테스트, 경계면 검사

### 작업 흐름
1. researcher가 주제를 조사하고 기술 분석 문서 작성
2. architect가 구현 설계 (컴포넌트 구조, API 설계, DB 스키마)
3. developer가 코드 구현
4. qa가 각 모듈 완성 직후 점진적 검증 (incremental QA)

## 코딩 컨벤션
- TypeScript strict mode 필수
- 컴포넌트는 함수형 + Server Components 우선
- API는 RESTful, 응답은 일관된 JSON 구조
- MDX 콘텐츠에는 frontmatter (title, date, category, tags, summary) 필수
- 테스트: Vitest (unit) + Playwright (e2e)

## 콘텐츠 카테고리
- `npu/` — Neural Processing Unit (Rebellions, FuriosaAI, etc.)
- `tpu/` — Tensor Processing Unit (Google TPU)
- `lpu/` — Language Processing Unit (Groq)
- `gpgpu/` — General Purpose GPU (NVIDIA, AMD)
- `compiler/` — AI Compiler (MLIR, TVM, ONNX)
- `inference/` — Inference Optimization (vLLM, quantization)
- `model/` — AI Model Architecture (Transformer, MoE, etc.)
