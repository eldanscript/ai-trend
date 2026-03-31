# NPU 서버 마스터 30일 속성 플랜

## JUST DO IT — 2026.03.30 (월) ~ 2026.04.28 (화)

> **원칙:** 매일 아침 Claude와 해당 날짜의 프롬프트로 세션을 시작하고, 과제를 수행하고, 저녁에 검증 프롬프트로 마무리한다.
> **하루 투입:** 최소 4시간 (이론 2h + 실습/과제 2h)
> **도구:** Claude (이 문서 + 프롬프트), 터미널 (Python/Verilog), 브라우저 (논문/영상)

---

## 전체 캘린더 (30일)

| 주차 | 날짜 | 단계 | 주제 | 유형 |
|------|------|------|------|------|
| **W1** | 3/30 월 | Ⅰ-1 | MOSFET, CMOS, 공정 노드 | 이론 |
| | 3/31 화 | Ⅰ-2 | MAC 유닛, 디지털 로직 | 이론+실습 |
| | 4/1 수 | Ⅰ-3 | VLSI 흐름, FPGA vs ASIC, 칩렛 | 이론 |
| | 4/2 목 | Ⅱ-1 | CPU/GPU 아키텍처 복습 | 이론 |
| | 4/3 금 | Ⅱ-2 | 메모리 계층: SRAM, DRAM, HBM | 이론 |
| | 4/4 토 | Ⅱ-3 | Roofline 모델 + 인터커넥트 | 이론+실습 |
| | 4/5 일 | Ⅱ-4 | 이종 컴퓨팅 + **W1 종합 검증** | 실습+시험 |
| **W2** | 4/6 월 | Ⅲ-1 | GEMM 해부 + Tiling | 이론+실습 |
| | 4/7 화 | Ⅲ-2 | Attention, FlashAttention, KV-Cache | 이론 |
| | 4/8 수 | Ⅲ-3 | LLM 추론 파이프라인 (Prefill/Decode) | 이론+실습 |
| | 4/9 목 | Ⅲ-4 | 양자화: FP16→INT4, MoE | 이론 |
| | 4/10 금 | Ⅳ-1 | NPU 핵심 패턴: Systolic Array | 이론 |
| | 4/11 토 | Ⅳ-2 | Dataflow + SRAM-Only 아키텍처 | 이론 |
| | 4/12 일 | Ⅳ-3 | Wafer-Scale + **W2 종합 검증** | 이론+시험 |
| **W3** | 4/13 월 | Ⅳ-4 | Programmable NPU (Rebellions, FuriosaAI) | 이론 |
| | 4/14 화 | Ⅳ-5 | 주요 NPU 비교 매트릭스 작성 | 실습 |
| | 4/15 수 | Ⅳ-6 | 한국 NPU 생태계 + 시장 분석 | 이론+실습 |
| | 4/16 목 | Ⅴ-1 | AI 컴파일러 기초 + Computation Graph | 이론 |
| | 4/17 금 | Ⅴ-2 | MLIR 심층: Dialect, Pass, Lowering | 이론+실습 |
| | 4/18 토 | Ⅴ-3 | TVM, ONNX RT, 벤더 SDK | 이론 |
| | 4/19 일 | Ⅴ-4 | vLLM + 양자화 엔진 + **W3 종합 검증** | 실습+시험 |
| **W4** | 4/20 월 | Ⅵ-1 | 서버 하드웨어 아키텍처 | 이론 |
| | 4/21 화 | Ⅵ-2 | 전력·냉각·물리 인프라 | 이론 |
| | 4/22 수 | Ⅵ-3 | 네트워크 토폴로지 + 클러스터 설계 | 이론+실습 |
| | 4/23 목 | Ⅵ-4 | 멀티클라우드 + 하이브리드 NPU 인프라 | 이론+실습 |
| | 4/24 금 | Ⅶ-1 | 프로파일링 + 벤치마킹 | 이론+실습 |
| | 4/25 토 | Ⅶ-2 | TCO 분석 + 서빙 최적화 | 실습 |
| | 4/26 일 | Ⅶ-3 | **W4 종합 검증** | 시험 |
| | 4/27 월 | Ⅷ-1 | 마스터 프로젝트 (설계 + 작성) | 프로젝트 |
| | 4/28 화 | Ⅷ-2 | 마스터 프로젝트 (완성 + 최종 리뷰) | 프로젝트 |

---

## 일일 학습 구조

매일 동일한 리듬으로 진행합니다:

```
[08:00-08:10] 🌅 시작 프롬프트 → Claude와 세션 오픈
[08:10-09:30] 📖 이론 학습 — Claude 대화 + 추천 자원 읽기
[09:30-10:00] ☕ 정리 — 핵심 개념 3줄 요약 직접 작성
[10:00-11:30] 🔧 실습/과제 — 코드 작성, 분석, 리포트
[11:30-12:00] 🧪 검증 프롬프트 — Claude에게 시험 받기
```

---

## W1: 하드웨어 기초 (3/30 월 ~ 4/5 일)

---

### DAY 01 — 3/30 (월): MOSFET, CMOS, 공정 노드

**오늘의 목표:** 트랜지스터가 어떻게 논리 연산을 수행하는지, 공정 노드가 성능/전력에 어떤 영향을 주는지 이해한다.

**학습 자원 (순서대로):**
1. 영상: "How Do Transistors Work?" — Veritasium (YouTube, 15분)
2. 영상: "How are Microchips Made?" — Branch Education (YouTube, 20분)
3. 읽기: Ken Shirriff 블로그 — "Inside the Intel 4004" (righto.com)
4. 읽기: WikiChip — TSMC N4 / Samsung 4LPP 공정 비교 페이지

**시작 프롬프트 (아침):**
```
나는 AI용 NPU 서버 30일 마스터 과정 DAY 01을 진행 중이다.
오늘 주제: MOSFET, CMOS 로직, 반도체 공정 노드

다음을 단계별로 가르쳐 달라:
1. MOSFET(N-type, P-type)의 동작 원리를 물 흐름 비유로 설명
2. CMOS 인버터가 어떻게 NAND 게이트가 되는지
3. 반도체 공정 노드(4nm, 5nm, 7nm)가 실제로 뜻하는 것
   — "4nm이 정말 4나노미터인가?"
4. 누설 전류(Leakage)와 동적 전력(Dynamic Power)의 관계
5. NPU 설계에서 공정 선택이 중요한 이유
   — Rebellions(삼성 4nm) vs Groq(삼성 14nm) 사례로 설명

각 개념마다 "이것을 모르면 NPU를 이해할 때 어디서 막히는가"를
함께 설명해 달라. 도표나 비유를 적극 사용해라.
```

**실습 과제:**
```
[과제 DAY01] Python으로 CMOS 인버터 시뮬레이션
- NMOS와 PMOS의 전압-전류 특성을 간단한 모델로 구현
- 입력 0V → 출력 Vdd, 입력 Vdd → 출력 0V 확인
- matplotlib으로 전달 특성 곡선(VTC) 플로팅
```

**검증 프롬프트 (저녁):**
```
나는 NPU 30일 과정 DAY 01을 마쳤다. 내 이해를 검증해 달라.

다음 질문에 답하겠다. 각 답변 후 정확도를 채점하고 보완 설명을 해 달라:

Q1: TSMC 4nm과 삼성 4nm은 같은 성능인가? 왜?
Q2: Groq가 14nm 구형 공정을 선택한 합리적 이유 3가지는?
Q3: NPU의 MAC 유닛 수를 2배로 늘리면 전력은 어떻게 변하나?
Q4: 누설 전류가 NPU 서버 설계에서 중요한 이유는?
Q5: "공정 미세화 한계"가 NPU 산업에 미치는 영향을 설명하라.

채점 기준: 각 10점, 총 50점. 40점 이상이면 PASS.
```

---

### DAY 02 — 3/31 (화): MAC 유닛과 디지털 로직

**오늘의 목표:** NPU의 핵심 빌딩 블록인 MAC(Multiply-Accumulate) 유닛을 설계 수준으로 이해한다.

**학습 자원:**
1. nand2tetris — Chapter 1-2 (Boolean Logic, Boolean Arithmetic)
2. 영상: "Binary Multiplication in Hardware" — Ben Eater (YouTube)
3. 읽기: Google TPU v1 논문 Section 3 "Architecture" (systolic array 부분)

**시작 프롬프트:**
```
NPU 30일 과정 DAY 02. 주제: MAC 유닛과 디지털 로직

다음을 가르쳐 달라:
1. 이진수 곱셈기(Binary Multiplier)의 하드웨어 구현
   — 부분곱(Partial Product) + 가산기 트리
2. MAC(Multiply-Accumulate) 연산이란 무엇인가
   — a = a + (b × c)가 왜 AI의 기본 연산인가
3. INT8 MAC vs FP16 MAC의 하드웨어 복잡도 차이
   — 면적, 전력, 속도 측면
4. MAC 유닛을 배열로 구성하는 방법
   — 1D 배열 vs 2D 배열(Systolic Array 맛보기)
5. TOPS(Tera Operations Per Second) 계산법
   — MAC 256개, 클록 1GHz이면 이론적 TOPS는?

Python 코드 예시를 포함해서 8-bit MAC 유닛의 동작을 보여 달라.
```

