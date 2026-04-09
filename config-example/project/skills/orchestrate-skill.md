---
name: wiki-orchestrate
description: "AI Wiki webapp 프로젝트의 에이전트 팀을 조율한다. 새로운 위키 문서 생성, 기능 개발, 리팩토링 등 여러 에이전트의 협업이 필요한 복합 작업을 수행할 때 사용. '위키 문서 작성해줘', '기능 개발해줘', '팀으로 작업', '전체 파이프라인 실행' 등의 요청 시 트리거한다."
---

# Wiki Orchestrator 스킬

AI Wiki webapp 에이전트 팀의 작업 흐름을 조율한다.

## 워크플로우 A: 위키 문서 생성 파이프라인

새로운 AI 기술 주제에 대한 위키 문서를 생성할 때 사용한다.

### Phase 1: 리서치
- **담당**: researcher
- **작업**: 주제 조사 및 기술 분석 문서 작성
- **산출물**: `_workspace/01_researcher_{topic}.md`
- **완료 조건**: 핵심 기술, 성능 데이터, 참고 자료가 포함된 분석 문서

### Phase 2: 콘텐츠 생성
- **담당**: developer (researcher 산출물 기반)
- **작업**: MDX 위키 문서 작성 및 `src/content/{category}/` 배치
- **산출물**: `src/content/{category}/{slug}.mdx`
- **완료 조건**: frontmatter 완비, 코드 예제 포함, 이미지 참조 정리

### Phase 3: 검증
- **담당**: qa
- **작업**: MDX frontmatter 유효성, 내부 링크, 렌더링 확인
- **산출물**: `_workspace/04_qa_{topic}_report.md`

## 워크플로우 B: 기능 개발 파이프라인

새로운 기능을 추가하거나 기존 기능을 개선할 때 사용한다.

### Phase 1: 설계
- **담당**: architect
- **작업**: 기능 설계 (컴포넌트 구조, API, DB 스키마)
- **산출물**: `_workspace/02_architect_{feature}.md`

### Phase 2: 구현
- **담당**: developer (architect 산출물 기반)
- **작업**: 코드 구현 + 단위 테스트
- **산출물**: 소스 코드 + 테스트 파일

### Phase 3: 검증 (모듈 단위 점진적)
- **담당**: qa
- **작업**: 경계면 교차 비교, 타입 정합성, 보안 검사
- **산출물**: `_workspace/04_qa_{feature}_report.md`

### Phase 4: 수정 (필요 시)
- **담당**: developer (qa 리포트 기반)
- **작업**: 발견된 이슈 수정
- **완료 조건**: qa의 모든 critical 이슈가 resolved

## 데이터 전달 프로토콜
- **태스크 기반**: TaskCreate로 작업 할당, TaskUpdate로 진행상황 추적
- **파일 기반**: `_workspace/` 폴더에 중간 산출물 저장
- **메시지 기반**: SendMessage로 실시간 소통 (추가 조사 요청, 설계 명확화 등)
- **파일명 컨벤션**: `{phase번호}_{에이전트}_{산출물명}.md`

## 에러 핸들링
- 에이전트 작업 실패 시: 1회 재시도 후 재실패 시 해당 결과 없이 진행 (보고서에 누락 명시)
- 상충 데이터: 삭제하지 않고 출처 병기
- critical 이슈: 다음 Phase 진행 전 반드시 해결

## 팀 크기
이 프로젝트는 4명 팀 (researcher, architect, developer, qa)으로 운영한다.
워크플로우에 따라 필요한 에이전트만 선택적으로 활성화한다.

## 테스트 시나리오

### 정상 흐름
```
사용자: "NPU Systolic Array에 대한 위키 문서를 작성해줘"
→ 워크플로우 A 실행
→ researcher: Systolic Array 기술 분석
→ developer: MDX 문서 생성 (src/content/npu/systolic-array.mdx)
→ qa: frontmatter, 내부 링크, 렌더링 검증
→ 최종 산출물 전달
```

### 에러 흐름
```
사용자: "검색 기능을 추가해줘"
→ 워크플로우 B 실행
→ architect: pgvector + full-text search 하이브리드 설계
→ developer: 구현 중 pgvector 확장 미설치 발견
→ developer → architect: SendMessage로 대안 요청
→ architect: SQLite FTS5 폴백 설계 제시
→ developer: 폴백 구현 + 테스트
→ qa: 검색 정확도 + API 응답 shape 검증
```
