# 🧠 AI용 NPU 서버 마스터 트레이닝 프로그램

> **목표:** 초보자에서 NPU 서버 아키텍트/엔지니어로 성장하는 체계적 훈련 과정
> **대상:** 클라우드 아키텍처 경험이 있는 엔지니어 (AWS/GCP/Azure 숙련자)
> **기간:** 약 6–9개월 (풀타임 학습 기준 4개월 압축 가능)

---

## 전체 로드맵 개요

| 단계 | 주제 | 기간 | 핵심 역량 |
|------|------|------|-----------|
| **Ⅰ** | 반도체 물리 + 디지털 로직 기초 | 3주 | 트랜지스터→게이트→ALU 흐름 이해 |
| **Ⅱ** | 컴퓨터 아키텍처 + 메모리 계층 | 4주 | 파이프라인, 캐시, DRAM/SRAM/HBM |
| **Ⅲ** | AI 워크로드 해부 | 3주 | GEMM, Convolution, Attention의 하드웨어 매핑 |
| **Ⅳ** | NPU 아키텍처 심층 분석 | 5주 | Systolic Array, Dataflow, 주요 NPU 칩 비교 |
| **Ⅴ** | NPU 컴파일러 스택 | 4주 | MLIR, TVM, ONNX Runtime, 양자화 |
| **Ⅵ** | NPU 서버 시스템 설계 | 4주 | 랙 설계, 냉각, 전력, 네트워크 토폴로지 |
| **Ⅶ** | 운영·최적화·벤치마킹 | 3주 | 프로파일링, 서빙 파이프라인, TCO 분석 |
| **Ⅷ** | 마스터 프로젝트 | 4주 | 종합 설계 + 논문급 기술 보고서 |

---

## 단계 Ⅰ: 반도체 물리 + 디지털 로직 기초 (3주)

### 왜 이것부터?
NPU를 진정으로 이해하려면 "트랜지스터가 어떻게 행렬곱을 하는가"의 물리적 직관이 필요합니다. 이것 없이는 데이터시트의 숫자가 의미 없는 마케팅 수치로 전락합니다.

### 학습 내용

**Week 1: 반도체 물리 핵심**
- MOSFET 동작 원리 (N-type, P-type)
- CMOS 인버터 → NAND/NOR 게이트 구성
- 공정 노드의 의미: 4nm vs 5nm vs 7nm이 실제로 뜻하는 것
- 누설 전류(Leakage)와 동적 전력(Dynamic Power)의 트레이드오프
- **지름길:** "공정 미세화 = 무조건 좋다"가 아닌 이유를 이해하면, TSMC N3 vs N5 논쟁을 즉시 해석 가능

**Week 2: 디지털 로직 설계**
- 가산기(Adder), 곱셈기(Multiplier) 회로
- MAC(Multiply-Accumulate) 유닛 — NPU의 핵심 빌딩 블록
- 클록 주파수 vs 처리량(Throughput) 관계
- 파이프라인의 개념과 하드웨어 구현

**Week 3: VLSI 설계 흐름 개요**
- RTL → 합성 → 배치배선 → 테이프아웃 파이프라인
- FPGA vs ASIC 차이점과 각각의 적용 시나리오
- IP 코어, 다이(Die), 패키지, 칩렛(Chiplet) 개념
- UCIe (Universal Chiplet Interconnect Express) 표준 이해

### 비주류 자원
1. **"From Nand to Tetris"** (nand2tetris.org) — 게이트부터 컴퓨터까지 직접 구축하는 프로젝트 기반 학습
2. **Sam Zeloof의 YouTube** — 자택 차고에서 반도체 만드는 DIY 영상 (물리적 직관 형성)
3. **Ken Shirriff 블로그** (righto.com) — 빈티지 칩을 현미경으로 해부하며 설명
4. **ZeroToASIC** (zerotoasic.com) — 실제 ASIC 테이프아웃 경험을 단축하는 커뮤니티 프로젝트
5. **OpenROAD 프로젝트** — 오픈소스 ASIC 설계 도구 체인 (실습 가능)

### 검증 과제
```
[과제 Ⅰ-1] MAC 유닛 시뮬레이션
- Verilog 또는 Python으로 8-bit MAC 유닛을 구현하라
- INT8 두 수의 곱셈 + 누적을 시뮬레이션하라
- 이 MAC 유닛 256개를 2D 배열로 구성했을 때
  초당 가능한 연산 수(OPS)를 클록 1GHz 기준으로 계산하라

[과제 Ⅰ-2] 전력 분석 미니 리포트
- TSMC 4nm vs 5nm 공정에서 동일 설계의 전력 차이를 
  공개 논문/데이터시트로 조사하여 1페이지 요약을 작성하라
- "왜 Rebellions는 삼성 4nm을, Groq는 삼성 14nm을 선택했는가?"에 답하라

[검증 시뮬레이션 Ⅰ]
질문: "어떤 NPU가 TOPS/W 기준으로 우수하다고 주장합니다. 
이 주장을 검증하려면 어떤 정보가 추가로 필요합니까?"
→ 최소 5가지 항목을 열거하고 각각의 이유를 설명하라.
```

---