**실습 과제:**
```
[과제 DAY02] 8-bit MAC 유닛 및 배열 시뮬레이션

1. Python으로 8-bit integer MAC 유닛을 구현하라
   - multiply_accumulate(a, b, accumulator) → new_accumulator
   - 오버플로 처리 포함

2. 16×16 MAC 배열을 구현하고 4×4 행렬곱을 수행하라
   - numpy 결과와 비교하여 정확성 검증

3. 다음을 계산하라:
   - MAC 배열 256개 × 2 (곱+덧), 클록 1.5GHz = ? TOPS
   - 이 스펙이 실제 어떤 NPU에 가까운지 조사하라

코드는 Claude에게 리뷰 받을 것.
```

**검증 프롬프트:**
```
NPU 30일 DAY 02 검증.

내가 작성한 MAC 배열 코드를 리뷰하고 다음을 평가해 달라:
1. 코드의 정확성 (행렬곱 결과가 맞는가)
2. INT8 오버플로 처리가 적절한가
3. TOPS 계산이 정확한가

추가 질문:
Q1: FP16 MAC이 INT8 MAC보다 약 4배 큰 면적을 차지하는 이유는?
Q2: NPU가 INT8을 선호하는 이유와 정확도 트레이드오프는?
Q3: "TOPS는 마케팅 수치"라는 말의 의미는?
   실제 성능과 TOPS가 괴리되는 사례 3가지를 들어라.
```

---

### DAY 03 — 4/1 (수): VLSI 흐름, FPGA vs ASIC, 칩렛

**오늘의 목표:** 칩이 만들어지는 전체 흐름과, NPU 산업의 새 패러다임인 칩렛 아키텍처를 이해한다.

**학습 자원:**
1. ZeroToASIC (zerotoasic.com) — "ASIC Design Flow" 페이지
2. 읽기: UCIe Specification Overview (공식 사이트)
3. 읽기: Rebellions의 UCIe 칩렛 전략 (EE Times 기사)
4. 영상: "What are Chiplets?" — Asianometry (YouTube)

**시작 프롬프트:**
```
NPU 30일 과정 DAY 03. 주제: VLSI 설계 흐름, FPGA vs ASIC, 칩렛

다음을 가르쳐 달라:
1. ASIC 설계 흐름 전체를 한 장의 파이프라인으로
   RTL → 합성 → 배치배선 → 검증 → 테이프아웃 → 패키징
   각 단계를 1문장으로 설명

2. FPGA vs ASIC
   - FPGA: 프로그래머블, 소량 생산, 프로토타이핑
   - ASIC: 고정 기능, 대량 생산, 최고 성능/효율
   - NPU는 왜 ASIC으로 만드는가? (AMD XDNA는 왜 FPGA 기반?)

3. 칩렛(Chiplet) 아키텍처
   - 모놀리식 다이 vs 칩렛 방식
   - UCIe 표준이 NPU 산업에 미치는 영향
   - Rebellions REBEL-Quad: 4개 다이를 UCIe로 연결
   - 수율(Yield) 문제 해결과 확장성

4. 패키징 기술
   - 2.5D (인터포저), 3D 스태킹, CoWoS
   - HBM이 칩과 어떻게 물리적으로 연결되는가
   - Cerebras WSE의 극단적 패키징

각 개념이 "NPU 서버 아키텍트가 왜 이걸 알아야 하는가"와 연결해서 설명해 달라.
```

**실습 과제:**
```
[과제 DAY03] NPU 칩 설계 의사결정 분석

다음 시나리오에 대해 1페이지 분석 리포트를 작성하라:

"당신이 NPU 스타트업의 CTO입니다. 첫 번째 칩을 설계해야 합니다."

결정해야 할 사항:
1. 공정 노드 선택: TSMC 4nm vs 삼성 4nm vs TSMC 7nm
   → 각각의 비용, 성능, 납기 트레이드오프
2. 모놀리식 vs 칩렛: 첫 제품에서 어떤 전략이 유리한가?
3. FPGA 프로토타이핑을 먼저 할 것인가, 바로 ASIC으로 갈 것인가?

Rebellions과 FuriosaAI의 실제 선택을 참고하여 결정을 정당화하라.
```

**검증 프롬프트:**
```
NPU 30일 DAY 03 검증.

Q1: REBEL-Quad가 UCIe를 사용하는 이유와 
    모놀리식 대비 장단점을 3가지씩 들어라
Q2: Cerebras WSE-3은 "칩렛의 극단적 반대편"이다. 
    이 접근의 장점과 치명적 단점은?
Q3: TSMC CoWoS 패키징이 병목이 되는 이유와
    이것이 NPU 공급에 미치는 영향은?
Q4: 한국에서 NPU를 설계할 때 삼성 파운드리 vs TSMC 선택의
    지정학적 리스크를 분석하라.

각 10점, 40점 이상 PASS.
```

---

### DAY 04 — 4/2 (목): CPU/GPU 아키텍처 복습

**시작 프롬프트:**
```
NPU 30일 과정 DAY 04. 주제: CPU와 GPU 아키텍처 복습

나는 클라우드 아키텍트로서 CPU/GPU를 "사용자"로 알고 있지만,
"설계자" 관점에서 재학습이 필요하다.

다음을 가르쳐 달라:
1. CPU 아키텍처의 핵심 설계 원칙
   - 파이프라인, 비순차 실행(OoO), 분기 예측, 캐시 계층
   - "왜 CPU는 AI에 느린가?"를 아키텍처 관점에서

2. GPU 아키텍처 (NVIDIA 중심)
   - SM(Streaming Multiprocessor) 구조
   - Warp, Thread Block, Grid 계층
   - Tensor Core의 역할과 한계
   - CUDA 프로그래밍 모델이 하드웨어에 매핑되는 방식

3. 병렬화 모델 비교
   - SIMD (CPU 벡터 명령어)
   - SIMT (GPU)
   - Spatial (NPU — 맛보기)
   - 각 모델이 행렬곱에 어떻게 매핑되는가

4. GPU의 한계 = NPU의 존재 이유
   - GPU는 "범용 병렬 프로세서"이지 "AI 전용"이 아니다
   - 에너지 효율, 레이턴시 예측성, 비용 측면

CPU와 GPU의 블록 다이어그램을 텍스트로 그려서 비교해 달라.
```

**실습 과제:**
```
[과제 DAY04] CPU vs GPU vs NPU 연산 시뮬레이션

Python으로 다음을 구현하고 비교하라:

1. 512×512 행렬곱을 세 가지 방식으로 시뮬레이션:
   (a) Sequential (CPU 시뮬레이션) — 3중 for 루프
   (b) Parallel (GPU 시뮬레이션) — NumPy (내부적으로 BLAS)
   (c) Systolic (NPU 시뮬레이션) — 타일링 + 순차 데이터 흐름

2. 각 방식의 실행 시간 측정 및 비교

3. "연산당 메모리 접근 횟수"를 각 방식에서 추정하라
   → 이것이 Operational Intensity 개념으로 연결됨
```

**검증 프롬프트:**
```
NPU 30일 DAY 04 검증.

Q1: NVIDIA H100의 Tensor Core가 하는 일을 
    "4×4 행렬곱을 1클록에" 이상으로 상세히 설명하라
Q2: GPU의 Warp Divergence가 AI 워크로드에서 문제가 되는 경우는?
Q3: "GPU는 범용이라 비효율적"이라는 NPU 진영의 주장을
    반박하는 논거 2가지와 지지하는 논거 2가지를 들어라
Q4: NVIDIA가 Groq를 $20B에 인수한 전략적 의미를
    하드웨어 아키텍처 관점에서 분석하라.
```

---

### DAY 05 — 4/3 (금): 메모리 계층 — SRAM, DRAM, HBM

**시작 프롬프트:**
```
NPU 30일 과정 DAY 05. 주제: 메모리 계층 심층 분석

"NPU 성능의 80%는 메모리가 결정한다"는 명제를 증명해 달라.

다음을 가르쳐 달라:
1. 메모리 계층 전체 지도
   - 레지스터 (< 1ns) → SRAM (1-2ns) → DRAM (50-100ns)
   - 각 계층의 용량, 대역폭, 에너지/접근 비교 표

2. HBM (High Bandwidth Memory) 심층
   - HBM2e → HBM3 → HBM3E → HBM4 진화 로드맵
   - HBM의 물리적 구조: TSV(Through-Silicon Via), 스택
   - SK Hynix의 HBM4 2026 양산이 NPU 생태계에 미치는 영향
   - HBM 대역폭 계산법: 채널 수 × 채널 대역폭

3. SRAM-Only 접근법 (Groq)
   - 230MB SRAM으로 LLM을 어떻게 서빙하는가?
   - 모델 파티셔닝과 칩 간 통신
   - 결정론적 실행의 의미와 가치

4. Memory Wall 문제
   - "연산 속도는 매년 2배 증가하지만 메모리 대역폭은 1.3배"
   - 이 문제에 대한 각 NPU 벤더의 해결 접근법:
     (a) Cerebras: 44GB on-die SRAM, 21 PB/s 내부 대역폭
     (b) SambaNova: 3-tier 메모리 아키텍처
     (c) d-Matrix: In-Memory Computing
     (d) Qualcomm AI250: Near-Memory Computing (2027)

5. LLM에서 메모리가 병목인 구체적 증거
   - Decode 단계: 모델 전체 가중치를 매 토큰마다 로드
   - Mobile NPU (50-90 GB/s) vs GPU (2-3 TB/s) = 30-50배 격차
```

