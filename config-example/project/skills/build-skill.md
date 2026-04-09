---
name: wiki-build
description: "AI Wiki webapp의 기능을 구현한다. Next.js 페이지, React 컴포넌트, API 라우트, DB 스키마, MDX 렌더링, 검색 기능 등 웹앱의 모든 코딩 작업을 수행할 때 사용. '구현해줘', '만들어줘', '페이지 추가', '컴포넌트 생성', 'API 만들어', '검색 기능', '스타일링' 등의 키워드가 포함되면 이 스킬을 트리거한다."
---

# Wiki Build 스킬

AI Wiki webapp의 기능을 설계하고 구현한다.

## 기술 스택 규칙

### Next.js 15
- App Router 사용, pages/ 디렉토리 사용 금지
- Server Components 기본, "use client"는 인터랙션이 필요한 경우만
- `loading.tsx`, `error.tsx`로 로딩/에러 상태 처리
- Dynamic routes: `[slug]`, catch-all: `[...slug]`

### TypeScript
- strict: true, noUncheckedIndexedAccess: true
- API 응답 타입은 `src/types/`에 중앙 관리
- Zod로 런타임 유효성 검사 (API boundary)

### MDX 콘텐츠
- frontmatter 필수 필드: title, date, category, tags, summary
- 코드 블록: rehype-pretty-code로 하이라이팅
- 자동 목차(ToC): rehype-slug + rehype-autolink-headings
- 수식: rehype-katex (필요 시)

### 스타일링
- Tailwind CSS 유틸리티 클래스 우선
- 다크 모드: `dark:` 프리픽스로 지원
- 반응형: mobile-first (sm → md → lg)

### DB / ORM
- Prisma로 스키마 정의, PostgreSQL 대상
- 마이그레이션: `npx prisma migrate dev`
- pgvector: 벡터 검색 인덱스

## 구현 체크리스트
- [ ] TypeScript 컴파일 에러 없음 (`npx tsc --noEmit`)
- [ ] ESLint 경고/에러 없음 (`npx eslint .`)
- [ ] 단위 테스트 작성 및 통과 (`npx vitest`)
- [ ] 접근성 기본 요건 (semantic HTML, alt text)
- [ ] 에러 바운더리 처리