## 단계 Ⅱ: 컴퓨터 아키텍처 + 메모리 계층 (4주)

### 왜 중요한가?
NPU 서버 성능의 80%는 "계산 능력"이 아니라 "데이터를 얼마나 빨리 가져올 수 있는가"에 달려 있습니다. 메모리 대역폭이 진짜 병목입니다.

### 학습 내용

**Week 4: CPU/GPU 아키텍처 복습**
- von Neumann vs Harvard 아키텍처
- SIMD, SIMT, MIMD 병렬화 모델
- GPU 아키텍처: SM(Streaming Multiprocessor), Warp, Thread Block
- CPU와 GPU의 근본적 설계 철학 차이

**Week 5: 메모리 계층 심층 분석**
- 레지스터 → SRAM → DRAM → Storage 계층 별 지연시간과 대역폭
- **HBM (High Bandwidth Memory):** HBM2e → HBM3 → HBM3E → HBM4 진화
  - HBM3E: ~1.2 TB/s per stack
  - HBM4: 2026년 양산, ~1.65 TB/s 예상
- **SRAM vs HBM 트레이드오프:** Groq (230MB SRAM only) vs NVIDIA (HBM 중심) 설계 철학
- Memory Wall 문제와 해결 접근법

**Week 6: 인터커넥트와 데이터 이동**
- PCIe Gen 5/6, CXL (Compute Express Link)
- NVLink, NVSwitch, InfiniBand vs RoCE
- 칩 간(Die-to-Die) 인터커넥트: UCIe, BoW(Bunch of Wires)
- **Roofline 모델:** 모든 AI 하드웨어 분석의 핵심 도구
  - Operational Intensity = FLOPs / Bytes Accessed
  - Compute-bound vs Memory-bound 워크로드 구분

**Week 7: 이종 컴퓨팅(Heterogeneous Computing)**
- CPU + GPU + NPU 협업 모델
- DMA(Direct Memory Access) 엔진의 역할
- 호스트-디바이스 메모리 전송 최적화
- NUMA(Non-Uniform Memory Access) 아키텍처

### 비주류 자원
1. **Onur Mutlu의 강의 (ETH Zürich)** — YouTube에 전체 공개된 최고 수준의 컴퓨터 아키텍처 강의
2. **"A Primer on Memory Consistency and Cache Coherence" (2nd ed.)** — 무료 PDF 공개
3. **WikiChip** (wikichip.org) — 비공식이지만 가장 상세한 프로세서 아키텍처 데이터베이스
4. **Chips and Cheese 블로그** (chipsandcheese.com) — 마이크로아키텍처 리버스 엔지니어링 분석
5. **"Computer Architecture: A Quantitative Approach" (Hennessy & Patterson)** — 6th ed., 특히 Ch.7 Domain-Specific Architectures

### 검증 과제
```
[과제 Ⅱ-1] Roofline 분석
- LLaMA 2 7B 모델의 Decode 단계를 Roofline 모델로 분석하라
- 다음 세 플랫폼에서의 위치를 그래프에 표시하라:
  (a) NVIDIA A100 (HBM2e, 2TB/s, 312 TFLOPS FP16)
  (b) Groq LPU (SRAM 80TB/s on-die, 750 TOPS INT8)
  (c) Mobile NPU (50-90 GB/s, ~50 TOPS)
- 각각이 compute-bound인지 memory-bound인지 판단하고 이유를 설명하라

[과제 Ⅱ-2] 메모리 대역폭 계산
- 70B 파라미터 LLM을 INT4로 양자화하면 모델 크기는?
- 이 모델의 Decode 단계에서 token/s를 최대화하려면
  필요한 최소 메모리 대역폭을 계산하라 (목표: 100 tok/s)
- HBM3E 1스택 vs 4스택의 이론적 대역폭을 비교하라

[검증 시뮬레이션 Ⅱ]
시나리오: "고객이 'GPU 대비 NPU가 왜 더 효율적인가?'라고 묻습니다."
→ Roofline 모델, 전력 효율, 메모리 대역폭 활용률 세 축으로 
   5분 분량의 기술 브리핑 자료를 작성하라.
```

---

## 단계 Ⅲ: AI 워크로드 해부 (3주)

### 핵심 통찰
NPU를 이해하려면 NPU가 가속하는 "대상"을 해부해야 합니다. 모든 AI 모델은 결국 소수의 수학 연산으로 분해됩니다.

### 학습 내용

**Week 8: 핵심 연산 해부**
- **GEMM (General Matrix Multiply):** AI 컴퓨팅의 80%를 차지하는 연산
  - Tiling 전략: M×N×K 분할과 데이터 재사용
  - 배치 GEMM과 Transformer의 관계
- **Convolution → im2col → GEMM 변환** 원리
- **Attention 메커니즘:** Self-Attention, Multi-Head Attention, GQA, MQA
  - FlashAttention의 핵심: Tiling으로 HBM 접근 최소화
  - KV-Cache의 메모리 요구량과 관리

**Week 9: LLM 추론 파이프라인**
- **Prefill (Prompt Processing):** Compute-bound, 높은 FLOPS 필요
- **Decode (Token Generation):** Memory-bound, 높은 대역폭 필요
- 이 두 단계의 근본적 차이가 하드웨어 설계에 미치는 영향
- **MoE (Mixture of Experts):** Expert 라우팅과 동적 메모리 로드
  - GPT-OSS-20B의 AMD NPU 배포 사례: 동적 expert weight 로딩