**실습 과제:**
```
[과제 DAY05] 메모리 대역폭 계산기

Python 스크립트를 작성하라:

입력: 모델 파라미터 수, 양자화 비트, 목표 tok/s
출력: 필요 메모리 대역폭 (GB/s)

예시 계산:
- LLaMA 70B, INT4 = 35GB 모델 크기
- 100 tok/s 목표 → 매 토큰마다 전체 가중치 로드
- 필요 대역폭 = 35GB × 100 = 3,500 GB/s = 3.5 TB/s
- HBM3E 1스택 (~1.2 TB/s) → 최소 3스택 필요

이 계산기로 다음을 분석:
(a) 7B FP16 모델, 50 tok/s 목표
(b) 70B INT4 모델, 30 tok/s 목표  
(c) 405B INT4 모델, 10 tok/s 목표
각각에 필요한 HBM 스택 수를 산출하라.
```

**검증 프롬프트:**
```
NPU 30일 DAY 05 검증.

Q1: HBM3E와 HBM4의 핵심 차이 3가지는?
Q2: Groq가 HBM 없이 SRAM만 쓰는데, 70B 모델을 서빙할 수 있는가?
    가능하다면 어떻게? 불가능하다면 왜?
Q3: "Memory Wall"을 해결하는 가장 유망한 접근법은 무엇이고 왜인가?
Q4: 내가 작성한 메모리 대역폭 계산기의 결과를 검증해 달라.
    계산에 빠진 요소(KV-cache 등)가 있다면 지적해 달라.

각 10점, 40점 이상 PASS.
```

---

### DAY 06 — 4/4 (토): Roofline 모델 + 인터커넥트

**시작 프롬프트:**
```
NPU 30일 과정 DAY 06. 주제: Roofline 모델과 인터커넥트

Roofline 모델은 "모든 하드웨어를 30초 만에 분석하는 도구"이다.
이것을 완벽히 마스터시켜 달라.

1. Roofline 모델 완전 해설
   - X축: Operational Intensity (FLOPs/Byte)
   - Y축: 달성 가능한 성능 (FLOPS)
   - "지붕선"의 의미: 수평선(compute-bound) + 경사선(memory-bound)
   - 어떤 워크로드가 어디에 위치하는가:
     GEMM (높은 OI) vs Attention (낮은 OI) vs Embedding (매우 낮은 OI)

2. 실전 Roofline 분석
   - A100: 312 TFLOPS FP16, 2TB/s HBM 대역폭
     → 전환점 OI = 312T / 2T = 156 FLOPs/Byte
   - Groq LPU: 750 TOPS INT8, ~80 TB/s SRAM 대역폭
     → 전환점 OI = ?
   - 이 차이가 의미하는 것은?

3. 인터커넥트 기술 비교
   - 칩 내부: On-die mesh, ring bus
   - 칩 간(Die-to-Die): UCIe, NVLink, BoW
   - 서버 간: InfiniBand (400Gbps), RoCE, Ethernet
   - 랙 간: Fat Tree vs Dragonfly 토폴로지
   
4. CXL (Compute Express Link) 심층
   - CXL 3.0: 메모리 풀링, 메모리 확장
   - NPU 서버에서 CXL의 잠재적 역할
   - 한국 기업(Samsung, SK Hynix)의 CXL 전략

Roofline 그래프를 텍스트 아트로 그려서 보여 달라.
```

**실습 과제:**
```
[과제 DAY06] Roofline 모델 시각화 도구

Python + matplotlib로 Roofline 모델 그래프 생성기를 만들어라:

1. 입력: peak FLOPS, peak 메모리 대역폭
2. 출력: Roofline 그래프 + 워크로드 포인트 표시

다음 5개 플랫폼의 Roofline을 하나의 그래프에 그려라:
(a) NVIDIA A100 (312 TFLOPS, 2 TB/s)
(b) NVIDIA H100 (990 TFLOPS, 3.35 TB/s)
(c) Groq LPU (추정: 750 TOPS INT8, 80 TB/s on-die)
(d) Google TPU v5e (추정: 400 TFLOPS, 1.6 TB/s)
(e) Mobile NPU (50 TOPS, 70 GB/s)

그 위에 다음 워크로드를 점으로 표시:
- LLM Decode (OI ≈ 1)
- LLM Prefill (OI ≈ 100-200)  
- CNN Inference (OI ≈ 50-100)
- Embedding Lookup (OI ≈ 0.25)
```

**검증 프롬프트:**
```
NPU 30일 DAY 06 검증.

Q1: LLM Decode가 거의 모든 하드웨어에서 memory-bound인 이유를
    Roofline 모델로 정량적으로 증명하라.
Q2: Groq LPU의 Roofline이 A100과 질적으로 다른 이유는?
    SRAM 대역폭이 핵심인 이유를 설명하라.
Q3: InfiniBand 400Gbps와 PCIe Gen5 x16의 대역폭을 비교하고,
    NPU 서버 클러스터에서 어떤 상황에 각각이 필요한지 설명하라.
Q4: CXL 메모리 풀링이 NPU 서버에 적용되면 
    어떤 워크로드에서 가장 큰 이점이 있을까?

내가 만든 Roofline 그래프를 리뷰해 달라.
```

---

### DAY 07 — 4/5 (일): 이종 컴퓨팅 + W1 종합 검증

**시작 프롬프트:**
```
NPU 30일 과정 DAY 07. 주제: 이종 컴퓨팅 + W1 종합 복습

오전: 이종 컴퓨팅 학습
1. CPU + GPU + NPU 협업 모델
   - 호스트(CPU)가 디바이스(NPU)에 작업을 위임하는 흐름
   - DMA 엔진의 역할: CPU 개입 없이 메모리 전송
   - Zero-Copy vs Staged Transfer
   
2. NUMA와 NPU 서버
   - 다중 소켓 서버에서 NPU 배치 전략
   - PCIe 토폴로지가 NPU 성능에 미치는 영향
   - NVIDIA의 NVSwitch vs NPU 진영의 접근법

오후: W1 전체 종합 검증 시험을 출제해 달라.
- 범위: DAY 01 ~ DAY 07 전체
- 형식: 객관식 5문제 + 서술형 3문제 + 실전 시나리오 2문제
- 난이도: 중상 (개념 이해 + 응용)
- 채점 기준 포함
- 70점 이상 PASS, 미달 시 보충 학습 범위 지정
```

---

## W2: AI 워크로드 + NPU 아키텍처 (4/6 월 ~ 4/12 일)

---

### DAY 08 — 4/6 (월): GEMM 해부 + Tiling

**시작 프롬프트:**
```
NPU 30일 과정 DAY 08. 주제: GEMM(General Matrix Multiply) 심층 해부

GEMM은 AI 컴퓨팅의 80%다. 이것을 뼛속까지 이해시켜 달라.

1. GEMM의 정의와 AI에서의 위치
   - C = α·A·B + β·C (BLAS Level 3)
   - Transformer의 모든 Linear Layer = GEMM
   - Attention = Batched GEMM
   - Convolution = im2col → GEMM 변환

2. GEMM 최적화의 핵심: 데이터 재사용
   - Naive 3중 루프: O(N³) 연산, O(N²) 메모리 접근
   - 타일링(Tiling): 데이터를 캐시에 맞는 블록으로 분할
   - 타일 크기와 캐시 크기의 관계
   - NPU에서 타일 = Systolic Array의 한 "펌프" 분량

3. GEMM과 하드웨어의 매핑
   - CPU: BLAS 라이브러리 (MKL, OpenBLAS)
   - GPU: Tensor Core (4×4, 8×8, 16×16 타일)
   - NPU: Systolic Array (128×128 또는 256×256)
   - 왜 타일 크기가 다른가? → 하드웨어 자원과의 매칭

4. NVIDIA CUTLASS와 Triton
   - GEMM 커널을 직접 최적화하는 도구들
   - NPU 컴파일러가 내부적으로 하는 일과의 유사성

코드 예시를 단계별로 보여주며 naive → tiled → optimized 진화를 보여달라.
```

