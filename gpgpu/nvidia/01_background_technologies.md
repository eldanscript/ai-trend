# 배경 기술 Deep Dive

> 원문 이해를 위해 반드시 숙지해야 하는 배경 기술들을 나열하고 각각 세부적으로 정리한다.
> **소스**: https://www.aleksagordic.com/blog/matmul

---

## 목차

1. [반도체 물리 및 DRAM 동작 원리](#1-반도체-물리-및-dram-동작-원리)
2. [GPU 하드웨어 아키텍처](#2-gpu-하드웨어-아키텍처)
3. [메모리 계층 구조](#3-메모리-계층-구조)
4. [CUDA 프로그래밍 모델](#4-cuda-프로그래밍-모델)
5. [행렬 곱셈의 수학적 기초](#5-행렬-곱셈의-수학적-기초)
6. [Roofline 모델과 산술 강도](#6-roofline-모델과-산술-강도)
7. [부동소수점 수 체계](#7-부동소수점-수-체계)
8. [ISA 계층: PTX와 SASS](#8-isa-계층-ptx와-sass)
9. [동기화 프리미티브](#9-동기화-프리미티브)
10. [비트 연산과 스위즐링 수학](#10-비트-연산과-스위즐링-수학)

---

## 1. 반도체 물리 및 DRAM 동작 원리

### 1-1. DRAM Row Activation

원문에서 GMEM 코얼레싱의 중요성을 설명할 때, **DRAM의 물리적 동작**을 이해해야 한다.

**DRAM 셀 구조:**
- 하나의 DRAM 셀 = 1 트랜지스터 + 1 커패시터
- 커패시터에 전하를 저장하여 1비트를 표현
- 전하가 자연 방전되므로 주기적 리프레시 필요 (64ms 주기)

**Row Activation 과정:**
```
1. 행 주소 전송 → Row Address Strobe (RAS) 신호
2. 워드라인 활성화 → 해당 행의 모든 셀이 센스 앰프로 전송
3. 센스 앰프가 미세 전하를 감지하여 디지털 값으로 증폭
4. 열 주소 전송 → Column Address Strobe (CAS) 신호
5. 해당 열의 데이터 출력
```

**핵심 지연 시간:**
| 동작 | 시간 | 설명 |
|------|------|------|
| tRCD (RAS to CAS Delay) | ~13ns | 행 활성화 → 열 접근 가능까지 |
| tCAS (CAS Latency) | ~13ns | 열 주소 → 데이터 출력까지 |
| tRP (Row Precharge) | ~13ns | 행 닫기 (다른 행 접근 전) |
| tRAS (Row Active Time) | ~32ns | 행이 열린 상태 유지 최소 시간 |

**코얼레싱과의 관계:**
- 같은 행(row) 내의 연속 열 접근 → tRCD 1회만 지불 (버스트 모드)
- 다른 행 접근 → tRP + tRCD 추가 지불 (row miss)
- **GMEM 코얼레싱** = 워프 내 32개 스레드가 같은 DRAM 행의 연속 주소를 접근하도록 보장
- 코얼레싱된 접근: 1회 트랜잭션 (128바이트)
- 비코얼레싱: 최대 32회 트랜잭션 → **13배 성능 저하** (원문 실측)

### 1-2. HBM (High Bandwidth Memory)

H100이 사용하는 **HBM3** 메모리:

**구조:**
```
┌─────────────────────────┐
│      GPU Die            │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐  │
│  │H1│ │H2│ │H3│ │H4│  │  ← HBM3 스택 (각 8-12 DRAM 다이)
│  └──┘ └──┘ └──┘ └──┘  │
│    TSV   TSV  TSV  TSV  │  ← Through-Silicon Via 연결
│     인터포저 (Si)        │
└─────────────────────────┘
```

| 사양 | HBM2e | HBM3 | HBM3E |
|------|-------|------|-------|
| 핀당 대역폭 | 3.6 Gbps | 6.4 Gbps | 9.6 Gbps |
| 스택당 용량 | 16 GB | 24 GB | 36 GB |
| 스택당 대역폭 | 461 GB/s | 819 GB/s | 1.2 TB/s |
| H100 총 대역폭 | - | **3.35 TB/s** (SXM5) | - |

**왜 중요한가:**
- matmul 성능의 궁극적 한계 = 데이터를 HBM에서 얼마나 빨리 가져올 수 있는가
- Roofline 모델의 메모리 대역폭 천장이 HBM 사양에 의해 결정

---

## 2. GPU 하드웨어 아키텍처

### 2-1. H100 전체 구조

```
H100 SXM5 Die
├── 8 GPCs (Graphics Processing Clusters)
│   └── 각 GPC: 최대 18 SMs
│       └── 132 SMs 총 (일부 비활성화: yield 관리)
├── 6 HBM3 스택 (80 GB 총)
├── 60 MB L2 캐시
├── PCIe Gen5 x16 또는 NVLink 4.0
└── 전용 유닛: TMA, NVLink Engine, Copy Engine
```

### 2-2. Streaming Multiprocessor (SM) 내부 구조

```
SM (1개)
├── 4 Processing Blocks (Quadrants)
│   ├── 각 Quadrant:
│   │   ├── Warp Scheduler (1개) + Dispatch Unit (1개)
│   │   ├── 16 FP32 CUDA Cores
│   │   ├── 16 INT32 Cores
│   │   ├── 1 Tensor Core (4세대)
│   │   ├── 4 Load/Store Units
│   │   └── 4 SFU (Special Function Units)
│   │
│   └── 총 SM당:
│       ├── 64 FP32 CUDA Cores → 128 FP32 ops/clk
│       ├── 4 Tensor Cores
│       ├── 16 Load/Store Units
│       └── 16 SFUs
│
├── Register File: 65,536 × 32-bit 레지스터
├── L1/Shared Memory: 최대 228 KiB (설정 가능 비율)
├── L0 Instruction Cache
└── Texture/Surface Units
```

### 2-3. Tensor Core 동작 원리

**4세대 Tensor Core (Hopper):**
- 한 클럭에 `D = A × B + C` 수행
- 행렬 크기: 다양한 shape 지원 (m16n8k16 등)
- **WGMMA**: 워프 그룹(128 스레드) 단위로 협력 연산

```
입력 A (m×k, bf16) × 입력 B (k×n, bf16) → 출력 D (m×n, fp32)

Tensor Core 1회 연산:
- m64n64k16: 64×16 @ 16×64 → 64×64 부분합
- FLOPs per op: 2 × 64 × 64 × 16 = 131,072 FLOPs
```

**Speed of Light 계산:**
```
SoL = freq × num_SMs × TCs_per_SM × FLOPs_per_TC_per_clk
    = 2.0 GHz × 132 × 4 × 256
    = 약 270 TFLOP/s (단일 정밀도 등가)

BF16 Tensor Core:
    = 2.0 GHz × 132 × 4 × 512
    = 약 540 TFLOP/s (bf16 기준)
```

> 주의: 전력 쓰로틀링으로 실제 클럭이 1.5~1.8 GHz로 떨어지면 SoL도 비례 감소

### 2-4. 워프 스케줄링

**워프 = 32개 스레드의 SIMT 실행 단위**

```
SM Quadrant의 워프 스케줄러:
  매 클럭마다:
  1. 실행 가능한 워프(eligible warp) 목록 확인
  2. 스케줄링 정책에 따라 1개 워프 선택
  3. 선택된 워프의 다음 명령어 발행(issue)
  4. 모든 32개 스레드가 동일 명령어를 SIMT로 실행
```

**Occupancy (점유율):**
```
Occupancy = 활성 워프 수 / SM 최대 워프 수

SM 최대 워프: 64 (= 2048 스레드 / 32)
제한 요인:
  - 레지스터: 65,536 / (워프당 스레드 × 스레드당 레지스터)
  - SMEM: 228 KiB / 블록당 SMEM
  - 스레드: 2,048 / 블록당 스레드
```

---

## 3. 메모리 계층 구조

### 3-1. 전체 계층

```
레벨            용량         대역폭           레이턴시     공유 범위
─────────────────────────────────────────────────────────
레지스터         256 KB/SM    ~20 TB/s (추정)   0 clk       스레드
L0 I-Cache      ~16 KB/SM    -                  ~1 clk      SM
SMEM            228 KiB/SM   ~20 TB/s           ~20 clk     블록
L1 D-Cache      ~128 KB/SM   ~10 TB/s           ~30 clk     SM
L2 Cache        60 MB        ~5 TB/s            ~200 clk    전체 GPU
GMEM (HBM3)     80 GB        3.35 TB/s          ~400 clk    전체 GPU
Host (CPU)      TB급          ~64 GB/s (PCIe5)   ~10,000 clk 시스템
```

### 3-2. Shared Memory (SMEM) 상세

**물리 구조:**
```
32개 뱅크, 각 뱅크 폭 4바이트 (32비트)
  Bank 0  Bank 1  Bank 2  ...  Bank 31
  ┌────┐  ┌────┐  ┌────┐      ┌────┐
  │0x00│  │0x04│  │0x08│      │0x7C│   ← 주소 0~127 (128바이트 = 1행)
  │0x80│  │0x84│  │0x88│      │0xFC│   ← 주소 128~255
  │... │  │... │  │... │      │... │
  └────┘  └────┘  └────┘      └────┘

주소-뱅크 매핑: bank_id = (byte_address / 4) % 32
```

**뱅크 충돌 (Bank Conflict):**
```
상황 1: 충돌 없음 (No Conflict)
  스레드 0 → Bank 0, 스레드 1 → Bank 1, ..., 스레드 31 → Bank 31
  → 1 사이클에 32개 동시 서비스

상황 2: 2-way 충돌
  스레드 0,16 → Bank 0 (같은 뱅크, 다른 주소)
  → 2 사이클 필요 (직렬화)

상황 3: 32-way 충돌 (최악)
  모든 스레드 → Bank 0 (같은 뱅크, 다른 주소)
  → 32 사이클 필요

예외: 멀티캐스트 (같은 주소)
  스레드 0,1,2,...,31 → Bank 0의 동일 주소
  → 1 사이클 (브로드캐스트)
```

**원문과의 연결:**
- 스위즐링은 이 뱅크 충돌을 **XOR 비트 변환**으로 제거하는 기법
- TMA가 자동으로 스위즐링을 적용/해제

### 3-3. L1 Cache / Data Cache

**Set-Associative 캐시 구조:**
```
캐시 조회 과정:
1. 주소를 tag | set_index | offset 으로 분리
2. set_index로 캐시 세트 선택
3. 세트 내 모든 way의 tag와 비교
4. tag 일치 → hit (L1 레이턴시로 서비스)
5. tag 불일치 → miss → L2로 요청 전달
```

### 3-4. L2 Cache

- H100: **60 MB** 통합 L2
- 모든 SM이 공유
- GMEM(HBM) 접근 전 마지막 캐시 레벨
- matmul에서 **타일 스케줄링**이 L2 히트율에 직접 영향
  - Hilbert 곡선 스케줄링: 인접 타일이 같은 L2 라인을 공유 → 히트율 증가

---

## 4. CUDA 프로그래밍 모델

### 4-1. 실행 계층 구조

```
Grid (전체 작업)
├── Cluster (Hopper 신규, 선택적)
│   ├── Thread Block 0
│   │   ├── Warp Group 0 (128 threads = 4 warps)
│   │   │   ├── Warp 0 (threads 0-31)
│   │   │   ├── Warp 1 (threads 32-63)
│   │   │   ├── Warp 2 (threads 64-95)
│   │   │   └── Warp 3 (threads 96-127)
│   │   └── Warp Group 1 ...
│   ├── Thread Block 1
│   └── ...
└── ...
```

### 4-2. 스레드-위치 매핑

```cuda
// 2D 그리드에서의 위치 계산
int row = blockIdx.y * blockDim.y + threadIdx.y;
int col = blockIdx.x * blockDim.x + threadIdx.x;

// matmul에서의 출력 매핑
C[row * N + col] = dot_product(A[row,:], B[:,col]);
```

### 4-3. 메모리 공간

| 메모리 | 선언 | 스코프 | 수명 | 속도 |
|--------|------|--------|------|------|
| 레지스터 | `int x;` | 스레드 | 스레드 | 최고 |
| 로컬 | 레지스터 스필 | 스레드 | 스레드 | GMEM급 |
| Shared | `__shared__` | 블록 | 블록 | 높음 |
| Global | `__device__` | 전체 | 애플리케이션 | 낮음 |
| Constant | `__constant__` | 전체 | 애플리케이션 | 캐시됨 |

### 4-4. 동기화 메커니즘

```
레벨           API/명령어                    범위
──────────────────────────────────────────────────
워프 내        __syncwarp()                   32 스레드
블록 내        __syncthreads()                블록 전체
그리드          cooperative_groups            그리드 전체
비동기          cuda::barrier                 프로듀서-컨슈머
```

---

## 5. 행렬 곱셈의 수학적 기초

### 5-1. 기본 정의

```
C = A × B

A: M×K 행렬
B: K×N 행렬
C: M×N 행렬

C[i][j] = Σ(k=0 to K-1) A[i][k] × B[k][j]

FLOPs = 2 × M × N × K  (곱셈 M×N×K + 덧셈 M×N×K)
```

### 5-2. 타일링 분해

원문의 핵심 통찰: **matmul = 부분 외적(outer product)의 합**

```
표준 순서 (m-n-k): 각 출력 원소에 대해 내적(dot product) 계산
  for m: for n: for k: C[m,n] += A[m,k] * B[k,n]

변환된 순서 (k-m-n): 외적의 합
  for k: for m: for n: C[m,n] += A[m,k] * B[k,n]
  
  각 k에 대해: C += A[:,k] ⊗ B[k,:]  (외적 = rank-1 update)
```

**왜 외적 관점이 중요한가:**
- 블록 타일링 시, K 차원을 BK 크기로 분할
- 각 BK 청크를 SMEM에 로드 → 해당 청크의 외적 계산 → 다음 청크
- **산술 강도**가 BM, BN에 비례하여 증가

### 5-3. 타일링의 산술 강도 분석

```
타일 크기: BM × BN (출력), BK (K 방향)

K/BK 반복에서:
  로드 바이트: (BM × BK + BK × BN) × sizeof(element)
  계산 FLOPs: 2 × BM × BN × BK

산술 강도(AI) = (2 × BM × BN × BK) / ((BM + BN) × BK × sizeof)
             = (2 × BM × BN) / ((BM + BN) × sizeof)

BM = BN = 128, bf16(2바이트):
  AI = 2 × 128 × 128 / (256 × 2) = 64 FLOPs/byte

H100 Roofline 릿지 포인트: ~990 TFLOP/s / 3.35 TB/s ≈ 295 FLOPs/byte
→ AI=64로는 여전히 메모리 바운드 → 더 큰 타일 또는 재사용 필요
```

---

## 6. Roofline 모델과 산술 강도

### 6-1. Roofline 모델 개념

```
                    ┌─────────────────── 연산 천장 (Peak FLOP/s)
                    │
성능 (FLOP/s)       │         ────────────
                    │        /
                    │       / ← 대역폭 제한 영역
                    │      /   (기울기 = 메모리 대역폭)
                    │     /
                    │    /
                    │   /   │
                    │  /    │ 릿지 포인트
                    │ /     │
                    └───────┴──────────────
                    산술 강도 (FLOPs/byte)
```

### 6-2. H100 Roofline 파라미터

```
연산 천장:
  - FP32 CUDA Core: ~67 TFLOP/s
  - BF16 Tensor Core: ~990 TFLOP/s (SXM5)
  - BF16 Tensor Core: ~756 TFLOP/s (PCIe, 114 SM)

메모리 대역폭:
  - HBM3 (SXM5): 3.35 TB/s
  - HBM3 (PCIe): ~2.0 TB/s

릿지 포인트:
  - SXM5 BF16 TC: 990 / 3.35 ≈ 295 FLOPs/byte
  - PCIe BF16 TC: 756 / 2.0 ≈ 378 FLOPs/byte

해석:
  산술 강도 > 릿지 포인트 → 연산 바운드 (compute-bound)
  산술 강도 < 릿지 포인트 → 메모리 바운드 (memory-bound)
```

### 6-3. matmul과 Roofline

```
M=N=K=4096, bf16 기준:
  FLOPs = 2 × 4096³ = 137.4 GFLOPs
  바이트 로드 = 2 × 4096² × 2 = 67.1 MB (A + B)
  AI = 137.4G / 67.1M ≈ 2048 FLOPs/byte → 연산 바운드!

→ 대규모 matmul은 본질적으로 compute-bound
→ 핵심은 텐서 코어 활용률 극대화
```

---

## 7. 부동소수점 수 체계

### 7-1. 원문에서 사용되는 데이터 타입

| 형식 | 비트 | 부호 | 지수 | 가수 | 범위 | 용도 |
|------|------|------|------|------|------|------|
| FP32 | 32 | 1 | 8 | 23 | ±3.4×10³⁸ | Naive 커널, 어큐뮬레이터 |
| BF16 | 16 | 1 | 8 | 7 | ±3.4×10³⁸ | Hopper 커널 입력 |
| FP16 | 16 | 1 | 5 | 10 | ±65,504 | 대안 (더 높은 정밀도) |
| FP8 (E4M3) | 8 | 1 | 4 | 3 | ±448 | 추론 최적화 |
| FP8 (E5M2) | 8 | 1 | 5 | 2 | ±57,344 | 학습 (더 넓은 범위) |

### 7-2. BF16 (Brain Floating Point) 상세

```
비트 레이아웃: [S][EEEEEEEE][MMMMMMM]
               1    8           7

FP32와 비교:
  FP32: [S][EEEEEEEE][MMMMMMMMMMMMMMMMMMMMMMM]
  BF16: [S][EEEEEEEE][MMMMMMM]  ← FP32의 상위 16비트와 동일 구조!

변환: FP32 → BF16 = 단순히 하위 16비트 절삭 (truncation)
     BF16 → FP32 = 하위 16비트에 0 패딩

장점:
  - FP32와 동일한 동적 범위 (지수 8비트)
  - FP32 ↔ BF16 변환이 매우 저렴
  - 텐서 코어에서 2배 처리량 (16비트 vs 32비트)

단점:
  - FP16 대비 정밀도 낮음 (가수 7비트 vs 10비트)
```

### 7-3. 혼합 정밀도 (Mixed Precision)

원문의 Hopper 커널 전략:
```
입력:    BF16 (A, B 행렬) → 메모리 대역폭 절약, TC 처리량 2배
누적기:  FP32 (C 행렬)   → 정밀도 유지, 누적 오차 방지
출력:    BF16 또는 FP32   → 용도에 따라 선택

wgmma.mma_async.sync.aligned.m64n64k16.f32.bf16.bf16
                                        ^^^  ^^^^  ^^^^
                                        출력  입력A 입력B
```

---

## 8. ISA 계층: PTX와 SASS

### 8-1. 컴파일 파이프라인

```
CUDA C/C++ (.cu)
    │ nvcc (프론트엔드)
    ▼
PTX (Parallel Thread eXecution) ← 가상 ISA, 아키텍처 독립적
    │ ptxas (백엔드)
    ▼
SASS (Shader ASSembly) ← 실제 GPU 기계어, 아키텍처 종속적
    │ GPU 실행
    ▼
하드웨어 실행
```

### 8-2. PTX 주요 명령어 (원문 참조)

```
// 메모리 명령어
ld.global.f32       rd, [addr]        // GMEM → 레지스터
st.global.f32       [addr], rs        // 레지스터 → GMEM
ld.shared.f32       rd, [addr]        // SMEM → 레지스터
st.shared.f32       [addr], rs        // 레지스터 → SMEM

// 연산 명령어
fma.rn.f32          rd, ra, rb, rc    // rd = ra * rb + rc (FMA)

// Tensor Core 명령어 (Hopper)
wgmma.mma_async.sync.aligned.m64n64k16.f32.bf16.bf16
wgmma.fence.sync.aligned
wgmma.commit_group.sync.aligned
wgmma.wait_group.sync.aligned

// TMA 명령어
cp.async.bulk.tensor.2d.shared::cluster.global.tile
    [smem_addr], [tensor_map, {coords}], [mbar]

// 레지스터 관리
setmaxnreg.inc.sync.aligned.u32  {count}  // 레지스터 예산 증가
setmaxnreg.dec.sync.aligned.u32  {count}  // 레지스터 예산 감소
```

### 8-3. SASS 명령어 (원문 참조)

```
// 벡터화된 로드/스토어
LDG.32    R0, [R2]          // Global 4바이트 로드
LDG.128   R0, [R2]          // Global 16바이트 로드 (4×fp32)
STS.128   [R4], R0          // Shared 16바이트 스토어
LDS.128   R0, [R4]          // Shared 16바이트 로드

// 제어 흐름
BRA       label             // 분기
EXIT                        // 스레드 종료

// 비트 연산
BFI       R0, R1, R2, R3   // Bit Field Insert
```

### 8-4. 컴파일러 최적화 관찰 (원문)

원문에서 SASS 디스어셈블리를 분석하여 발견한 것들:
```
최적화:
  - 루프 언롤링: PTX의 4× → SASS의 16×
  - 벡터화: 32개 LDG.32 → 1개 LDG.128 (코얼레싱 시)
  - 테일 루프: 언롤 팩터를 점진적으로 줄이는 다중 테일 루프

비효율:
  - 불필요한 변수 0 초기화
  - 복잡한 주소 계산 (상수 접기 미적용)
  - 사용되지 않는 레지스터 (dead code)
  - EXIT 이후 무한 while 루프
  - NOP 패딩 (정렬용)
```

---

## 9. 동기화 프리미티브

### 9-1. 전통적 동기화

```cuda
// 블록 내 배리어
__syncthreads();  // 블록 내 모든 스레드가 이 지점에 도달할 때까지 대기

// 워프 내 동기화
__syncwarp(mask);  // 지정 마스크의 스레드들만 동기화

// 메모리 펜스
__threadfence();       // 디바이스 수준 메모리 가시성 보장
__threadfence_block(); // 블록 수준 메모리 가시성 보장
```

### 9-2. 비동기 배리어 (Hopper)

원문의 프로듀서-컨슈머 파이프라인 핵심:

```cuda
// 배리어 초기화 (SMEM에 위치)
cuda::barrier bar;
init(&bar, thread_count);

// 트랜잭션 기반 배리어 (TMA용)
barrier_arrive_tx(bar, 1, byte_count);
// → "나는 도착했고, byte_count 만큼의 데이터 전송을 기대한다"

// 대기 조건: 두 조건 모두 충족 시 해제
// 1) 모든 스레드가 arrive()
// 2) 전송된 바이트 합 ≥ 등록된 byte_count 합

// Circular buffer 패턴
barrier full[QSIZE];   // "이 슬롯에 데이터가 채워졌다"
barrier empty[QSIZE];  // "이 슬롯이 비었다 (소비 완료)"

// 프로듀서:
empty[slot].wait();               // 슬롯이 비기를 대기
TMA_load(smem[slot], gmem);       // 비동기 로드 시작
full[slot].arrive_tx(byte_count); // 데이터 도착 알림

// 컨슈머:
full[slot].wait();                // 데이터가 채워지기를 대기
compute(smem[slot]);              // 텐서 코어 연산
empty[slot].arrive();             // 소비 완료 알림
```

### 9-3. WGMMA 동기화 프로토콜

```cuda
// 1. 펜스: 이전 메모리 연산 완료 보장
wgmma.fence.sync.aligned;

// 2. 비동기 MMA 발행 (여러 개 연속 가능)
wgmma.mma_async ...;
wgmma.mma_async ...;
wgmma.mma_async ...;
wgmma.mma_async ...;

// 3. 커밋: 발행된 MMA들을 그룹으로 묶음
wgmma.commit_group.sync.aligned;

// 4. 대기: 그룹 완료까지 대기
wgmma.wait_group.sync.aligned %0;  // 0 = 모든 그룹 완료
```

---

## 10. 비트 연산과 스위즐링 수학

### 10-1. XOR 연산 기초

```
XOR (⊕) 진리표:
  0 ⊕ 0 = 0
  0 ⊕ 1 = 1
  1 ⊕ 0 = 1
  1 ⊕ 1 = 0

핵심 성질:
  a ⊕ a = 0  (자기 역원)
  a ⊕ 0 = a  (항등원)
  (a ⊕ b) ⊕ b = a  (가역성 → TMA가 역변환 가능)
```

### 10-2. Swizzle<B, M, S> 함수

원문의 `Swizzle<3,4,3>` (128바이트 모드):

```
입력 주소: ... G H I J K L ...
           ↑↑↑       ↑↑↑
           비트 9,8,7  비트 6,5,4

변환 과정:
  Step 1: bit_msk = (1 << B) - 1 = (1 << 3) - 1 = 0b111
  Step 2: yyy_msk = bit_msk << (M + S) = 0b111 << 7 = 0b111_0000000
  Step 3: masked  = input & yyy_msk     → 비트 9,8,7 추출 (GHI)
  Step 4: shifted = masked >> S          → 3비트 오른쪽 시프트 → 비트 6,5,4 위치로
  Step 5: output  = input ^ shifted      → XOR로 비트 6,5,4를 변환

결과: 비트 9,8,7 (행 인덱스)이 비트 6,5,4 (뱅크 선택)을 XOR로 변조
→ 같은 뱅크에 매핑되던 다른 행의 데이터가 분산됨
→ 행 방향 AND 열 방향 접근 모두 뱅크 충돌 없음!
```

**시각화:**
```
스위즐링 전 (뱅크 충돌 발생):
  Row 0: Bank 0,1,2,3,4,5,6,7
  Row 1: Bank 0,1,2,3,4,5,6,7  ← 같은 열 접근 시 충돌!
  Row 2: Bank 0,1,2,3,4,5,6,7

스위즐링 후 (충돌 제거):
  Row 0: Bank 0,1,2,3,4,5,6,7
  Row 1: Bank 1,0,3,2,5,4,7,6  ← XOR로 뱅크 번호 변조
  Row 2: Bank 2,3,0,1,6,7,4,5
  Row 3: Bank 3,2,1,0,7,6,5,4

→ 어떤 열을 선택해도 32개 뱅크 모두 다른 뱅크!
```

### 10-3. Bit Field Insert (BFI)

원문에서 언급된 SASS 명령어:
```
BFI Rd, Ra, Rb, Rc
→ Rd의 특정 비트 필드를 Ra의 값으로 교체
→ 스위즐링된 SMEM 주소 계산에 사용
```