**Week 10: 양자화와 수치 형식**
- FP32 → FP16 → BF16 → FP8 (E4M3, E5M2) → INT8 → INT4
- **MXFP4, NF4** 등 최신 형식
- 양자화가 정확도에 미치는 영향: MMLU 벤치마크로 검증
- Mixed-Precision 전략: 가중치 INT4 + 활성화 BF16 조합
- PTQ(Post-Training Quantization) vs QAT(Quantization-Aware Training)

### 비주류 자원
1. **"Efficient Transformers: A Survey" (Tay et al., 2022)** — Attention 변형 총정리
2. **FlashAttention 논문 + Tri Dao의 블로그** — Tiling 기반 최적화의 원전
3. **"LLM Inference Unveiled" (Dissecting Batched Prefill and Decode, 2024)** — Prefill/Decode 분리 전략
4. **NVIDIA Cutlass GitHub** — GEMM 커널 최적화의 바이블 코드
5. **"A Survey on Efficient Inference for LLMs" (Xiao et al.)** — 양자화 총정리

### 검증 과제
```
[과제 Ⅲ-1] GEMM 타일링 실습
- Python/NumPy로 naive GEMM vs tiled GEMM을 구현하라
- 타일 크기(32, 64, 128, 256)별 실행시간을 측정하고
- 왜 특정 타일 크기에서 성능이 최적인지 캐시 크기와 연관지어 설명하라

[과제 Ⅲ-2] LLM 추론 분석
- LLaMA 2 7B (FP16)의 단일 토큰 생성에 필요한:
  (a) 총 FLOPs
  (b) 필요 메모리 접근량 (bytes)
  (c) Arithmetic Intensity (FLOPs/byte)
  를 계산하라
- 이 수치를 Roofline 모델에 배치하고, A100 vs Groq LPU에서
  각각 이론적 최대 tok/s를 추정하라

[검증 시뮬레이션 Ⅲ]
시나리오: "DeepSeek-V3 (MoE, 671B total / 37B active)를 
NPU 서버에 배포해야 합니다."
→ MoE의 expert routing이 NPU 설계에 주는 도전과제 3가지를 
   기술적으로 설명하고, 각각의 해결 접근법을 제시하라.
```

---

## 단계 Ⅳ: NPU 아키텍처 심층 분석 (5주)

### 이 단계가 핵심입니다
여기서부터 GPU 엔지니어와 NPU 엔지니어가 갈라집니다.

### 학습 내용

**Week 11–12: NPU 핵심 아키텍처 패턴**

**(1) Systolic Array**
- Google TPU의 근간 아키텍처
- 데이터가 PE(Processing Element) 배열을 "맥박"처럼 흐르는 구조
- Weight-Stationary vs Output-Stationary vs Row-Stationary 데이터플로우
- 장점: 높은 에너지 효율, 데이터 재사용 극대화
- 한계: 불규칙한 연산(Sparse, Dynamic shape)에 비효율적

**(2) Dataflow Architecture**
- SambaNova RDU (Reconfigurable Dataflow Unit)
- 각 연산 단계가 공간적으로 매핑되어 파이프라인처럼 실행
- 재구성 가능성(Reconfigurability)의 장단점

**(3) SRAM-Only / Deterministic Architecture**
- Groq LPU: HBM 없이 230MB SRAM만 사용
- 결정론적 실행(Deterministic Execution): 지연시간 예측 가능
- 장점: 극단적으로 낮은 레이턴시 (TTFB ~0.22s)
- 한계: 모델 크기 제한, 대규모 배치 처리 비효율

**(4) Wafer-Scale Engine**
- Cerebras WSE-3: 900,000 AI 코어, 4조 트랜지스터
- 단일 웨이퍼 전체를 하나의 칩으로 사용
- 44GB on-die SRAM, 21 PB/s 내부 대역폭
- 장점: GPU 클러스터 대비 통신 병목 제거
- 한계: 수율, 냉각, 비용

**(5) 프로그래머블 NPU**
- Rebellions ATOM/Rebel 시리즈: Programmable Dataflow Architecture
  - Instruction-level + Data-level 병렬화
  - UCIe 기반 칩렛 확장 (REBEL-Quad = 4×Rebel)
  - Samsung 4nm 공정, HBM3E 탑재
- FuriosaAI RNGD: 한국 AI 반도체의 또 다른 축
  - 2025년 말부터 20,000유닛 공급 목표
- AMD XDNA (Ryzen AI NPU): AI Engine tile 기반 spatial array
  - MLIR-AIE를 통한 프로그래밍 가능

**Week 13–14: 주요 NPU/가속기 비교 분석**