**실습 과제:**
```
[과제 DAY08] Tiled GEMM 구현 및 성능 분석

Python으로 구현:
1. naive_gemm(A, B): 3중 for 루프
2. tiled_gemm(A, B, tile_size): 타일링 적용
3. numpy_gemm(A, B): np.matmul (BLAS)

행렬 크기: 256, 512, 1024, 2048
타일 크기: 16, 32, 64, 128, 256

측정 및 분석:
- 각 조합의 실행 시간
- 타일 크기별 성능 변화 그래프
- "최적 타일 크기"가 CPU L1/L2 캐시 크기와 일치하는지 확인
- 이것이 NPU의 SRAM 크기 결정에 주는 시사점
```

**검증 프롬프트:**
```
NPU 30일 DAY 08 검증.

내 tiled GEMM 코드와 결과를 리뷰해 달라.

Q1: 1024×1024 FP16 GEMM의 총 FLOPs와 
    필요 메모리 접근량(bytes)을 계산하라. OI는?
Q2: NPU의 systolic array 크기가 128×128이면
    2048×2048 GEMM을 처리하는 데 몇 "타일 스텝"이 필요한가?
Q3: Convolution이 GEMM으로 변환되는 im2col 과정을 
    3×3 커널, 4×4 입력 예시로 보여 달라.
```

---

### DAY 09 — 4/7 (화): Attention, FlashAttention, KV-Cache

**시작 프롬프트:**
```
NPU 30일 과정 DAY 09. 주제: Attention 메커니즘 하드웨어 관점

Attention은 GEMM 다음으로 NPU가 가속하는 핵심 연산이다.
하드웨어 엔지니어의 눈으로 해부해 달라.

1. Self-Attention의 수학
   - Q, K, V 행렬 생성 (Linear projection = GEMM)
   - Attention Score = Q × K^T / √d_k
   - Attention Weight = Softmax(Score)
   - Output = Weight × V
   - 총 연산량: 시퀀스 길이 N에 대해 O(N²d)

2. Multi-Head Attention → GQA → MQA
   - MHA: H개의 독립적 어텐션 헤드
   - GQA (Grouped Query Attention): K,V 공유 → 메모리 절약
   - MQA: 극단적 K,V 공유
   - 각각의 메모리 요구량 차이 계산

3. KV-Cache: 추론의 핵심 메모리 소비자
   - 왜 필요한가: 이전 토큰의 K,V를 재계산하지 않기 위해
   - 크기 계산: layers × heads × seq_len × d_head × 2(K+V) × dtype_size
   - 70B 모델, 4K 컨텍스트 FP16 KV-Cache = ? GB
   - KV-Cache가 NPU의 SRAM/HBM을 얼마나 차지하는가

4. FlashAttention의 핵심
   - 문제: Standard Attention은 N×N 행렬을 HBM에 저장
   - 해결: Tiling으로 SRAM에서 블록 단위로 처리
   - Online Softmax 트릭
   - NPU에서 FlashAttention 구현의 도전

5. NPU 설계에 주는 시사점
   - SRAM 크기가 FlashAttention 타일 크기를 결정
   - GQA/MQA 지원이 NPU 메모리 효율에 미치는 영향
   - Long-context (100K+ tokens) 처리의 하드웨어 도전
```

**검증 프롬프트:**
```
NPU 30일 DAY 09 검증.

Q1: LLaMA 2 70B (GQA, 80 layers, 8 KV heads, d=128)의 
    KV-Cache 크기를 4K, 16K, 128K 컨텍스트에서 각각 계산하라 (FP16 기준)
Q2: FlashAttention이 NPU SRAM에서 더 자연스러운 이유를
    GPU HBM 기반 구현과 비교하여 설명하라
Q3: MoE 모델에서 Attention의 KV-Cache가 추가로 복잡해지는 이유는?
Q4: "Attention is O(N²)"를 O(N)으로 줄이려는 접근법 3가지와
    각각의 NPU 하드웨어 지원 가능성을 논하라.
```

---

### DAY 10 — 4/8 (수): LLM 추론 파이프라인

**시작 프롬프트:**
```
NPU 30일 과정 DAY 10. 주제: LLM 추론 파이프라인 (Prefill vs Decode)

이것이 NPU 서버 설계의 가장 핵심적인 워크로드 분석이다.

1. Prefill (Prompt Processing) 단계
   - 전체 프롬프트를 한 번에 처리
   - Compute-bound: 높은 FLOPS 필요
   - 높은 Operational Intensity → compute 활용률 높음
   - 배치 가능, 처리량 중심

2. Decode (Token Generation) 단계  
   - 토큰을 하나씩 순차 생성
   - Memory-bound: 매 토큰마다 전체 모델 가중치 로드
   - 낮은 OI (≈1-2 FLOPs/Byte)
   - 레이턴시 중심

3. 이 두 단계의 극적 차이가 NPU 설계에 주는 영향
   - "Prefill 최적화 NPU" vs "Decode 최적화 NPU"
   - Prefill: 높은 TFLOPS 필요 → Cerebras, GPU 유리
   - Decode: 높은 메모리 대역폭 필요 → Groq, SRAM 유리
   - Disaggregated Prefill/Decode: 다른 하드웨어로 각 단계 처리

4. Continuous Batching
   - 정적 배치 vs 동적 배치
   - vLLM의 PagedAttention
   - Iteration-level scheduling

5. Speculative Decoding
   - Draft 모델(작은 모델)이 토큰 후보 생성
   - Verify 모델(큰 모델)이 한 번에 검증
   - Memory-bound → Compute-bound로 변환하는 트릭

6. MoE (Mixture of Experts) 추론
   - Expert routing의 동적 특성
   - 활성 파라미터 vs 전체 파라미터
   - Expert weight 동적 로딩 (AMD NPU 사례)
   - NPU에서 MoE의 도전: 불규칙 메모리 접근 패턴
```

**실습 과제:**
```
[과제 DAY10] LLM 추론 성능 계산기

Python으로 LLM 추론 성능 추정 도구를 만들어라:

입력:
- 모델: 파라미터 수, 레이어 수, hidden dim, heads, KV heads
- 양자화: FP16/INT8/INT4
- 하드웨어: compute FLOPS, 메모리 대역폭, 메모리 용량

출력:
- Prefill 처리량 (tokens/s) - compute-bound 추정
- Decode 처리량 (tokens/s) - memory-bound 추정
- TTFB (Time to First Byte)
- KV-Cache 메모리 사용량
- 최대 동시 요청 수 (메모리 한도 기준)

다음 조합으로 시뮬레이션:
- LLaMA 70B INT4 on A100 (80GB)
- LLaMA 70B INT4 on Groq LPU (추정 스펙)
- LLaMA 70B INT4 on Inferentia2
```

---

### DAY 11 — 4/9 (목): 양자화와 수치 형식

**시작 프롬프트:**
```
NPU 30일 과정 DAY 11. 주제: 양자화 + 수치 형식 심층

양자화는 NPU의 존재 이유와 직결된다.
NPU가 INT8/INT4에서 GPU를 압도하는 이유를 설명해 달라.

1. 수치 형식 계보
   - FP32 (32비트) → FP16 (16비트) → BF16 (16비트) → FP8 → INT8 → INT4
   - 각 형식의 표현 범위와 정밀도
   - BF16이 AI에서 FP16보다 선호되는 이유 (지수부 크기)
   - FP8의 두 변형: E4M3 (추론) vs E5M2 (학습)

2. 양자화 방법론
   - PTQ (Post-Training Quantization): 학습 후 양자화
   - QAT (Quantization-Aware Training): 학습 중 양자화
   - GPTQ, AWQ, SmoothQuant 비교
   - GGUF 포맷과 llama.cpp 생태계

3. Mixed Precision 전략
   - 가중치: INT4, 활성화: BF16 (AMD NPU 사례)
   - KV-Cache: FP16 → INT8/INT4 압축
   - Embedding: FP16 유지 (양자화에 민감)

4. NPU에서 양자화의 하드웨어 의미
   - INT8 MAC vs FP16 MAC: 면적 1/4, 전력 1/4
   - INT4: 같은 면적에 MAC 유닛 4배 탑재 가능
   - NPU의 "TOPS" 수치가 대부분 INT8 기준인 이유
   - 정확도 손실 정량화: MMLU 벤치마크 비교

5. 최신 트렌드
   - MXFP4: Microscaling 기반 4-bit 부동소수점
   - NF4: QLoRA에서 사용하는 Normal Float
   - 1-bit LLM (BitNet): 극단적 양자화의 가능성과 한계
```

**검증 프롬프트:**
```
NPU 30일 DAY 11 검증.

Q1: 70B 모델을 FP16, INT8, INT4로 양자화했을 때
    각각의 모델 크기와 필요 메모리 대역폭(100 tok/s 기준)을 계산하라
Q2: GPTQ vs AWQ의 핵심 차이를 한 문장으로 설명하라
Q3: "NPU X는 500 TOPS (INT8)"와 "GPU Y는 500 TFLOPS (FP16)"
    — 이 두 수치를 직접 비교할 수 있는가? 왜 안 되는가?
Q4: Mixed Precision(W4A16)이 W8A8보다 나은 경우와 
    그 반대인 경우를 각각 설명하라.
```

---

### DAY 12 — 4/10 (금): NPU 핵심 패턴 - Systolic Array

