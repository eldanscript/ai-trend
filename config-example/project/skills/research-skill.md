---
name: ai-research
description: "AI 기술 트렌드 리서치 및 deep dive 분석을 수행한다. NPU, TPU, LPU, GPU, AI 컴파일러, 추론 최적화, 모델 아키텍처 등 AI 하드웨어/소프트웨어 주제에 대해 기술적으로 깊이 있는 분석 문서를 작성할 때 사용. '리서치해줘', '분석해줘', '조사해줘', 'deep dive', '기술 트렌드', 'AI 하드웨어' 등의 키워드가 포함되면 이 스킬을 트리거한다."
---

# AI 기술 리서치 스킬

AI 도메인의 기술 주제를 다각도로 조사하고 구조화된 분석 문서를 생성한다.

## 워크플로우

### 1단계: 주제 범위 설정
- 사용자 요청에서 핵심 기술 키워드 추출
- 카테고리 분류: npu / tpu / lpu / gpgpu / compiler / inference / model
- 분석 깊이 결정: overview(개요) / deep-dive(심층) / comparison(비교)

### 2단계: 정보 수집
- 웹 검색으로 최신 발표, 벤치마크, 기술 블로그 수집
- 학술 논문 키워드 검색 (arXiv, Google Scholar)
- 공식 문서/스펙시트 확인
- 한국 시장 관련 정보 (Rebellions, FuriosaAI, SK Hynix 등) 별도 수집

### 3단계: 분석 문서 작성

출력 형식 — MDX frontmatter를 포함한 위키 문서:

```mdx
---
title: "{주제 제목}"
date: "{YYYY-MM-DD}"
category: "{카테고리}"
tags: ["{태그1}", "{태그2}"]
summary: "{1-2문장 요약}"
depth: "{overview | deep-dive | comparison}"
---

## 개요
{기술의 배경과 필요성}

## 핵심 기술
{아키텍처, 동작 원리, 핵심 혁신}

## 성능 분석
{벤치마크 데이터, 비교 테이블, Roofline 분석}

## 시장 현황
{주요 플레이어, 시장 점유율, 한국 시장 동향}

## 실무 적용
{사용 사례, 도입 시 고려사항, TCO 분석}

## 참고 자료
{논문, 공식 문서, 블로그 링크}
```

### 4단계: 검증
- 수치 데이터의 출처 확인
- 상충 정보 양쪽 출처 병기
- 최신성 확인 (6개월 이내 데이터 우선)