| 칩 | 제조사 | 아키텍처 | 공정 | 메모리 | 핵심 특징 |
|----|--------|----------|------|--------|-----------|
| WSE-3 | Cerebras | Wafer-Scale | TSMC 5nm | 44GB SRAM | 단일 웨이퍼 칩 |
| LPU | Groq | TSP (SRAM-only) | Samsung 14nm | 230MB SRAM | 결정론적 초저레이턴시 |
| SN50 | SambaNova | RDU (Dataflow) | - | 3-tier 메모리 | 10T+ 파라미터 지원 |
| TPU v6 (Trillium) | Google | Systolic Array | - | HBM | 4.7x/chip 성능 향상 |
| Trainium2 | AWS | Custom | - | HBM | 83.2 PFLOPS UltraServer |
| Inferentia2 | AWS | Custom | - | HBM | 추론 특화, 70% 비용절감 |
| Gaudi3 | Intel | Custom | TSMC 5nm | HBM2E | 긴 출력 LLM 추론 강점 |
| Rebel | Rebellions | Programmable Dataflow | Samsung 4nm | HBM3E | UCIe 칩렛, 한국 토종 |
| RNGD | FuriosaAI | Custom | - | - | LLM 추론 특화 |
| Maia 100 | Microsoft | Custom ASIC | TSMC N5 | HBM | Azure 전용 |
| AI200/250 | Qualcomm | Hexagon NPU | - | Near-memory (AI250) | 2026-2027 출시 |
| "Titan" | OpenAI/Broadcom | Systolic Array | TSMC 3nm | HBM3E/HBM4 | 2026 H2 배포 |

**Week 15: 한국 NPU 생태계 심층 분석**
- **Rebellions:** SAPEON 합병 배경, SK 그룹 연결고리, 삼성 투자
  - REBEL-Quad 2026 상반기 양산 계획
  - 정부 실증 인프라 확보 필요성 논의
- **FuriosaAI:** 한국 정부 프로젝트, RNGD 로드맵
- **한국 AI 반도체 생태계:**
  - SK Hynix HBM4 2026 양산 → NPU 생태계 시너지
  - 삼성 파운드리 4nm/3nm GAA 공정
  - 국가 AI 반도체 전략과 예산 (NVIDIA 대비 1/10 미만 현실)

### 비주류 자원
1. **Hot Chips / ISSCC / MICRO 학회 논문들** — 각 칩의 원본 아키텍처 공개 자료
2. **The Next Platform** (nextplatform.com) — Timothy Prickett Morgan의 심층 하드웨어 분석 (Rebellions 기사 등)
3. **SemiAnalysis 뉴스레터** (semianalysis.com) — Dylan Patel의 반도체 업계 인사이드 분석
4. **"Efficiently Scaling Transformer Inference" (Pope et al., Google, 2022)** — TPU에서의 LLM 추론 최적화 원전
5. **Rebellions 기술 블로그 / GitHub** — RBLN SDK 문서
6. **EE Times / AnandTech (via Chips and Cheese)** — 칩 아키텍처 리뷰

### 검증 과제
```
[과제 Ⅳ-1] 아키텍처 비교 매트릭스
- Systolic Array, Dataflow, SRAM-Only, Wafer-Scale 4가지 아키텍처를
  다음 7가지 축으로 비교하는 상세 매트릭스를 작성하라:
  (1) 에너지 효율 (TOPS/W)
  (2) 레이턴시 예측 가능성
  (3) 모델 크기 확장성
  (4) 배치 처리 효율
  (5) Sparse 연산 지원
  (6) 프로그래밍 용이성
  (7) 총 소유 비용 (TCO)

[과제 Ⅳ-2] NPU 선택 의사결정 트리
- 다음 시나리오별로 최적의 NPU를 추천하고 근거를 제시하라:
  (a) 실시간 챗봇 (레이턴시 < 100ms TTFB)
  (b) 배치 문서 처리 (처리량 극대화, 비용 최소화)
  (c) 70B LLM 서빙 (높은 동시 사용자)
  (d) 한국 정부 프로젝트 (보안 요구사항 + 국산 선호)
  (e) 엣지 추론 (전력 < 75W)

[과제 Ⅳ-3] Rebellions REBEL 분석 리포트
- 공개 자료를 기반으로 REBEL 칩의:
  (a) 아키텍처 특징 (Programmable Dataflow, UCIe)
  (b) NVIDIA H100 대비 예상 강점/약점
  (c) 타겟 워크로드 및 시장 포지셔닝
  을 2페이지 기술 리포트로 작성하라

[검증 시뮬레이션 Ⅳ]
역할극: 당신은 NPU 서버 아키텍트입니다. CTO가 묻습니다:
"NVIDIA H100 8-GPU 서버 vs Cerebras CS-3 2대 vs Groq GroqRack —
 우리 서비스(실시간 LLM 추론, 70B, 동시 500명)에 뭘 사야 하나?"
→ 각 옵션의 TCO, 성능, 운영 복잡도를 비교하는 의사결정 보고서를 작성하라.
```

---

## 단계 Ⅴ: NPU 컴파일러 스택 (4주)

### 왜 컴파일러?
하드웨어가 아무리 좋아도, 모델을 하드웨어에 효율적으로 매핑하는 컴파일러 없이는 성능의 10%도 끌어내지 못합니다.

### 학습 내용

**Week 16: AI 컴파일러 기초**
- 전통 컴파일러 vs AI 컴파일러의 차이
- **Computation Graph:** 모델 → 연산 그래프 → 최적화 → 하드웨어 매핑
- Graph-level 최적화: Operator Fusion, Constant Folding, Layout Transform
- Kernel-level 최적화: Loop Tiling, Vectorization, Unrolling