**시작 프롬프트:**
```
NPU 30일 과정 DAY 12. 주제: Systolic Array 아키텍처 심층

Systolic Array는 NPU의 가장 근본적인 아키텍처 패턴이다.
Google TPU에서 시작하여 원리를 완벽히 이해시켜 달라.

1. Systolic Array의 기원과 원리
   - H.T. Kung (1982)의 원논문 아이디어
   - "데이터가 심장 박동처럼 배열을 흐른다"
   - PE(Processing Element)의 구조: MAC + 레지스터
   - 2D 배열에서 데이터 흐름 시뮬레이션

2. 데이터플로우 분류 (핵심!)
   - Weight-Stationary: 가중치가 PE에 고정, 입력이 흘러감
     → Google TPU v1
   - Output-Stationary: 출력이 PE에 고정, 입력+가중치가 흘러감
     → 일부 ASIC 설계
   - Row-Stationary: 행 단위 데이터 재사용 최적화
     → MIT Eyeriss (2016)
   - 각각의 에너지 효율, 대역폭 요구량 비교

3. Google TPU 시리즈의 진화
   - TPU v1 (2015): 256×256 systolic array, 추론 전용
   - TPU v2-v3: 학습 지원, HBM 추가
   - TPU v4: 칩 간 ICI (Inter-Chip Interconnect)
   - TPU v5e/v5p: 비용 최적화 vs 성능 최적화
   - TPU v6 (Trillium): 4.7x 성능/chip 향상

4. Systolic Array의 강점과 한계
   - 강점: GEMM에 이상적, 높은 에너지 효율, 데이터 재사용
   - 한계: 불규칙 연산, Dynamic Shape, Sparse 연산에 비효율
   - Attention의 Softmax가 systolic array에 도전이 되는 이유

5×5 PE 배열에서 3×3 행렬곱의 데이터 흐름을 
클록 사이클별로 step-by-step 보여 달라.
```

**실습 과제:**
```
[과제 DAY12] Systolic Array 시뮬레이터

Python으로 N×N Systolic Array 시뮬레이터를 구현하라:

1. PE 클래스: multiply_accumulate, shift_data
2. SystolicArray 클래스: NxN PE 배열, step() 메서드
3. 클록 사이클별 데이터 흐름 시각화 (텍스트 출력)
4. 4×4 행렬곱을 4×4 systolic array에서 실행
5. 총 사이클 수 계산: 2N-1 + N = 3N-1 사이클

Weight-Stationary vs Output-Stationary 모두 구현하고
에너지 효율(메모리 접근 횟수) 비교.
```

---

### DAY 13 — 4/11 (토): Dataflow + SRAM-Only 아키텍처

**시작 프롬프트:**
```
NPU 30일 과정 DAY 13. 주제: Dataflow 아키텍처와 SRAM-Only 접근법

Systolic Array의 대안 아키텍처들을 심층 분석해 달라.

1. Dataflow 아키텍처 (SambaNova RDU)
   - 연산 그래프가 하드웨어에 "공간적으로 매핑"
   - 각 연산 노드가 전용 하드웨어 영역에 배치
   - 데이터가 파이프라인처럼 노드 간 흐름
   - Reconfigurable: 다른 모델에 대해 재배치 가능
   - SN50 칩: 5x compute/accelerator, 3-tier 메모리
   - 10T+ 파라미터, 10M+ 토큰 컨텍스트 지원 주장

2. SRAM-Only / Deterministic 아키텍처 (Groq LPU)
   - TSP (Tensor Streaming Processor) 아키텍처
   - 230MB SRAM, HBM 없음
   - 80 TB/s on-die 메모리 대역폭
   - 결정론적 실행: 모든 지연시간이 컴파일 타임에 결정
   - 장점: TTFB 0.22s, 241 tok/s (독립 벤치마크)
   - 한계: 모델 크기 제한, 높은 칩 수 요구

3. In-Memory Computing (d-Matrix)
   - 연산을 메모리 내부에서 수행
   - von Neumann 병목 근본적 해결 시도
   - 아직 초기 단계, 미검증 접근법

4. Dataflow vs Systolic vs SRAM-Only 비교
   | 축 | Systolic | Dataflow | SRAM-Only |
   | --- | --- | --- | --- |
   | GEMM 효율 | ★★★ | ★★☆ | ★★☆ |
   | 유연성 | ★☆☆ | ★★★ | ★☆☆ |
   | 에너지 효율 | ★★★ | ★★☆ | ★★★ |
   | 레이턴시 | ★★☆ | ★★☆ | ★★★ |
   | 확장성 | ★★☆ | ★★★ | ★☆☆ |

각 아키텍처의 "킬러 유즈케이스"를 하나씩 제시해 달라.
```

---

### DAY 14 — 4/12 (일): Wafer-Scale + W2 종합 검증

**시작 프롬프트:**
```
NPU 30일 과정 DAY 14.

오전 주제: Cerebras Wafer-Scale Engine

1. WSE-3 아키텍처 상세
   - 전체 300mm 웨이퍼 = 하나의 칩
   - 900,000 AI 코어, 4조 트랜지스터
   - 44GB on-chip SRAM, 21 PB/s 패브릭 대역폭
   - CS-3 시스템: ~20kW 전력, 액냉 필수

2. 장점
   - GPU 클러스터의 통신 병목 완전 제거
   - 단일 칩에서 거대 모델 학습/추론 가능
   - OpenAI 750MW 추론 계약의 의미

3. 한계와 도전
   - 수율: 하나의 결함도 칩 전체에 영향 → redundancy 설계
   - 냉각: 거대한 열 밀도
   - 비용: 단일 시스템 수억원
   - 소프트웨어: 전용 SDK, GPU 생태계와 호환 불가

오후: W2 종합 검증 시험

범위: DAY 08 ~ DAY 14 (GEMM, Attention, LLM 추론, 양자화, 
      Systolic Array, Dataflow, SRAM-Only, Wafer-Scale)

형식: 
- 아키텍처 비교 에세이 (1000자) 1문제
- 계산 문제 3문제
- 시나리오 의사결정 2문제
- 70점 이상 PASS
```

---

## W3: 컴파일러 + 한국 생태계 (4/13 월 ~ 4/19 일)

---

### DAY 15 — 4/13 (월): Programmable NPU (Rebellions, FuriosaAI, AMD)

**시작 프롬프트:**
```
NPU 30일 과정 DAY 15. 주제: 프로그래머블 NPU 아키텍처

한국의 NPU 기업들과 AMD의 접근법을 심층 분석해 달라.

1. Rebellions ATOM → Rebel → REBEL-Quad
   - Programmable Dataflow Architecture
   - Instruction-level + Data-level 병렬화
   - Task Manager: 스케줄링과 태스크 병렬화
   - DMA 엔진
   - UCIe 기반 칩렛 확장 (4개 다이 = REBEL-Quad)
   - 삼성 4nm, HBM3E 탑재
   - LLM 시대 대응: 새로운 precision 지원
   - SAPEON 합병으로 비디오 코덱 + RISC-V 역량 추가

2. FuriosaAI RNGD
   - LLM 추론 특화 설계
   - 2025년 말부터 20,000유닛 시장 공급 계획
   - 한국 정부 프로젝트 참여

3. AMD XDNA (Ryzen AI NPU)
   - AI Engine: 공간 배열(Spatial Array) 타일 기반
   - 각 타일: AI Engine 코어 + 메모리
   - Stream switch로 타일 간 데이터 라우팅
   - DMA로 프로그래밍 가능한 데이터 이동
   - MLIR-AIE로 직접 프로그래밍 가능 (오픈소스!)

4. "프로그래머블"이 왜 중요한가
   - 고정 기능 NPU: 특정 연산만 가속
   - 프로그래머블 NPU: 새로운 모델 아키텍처 대응 가능
   - MoE, Sparse Attention 등 비정형 워크로드 처리
   
각 칩의 블록 다이어그램을 텍스트로 비교해 달라.
```

---

### DAY 16 — 4/14 (화): 주요 NPU 비교 매트릭스 작성

**시작 프롬프트:**
```
NPU 30일 과정 DAY 16. 주제: NPU 비교 매트릭스 작성 (실습일)

오늘은 순수 실습이다. 내가 직접 매트릭스를 작성하면 검증해 달라.

작성할 매트릭스:
12개 NPU/가속기를 10개 축으로 비교

칩 목록:
1. Cerebras WSE-3    7. Intel Gaudi3
2. Groq LPU          8. Rebellions Rebel
3. SambaNova SN50    9. FuriosaAI RNGD
4. Google TPU v6     10. Qualcomm AI200
5. AWS Trainium2     11. OpenAI "Titan"
6. AWS Inferentia2   12. NVIDIA H100 (기준선)

비교 축:
(1) 아키텍처 유형
(2) 공정 노드
(3) 메모리 종류 및 용량
(4) Peak 연산 성능 (TOPS/TFLOPS)
(5) 메모리 대역폭
(6) TDP(전력)
(7) 추정 TOPS/W
(8) 주요 소프트웨어 스택
(9) 타겟 워크로드
(10) 가용성 (클라우드/온프레미스/미출시)

내가 작성하면 빈 칸을 채워주고, 잘못된 정보를 교정해 달라.
```

