# Deep Dive 분석: 고성능 Matmul 커널 최적화

> 원문의 핵심 주제들에 대한 심층 분석.
> **소스**: https://www.aleksagordic.com/blog/matmul

---

## 목차

1. [Naive에서 SOTA까지: 커널 진화 단계별 분석](#1-naive에서-sota까지-커널-진화-단계별-분석)
2. [TMA (Tensor Memory Accelerator) 심층 분석](#2-tma-tensor-memory-accelerator-심층-분석)
3. [WGMMA 명령어와 텐서 코어 프로그래밍](#3-wgmma-명령어와-텐서-코어-프로그래밍)
4. [스위즐링 메커니즘 완전 분석](#4-스위즐링-메커니즘-완전-분석)
5. [프로듀서-컨슈머 파이프라인 설계](#5-프로듀서-컨슈머-파이프라인-설계)
6. [Persistent 커널과 타일 스케줄링](#6-persistent-커널과-타일-스케줄링)
7. [레지스터 압력과 setmaxnreg](#7-레지스터-압력과-setmaxnreg)
8. [Thread Block Cluster와 DSMEM](#8-thread-block-cluster와-dsmem)
9. [Wave/Tile Quantization 문제](#9-wavetile-quantization-문제)
10. [성능 프로파일링과 병목 분석](#10-성능-프로파일링과-병목-분석)

---

## 1. Naive에서 SOTA까지: 커널 진화 단계별 분석

### 1-1. Kernel 1: Naive Matmul

**구조:**
```cuda
__global__ void matmul_naive(float* A, float* B, float* C, int M, int N, int K) {
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;
    
    if (row < M && col < N) {
        float sum = 0.0f;
        for (int k = 0; k < K; k++) {
            sum += A[row * K + k] * B[k * N + col];
        }
        C[row * N + col] = sum;
    }
}
```

**분석:**
```
스레드당 작업: K번의 FMA (Fused Multiply-Add)
스레드 수: M × N (출력 원소당 1개)
블록 크기: 32×32 = 1024 스레드

메모리 접근 패턴:
  A[row * K + k]: 같은 워프 내 스레드들이 같은 행의 다른 열 접근
    → 워프 내 스레드 간 row 값 상이 (threadIdx.y 기반)
    → 비코얼레싱 위험!
    
  B[k * N + col]: 같은 워프 내 스레드들이 같은 행의 연속 열 접근
    → 워프 내 스레드 간 col 값 연속 (threadIdx.x 기반)
    → 코얼레싱됨 ✓

산술 강도:
  스레드당: 2K FLOPs / (2K × 4 bytes) = 0.25 FLOPs/byte
  → 극도로 메모리 바운드
```

**성능:** ~3,171 GFLOP/s (H100 PCIe)

**코얼레싱 vs 비코얼레싱 실험:**
```
코얼레싱된 접근 (col 연속): 3,171 GFLOP/s
비코얼레싱 (row/col 교환):    243 GFLOP/s
차이: 13× → DRAM row activation 비용 실증
```

### 1-2. Kernel 10: Warp-Tiling (Simon Boehm 방식)

**핵심 아이디어:** 스레드당 1개 출력 → 스레드당 여러 개 출력

```
타일 파라미터:
  BM = BN = 128  (출력 타일)
  BK = 16        (K 방향 청크)
  WM = 64, WN = 64  (워프 타일)
  TM = 8, TN = 4    (스레드 타일)
```

**동작 과정:**
```
1. 블록이 128×128 출력 타일을 담당
2. K를 BK=16 단위로 반복:
   a. 협력적으로 A의 128×16, B의 16×128을 SMEM에 로드
      - LDG.128로 글로벌 → 레지스터
      - STS.128로 레지스터 → SMEM
      - __syncthreads()
   b. 각 스레드가 자신의 TM×TN=8×4=32개 출력에 대해
      외적(outer product) 누적:
      - LDS.128로 SMEM → 레지스터
      - 8×4 FMA 연산 수행
   c. __syncthreads()
3. 최종 결과를 GMEM에 기록
```

**산술 강도 개선:**
```
SMEM 로드: (128×16 + 16×128) × 4 = 16,384 bytes
계산: 2 × 128 × 128 × 16 = 524,288 FLOPs
AI = 524,288 / 16,384 = 32 FLOPs/byte → naive의 128배!
```

**성능:** ~32 TFLOP/s (fp32, CUDA 코어만 사용)

### 1-3. Hopper Kernel: TMA + Tensor Core

**도약의 핵심:**
1. **BF16**: 2배 처리량 + 메모리 절약
2. **Tensor Core**: CUDA 코어 대비 수십 배 처리량
3. **TMA**: 비동기 데이터 이동, 자동 스위즐링

```
수동 로딩 코드 (Warp-Tiling):
  ~50줄의 인덱스 계산 + LDG/STS 코드
  
TMA 로딩 코드:
  cp_async_bulk_tensor_2d_global_to_shared(dst, tensorMap, coord0, coord1, bar);
  → 1줄. TMA 엔진이 나머지 처리.
```

**성능 도약:** 32 → 317 TFLOP/s (**10배**, bf16 + TC + TMA)

### 1-4. 파이프라인 → Persistent → Cluster

```
성능 진화 요약:

317 TFLOP/s  ← TC + TMA 기본
  ↓ +33% 타일 확대
423 TFLOP/s  
  ↓ +18% TMA/TC 오버랩
498 TFLOP/s  
  ↓ +22% 듀얼 컨슈머
610 TFLOP/s  
  ↓ +8% Persistent
660 TFLOP/s  
  ↓ +7% 빠른 배리어
704 TFLOP/s  
  ↓ +4% 클러스터
734 TFLOP/s  
  ↓ +4% 마이크로 최적화 + 비동기 스토어 + 힐베르트
764 TFLOP/s  ← cuBLAS의 107%
```

---

## 2. TMA (Tensor Memory Accelerator) 심층 분석

### 2-1. TMA란?

Hopper에 도입된 **전용 하드웨어 유닛**으로, CPU의 DMA 엔진과 유사하게 데이터 이동을 SM과 독립적으로 수행한다.

```
기존 방식 (Ampere 이전):
  SM 스레드 → LDG 명령어 → GMEM 읽기 → 레지스터 → STS → SMEM
  → SM의 연산 사이클을 데이터 이동에 소비

TMA 방식 (Hopper):
  스레드 1개가 TMA에 요청 → TMA 엔진이 독립적으로 GMEM→SMEM 전송
  → SM은 즉시 다른 연산 수행 가능 (비동기)
```

### 2-2. 텐서 맵 디스크립터

TMA는 **텐서 맵(Tensor Map)**이라는 메타데이터 구조로 전송을 설정한다:

```c
CUtensorMap tensorMap;

cuTensorMapEncodeTiled(
    &tensorMap,           // 출력: 텐서 맵 디스크립터
    CU_TENSOR_MAP_DATA_TYPE_BFLOAT16, // 데이터 타입
    2,                    // 랭크 (2D 텐서)
    d_A,                  // 글로벌 메모리 포인터
    globalDim,            // 텐서 전체 크기 {K, M}
    globalStride,         // 행 스트라이드 {K * sizeof(bf16)}
    boxDim,               // SMEM 타일 크기 {BK, BM}
    elementStride,        // 원소 간 스트라이드 {1, 1}
    CU_TENSOR_MAP_INTERLEAVE_NONE,
    CU_TENSOR_MAP_SWIZZLE_128B, // 스위즐 모드!
    CU_TENSOR_MAP_L2_PROMOTION_L2_128B,
    CU_TENSOR_MAP_FLOAT_OOB_FILL_NONE
);
```

**핵심 파라미터:**
| 파라미터 | 설명 | 중요도 |
|---------|------|--------|
| `dataType` | bf16, fp16, fp32, fp8 등 | 대역폭 효율 결정 |
| `rank` | 텐서 차원 (1D~5D) | 2D가 matmul 표준 |
| `globalDim` | 전체 텐서 크기 | 경계 검사에 사용 |
| `boxDim` | SMEM 타일 크기 | 1회 전송 크기 |
| `swizzle` | 32B/64B/128B/None | 뱅크 충돌 제거 |
| `l2Promotion` | L2 캐시 정책 | 히트율 영향 |

### 2-3. TMA 비동기 로드

```cuda
// 프로듀서 워프 그룹의 스레드 0만 실행:
if (threadIdx.x == 0) {
    // 배리어에 전송 바이트 수 등록
    barrier_arrive_tx(full_barrier[slot], 1, sizeof(smem_tile));
    
    // TMA 로드 명령 발행 (즉시 반환, 비동기 실행)
    cp_async_bulk_tensor_2d_global_to_shared(
        &smem_A[slot][0],      // SMEM 목적지
        tensorMapA,             // 디스크립터
        k_iter * BK,            // GMEM 좌표 (열)
        block_m * BM,           // GMEM 좌표 (행)
        full_barrier[slot]      // 완료 시 이 배리어에 신호
    );
}

// 컨슈머 워프 그룹:
full_barrier[slot].wait(token);  // 전송 완료 대기
// 이제 smem_A[slot]에 데이터가 있음 → 텐서 코어 연산
```

### 2-4. TMA의 추가 기능

| 기능 | 설명 |
|------|------|
| **자동 경계 검사** | 텐서 경계 밖 접근 시 0으로 채움 (OOB fill) |
| **자동 스위즐링** | 로드 시 적용, 스토어 시 역변환 |
| **L2 프로모션** | 로드 데이터를 L2에 캐시하도록 힌트 |
| **멀티캐스트** | 클러스터 내 여러 SM의 SMEM에 동시 전송 |
| **SMEM→GMEM 스토어** | 출력 타일을 비동기로 GMEM에 기록 |

---

## 3. WGMMA 명령어와 텐서 코어 프로그래밍

### 3-1. 텐서 코어 명령어 진화

| 세대 | 명령어 | 단위 | GPU |
|------|--------|------|-----|
| Volta | `wmma` | 워프 (32 스레드) | V100 |
| Ampere | `mma.sync` | 워프 (32 스레드) | A100 |
| Hopper | `wgmma.mma_async` | **워프 그룹 (128 스레드)** | H100 |

### 3-2. WGMMA 상세

**명령어 시그니처:**
```
wgmma.mma_async.sync.aligned.m64n{N}k16.f32.bf16.bf16
                               ^^^  ^   ^^
                               M고정 N가변 K고정
N ∈ {8, 16, 24, 32, 48, 64, 96, 128, 192, 256}
```

**m64n64k16 예시:**
```
입력 A: 64×16 (bf16)  → 레지스터 또는 SMEM
입력 B: 16×64 (bf16)  → SMEM만 가능
출력 D: 64×64 (fp32)  → 레지스터 (분산)

FLOPs = 2 × 64 × 64 × 16 = 131,072
```

### 3-3. 어큐뮬레이터 레지스터 분배

```
128 스레드(워프 그룹)이 64×64 fp32 어큐뮬레이터를 분산 보유

스레드당 레지스터:
  float d[WGMMA_N / 16][8]
  
  WGMMA_N = 64일 때: d[4][8] = 32개 fp32 레지스터

전체 워프 그룹:
  128 스레드 × 32 레지스터 = 4,096 fp32 값
  = 64 × 64 = 4,096 ✓ (정확히 일치)
```

### 3-4. BK=64를 처리하는 4번의 WGMMA

```
BK = 64, WGMMA_K = 16 → 4번 반복 필요

wgmma.fence.sync.aligned;                          // 메모리 펜스

// 4번의 부분합 누적 (k=0,16,32,48)
wgmma64(d, &sA[0*16],  &sB[0*16]);   // A[64×16] × B[16×64]
wgmma64(d, &sA[1*16],  &sB[1*16]);   // A[64×16] × B[16×64]
wgmma64(d, &sA[2*16],  &sB[2*16]);   // A[64×16] × B[16×64]
wgmma64(d, &sA[3*16],  &sB[3*16]);   // A[64×16] × B[16×64]

wgmma.commit_group.sync.aligned;                   // 그룹 커밋
wgmma.wait_group.sync.aligned %0;                  // 완료 대기

// 이 시점에서 d[]는 A[64×64] × B[64×64]의 64×64 부분합을 보유
```

### 3-5. 출력 타일 크기와 WGMMA 조합

```
128×128 출력 타일 (1 컨슈머 워프 그룹):
  → 워프 그룹이 64×64를 4번 = 128×128
  → d[4][8] × 4 위치 = 128 레지스터/스레드

128×256 출력 타일 (2 컨슈머 워프 그룹):
  → 각 워프 그룹이 64×256 담당
  → 한 워프 그룹에서 d[16][8] = 128 레지스터/스레드
  → SM 레지스터 예산에 맞추기 위해 setmaxnreg 필요
```

---

## 4. 스위즐링 메커니즘 완전 분석

### 4-1. 문제: 왜 스위즐링이 필요한가?

```
SMEM에 128×64 bf16 타일을 행 우선(row-major)으로 저장:

Row 0:  [e0,  e1,  e2,  ... e63]   → 128 bytes → Bank 0~31 (2바이트×64 = 128바이트)
Row 1:  [e64, e65, e66, ... e127]  → 128 bytes → Bank 0~31 (동일 패턴!)
Row 2:  [e128,...]                  → 128 bytes → Bank 0~31
...

문제: 열(column) 방향 접근 시
  e0 (Row0, Bank0), e64 (Row1, Bank0), e128 (Row2, Bank0)...
  → 모든 접근이 Bank 0 → 32-way 뱅크 충돌!
```

### 4-2. Swizzle<3,4,3>의 수학적 동작

```
SMEM 주소 비트 구조 (128바이트 모드):
  주소: [...] [G H I] [J K L] [M N O P Q R S T U V W X Y Z]
              bit 9-7  bit 6-4  bit 3-0

  bit 9-7: 행 인덱스의 하위 3비트 (8행 주기)
  bit 6-4: 128바이트 행 내의 뱅크 그룹 (8개 그룹 × 16바이트)
  bit 3-0: 뱅크 그룹 내 오프셋

Swizzle<B=3, M=4, S=3> 적용:
  1. bit_msk = (1 << 3) - 1 = 0b111
  2. yyy_msk = 0b111 << (4+3) = 0b111 << 7
  3. masked  = addr & yyy_msk  → bit 9,8,7 추출
  4. shifted = masked >> 3      → bit 9,8,7을 bit 6,5,4 위치로 이동
  5. result  = addr ^ shifted   → bit 6,5,4를 XOR 변조

실질적 효과:
  swizzled_bank_group = original_bank_group ⊕ row_index[2:0]
```

### 4-3. 스위즐링 검증 예시

```
8행 × 8뱅크그룹 매핑 (뱅크 그룹 = bit 6:4):

스위즐링 전 (row_idx ⊕ 0):
  Row 0: BG0 BG1 BG2 BG3 BG4 BG5 BG6 BG7
  Row 1: BG0 BG1 BG2 BG3 BG4 BG5 BG6 BG7  ← 충돌!
  Row 2: BG0 BG1 BG2 BG3 BG4 BG5 BG6 BG7
  ...

스위즐링 후 (row_idx ⊕ bank_group):
  Row 0 (000⊕): BG0 BG1 BG2 BG3 BG4 BG5 BG6 BG7
  Row 1 (001⊕): BG1 BG0 BG3 BG2 BG5 BG4 BG7 BG6
  Row 2 (010⊕): BG2 BG3 BG0 BG1 BG6 BG7 BG4 BG5
  Row 3 (011⊕): BG3 BG2 BG1 BG0 BG7 BG6 BG5 BG4
  Row 4 (100⊕): BG4 BG5 BG6 BG7 BG0 BG1 BG2 BG3
  Row 5 (101⊕): BG5 BG4 BG7 BG6 BG1 BG0 BG3 BG2
  Row 6 (110⊕): BG6 BG7 BG4 BG5 BG2 BG3 BG0 BG1
  Row 7 (111⊕): BG7 BG6 BG5 BG4 BG3 BG2 BG1 BG0

검증 - 열 0 접근: BG0,BG1,BG2,BG3,BG4,BG5,BG6,BG7 → 모두 다른 뱅크 그룹 ✓
검증 - 행 0 접근: BG0,BG1,BG2,BG3,BG4,BG5,BG6,BG7 → 원래 순서 ✓
```

### 4-4. 3가지 스위즐 모드

| 모드 | Swizzle 파라미터 | 적용 범위 | 적합한 경우 |
|------|-----------------|----------|-----------|
| 128B | `Swizzle<3,4,3>` | 128바이트 단위 | bf16/fp16 대규모 타일 |
| 64B | `Swizzle<2,4,3>` | 64바이트 단위 | 작은 타일 |
| 32B | `Swizzle<1,4,3>` | 32바이트 단위 | fp32 또는 소규모 |
| None | - | 스위즐 없음 | 이미 충돌 없는 패턴 |

---

## 5. 프로듀서-컨슈머 파이프라인 설계

### 5-1. 기본 아이디어

```
시간 →
                Slot 0      Slot 1      Slot 2      Slot 0 ...
프로듀서:     [LOAD tile0] [LOAD tile1] [LOAD tile2] [LOAD tile3]
컨슈머:                    [COMP tile0] [COMP tile1] [COMP tile2]
                            ↑
                            로드와 연산 오버랩!
```

### 5-2. Circular Buffer 구조

```cuda
// SMEM 배열: QSIZE 슬롯의 순환 버퍼
__shared__ bf16 smem_A[QSIZE][BM * BK];
__shared__ bf16 smem_B[QSIZE][BK * BN];

// 배리어 배열
__shared__ cuda::barrier full[QSIZE];   // 프로듀서→컨슈머: "데이터 준비됨"
__shared__ cuda::barrier empty[QSIZE];  // 컨슈머→프로듀서: "슬롯 사용 완료"
```

### 5-3. 프로듀서 워프 그룹 동작

```cuda
// 프로듀서: TMA를 이용한 비동기 로드
for (int k = 0; k < K / BK; k++) {
    int slot = k % QSIZE;
    
    // 1. 슬롯이 비기를 대기 (컨슈머가 이전 데이터 소비 완료)
    empty[slot].wait(empty_token[slot]);
    empty_token[slot] ^= 1;  // 토큰 플립 (phase bit)
    
    // 2. TMA 비동기 로드 발행 (스레드 0만)
    if (threadIdx.x == 0) {
        barrier_arrive_tx(full[slot], 1, tile_bytes);
        
        cp_async_bulk_tensor_2d_global_to_shared(
            &smem_A[slot][0], tensorMapA,
            k * BK, blockIdx.x * BM, full[slot]);
            
        cp_async_bulk_tensor_2d_global_to_shared(
            &smem_B[slot][0], tensorMapB,
            blockIdx.y * BN, k * BK, full[slot]);
    }
    // 나머지 스레드도 배리어 arrive (바이트 수 0)
    else {
        full[slot].arrive();
    }
}
```

### 5-4. 컨슈머 워프 그룹 동작

```cuda
// 컨슈머: 텐서 코어 연산
float d[WGMMA_N/16][8] = {0};  // 어큐뮬레이터 초기화

for (int k = 0; k < K / BK; k++) {
    int slot = k % QSIZE;
    
    // 1. 데이터가 준비되기를 대기
    full[slot].wait(full_token[slot]);
    full_token[slot] ^= 1;
    
    // 2. WGMMA 연산 (SMEM에서 직접 읽음)
    wgmma.fence.sync.aligned;
    for (int i = 0; i < BK / WGMMA_K; i++) {
        wgmma64(d, &smem_A[slot][i*WGMMA_K], &smem_B[slot][i*WGMMA_K]);
    }
    wgmma.commit_group.sync.aligned;
    wgmma.wait_group.sync.aligned %0;
    
    // 3. 슬롯 소비 완료 알림
    empty[slot].arrive();
}

// 4. 어큐뮬레이터를 GMEM에 기록
store_result(d, C, ...);
```

### 5-5. QSIZE 선택 가이드

```
QSIZE = 1: 파이프라인 없음 (로드→연산→로드→연산)
QSIZE = 2: 기본 더블 버퍼링 (로드N+1 || 연산N)
QSIZE = 3: 트리플 버퍼링 (여유 슬롯으로 레이턴시 은닉 향상)
QSIZE = 5: 깊은 파이프라인 (SMEM 소비 증가, 레이턴시 은닉 극대화)

트레이드오프:
  ↑ QSIZE → ↑ SMEM 소비 → ↓ 동시 블록 수 (occupancy)
  ↑ QSIZE → ↑ 레이턴시 은닉 → ↑ 처리량
```

---

## 6. Persistent 커널과 타일 스케줄링

### 6-1. Persistent 커널이란?

```
일반 커널:
  Grid: (ceil(M/BM) × ceil(N/BN)) 블록 → SM에 분배 → 각 블록 1개 타일 처리 → 종료

Persistent 커널:
  Grid: (SM 수) 블록 → SM당 정확히 1개 블록 → 내부 루프로 여러 타일 처리 → 종료

예시 (H100 PCIe):
  일반: 4096/128 × 4096/128 = 32 × 32 = 1024 블록 스케줄링
  Persistent: 114 블록 (SM 수), 각각 ~9개 타일 처리
```

**장점:**
- 블록 스케줄링 오버헤드 제거
- 출력 스토어와 다음 타일 로드 오버랩 가능
- 타일 순서를 커널 내에서 직접 제어 → 캐시 최적화

### 6-2. 타일 스케줄링 전략

**전략 1: Naive (행 우선 래스터)**
```
타일 순서: (0,0) (0,1) (0,2) (0,3) ...
           (1,0) (1,1) (1,2) (1,3) ...

문제: 같은 행의 타일은 A를 공유하지만 B를 공유하지 않음
     → L2 캐시에서 B의 재사용 없음
```

**전략 2: 블록 단위 캐시 인지 (Block-wise Cache-Aware)**
```
타일 그리드를 S×S 블록으로 분할:

┌────┬────┬────┐
│ 1  │ 2  │ 5  │
│ 3  │ 4  │ 6  │
├────┼────┼────┤
│ 7  │ 8  │ 11 │
│ 9  │ 10 │ 12 │
└────┴────┴────┘

S×S 블록 내의 타일들은 A의 같은 행들과 B의 같은 열들을 공유
→ L2 히트율 증가
```

**전략 3: 힐베르트 곡선 (Space-Filling Curve)**
```
2D 공간을 1D 경로로 변환하되 공간적 인접성을 최대한 보존:

┌───┐   ┌───┐
│ 1→2   3→4 │
│   ↓   ↑   │
│   ↓   ↑   │
│ 8→7   6→5 │
│ ↓         ↑
│ 9→10  13→14│
│    ↓   ↑   │
│ 12→11 16→15│
└───────────┘

특성:
  - 연속된 타일이 항상 2D 이웃
  - 블록 단위보다 더 균일한 인접성
  - 다양한 크기의 타일 그리드에 일반적으로 적용 가능
```

**성능 비교 (원문):**
```
Naive 래스터:           기준선
블록 단위 캐시 인지:     +2~3%
힐베르트 곡선:          +4~5% (758 → 764 TFLOP/s)
```

### 6-3. CTA Rasterization 구현

```cuda
// Persistent 커널 내부
__global__ void matmul_persistent(params...) {
    int sm_id = blockIdx.x;  // SM당 1개 블록
    int total_tiles = grid_m * grid_n;
    
    for (int tile_idx = sm_id; tile_idx < total_tiles; tile_idx += num_sms) {
        // 힐베르트 곡선으로 2D 좌표 계산
        auto [tile_m, tile_n] = hilbert_d2xy(tile_idx, grid_size);
        
        // 이 타일의 K-loop 실행 (프로듀서-컨슈머 파이프라인)
        run_tile(tile_m, tile_n, ...);
        
        // 출력 스토어 (비동기, 다음 타일 로드와 오버랩)
        async_store_result(...);
    }
}
```

---

## 7. 레지스터 압력과 setmaxnreg

### 7-1. 레지스터 예산 문제

```
SM당 레지스터: 65,536 (32비트)

128×128 타일 (1 프로듀서 + 1 컨슈머):
  블록: 256 스레드 (2 워프 그룹)
  컨슈머 어큐뮬레이터: 128 × 128 / 128 스레드 = 128 fp32 = 128 레지스터
  + 주소 계산, 루프 변수 등: ~30 레지스터
  컨슈머 합계: ~158 레지스터/스레드
  
  프로듀서: TMA 발행만 → ~32 레지스터/스레드
  
  평균: (158 + 32) / 2 ≈ 95 레지스터/스레드 (불균형!)

128×256 타일 (1 프로듀서 + 2 컨슈머):
  블록: 384 스레드 (3 워프 그룹)
  컨슈머 어큐뮬레이터: 64 × 256 / 128 = 128 fp32 = 128 레지스터
  + 기타: ~30
  컨슈머: ~158/스레드
  
  65,536 / 384 = 170 레지스터/스레드 (상한)
  프로듀서에 170 할당하면 낭비!
```

### 7-2. setmaxnreg로 동적 재분배

```
Hopper 신규 PTX 명령어:
  setmaxnreg.inc.sync.aligned.u32 {count}  // 레지스터 예산 증가
  setmaxnreg.dec.sync.aligned.u32 {count}  // 레지스터 예산 감소

사용 패턴:
  프로듀서 워프 그룹:
    setmaxnreg.dec ... 40    // "나는 40개만 쓸게"
    → 남는 레지스터가 컨슈머에게 이전

  컨슈머 워프 그룹:
    setmaxnreg.inc ... 232   // "나는 232개 필요해"
    → 프로듀서가 반환한 레지스터 확보

결과:
  프로듀서: 40 레지스터 (TMA 발행에 충분)
  컨슈머: 232 레지스터 (대규모 어큐뮬레이터 + 여유)
  합계: 40 + 232 + 232 = 504 × 128스레드 = 64,512 ≤ 65,536 ✓
```

---

## 8. Thread Block Cluster와 DSMEM

### 8-1. Cluster 개념

```
Hopper 신규 프로그래밍 모델 계층:

Grid
└── Cluster (2~16 블록, 같은 GPC 내 SM에 배치)
    ├── Block 0 (SM 0) ─── SMEM 0
    ├── Block 1 (SM 1) ─── SMEM 1
    └── DSMEM: SMEM 0 + SMEM 1을 통합 접근 가능
```

### 8-2. matmul에서의 활용

```
2-SM 클러스터:
  SM 0: 타일의 상반부 (64×256)
  SM 1: 타일의 하반부 (64×256)
  
  B 행렬의 같은 열 타일을 공유:
    SM 0이 TMA로 B 타일 로드 → DSMEM을 통해 SM 1도 접근 가능
    → GMEM 대역폭 절반 절약!

TMA 멀티캐스트:
  cp_async_bulk_tensor_2d_global_to_shared::cluster(
      &smem[0], tensorMap, coord0, coord1, 
      cluster_mask,  // 어느 SM들에게 전달할지
      barrier
  );
  → 1회 TMA로 클러스터 내 모든 SM의 SMEM에 동시 기록
```

### 8-3. DSMEM 통신 패턴

```
직접 접근:
  SM 0의 스레드가 SM 1의 SMEM을 직접 load/store
  → L2를 거치지 않고 GPC 내부 인터커넥트 사용

원자적 연산:
  SM 0의 스레드가 SM 1의 SMEM에 atomicAdd
  → 부분합 동기화에 활용 가능

제약:
  - 같은 GPC 내 SM만 가능 (GPC 간 불가)
  - 레이턴시: SMEM보다 높음 (SM 간 통신 비용)
  - 대역폭: SMEM보다 낮음
```

---

## 9. Wave/Tile Quantization 문제

### 9-1. Tile Quantization

```
M=N=4096, BM=BN=128:
  타일 수: 32 × 32 = 1,024

M=N=4000, BM=BN=128:
  타일 수: ceil(4000/128) × ceil(4000/128) = 32 × 32 = 1,024
  
  BUT: 마지막 타일 (31번째)은 4000 - 31×128 = 32 원소만 유효
  → 128개 중 32개만 유효 = 75% 낭비
```

### 9-2. Wave Quantization

```
H100 PCIe: 114 SM, 블록당 1 SM (persistent 모드)

1,024 타일 / 114 SM = 8.98 → 9 wave 필요

wave 8까지: 114 SM × 8 = 912 타일 처리 (모든 SM 활용)
wave 9:    1,024 - 912 = 112 타일 → 114 SM 중 112만 활용 (2 SM 유휴)

더 나쁜 경우:
  115 타일 / 114 SM = 1.009 → 2 wave
  wave 1: 114 타일 (풀 활용)
  wave 2: 1 타일 (113 SM 유휴!) → ~50% GPU 활용률
```

### 9-3. 대응 전략

| 전략 | 설명 |
|------|------|
| 타일 크기 조정 | 문제 크기에 맞춰 BM,BN을 선택 |
| Split-K | K 차원도 병렬화하여 블록 수 증가 |
| Stream-K | K를 불균등 분할하여 마지막 wave 부분 처리 |
| Persistent + 힐베르트 | wave 내에서도 효율적 스케줄링 |

---

## 10. 성능 프로파일링과 병목 분석

### 10-1. NSight Compute 사용법

```bash
# 전체 프로파일 수집
ncu --set full -o profile_output.ncu-rep ./matmul_kernel

# 리소스 사용량 출력
nvcc --ptxas-options=-v matmul.cu

# SASS 디스어셈블리
nvdisasm matmul.cubin
```

### 10-2. 핵심 프로파일링 메트릭

| 메트릭 | 의미 | 목표 |
|--------|------|------|
| SM Throughput | SM 활용률 (%) | >80% |
| Memory Throughput | 메모리 대역폭 활용률 (%) | compute-bound면 낮아도 OK |
| Occupancy | 활성 워프 / 최대 워프 | 높을수록 좋지만 절대적이지 않음 |
| Warp Stall Reasons | 워프 대기 원인 분포 | barrier 대기가 지배적이면 정상 |
| Bank Conflicts | SMEM 뱅크 충돌 수 | 0에 가까울수록 좋음 |
| L2 Hit Rate | L2 캐시 히트율 | 타일 스케줄링 품질 지표 |
| Achieved FLOP/s | 실제 달성 처리량 | SoL 대비 비율로 평가 |

### 10-3. 병목 진단 흐름

```
1. Roofline 위치 확인
   → 연산 바운드? 메모리 바운드?

2. 연산 바운드라면:
   → TC 활용률 확인 → 낮으면 파이프라인 버블, 동기화 대기
   → ILP 확인 → 독립 명령어 부족하면 instruction 레벨에서 병목

3. 메모리 바운드라면:
   → GMEM 코얼레싱 확인 → 비코얼레싱이면 접근 패턴 수정
   → SMEM 뱅크 충돌 확인 → 있으면 스위즐링 적용
   → L2 히트율 확인 → 낮으면 타일 스케줄링 개선

4. 레이턴시 바운드라면:
   → occupancy 확인 → 낮으면 레지스터/SMEM 절감
   → 파이프라인 깊이 확인 → QSIZE 증가
   → 프로듀서-컨슈머 밸런스 → setmaxnreg 조정
```

### 10-4. Speed of Light 달성률

```
원문 최종 결과:
  달성: 764 TFLOP/s
  cuBLAS: ~714 TFLOP/s
  이론 SoL: ~756 TFLOP/s (PCIe, 쓰로틀링 감안)
  
  SoL 대비: 764 / 756 ≈ 101% 
  → 전력 쓰로틀링 지점이 다르거나 측정 조건 차이로 이론치 초과 가능
  cuBLAS 대비: 764 / 714 ≈ 107%
```