**Week 17: MLIR (Multi-Level Intermediate Representation)**
- LLVM 생태계 안에서 MLIR의 위치
- **Dialect 시스템:** 다단계 추상화의 핵심
  - High-level: Torch-MLIR, TOSA, StableHLO
  - Mid-level: Linalg, SCF (Structured Control Flow), Affine
  - Low-level: LLVM IR dialect, vendor-specific dialects
- MLIR이 NPU 벤더 백엔드의 "공통 IR 레이어" 역할을 하는 구조
- 실습: Qualcomm Hexagon-MLIR (오픈소스)
  - Triton 커널 → Hexagon NPU 매핑 파이프라인
  - TCM(Tightly Coupled Memory) 활용, Double Buffering
- 실습: AMD MLIR-AIE (오픈소스)
  - AIR dialect → NPU spatial array 매핑
  - ARIES 프레임워크: Loop tiling → AIE core 분배

**Week 18: TVM / ONNX Runtime / 벤더 SDK**
- **Apache TVM:** Python-first, 범용 ML 컴파일러
  - Relay IR → TIR → Target code
  - AutoTVM / MetaSchedule: 자동 최적화 탐색
  - BYOC (Bring Your Own Codegen): NPU 백엔드 통합
- **ONNX Runtime:** 실무에서 가장 많이 쓰이는 추론 프레임워크
  - Execution Provider 아키텍처
  - AMD Ryzen AI: OGA(ONNX Runtime GenAI) 통합
- **벤더 SDK:**
  - Intel OpenVINO (NPU 지원 성숙)
  - Qualcomm AI Engine Direct
  - Rebellions RBLN SDK
  - Cerebras SDK (Model Zoo)

**Week 19: 양자화 엔진 + 모델 최적화**
- GPTQ, AWQ, SmoothQuant, GGUF 포맷
- **vLLM:** PagedAttention 기반 서빙 최적화
  - NPU에서 vLLM 적용 시 tok/s 2배, 전력 효율 92% 향상 사례
- KV-Cache 압축: INT8/INT4 KV, Sliding Window
- Speculative Decoding: 소형 모델 draft + 대형 모델 verify
- Continuous Batching: 동적 배치로 GPU/NPU 활용률 극대화

### 비주류 자원
1. **Stephen Diehl "MLIR Tutorial" 시리즈** (stephendiehl.com) — GPT-2를 GPU 커널로 컴파일하는 과정
2. **Xilinx/AMD MLIR-AIE GitHub** (github.com/Xilinx/mlir-aie) — Ryzen AI NPU 직접 프로그래밍
3. **"Hexagon-MLIR" 논문 (arXiv:2602.19762)** — Qualcomm NPU 컴파일 스택 상세
4. **"MLIR-AIR" 논문 (arXiv:2510.14871)** — Loop nest를 AMD NPU 실리콘에 매핑
5. **TVM Conference 발표 영상들** — 매년 최신 AI 컴파일러 트렌드
6. **llama.cpp 소스 코드** — CPU 추론 최적화의 실전 바이블

### 검증 과제
```
[과제 Ⅴ-1] MLIR-AIE 실습
- AMD MLIR-AIE GitHub를 clone하고 환경을 설정하라
  (Ubuntu + Ryzen AI 하드웨어 없이도 에뮬레이션 가능)
- 제공된 matrix multiplication 예제를 빌드하고 실행하라
- 타일 크기를 변경하며 compute efficiency 변화를 관찰하라

[과제 Ⅴ-2] 컴파일 파이프라인 도식화
- PyTorch 모델이 다음 NPU에서 실행되기까지의 전체 컴파일 경로를 
  도식화하라 (각 단계의 IR 변환 포함):
  (a) Google TPU (JAX/XLA → StableHLO → TPU 코드)
  (b) AMD Ryzen AI NPU (PyTorch → Torch-MLIR → MLIR-AIR → MLIR-AIE → binary)
  (c) Qualcomm Hexagon NPU (Triton → Hexagon-MLIR → binary)

[과제 Ⅴ-3] vLLM + NPU 최적화 분석
- vLLM의 PagedAttention 메커니즘을 설명하라
- RBLN-CA12 NPU에서 vLLM 적용 시 92% 전력 효율 향상이
  달성된 원리를 추론하라 (MDPI 논문 참조)

[검증 시뮬레이션 Ⅴ]
시나리오: "새로운 NPU 칩을 출시했는데, PyTorch 모델이 실행되지 않습니다."
→ 컴파일 스택의 어느 단계에서 문제가 발생할 수 있는지 
   계층별로 디버깅 체크리스트를 작성하라 (최소 10개 항목).
```

---

## 단계 Ⅵ: NPU 서버 시스템 설계 (4주)

### 핵심
칩은 하나의 부품일 뿐입니다. 서버는 칩 + 메모리 + 인터커넥트 + 전력 + 냉각의 종합 시스템입니다.

### 학습 내용