---

### DAY 17 — 4/15 (수): 한국 NPU 생태계 + 시장 분석

**시작 프롬프트:**
```
NPU 30일 과정 DAY 17. 주제: 한국 AI 반도체 생태계 심층

나는 한국에서 일하는 클라우드 아키텍트다. 
한국 NPU 생태계를 투자자/아키텍트 관점에서 분석해 달라.

1. 한국 NPU 기업 지형도
   - Rebellions: SK그룹(SK Telecom, SK Hynix) + 삼성 투자, 
     Arm Holdings 주도 Series C, 유니콘 ($1.5B+)
   - FuriosaAI: $124M 펀딩 (2024), LLM 추론 특화
   - 비교: Rebellions 정부 매출 70억원 vs Groq 사우디 7,500억원

2. 한국의 구조적 강점
   - SK Hynix: HBM 세계 시장 50%+ → NPU 생태계 시너지
   - 삼성 파운드리: GAA 공정 (3nm) 경쟁력
   - KAIST/서울대 반도체 인재 파이프라인
   - 정부 AI 반도체 전략 및 예산

3. 한국의 구조적 약점
   - 정부 예산: NVIDIA 대비 1/10 미만 (Rebellions CEO 발언)
   - 소프트웨어 생태계 미성숙 (CUDA 대항마 부재)
   - 해외 시장 개척 경험 부족
   - 미-중 지정학 리스크 (수출 통제, 공급망)

4. NPU 서버 아키텍트로서의 시사점
   - 한국 정부 프로젝트에서 국산 NPU 사용 의무화 가능성
   - SK/삼성 그룹사 내부 수요
   - REBEL-Quad의 실제 성능 데이터 언제 공개되나?
   
5. 글로벌 대비 포지셔닝
   - 사우디/UAE의 비-NVIDIA 칩 구매 트렌드
   - 공급망 다변화 수요
   - 한국 NPU의 중동/동남아 수출 기회

최신 뉴스와 데이터를 반영하여 분석해 달라.
```

---

### DAY 18 — 4/16 (목): AI 컴파일러 기초

**시작 프롬프트:**
```
NPU 30일 과정 DAY 18. 주제: AI 컴파일러 기초

"하드웨어가 아무리 좋아도 컴파일러 없이는 10%도 못 쓴다."
이 명제를 이해시켜 달라.

1. 전통 컴파일러 vs AI 컴파일러
   - 전통: 소스 코드 → 기계어 (C++ → x86)
   - AI: 모델 그래프 → 가속기 코드 (PyTorch → NPU 바이너리)
   - 공통점: 최적화 Pass 체인, IR(중간 표현)

2. Computation Graph
   - PyTorch 모델이 어떻게 연산 그래프로 변환되는가
   - 노드 = 연산 (MatMul, Softmax, ReLU...)
   - 엣지 = 텐서 데이터 흐름
   - 그래프 시각화: torch.fx, ONNX

3. Graph-level 최적화
   - Operator Fusion: MatMul + Bias + ReLU → 하나의 커널
   - Constant Folding: 컴파일 타임에 계산 가능한 것은 미리 계산
   - Dead Code Elimination
   - Layout Transformation: NCHW vs NHWC 

4. Kernel-level 최적화
   - Loop Tiling (타일링): 캐시 활용
   - Vectorization: SIMD 명령어 활용
   - Loop Unrolling: 분기 오버헤드 제거
   - Double Buffering: 계산과 데이터 전송 오버랩

5. NPU 컴파일러의 특수한 도전
   - Spatial Mapping: 연산을 2D PE 배열에 배치
   - DMA Scheduling: 데이터 이동 스케줄링
   - 메모리 할당: 제한된 SRAM에 텐서 배치
   - 파이프라인 스테이징: Prefill/Decode 최적화
```

---

### DAY 19 — 4/17 (금): MLIR 심층

**시작 프롬프트:**
```
NPU 30일 과정 DAY 19. 주제: MLIR (Multi-Level Intermediate Representation)

MLIR을 배우면 모든 NPU 컴파일러의 기반을 이해하게 된다.

1. MLIR이 필요한 이유
   - LLVM IR은 CPU/GPU에 최적화, AI 가속기에는 추상화 부족
   - 각 NPU 벤더가 자체 IR을 만들면 → 생태계 파편화
   - MLIR: "모든 수준의 추상화를 하나의 프레임워크에서"

2. Dialect 시스템 (MLIR의 핵심)
   - High-level dialects:
     - Torch-MLIR: PyTorch → MLIR 변환
     - TOSA: Tensor Operator Set Architecture (이식성)
     - StableHLO: Google의 XLA에서 파생
   - Mid-level dialects:
     - Linalg: 선형 대수 연산
     - SCF: 구조적 제어 흐름 (for loops)
     - Affine: 다차원 배열 접근 패턴
   - Low-level dialects:
     - LLVM IR dialect: 최종 코드 생성
     - Vendor-specific: AIE (AMD), Hexagon (Qualcomm)

3. Pass와 Lowering
   - Pass: IR을 변환하는 단위 작업 (최적화, 검증, 변환)
   - Progressive Lowering: 높은 추상화 → 낮은 추상화 단계적 변환
   - 예: Torch-MLIR → Linalg → SCF → LLVM IR

4. NPU 벤더의 MLIR 활용 사례
   - AMD: MLIR-AIE, MLIR-AIR (오픈소스!)
     - AIR dialect: 비동기 계층적 연산
     - 78.7% compute efficiency on 행렬곱
   - Qualcomm: Hexagon-MLIR (오픈소스!)
     - Triton 커널 → Hexagon NPU 매핑
     - TCM(Tightly Coupled Memory) 최적화
   - Google: StableHLO → XLA → TPU
   - Cerebras: 내부 MLIR 기반 컴파일러 (비공개)

5. MLIR이 "NPU 벤더 백엔드의 공통 IR 레이어"인 구조
   PyTorch/TF → Torch-MLIR/TOSA → [공통 최적화] → 벤더 dialect → 바이너리

다이어그램으로 전체 컴파일 파이프라인을 보여 달라.
```

**실습 과제:**
```
[과제 DAY19] MLIR 컴파일 파이프라인 도식화

3개 NPU의 전체 컴파일 경로를 Mermaid 다이어그램으로 그려라:

(a) Google TPU: JAX → StableHLO → XLA → TPU code
(b) AMD NPU: PyTorch → Torch-MLIR → Linalg → AIR → AIE → binary
(c) Qualcomm NPU: Triton → Hexagon-MLIR → Hexagon binary

각 단계에서 어떤 최적화가 적용되는지 주석을 달아라.
```

---

### DAY 20 — 4/18 (토): TVM, ONNX Runtime, 벤더 SDK

**시작 프롬프트:**
```
NPU 30일 과정 DAY 20. 주제: 실전 컴파일러/런타임 프레임워크

MLIR이 기반이라면, 실무에서 직접 쓰는 도구들을 학습한다.

1. Apache TVM
   - Python-first ML 컴파일러
   - Relay IR → TIR → Target code
   - AutoTVM / MetaSchedule: 자동 최적화 탐색
   - BYOC (Bring Your Own Codegen): NPU 백엔드 통합 방법
   - 장점: 범용, 오픈소스, Python 친화적
   - 한계: 최신 LLM 모델 지원 속도

2. ONNX Runtime
   - 실무에서 가장 널리 사용되는 추론 프레임워크
   - Execution Provider (EP) 아키텍처
   - NPU EP 추가 방법: 벤더가 EP 구현 제공
   - OGA (ONNX Runtime GenAI): LLM 추론 특화
   - AMD Ryzen AI의 OGA 통합 사례

3. 벤더별 SDK 비교
   - Intel OpenVINO: 가장 성숙한 NPU 지원
   - Qualcomm AI Engine Direct
   - Rebellions RBLN SDK
   - Cerebras SDK + Model Zoo
   - 각 SDK의 지원 모델, 성숙도, 문서화 수준

4. 실무 의사결정 프레임워크
   "내 모델을 NPU에서 돌리려면?"
   (a) ONNX로 변환 가능? → ONNX Runtime + NPU EP
   (b) 벤더 SDK에 모델 지원? → 벤더 SDK 직접 사용
   (c) 커스텀 최적화 필요? → TVM BYOC 또는 MLIR 직접 작성
   (d) 최고 성능 필요? → 벤더와 공동 최적화

AWS Inferentia2 사용 경험이 있는 나에게,
다른 NPU로의 전환 시 가장 큰 장벽이 무엇인지 설명해 달라.
```

---

### DAY 21 — 4/19 (일): vLLM + 양자화 엔진 + W3 종합 검증

