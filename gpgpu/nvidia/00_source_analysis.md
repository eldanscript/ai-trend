# 소스 분석: Inside NVIDIA GPUs — Anatomy of High Performance Matmul Kernels

> **원문**: https://www.aleksagordic.com/blog/matmul
> **저자**: Aleksa Gordić
> **발행일**: 2025년 9월 29일
> **분석일**: 2026-04-09

---

## 1. 문서 개요

이 블로그 포스트는 NVIDIA GPU에서의 **행렬 곱셈(matmul) 커널 최적화**를 하드웨어 기초부터 Hopper 아키텍처의 최첨단 비동기 커널까지 단계적으로 다룬다. Naive 커널에서 시작하여 cuBLAS 대비 **107% 성능**을 달성하는 과정을 상세히 기술한다.

## 2. 핵심 주제 분류

### 2-1. GPU 하드웨어 아키텍처
- H100 SXM5/PCIe 사양 (132/114 SM, HBM3)
- 메모리 계층: GMEM → L2 → DSMEM → L1 → SMEM → 레지스터
- Streaming Multiprocessor 내부 구조 (텐서 코어, CUDA 코어, Load/Store 유닛)
- Speed of Light (SoL) 계산법

### 2-2. CUDA 프로그래밍 모델
- 스레드 → 워프(32) → 스레드 블록 → 클러스터 → 그리드
- 워프 그룹 (128 스레드 = 4 워프) — Hopper 신규
- `blockIdx`, `threadIdx` 매핑

### 2-3. 메모리 접근 패턴
- GMEM 코얼레싱 (DRAM row activation 물리학)
- SMEM 뱅크 충돌 (32 뱅크, 4바이트 폭)
- L1 캐시 set-associative 구조

### 2-4. 커널 최적화 진화
- Naive → Warp-Tiling → TMA+Tensor Core → 파이프라인 → Persistent → Cluster
- 32 TFLOP/s → 764 TFLOP/s (24배 향상)

### 2-5. Hopper 전용 기능
- TMA (Tensor Memory Accelerator)
- WGMMA (Warp-Group Matrix Multiply Accumulate)
- 스위즐링 (XOR 기반 뱅크 충돌 제거)
- DSMEM (Distributed Shared Memory)
- 비동기 배리어 (트랜잭션 기반)
- Thread Block Cluster

## 3. 성능 진화 테이블

| 단계 | 최적화 | TFLOP/s | 누적 향상 |
|------|--------|---------|----------|
| 0 | Naive (CUDA 코어) | 3.17 | 기준선 |
| 1 | Warp-Tiling (fp32) | 32 | 10× |
| 2 | Tensor Core + TMA (bf16) | 317 | 100× |
| 3 | 출력 타일 확대 | 423 | 133× |
| 4 | TMA/TC 파이프라인 | 498 | 157× |
| 5 | 128×256 타일 + 듀얼 컨슈머 | 610 | 192× |
| 6 | Persistent 커널 | 660 | 208× |
| 7 | 빠른 PTX 배리어 | 704 | 222× |
| 8 | Cluster + TMA 멀티캐스트 | 734 | 231× |
| 9 | 마이크로 최적화 | 747 | 236× |
| 10 | TMA 비동기 스토어 | 758 | 239× |
| 11 | 힐베르트 곡선 스케줄링 | 764 | 241× |

## 4. 핵심 수치 요약

| 항목 | 값 |
|------|---|
| H100 SXM5 SM 수 | 132 |
| H100 PCIe SM 수 | 114 |
| SM당 최대 스레드 | 2,048 (64 워프) |
| SM당 레지스터 | 65,536 |
| SM당 SMEM | 최대 228 KiB |
| SMEM 뱅크 수 | 32 (각 4바이트) |
| BF16 이론 최대 성능 | ~990 TFLOP/s (SXM5) |
| 최종 달성 성능 | 764 TFLOP/s (PCIe) |
| cuBLAS 대비 | 107% |

## 5. 참조된 핵심 자료
- Simon Boehm의 warp-tiling 분석
- Pranjal의 Hopper worklog (cuBLAS 초과 달성)
- CUTLASS 공식 문서
- NVIDIA PTX ISA Reference
- Aroun의 stmatrix PR