**Week 20: 서버 하드웨어 아키텍처**
- NPU 서버 폼팩터: 1U, 2U, 4U, 커스텀
- PCIe 슬롯 구성: x16 레인, Bifurcation
- 호스트 CPU 선택: x86 (Intel Xeon, AMD EPYC) vs Arm (Ampere, Grace)
  - SoftBank "Silicon Trinity": Arm + Ampere + Graphcore
- 메모리 구성: DDR5 채널, DIMM 배치, CXL 메모리 확장
- 스토리지: NVMe SSD, 모델 로딩 시간 최적화

**Week 21: 전력·냉각·물리 인프라**
- **전력 설계:**
  - Cerebras CS-3: 단일 시스템 ~20kW
  - GPU 서버 (8×H100): ~10kW
  - NPU 서버 전력 효율 우위: 35-70% 전력 절감
  - PDU, UPS, 전력 이중화 설계
- **냉각 설계:**
  - 공랭(Air Cooling) 한계: ~40kW/rack
  - 직접 액냉(Direct Liquid Cooling, DLC): ~100kW+ /rack
  - Rear Door Heat Exchanger
  - Immersion Cooling: Cerebras WSE 필수
- **데이터센터 레벨:**
  - PUE(Power Usage Effectiveness) 목표: < 1.2
  - Tier III vs Tier IV 가용성

**Week 22: 네트워크 토폴로지**
- **Inference 클러스터 네트워크:**
  - Tensor Parallelism: 노드 내 고대역폭 (NVLink/UCIe)
  - Pipeline Parallelism: 노드 간 (InfiniBand/RoCE)
- **서빙 네트워크:**
  - Load Balancer → API Gateway → NPU 서버 풀
  - 요청 라우팅: 모델별, 레이턴시별, 배치 크기별
- **Fat Tree vs Dragonfly vs Rail-Optimized 토폴로지**
- **CXL 네트워크:** 메모리 풀링을 통한 자원 활용률 향상

**Week 23: 멀티클라우드 + 온프레미스 하이브리드**
- 클라우드 NPU 서비스:
  - AWS Inferentia2 / Trainium2 (EC2 inf2, trn2 인스턴스)
  - Google Cloud TPU v6 (Trillium)
  - GroqCloud (API 서비스)
  - Cerebras Cloud (온디맨드)
  - SambaNova Cloud
- 온프레미스 vs 클라우드 의사결정 프레임워크
- 하이브리드 아키텍처: 트래픽 기반 오토스케일링
  - 베이스라인: 온프레미스 NPU 서버
  - 피크: 클라우드 NPU burst
- Kubernetes 기반 NPU 오케스트레이션
  - Device Plugin, Resource Quota, Topology-Aware Scheduling

### 비주류 자원
1. **Open Compute Project (OCP)** — Facebook/Meta 주도 오픈소스 서버 설계
2. **NVIDIA DGX SuperPOD Reference Architecture** — 대규모 AI 인프라 설계의 레퍼런스
3. **Uptime Institute Tier Standards** — 데이터센터 가용성 기준
4. **"Sustainable AI Infrastructure" (Google, 2024)** — 탄소 효율적 AI 인프라 설계
5. **Dell PowerEdge XE9680 / HPE ProLiant DL380a** 기술 문서 — 실제 AI 서버 사양

### 검증 과제
```
[과제 Ⅵ-1] NPU 서버 랙 설계
- 다음 요구사항으로 NPU 서버 랙을 설계하라:
  목표: LLaMA 70B 서빙, 1000 req/s, p99 레이턴시 < 2s
  제약: 전력 40kW/rack, 공랭 기반
  - 칩 선택 및 근거
  - 서버당 NPU 수, 호스트 CPU, 메모리 구성
  - 랙 내 서버 수, 네트워크 토폴로지
  - 예상 TCO (3년 기준)

[과제 Ⅵ-2] 냉각 비교 분석
- Cerebras CS-3 (액냉 필수, ~20kW/시스템) vs 
  Groq GroqRack (공랭 가능) vs 
  8×H100 GPU 서버 (DLC 권장)
  세 시스템의 냉각 요구사항을 비교하고,
  한국 IDC(인터넷데이터센터) 환경에서의 실현 가능성을 분석하라

[과제 Ⅵ-3] 하이브리드 아키텍처 설계
- 한국 AI 스타트업 시나리오:
  - 평상시: 100 req/s, 피크: 2000 req/s
  - 모델: 70B LLM (한국어 특화)
  - 예산: 초기 투자 30억원, 월 운영비 5억원 한도
  - 온프레미스 NPU + 클라우드 NPU 하이브리드를 설계하라
  - Kubernetes 기반 오토스케일링 전략을 포함하라

[검증 시뮬레이션 Ⅵ]
역할극: 데이터센터 운영팀과 미팅. 시설팀이 말합니다:
"기존 공랭 IDC에 NPU 서버를 넣고 싶은데, 
랙당 20kW를 넘기면 안 됩니다."
→ 이 제약 하에서 가능한 최대 추론 성능을 설계하고,
   향후 액냉 전환 시 마이그레이션 계획도 제시하라.
```

---

## 단계 Ⅶ: 운영·최적화·벤치마킹 (3주)

### 학습 내용