**시작 프롬프트:**
```
NPU 30일 과정 DAY 21.

오전 주제: vLLM과 LLM 서빙 최적화

1. vLLM의 핵심: PagedAttention
   - OS의 Virtual Memory 개념을 KV-Cache에 적용
   - 물리 메모리 블록 + 페이지 테이블
   - 메모리 낭비 최소화 (평균 4% 미만)
   - NPU에서 vLLM 적용: tok/s 2배, 전력 효율 92% 향상 사례

2. 서빙 프레임워크 비교
   - vLLM: 범용, GPU 중심, NPU 포팅 진행 중
   - TensorRT-LLM: NVIDIA 전용, 최고 성능
   - Triton Inference Server: 멀티 모델 서빙
   - 각 NPU 벤더의 자체 서빙 솔루션

3. Speculative Decoding 실전
   - Draft 모델 선택: 원본의 1/10 크기
   - 검증 배치 크기 최적화
   - 수용률(Acceptance Rate)과 속도 향상 관계

오후: W3 종합 검증 시험
범위: DAY 15~21 (한국 NPU 생태계, 컴파일러, MLIR, TVM, vLLM)
형식: 기술 에세이 1문제 + 아키텍처 도식 1문제 + 시나리오 2문제
70점 이상 PASS
```

---

## W4: 시스템 설계 + 마스터 프로젝트 (4/20 월 ~ 4/28 화)

---

### DAY 22 — 4/20 (월): 서버 하드웨어 아키텍처

**시작 프롬프트:**
```
NPU 30일 과정 DAY 22. 주제: NPU 서버 하드웨어 아키텍처

나는 클라우드에서 EC2 인스턴스를 다뤘지만, 
물리 서버 설계는 처음이다. 기초부터 가르쳐 달라.

1. NPU 서버 폼팩터
   - 1U, 2U, 4U 서버의 물리적 차이
   - PCIe 슬롯 구성: x16 레인, Bifurcation (x16→x8x8)
   - 실제 AI 서버 예시:
     - Dell PowerEdge XE9680 (8 GPU/NPU)
     - HPE ProLiant DL380a Gen11

2. 호스트 CPU 선택
   - Intel Xeon (Sapphire Rapids, Emerald Rapids)
   - AMD EPYC (Genoa, Turin)
   - Arm: Ampere Altra, NVIDIA Grace
   - SoftBank "Silicon Trinity" 전략
   - CPU 선택이 NPU 성능에 미치는 영향

3. 메모리 구성
   - DDR5 채널 수, DIMM 배치
   - CXL 메모리 확장 가능성
   - 호스트 메모리 vs NPU 메모리의 역할 분담

4. 스토리지
   - NVMe SSD: 모델 로딩 시간 최적화
   - RAID 구성
   - 모델 웜업 전략

5. NPU 장착 구성
   - PCIe 카드 형태 (GroqCard, Inferentia2)
   - 독립 서버 형태 (Cerebras CS-3)
   - 커스텀 보드 (SambaNova DataScale)
   - OAM (OCP Accelerator Module) 표준

AWS EC2 inf2 인스턴스의 "물리적 실체"가 
어떤 서버 하드웨어인지 역추적해 달라.
```

---

### DAY 23 — 4/21 (화): 전력·냉각·물리 인프라

**시작 프롬프트:**
```
NPU 30일 과정 DAY 23. 주제: 전력·냉각·물리 인프라

NPU 서버는 열과 전기의 전쟁이다.

1. 전력 설계
   - NPU 서버 전력 비교:
     - 8×H100 GPU 서버: ~10kW
     - Cerebras CS-3: ~20kW
     - Groq GroqRack: 각 LPU ~300W × 칩 수
   - NPU 전력 효율 우위: 35-70% 절감 (MDPI 연구)
   - PDU(Power Distribution Unit), UPS, 이중화
   - 한국 산업용 전기요금: ~120원/kWh

2. 냉각 설계
   - 공랭(Air Cooling): 한계 ~40kW/rack
   - 직접 액냉(DLC): Cold plate 방식, ~100kW+/rack
   - Rear Door Heat Exchanger
   - Immersion Cooling: Cerebras WSE 필수
   - 한국 IDC 환경: 대부분 공랭, 액냉 전환 중

3. 랙 설계
   - 42U 표준 랙에 NPU 서버 배치
   - 전력 밀도 계획: kW/rack
   - 에어플로 관리: Hot aisle / Cold aisle
   - 무게 분산 (GPU/NPU 서버는 매우 무거움)

4. 데이터센터 레벨
   - PUE (Power Usage Effectiveness): 목표 < 1.2
   - Tier III vs Tier IV 가용성
   - 한국 주요 IDC: KT, LG U+, Naver, 삼성SDS
   - 전용 AI 데이터센터 트렌드

한국 수원(Suwon) 인근에서 NPU 서버를 운영한다면
어떤 IDC를 고려해야 하는지 실용적 조언을 해 달라.
```

---

### DAY 24 — 4/22 (수): 네트워크 토폴로지 + 클러스터 설계

**시작 프롬프트:**
```
NPU 30일 과정 DAY 24. 주제: 네트워크 토폴로지와 추론 클러스터

1. 추론 클러스터 내부 네트워크
   - Tensor Parallelism: 노드 내 고대역폭 필요
   - Pipeline Parallelism: 노드 간 네트워크
   - Expert Parallelism: MoE 전용

2. 네트워크 토폴로지
   - Fat Tree: 가장 보편적, 높은 양방향 대역폭
   - Dragonfly: 장거리 효율적
   - Rail-Optimized: NVIDIA SuperPOD에서 사용
   
3. 서빙 네트워크
   - Load Balancer → API Gateway → NPU 서버 풀
   - 요청 라우팅 전략: 라운드 로빈 vs 최소 레이턴시
   - 모델별 라우팅, 배치 크기별 라우팅

4. Kubernetes 기반 NPU 오케스트레이션
   - Device Plugin으로 NPU 리소스 등록
   - Resource Quota 관리
   - Topology-Aware Scheduling
   - 내가 AWS EKS 경험이 있으니, 
     온프레미스 K8s + NPU 설정과의 차이를 설명해 달라

Terraform + Kubernetes 기반 NPU 클러스터
IaC 템플릿의 핵심 구조를 보여 달라.
```

---

### DAY 25 — 4/23 (목): 멀티클라우드 + 하이브리드 NPU 인프라

**시작 프롬프트:**
```
NPU 30일 과정 DAY 25. 주제: 멀티클라우드 하이브리드 NPU 인프라

내 전문 분야(멀티클라우드 아키텍처)와 NPU를 결합한다.

1. 클라우드 NPU 서비스 비교
   - AWS: Inferentia2 (inf2), Trainium2 (trn2)
   - GCP: TPU v6 (Trillium), TPU v5e
   - GroqCloud: API 서비스
   - Cerebras Cloud: 온디맨드
   - SambaNova Cloud

2. 하이브리드 아키텍처 설계
   - 베이스라인: 온프레미스 NPU (예: Rebellions)
   - 버스트: 클라우드 NPU (AWS inf2)
   - 트래픽 기반 오토스케일링 전략

3. 멀티클라우드 NPU 전략
   - 모델별 최적 플랫폼 선택
   - 벤더 락인 회피: ONNX 기반 이식성
   - Terraform으로 멀티클라우드 NPU 프로비저닝

4. Bedrock이 내부적으로 쓰는 칩은?
   - Anthropic Claude on AWS: Trainium/Inferentia 추론 가능성
   - 서비스 뒤의 하드웨어를 이해하면 성능 예측 가능

5. CDK/Terraform 기반 NPU 파이프라인
   - inf2 인스턴스 프로비저닝
   - 모델 배포 자동화
   - 모니터링 (CloudWatch + Custom Metrics)

이미 내가 AWS CDK와 Terraform을 쓰고 있으니,
NPU 인프라에 바로 적용할 수 있는 코드 스니펫을 제공해 달라.
```

---

### DAY 26 — 4/24 (금): 프로파일링 + 벤치마킹

**시작 프롬프트:**
```
NPU 30일 과정 DAY 26. 주제: NPU 프로파일링과 벤치마킹

1. 핵심 성능 메트릭
   - TTFB (Time to First Byte): 첫 토큰 레이턴시
   - TPS (Tokens Per Second): 처리량
   - ITL (Inter-Token Latency): 토큰 간 지연
   - MFU (Model FLOPs Utilization): 이론 대비 실제 활용률
   - MBU (Memory Bandwidth Utilization): 대역폭 활용률

2. 프로파일링 도구
   - Chrome Tracing / Perfetto UI (MLIR-AIR)
   - NVIDIA Nsight (GPU 비교 기준)
   - Intel VTune, AMD uProf
   - 벤더별 프로파일러

3. 벤치마크 방법론
   - MLPerf Inference: 유일한 표준
   - ArtificialAnalysis.ai: 독립 벤치마크
   - 자체 워크로드 벤치마크 설계법

4. 벤더 마케팅 수치 해석법
   - "10x faster than GPU" 주장의 함정
   - 같은 조건 비교의 중요성: 모델, 배치 크기, 양자화, 입출력 길이
   - Peak vs Sustained 성능

5. $/token 계산법
   - 하드웨어 비용 + 전력 비용 + 운영 비용
   - 시간당 처리 토큰 수
   - 최종 $/million tokens 산출
```