**Week 24: 프로파일링과 병목 분석**
- NPU 프로파일링 도구:
  - Chrome Tracing / Perfetto UI (MLIR-AIR 실행 추적)
  - NVIDIA Nsight (비교 기준)
  - 벤더별 프로파일러: Intel VTune, AMD uProf
- 핵심 메트릭:
  - **Compute Utilization (MFU):** 이론 대비 실제 FLOPS 활용률
  - **Memory Bandwidth Utilization (MBU):** HBM 대역폭 활용률
  - **TTFB (Time to First Byte):** 첫 토큰 레이턴시
  - **TPS (Tokens Per Second):** 처리량
  - **Inter-Token Latency (ITL):** 토큰 간 지연시간

**Week 25: 서빙 인프라 최적화**
- **vLLM / TensorRT-LLM / Triton Inference Server** 비교
- Continuous Batching 구현 전략
- Speculative Decoding 실전 적용
- Model Parallelism 전략:
  - Tensor Parallelism: 단일 레이어를 여러 디바이스에 분할
  - Pipeline Parallelism: 레이어 그룹을 디바이스에 분배
  - Expert Parallelism: MoE 모델 전용

**Week 26: TCO 분석과 벤치마킹**
- **TCO(Total Cost of Ownership) 프레임워크:**
  - 하드웨어 비용 (칩 + 서버 + 네트워크)
  - 전력 비용 (한국 산업용 전기요금 기준)
  - 냉각 비용 (PUE 반영)
  - 인건비 (운영 + 개발)
  - 소프트웨어 라이선스
- **벤치마킹 방법론:**
  - MLPerf Inference (표준 벤치마크)
  - ArtificialAnalysis.ai (독립 벤치마크)
  - 자체 워크로드 벤치마크 설계
- **$/token 계산:** 최종 비즈니스 의사결정 지표

### 비주류 자원
1. **MLPerf Inference 결과 데이터베이스** (mlcommons.org) — 유일한 표준 벤치마크
2. **ArtificialAnalysis.ai** — 독립적 LLM 추론 벤치마크 (Groq 241 tok/s 검증한 곳)
3. **"Efficient Memory Management for LLM Serving with PagedAttention" (Kwon et al.)** — vLLM 원논문
4. **Anyscale 블로그** — LLM 서빙 최적화 실전 가이드
5. **한국전력 산업용 전기요금 자료** — TCO 계산 필수 데이터

### 검증 과제
```
[과제 Ⅶ-1] TCO 스프레드시트
- 3년 기준 TCO를 비교하는 스프레드시트를 작성하라:
  (a) NVIDIA H100 8-GPU 서버 × 4대
  (b) Groq GroqRack (320 LPU 칩)
  (c) AWS Inferentia2 (inf2.48xlarge × N대)
  (d) Rebellions REBEL-Quad × 8대 (예상 가격 기반)
  - 워크로드: LLaMA 70B, 평균 500 req/s
  - 한국 전기요금 적용 (약 120원/kWh 산업용)
  - $/million tokens 산출

[과제 Ⅶ-2] 벤치마크 설계
- 다음을 측정하는 벤치마크 스위트를 설계하라:
  (1) Single-stream latency (p50, p90, p99)
  (2) Multi-stream throughput (tok/s at varying batch sizes)
  (3) Power efficiency (tok/s/W)
  (4) Long-context performance (4K, 16K, 64K, 128K tokens)
  - 각 테스트의 목적, 방법론, 필요 도구를 명시하라

[검증 시뮬레이션 Ⅶ]
시나리오: "벤치마크 결과, 우리 NPU 서버가 NVIDIA 대비 
처리량은 80%지만 전력 효율은 200%입니다."
→ 이 결과를 경영진에게 보고하는 1페이지 임원 보고서를 작성하라.
   기술적 정확성과 비즈니스 임팩트를 모두 포함하라.
```

---

## 단계 Ⅷ: 마스터 프로젝트 (4주)

### 종합 프로젝트: "차세대 NPU 추론 인프라 설계서"

다음 요구사항을 충족하는 **20페이지 이상의 기술 설계서**를 작성하라:

```
[마스터 프로젝트 요구사항]

배경: 한국의 중견 AI 기업이 자체 LLM 추론 인프라를 구축하려 합니다.
- 모델: 70B 한국어 LLM + 13B 코드 생성 모델 + Vision-Language 모델
- 트래픽: 평균 2,000 req/s, 피크 10,000 req/s
- SLA: p99 레이턴시 < 3초 (70B), < 1초 (13B)
- 예산: 하드웨어 초기 투자 100억원, 연간 운영비 50억원
- 제약: 한국 IDC, 공랭+부분 액냉, 정부 보안 인증 필요

설계서에 반드시 포함할 내용:

1. 하드웨어 아키텍처
   - NPU 칩 선택 및 상세 근거 (최소 3개 후보 비교)
   - 서버 구성 (호스트 CPU, 메모리, 스토리지, 네트워크)
   - 랙 레이아웃 및 물리 인프라

2. 소프트웨어 스택
   - 컴파일러/런타임 선택
   - 서빙 프레임워크 (vLLM/TRT-LLM/커스텀)
   - 모니터링 + 오브저버빌리티

3. 네트워크 아키텍처
   - 클러스터 내부 네트워크 토폴로지
   - 외부 서빙 네트워크
   - 로드 밸런싱 전략

4. 운영 전략
   - Kubernetes 오케스트레이션
   - 오토스케일링 (온프레미스 + 클라우드 버스트)
   - 모델 업데이트/롤아웃 전략
   - 장애 대응 (HA/DR)

5. 경제성 분석
   - 3년 TCO 상세 분석
   - $/million tokens 비교
   - ROI 시나리오 (베스트/워스트/기본)

6. 마이그레이션 로드맵
   - Phase 1: 초기 구축 (6개월)
   - Phase 2: 최적화 (6개월)
   - Phase 3: 확장 (1년)

7. 리스크 분석
   - 기술 리스크 (칩 공급, SDK 성숙도)
   - 운영 리스크 (인력, 장애)
   - 비즈니스 리스크 (시장 변화, 경쟁)
```