---

### DAY 27 — 4/25 (토): TCO 분석 + 서빙 최적화

**시작 프롬프트:**
```
NPU 30일 과정 DAY 27. 주제: TCO 분석 실습

오늘은 전적으로 실습이다. 스프레드시트를 만들어라.

TCO 비교 대상 (3년 기준):
(a) NVIDIA H100 8-GPU 서버 × 4대
(b) Groq GroqRack (320 LPU)
(c) AWS inf2.48xlarge × N대 (온디맨드)
(d) AWS inf2.48xlarge × N대 (Reserved 3년)
(e) Rebellions REBEL-Quad × 8대 (추정)

워크로드: LLaMA 70B INT4, 평균 500 req/s, 입력 512 / 출력 256 토큰

비용 항목:
1. 하드웨어 구입비 (또는 클라우드 인스턴스 비용)
2. 전력 비용 (한국 산업용 ~120원/kWh)
3. 냉각 비용 (PUE 1.3 가정)
4. 네트워크 장비
5. IDC 상면 비용 (한국 기준 ~월 150-300만원/rack)
6. 인건비 (운영 엔지니어)
7. 소프트웨어 라이선스

출력: 
- 3년 총 비용
- $/million tokens
- 손익분기점 분석
- 민감도 분석 (전기요금 ±20%, 사용률 ±30%)

이 계산을 도와달라. 내가 스프레드시트를 만들면 검증해 달라.
```

---

### DAY 28 — 4/26 (일): W4 종합 검증

**시작 프롬프트:**
```
NPU 30일 과정 DAY 28. W4 종합 검증.

범위: DAY 22~27 (서버 하드웨어, 전력/냉각, 네트워크, 
      멀티클라우드, 프로파일링, TCO)

최종 검증 시험을 출제해 달라:

형식:
1. 서버 설계 문제 (1문제, 20점)
   "주어진 요구사항으로 NPU 서버 랙을 설계하라"
   
2. TCO 계산 문제 (1문제, 20점)
   "두 옵션의 3년 TCO를 비교하고 추천하라"

3. 시나리오 의사결정 (2문제, 각 15점)
   "CTO가 묻는다: GPU vs NPU, 뭘 사야 하나?"
   "시설팀이 공랭 한계를 걱정한다. 해결책은?"

4. 기술 에세이 (1문제, 30점)
   "2027년 NPU 서버 시장의 승자는 누구일 것인가? 
    기술적 근거를 들어 논하라."

총 100점, 80점 이상 PASS.
미달 시 보충 학습 범위와 추가 프롬프트 제공.
```

---

### DAY 29 — 4/27 (월): 마스터 프로젝트 (설계 + 작성)

**시작 프롬프트:**
```
NPU 30일 과정 DAY 29. 마스터 프로젝트 시작.

프로젝트: "한국 AI 기업의 차세대 NPU 추론 인프라 설계서"

요구사항:
- 모델: 70B 한국어 LLM + 13B 코드 생성 + VLM
- 트래픽: 평균 2,000 req/s, 피크 10,000 req/s
- SLA: p99 < 3초 (70B), p99 < 1초 (13B)
- 예산: HW 100억원, 연간 운영 50억원
- 한국 IDC, 공랭+부분 액냉, 정부 보안 인증

오늘 나는 다음을 작성한다:
1. 하드웨어 아키텍처 (칩 선택, 서버 구성, 랙 레이아웃)
2. 소프트웨어 스택 (컴파일러, 서빙, 모니터링)
3. 네트워크 아키텍처

내가 각 섹션의 초안을 작성하면:
- 기술적 오류를 지적하라
- 누락된 고려사항을 추가하라
- 비용 추정이 현실적인지 검증하라
- 대안을 제시하고 트레이드오프를 명시하라

내가 "섹션 1 초안"이라고 보내면 해당 리뷰를 시작해 달라.
```

---

### DAY 30 — 4/28 (화): 마스터 프로젝트 완성 + 최종 리뷰

**시작 프롬프트:**
```
NPU 30일 과정 DAY 30. 최종일.

오늘 완성할 섹션:
4. 운영 전략 (K8s, 오토스케일링, HA/DR)
5. 경제성 분석 (3년 TCO, $/M tokens, ROI)
6. 마이그레이션 로드맵 (Phase 1-3)
7. 리스크 분석

완성 후 최종 리뷰:
- 전체 설계서의 일관성 검증
- 실현 가능성 평가 (예산, 기술, 인력)
- "이 설계서를 CTO에게 발표한다면" 관점의 피드백
- 점수 산정 (100점 만점)

마지막으로, 30일 과정 전체를 돌아보며:
- 내가 가장 강해진 영역
- 추가 심화가 필요한 영역
- 앞으로 6개월간의 지속 학습 로드맵
을 정리해 달라.

수고했다, 제자여. (그리고 나 자신에게)
```

---

## 부록: 유틸리티 프롬프트

### 개념이 이해 안 될 때
```
NPU 30일 과정에서 [개념 이름]이 이해가 안 된다.

1. 5살짜리에게 설명하듯이 비유로 설명해 달라
2. 그 비유의 한계도 알려 달라
3. NPU 서버 맥락에서 왜 이것이 중요한지 연결해 달라
4. 이것을 모르면 다음 어느 단계에서 막히게 되는가?
```

### 코드 리뷰 요청
```
NPU 30일 과정 DAY [N] 과제 코드를 리뷰해 달라.

[코드 붙여넣기]

리뷰 기준:
1. 기술적 정확성
2. 최적화 가능한 부분
3. NPU 하드웨어 동작과의 매핑이 정확한가
4. 추가로 구현하면 좋을 기능
```

### 실전 시나리오 연습
```
NPU 30일 과정 실전 시나리오 연습.

역할: 나는 NPU 서버 아키텍트, 당신은 [고객/CTO/시설팀/동료].
상황: [구체적 시나리오]

내가 답하면 현실적으로 반론하고, 
내 답변의 강점과 약점을 평가해 달라.
```

### 최신 뉴스 연결
```
NPU 30일 과정과 관련된 최신 뉴스를 검색하고,
오늘의 학습 내용([주제])과 어떻게 연결되는지 분석해 달라.
특히 한국 NPU 생태계(Rebellions, FuriosaAI, SK Hynix)
관련 업데이트가 있는지 확인해 달라.
```

### 주간 복습
```
NPU 30일 과정 Week [N] 복습.

이번 주(DAY [X] ~ DAY [Y])에 배운 핵심 개념을 
마인드맵 형태로 정리해 달라.

그리고 다음 주 학습과의 연결 고리를 미리 보여 달라.
각 개념에 대한 내 이해도를 1-5로 자기 평가할 수 있게
체크리스트를 만들어 달라.
```

---

## 필수 자원 빠른 링크

| 자원 | URL | 용도 |
|------|-----|------|
| nand2tetris | nand2tetris.org | DAY 01-03 하드웨어 기초 |
| WikiChip | wikichip.org | 칩 스펙 데이터베이스 |
| Chips and Cheese | chipsandcheese.com | 마이크로아키텍처 분석 |
| MLIR 공식 | mlir.llvm.org | DAY 19 컴파일러 |
| MLIR-AIE GitHub | github.com/Xilinx/mlir-aie | DAY 19 실습 |
| Hexagon-MLIR 논문 | arxiv.org/html/2602.19762v1 | DAY 19 참고 |
| Apache TVM | tvm.apache.org | DAY 20 컴파일러 |
| vLLM GitHub | github.com/vllm-project/vllm | DAY 21 서빙 |
| llama.cpp | github.com/ggerganov/llama.cpp | 추론 최적화 |
| MLPerf | mlcommons.org | DAY 26 벤치마크 |
| ArtificialAnalysis | artificialanalysis.ai | 독립 벤치마크 |
| SemiAnalysis | semianalysis.com | 업계 분석 뉴스레터 |
| The Next Platform | nextplatform.com | 하드웨어 심층 분석 |
| Onur Mutlu 강의 | YouTube 검색 | 컴퓨터 아키텍처 |
| MIT 6.5940 | efficientml.ai | 효율적 AI 시스템 |

---

## 완주 후 다음 단계

1. **Hot Chips 2026 (8월)** 참석/시청 — 최신 칩 발표
2. **MLPerf Inference 결과** 분기별 추적
3. **Rebellions REBEL-Quad 벤치마크** 공개 시 분석
4. **MLIR-AIE 실습 프로젝트** — 실제 AMD NPU에서 커스텀 커널 작성
5. **기술 블로그 연재** — 학습 내용을 정리하여 전문성 증명
6. **한국 AI 반도체 포럼 / KAIST 세미나** 참여 — 네트워킹

---

> **"첫 번째 쥐는 덫에 걸리고, 두 번째 쥐가 치즈를 얻는다."**
> — Rebellions CBO Marshall Choy
>
> GPU 시대의 첫 번째 물결이 지나간 지금,
> NPU 서버의 두 번째 물결에 올라타는 것은 당신의 선택이다.