---

## 지름길 모음 (Shortcuts)

### 🔑 80/20 규칙 적용
1. **GEMM + Attention만 이해하면 AI 워크로드의 90%를 커버** — Convolution도 결국 GEMM
2. **Roofline 모델 하나면 모든 하드웨어를 30초 안에 분석 가능**
3. **MLIR을 배우면 모든 NPU 벤더의 컴파일러 스택을 이해** — 공통 레이어이므로
4. **vLLM 소스 코드를 읽으면 서빙 최적화의 80%를 습득**
5. **MLPerf 결과만 읽어도 시장 동향 파악 가능** — 벤더 마케팅 필터링

### 🚀 빠른 성장을 위한 커뮤니티
- **LLVM/MLIR Discord** — 컴파일러 질문 실시간 답변
- **r/LocalLLaMA (Reddit)** — 하드웨어 최적화 실전 경험담
- **Hot Chips 학회 (매년 8월)** — 칩 아키텍처 최신 발표
- **AI Hardware Summit** — 업계 네트워킹 + 최신 트렌드
- **한국 AI 반도체 포럼 / KAIST AI 세미나** — 국내 네트워크

### ⚡ 클라우드 아키텍트에서의 전환 지름길
Rainny의 기존 역량을 최대한 활용하는 전략:
1. **AWS Inferentia2/Trainium2부터 시작** → 이미 익숙한 AWS 생태계에서 NPU 경험
2. **Terraform으로 inf2 인스턴스 프로비저닝** → 멀티클라우드 NPU 비교 자동화
3. **Bedrock이 내부적으로 어떤 칩을 쓰는지 역추적** → 서비스 뒤의 하드웨어 이해
4. **CDK로 NPU 서빙 파이프라인 구축** → 기존 IaC 역량 + NPU 운영 경험 통합

---

## 참고 문헌 및 핵심 자원 종합

### 논문 (필독)
1. Jouppi et al. "In-Datacenter Performance Analysis of a Tensor Processing Unit" (2017) — TPU 원논문
2. Kwon et al. "Efficient Memory Management for LLM Serving with PagedAttention" (2023) — vLLM
3. Dao et al. "FlashAttention: Fast and Memory-Efficient Exact Attention" (2022)
4. Pope et al. "Efficiently Scaling Transformer Inference" (2022)
5. MDPI "Performance and Efficiency Gains of NPU-Based Servers over GPUs" (2025)
6. "From Loop Nests to Silicon: Mapping AI Workloads onto AMD NPUs with MLIR-AIR" (2025)
7. "Hexagon-MLIR: An AI Compilation Stack For Qualcomm's NPUs" (2026)

### 서적
1. Hennessy & Patterson, "Computer Architecture: A Quantitative Approach" 6th ed.
2. Sze et al. "Efficient Processing of Deep Neural Networks" (2020) — NPU 설계 교과서
3. "Primer on Memory Consistency and Cache Coherence" 2nd ed. (무료 PDF)

### 온라인 강의
1. Onur Mutlu (ETH Zürich) — Computer Architecture 전체 과정 (YouTube)
2. Stanford CS149 — Parallel Computing
3. MIT 6.5940 — EfficientML.ai (Song Han)
4. nand2tetris — 하드웨어 기초

### 오픈소스 프로젝트 (실습용)
1. github.com/Xilinx/mlir-aie — AMD NPU 프로그래밍
2. github.com/llvm/llvm-project (MLIR) — 컴파일러 인프라
3. github.com/vllm-project/vllm — LLM 서빙
4. github.com/ggerganov/llama.cpp — CPU/NPU 추론
5. github.com/NVIDIA/cutlass — GEMM 커널 최적화

---

> **마지막 조언:** NPU 서버 분야는 2026년 현재 급격히 재편되고 있습니다.
> NVIDIA의 Groq 인수($20B), OpenAI의 자체 칩("Titan"), 
> Cerebras의 OpenAI와 750MW 추론 계약,
> 한국의 Rebellions/FuriosaAI의 부상까지 —
> 지금이 이 분야에 진입하기에 최적의 시점입니다.
> 하드웨어와 소프트웨어 양쪽을 모두 이해하는 "풀스택 NPU 엔지니어"는
> 전 세계적으로 극소수이며, 그 희소성이 당신의 무기가 될 것입니다.
