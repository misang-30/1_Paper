# 0. 논문

- **제목:** DETRAM: End-to-end DEtection, Tracking and Recovery of HumAn Meshes
- **학회:** ECCV 2026 (사용자 제공 표기)
- **저자:** Chunggi Lee, Seonwook Park, Wanhua Li, Umar Iqbal, Hanspeter Pfister
- **소속:** Harvard University, NVIDIA, Nanyang Technological University
- **논문 버전:** arXiv:2607.09089v1, 2026년 7월 10일
- **분야:** Computer Vision, Human Mesh Recovery, Multi-Person Tracking, 3D Human Reconstruction

제공된 PDF의 본문 18페이지와 Supplementary Material 7페이지를 기준으로 연구 배경, 핵심 수식, 학습 방식, 구현 조건, 정량·정성 평가, Ablation Study 및 한계점을 정리하였다.

---

# 1. 요약

## 1.1. 연구 목적

DETRAM은 비디오에서 여러 사람을 동시에 검출(Detection), 추적(Tracking), 3D 인체 메시 복원(Human Mesh Recovery)하는 End-to-End 프레임워크이다.

기존 다중 인체 복원 시스템은 일반적으로 다음의 다단계 구조를 사용한다.

\[
\text{Detection}\rightarrow\text{Tracking}\rightarrow\text{HMR}
\]

이러한 구조에서는 객체 검출 오류가 추적과 3D 복원으로 전파되고, 여러 모델을 순차 실행하므로 처리 시간이 증가한다.

DETRAM은 세 작업을 하나의 Transformer Decoder에 통합하여 프레임마다 단일 네트워크의 Feed-forward 과정으로 처리한다. 이전 프레임의 인물 정보를 Tracking Query로 전달하여 인물별로 일관된 ID를 유지한다. Bounding Box Prompt로 사용자가 특정 인물을 지정하거나 놓친 인물을 추적 대상으로 추가할 수 있다.

## 1.2. 핵심 아이디어

DETRAM은 세 종류의 Query를 사용한다.

| Query | 역할 |
|---|---|
| Detection Query | 현재 프레임에서 새로운 사람 검출 |
| Tracking Query | 이전 프레임에서 추적 중이던 사람의 ID와 신체 정보 유지 |
| Prompt Query | 사용자가 지정한 사람을 추적하거나 놓친 인물의 추적 복구 |

세 Query를 하나의 Transformer Decoder에서 공동으로 처리한다. 핵심 기술은 다음과 같다.

1. Track-Modulated Cross-Attention(TMCA)을 통한 Tracking Query 갱신
2. Detection·Tracking·Prompt Query의 공동 디코딩
3. Track-Query Memory Bank를 이용한 과거 신원 정보 보존
4. Hungarian Matching 및 Tracking Loss를 이용한 End-to-End 학습
5. Bounding Box Prompt를 통한 사용자 지정 추적

기존 4DHumans처럼 Detection, HMR, Tracking을 별도의 모듈로 순차 수행하지 않으면서 새로운 사람의 등장과 기존 사람의 추적을 동시에 처리하는 것이 핵심이다.

## 1.3. 주요 연구 결과

| 데이터셋 | 주요 결과 |
|---|---|
| PoseTrack21 | MOTA 73.6, IDF1 80.6 |
| MuPoTS-3D | MOTA 95.9, IDF1 97.4 |
| BEDLAM | MOTA 99.18, IDF1 99.37, MPJPE 48.68 mm |
| Dyna3DPW | MOTA 99.8, IDF1 99.9, ID Switch 0 |
| 3DPW | MPJPE 60.0 mm, PVE 70.1 mm |

PoseTrack21에서 RTX A4000 GPU 기준 **10.87 FPS**를 기록하여 논문에서 비교한 CoMotion 대비 약 1.9배, 4DHumans 대비 약 21배 빠른 추론 속도를 보였다. 사용자 프롬프트 추가 실험에서는 PoseTrack21 MOTA가 73.6에서 75.7로 향상되었다. 다만 실제 사용자가 그린 박스가 아니라 GT로부터 생성한 박스를 사용한 Oracle 설정이다.

---

# 2. 문제 정의

## 2.1. 기존 Human Mesh Recovery의 한계

### 1) 다단계 파이프라인에서 발생하는 오류 전파

기존 Multi-Person HMR의 일반적인 구조는 다음과 같다.

```text
Input Video
    ↓
Human Detector
    ↓
Bounding Box
    ↓
Person Crop
    ↓
Human Mesh Recovery
    ↓
Identity Association / Tracking
    ↓
3D Human Mesh + Track ID
```

- 검출 Bounding Box가 부정확하면 사람의 신체 일부가 Crop에서 제외될 수 있다.
- 사람이 겹치면 신체를 잘못 추정하거나 서로 다른 사람의 특징을 혼동할 수 있다.
- Detection 오류가 HMR 및 Tracking에 영향을 미친다.
- 모듈별 독립적인 학습·실행으로 전체 추론 시간이 늘어난다.
- 비디오의 가림(Occlusion), 등장·퇴장, 카메라 움직임 때문에 같은 사람에게 동일한 ID를 유지하기 어렵다.

### 2) 단일 프레임 기반 HMR의 시간적 정보 부족

Multi-HMR, SAT-HMR, AIOS와 같은 One-Stage 방법은 전체 이미지에서 여러 사람의 3D Mesh를 공동으로 복원한다. 사람별 Crop에서 발생하는 정보 손실을 줄이고 전체 장면의 공간적 맥락을 이용할 수 있지만, 주로 프레임을 독립적으로 처리한다.

- 이전과 현재 프레임의 인물이 동일한 사람인지 구별하기 어렵다.
- 프레임 사이에서 ID가 바뀔 수 있다.
- 별도의 Tracking Module 또는 후처리 Association이 필요하다.
- 사용자가 원하는 사람만 지정하여 추적하는 기능이 부족하다.

### 3) 기존 Tracking-HMR 통합 방법의 한계

| 방법 | 특징 | 논문에서 지적한 차이 |
|---|---|---|
| T3DP / PHALP / 4DHumans | 3D Pose, 위치 및 특징을 이용한 추적 | 검출·복원·추적 모듈이 분리됨 |
| MaQ | Clip 단위 Motion Query 기반 통합 Motion Capture | Offline 방식, 별도의 학습 가능한 시간 모듈 사용 |
| CoMotion | 프레임별 검출기 + Pose Update Network | 검출과 Pose 갱신을 별도 구성 요소로 처리 |
| SAM 3 | Prompt 기반 비디오 분할과 추적 | 주로 2D Mask의 시간적 전파 수행 |

DETRAM은 Detection, Tracking, Prompt Query를 단일 Decoder에서 처리하며 사용자 Bounding Box를 인물의 ID Anchor로 활용한다.

## 2.2. 해결하고자 하는 문제

시간 순서에 따른 RGB 이미지 시퀀스를 입력한다.

\[
I=\{I^1,I^2,\ldots,I^T\}
\]

\[
I\in\mathbb{R}^{T\times H\times W\times3}
\]

여기서 $T$는 프레임 수, $H,W$는 이미지 해상도이다. 영상의 인원수 $N_p$는 사전에 알 수 없고 시간에 따라 달라진다. 목표는 인물별 3D Mesh를 복원하면서 프레임 간 동일 인물의 ID를 유지하는 것이다.

SMPL 모델 사용:

\[
H_i=\operatorname{SMPL}(\theta_i,\beta_i,\tau_i)
\]

| 파라미터 | 의미 | 차원 |
|---|---|---|
| $\theta_i$ | 신체 관절 회전 및 전역 방향 | $24\times3$ |
| $\beta_i$ | 체형 파라미터 | 10 |
| $\tau_i$ | 카메라 좌표계 기준 3D 위치 | 3 |
| $H_i$ | 복원된 Human Mesh | 6,890 vertices |

단순히 2D Keypoint가 아니라 3D 자세, 체형, 카메라 좌표계의 위치와 시간적으로 일관된 ID를 함께 추정해야 한다.

---

# 3. 아이디어

## 3.1. 하나의 Transformer Decoder에 세 가지 Query 통합

기본 구조는 **Figure 2(논문 5페이지)**에 제시된다.

```text
               RGB Image
                   │
                   ▼
          DINOv2 ViT Encoder
                   │
                   ▼
             Image Features
                   │
                   ▼
Detection Query ───┐
                   │
Tracking Query ────┼──► DETRAM Decoder
                   │           │
Prompt Query ──────┘           ▼
                       Prediction Heads
                              │
                 ┌────────────┼───────────┐
                 ▼            ▼           ▼
              2D Box       SMPL        Confidence
                              │
                              ▼
                     Tracking Query Update
                              │
                              ▼
                        Next Frame
```

Detection Query는 새로운 인물을 발견하고, Tracking Query는 이전 인물의 신원을 유지하며, Prompt Query는 사용자가 지정한 인물을 추적한다. 세 Query는 단일 Decoder에서 병렬로 처리되지만 서로 다른 역할과 감독을 유지한다.

## 3.2. Query의 구조

모든 Query는 Anchor, Content Embedding, Positional Embedding으로 표현된다.

\[
q=(A_q,C_q,P_q)
\]

| 구성 요소 | 의미 |
|---|---|
| Anchor $A_q$ | 대상의 공간적 위치와 크기 |
| Content Embedding $C_q$ | 대상의 특징 표현 |
| Positional Embedding $P_q$ | Anchor에서 생성한 위치 임베딩 |

\[
A_q=(x_q,y_q,w_q,h_q)
\]

- $x_q,y_q$: 정규화된 Bounding Box 중심 좌표
- $w_q,h_q$: 정규화된 너비 및 높이

Detection Query의 Anchor와 Content Embedding은 학습된다. Tracking Query는 이전 프레임의 Anchor를 전달받고 현재 Detection Query로 Content를 갱신한다. Prompt Query는 사용자가 제공한 Bounding Box를 Anchor로 사용한다.

## 3.3. Detection Query

현재 프레임의 사람을 발견한다. 외부 객체 검출기나 미리 지정한 Anchor Shape에 의존하지 않고 학습 가능한 Anchor를 사용한다. Image Visual Token과 Cross-Attention하여 사람의 위치, 자세, 체형, Confidence를 예측한다. 새 사람이 등장하면 결과를 Tracking Query로 승격할 수 있다. Detection Query 자체는 지속적인 ID를 보장하지 않으며 ID 유지는 Tracking Query가 담당한다.

## 3.4. Tracking Query 및 TMCA

Tracking Query는 추적 중인 사람의 정보를 다음 프레임으로 전달한다. Detection Query만 사용하면 움직임이나 외형 변화에 따라 사람과 Query의 대응이 달라질 수 있으므로, 이전 Tracking Query를 유지하면서 현재 Detection Query의 정보를 사용해 갱신한다.

**Track-Modulated Cross-Attention(TMCA)**에서는 Tracking Anchor와 Detection Anchor를 공통 Embedding Space로 투영한다.

\[
\phi(A_q^{trk}),\qquad\psi(A_k^{det})
\]

이들의 내적으로 Attention Weight를 구한다.

\[
w_{qk}=\operatorname{Softmax}_k\!\left(\phi(A_q^{trk})^\top\psi(A_k^{det})\right)
\]

- $q$: Tracking Query 인덱스
- $k$: Detection Query 인덱스
- $\phi,\psi$: Anchor 투영 함수
- $w_{qk}$: 두 Query의 관련성을 나타내는 가중치

안정성을 위해 Top-k Softmax를 사용하며 관련성이 높은 Detection Query의 Content를 가중합한다.

\[
C_q^{trk}=\sum_{k\in\mathcal N_k(q)}w_{qk}C_k^{det}
\]

$\mathcal N_k(q)$는 해당 Tracking Query와 관련성이 높은 상위 $k$개 Detection Query 집합이다. 이전 위치와 현재 검출된 위치의 관련성을 이용해 현재 특징으로 Tracking Query를 갱신한다. 위치·외형·자세가 바뀌어도 ID를 유지하도록 학습한다. **Figure 3(7페이지)**에 TMCA 구조가 제시된다.

## 3.5. Prompt Query

사용자는 특정 사람의 Bounding Box를 입력하여 Tracking Query를 초기화하거나, 놓친 Track을 재초기화할 수 있다.

```text
Frame t
   │
   ├── Detection 실패
   ▼
사용자가 Bounding Box 입력
   ↓
Prompt Query 초기화
   ↓
DETRAM Decoder
   ↓
Tracking Query 유지
   ↓
Frame t+1, t+2, ...
```

매 프레임 박스를 다시 입력할 필요 없이 지정 인물을 이후 프레임에서 추적한다. **현재 직접 지원되는 Prompt는 Bounding Box뿐**이다. Point, Mask, Text는 외부 Mapping 또는 Grounding을 통한 향후 확장 대상으로 설명된다.

## 3.6. Track-Query Memory Bank

장시간 가림이나 자세 변화에서 과거 신원 정보가 약해질 수 있으므로 선택적으로 Key–Value Memory Bank를 이용한다. XMem과 STCN에서 영감을 얻었다.

\[
m_k\in\mathbb R^{B\times(N_tM)\times D},\qquad q_k\in\mathbb R^{B\times N_t\times D}
\]

- $B$: Batch Size
- $N_t$: 활성 Tracking Query 수
- $M$: Memory Slot 수
- $D$: Feature Dimension

현재 Query와 저장된 Memory의 유사도를 구한 뒤 Top-k Softmax를 적용한다.

\[
\alpha_{b,n,m}=\operatorname{Softmax}_m(S'_{b,n,m})
\]

Memory Value 가중합:

\[
mem_{b,n}=\sum_m\alpha_{b,n,m}m_v(b,m)
\]

Memory Feature로 과거 신원 정보를 현재 추적에 반영한다. Memory Bank는 추가 학습 파라미터를 필요로 하지 않으며, 프레임 전체 기록 대신 활성 Track당 고정 Memory Slot을 사용한다.

\[
O(N_tMD)
\]

기본 설정은 **Memory Slot 4개, Decoder 이전 Early Injection**이다.

---

# 4. 구현

## 4.1. 전체 네트워크 구조

| 구성 요소 | 구현 및 기능 |
|---|---|
| Image Encoder | DINOv2 기반 Vision Transformer |
| Decoder | DETR 계열 Transformer Decoder |
| Query | Detection / Tracking / Prompt |
| Cross-Attention | TMCA 및 위치 기반 Conditional Cross-Attention |
| Prediction Heads | Confidence, Pose, Shape, Translation, Bounding Box |
| Temporal Module | Track-Query Memory Bank |
| Track Management | Confidence·IoU 기반 비학습 규칙 |

Encoder와 Decoder는 사전학습된 **SAT-HMR Checkpoint**로 초기화한다.

## 4.2. Image Encoder

입력 크기는 $1288\times1288$이다. 이미지를 $P\times P$ Patch로 분할하고 Patch Embedding과 Positional Encoding으로 Visual Token을 만든다.

\[
K=\frac HP\times\frac WP
\]

Transformer Encoder로 특징을 추출한다.

\[
F=\operatorname{Encoder}(I)
\]

Decoder는 이 Feature에서 여러 사람을 공동 추론한다.

## 4.3. Shared Decoder

Detection Query와 Tracking Query의 Anchor 및 Content Embedding을 각각 Concatenation한다.

\[
A_q=\operatorname{Cat}(A_q^{det},A_q^{trk})
\]

\[
C_q=\operatorname{Cat}(C_q^{det},C_q^{trk})
\]

이는 Query를 평균 내거나 하나의 사람 표현으로 합친다는 의미가 아니라 **서로 다른 Query를 한 Decoder에서 병렬 처리**한다는 의미이다.

Anchor 기반 Positional Embedding:

\[
P_q=\operatorname{MLP}(\operatorname{PE}(A_q))
\]

\[
\operatorname{PE}(A)=\operatorname{Cat}\left(\operatorname{PE}(x),\operatorname{PE}(y),\operatorname{PE}(w),\operatorname{PE}(h)\right)
\]

## 4.4. Self-Attention

Decoder Layer는 Self-Attention과 Cross-Attention으로 구성된다.

\[
Q_q=C_q+P_q,\qquad K_q=C_q+P_q,\qquad V_q=C_q
\]

Query와 Key에 위치 정보를 더하고 Value에는 Content를 사용한다. 같은 Decoder에서 Detection과 Tracking Query가 서로의 정보를 참조할 수 있다.

## 4.5. Conditional Cross-Attention

Conditional-DETR 계열의 공간 조건 기반 Cross-Attention:

\[
Q_q=\operatorname{Cat}\left(C_q,\operatorname{PE}(x_q,y_q)\odot\operatorname{MLP}^{csq}(C_q)\right)
\]

\[
K_{x,y}=\operatorname{Cat}\left(F_{x,y},\operatorname{PE}(x,y)\right)
\]

\[
V_{x,y}=F_{x,y}
\]

$\odot$는 Element-wise Multiplication이다. Content를 MLP에 통과시켜 Positional Embedding을 조절함으로써 의미 정보와 위치 정보를 결합한다. **Figure 3**에서는 TMCA로 Tracking Query를 갱신한 다음 위치·크기 정보를 반영한 Cross-Attention으로 Image Feature를 읽는다.

## 4.6. Prediction Heads

| Head | 출력 |
|---|---|
| Confidence Head | 검출 신뢰도 |
| Pose Head | SMPL Pose $\theta$ |
| Shape Head | SMPL Shape $\beta$ |
| Translation Head | 3D 위치 $\tau$ |
| Bounding Box Head | 2D Bounding Box |

Supplementary에서는 Root Depth 및 3D/2D Joint도 출력 및 감독 대상으로 설명한다. SMPL Pose·Shape는 평균 SMPL 파라미터로부터의 Offset으로 예측해 안정화한다. 각 Decoder Layer의 결과를 다음 Layer에 전달하는 **Iterative Refinement**로 위치, ID Association, 3D Mesh를 개선한다. 이는 별도 외부 모델의 반복 실행이 아닌 네트워크 내부 과정이다.

## 4.7. Track 생성·갱신

별도로 학습 가능한 Tracker는 없지만 Track 관리 규칙은 존재한다.

- 첫 프레임: $Confidence>\tau_{det}$인 Detection을 Tracking Query로 초기화한다.
- 이후 프레임: $Confidence>\tau_{det}$이고 기존 Track들과의 $IoU<\tau_{iou}$이면 새 Track을 생성한다.

| 상태 | 조건 및 동작 |
|---|---|
| Active | 충분한 신뢰도로 추적 중 |
| Inactive | Confidence가 임계값 아래로 떨어져 Query와 Memory 갱신 동결 |
| Removed | Inactive가 허용 프레임 수를 초과하면 제거 |

Inactive Track은 Confidence 또는 Association Score 회복 시 재활성화된다.

\[
L_{inactive}>L_{tol}\ \Rightarrow\ \text{Remove}
\]

따라서 ‘별도의 Tracker가 없음’은 Track 관리 로직까지 전혀 없다는 뜻이 아니다.

## 4.8. Camera Model

카메라 좌표계의 3D Point를 2D로 투영하기 위해 Perspective Projection을 사용한다.

\[
u=\frac{fx}{z}+p_u,\qquad v=\frac{fy}{z}+p_v
\]

- $f$: Focal Length
- $(p_u,p_v)$: Principal Point
- $(x,y,z)$: 카메라 좌표계 3D Point
- $(u,v)$: 이미지 좌표

내부 파라미터가 없는 데이터셋의 기본 설정:

\[
FOV=60^\circ
\]

\[
f=\frac{S_{hr}}{2\tan(FOV/2)},\qquad(p_u,p_v)=\left(\frac W2,\frac H2\right)
\]

$S_{hr}$는 이미지의 긴 변 길이이다.

## 4.9. Loss Function

Detection, Tracking, 3D Reconstruction을 하나의 Objective로 학습한다.

\[
\boxed{\mathcal L=\lambda_{matching}\mathcal L_{matching}+\lambda_{dn}\mathcal L_{dn}+\lambda_{trk}\mathcal L_{trk}}
\]

### 1) Matching Loss

Bounding Box, 투영 Joint, Confidence를 이용한 **Hungarian Matching**으로 Prediction과 GT를 일대일 대응시켜 검출 및 복원을 감독한다.

### 2) Denoising Loss

DN-DETR 방식으로 GT에 Noise를 주고 깨끗한 결과를 복원하도록 학습하여 수렴 속도와 안정성을 개선한다. Figure 2의 Attention Mask는 Matching, Denoising, Tracking/Prompt Query의 감독 역할을 구분하며 Denoising GT 정보의 누출을 방지한다.

### 3) Tracking Loss

각 Tracking Query를 인물별 하나의 GT Reference로 감독한다. Detection Query의 Denoising에는 여러 Noise 증강을 사용하지만 Tracking Query에는 단일 GT Reference를 써 ID 표현의 과도한 정규화를 방지한다.

### 4) 개별 Loss

세 Objective는 GT–Prediction Pair만 다르고 공통 Loss 항을 사용한다.

\[
\begin{aligned}
\mathcal L_{(\cdot)}={}&\lambda_{depth}\mathcal L_{depth}
+\lambda_{pose}\mathcal L_{pose}
+\lambda_{shape}\mathcal L_{shape}\\
&+\lambda_{j3d}\mathcal L_{j3d}
+\lambda_{j2d}\mathcal L_{j2d}
+\lambda_{box}\mathcal L_{box}
+\lambda_{conf}\mathcal L_{conf}
\end{aligned}
\]

Root Depth, SMPL Pose, Shape, 3D Joint, 2D Joint, Bounding Box, Confidence를 감독하며 목표에 따라 L1 또는 GIoU Loss를 쓴다. 제공 PDF에는 모든 개별 가중치 수치가 명시되어 있지는 않다.

## 4.10. 학습 데이터셋

| 데이터셋 | 데이터 규모 | 용도 |
|---|---|---|
| 3DPW | Train 약 17K, Validation 약 8K, Test 약 24K 이미지 | 실제 환경 3D SMPL·비디오 학습 |
| AGORA | Train 약 14K, Validation 약 2K, Test 약 3K 이미지 | 합성 인체 및 3D Mesh 감독 |
| BEDLAM | Train 약 286K 이미지 / 951K 인물, Validation 약 29K 이미지 / 96K 인물 | 3D Reconstruction 및 Tracking |
| COCO | 다운샘플 후 약 16K 이미지 / 66K 인물 | 실제 이미지 2D Joint 감독 |
| PoseTrack21 | Train 약 11K 프레임 / 153K 인물 | 장기 인물 추적 및 ID 유지 |

- BEDLAM Train을 6배 다운샘플링한다. 공식 Test Set이 공개되지 않아 Validation으로 평가하며 Supplementary는 이전 연구에서 사용하는 250개 Validation Image Subset을 명시한다.
- COCO는 NeuralAnnot Pseudo Annotation을 이용하되 모호성과 Noise를 고려하여 투영된 2D Joint만 감독한다.
- PoseTrack21은 Neural Localizer Fields(NLF)로 SMPL Pseudo Annotation을 생성한다.
- MuPoTS-3D는 평가 데이터셋으로 사용한다.

## 4.11. Training Protocol

**Stage 1 — Image-Based Training:** AGORA, BEDLAM, COCO, PoseTrack, 3DPW를 이용하여 이미지 기반 검출·3D 복원을 학습한다.

**Stage 2 — Video-Based Training:** BEDLAM, 3DPW, PoseTrack의 연속 프레임을 이용해 Tracking Query와 시간적 일관성을 학습한다.

\[
T=4\ \text{또는}\ 8
\]

Video Training에는 시간적으로 연속된 GT Track Segment만 사용하며, Clip 내부에서 대상 GT ID가 사라지거나 비연속적으로 Annotation되면 제외한다.

| 항목 | 설정 |
|---|---|
| GPU | NVIDIA A100 × 8 |
| Image Stage Batch Size | GPU당 6 |
| Video Stage Batch Size | GPU당 1 |
| Optimizer | AdamW |
| Learning Rate | $2\times10^{-5}$ |
| Encoder Learning Rate | $1\times10^{-5}$ |
| Scheduler | Cosine Learning Rate |
| Prompt Component LR | Base LR와 동일 |
| Data Augmentation | Random Rotation, Scaling, Horizontal Flipping |
| Video Sampling 비율 | BEDLAM 20%, 3DPW 100%, PoseTrack 100% |

**논문 내 불일치:** 본문에는 Batch Size GPU당 2라고 되어 있지만 Supplementary는 Image Stage 6, Video Stage 1로 구분한다. 재현 시 확인이 필요하다.

---

# 5. 성능

## 5.1. 평가 지표

| 지표 | 의미 | 방향 |
|---|---|---|
| MOTA | FP, FN, ID Switch 종합 추적 정확도 | ↑ |
| IDF1 | 동일 인물 ID 유지 정확도 | ↑ |
| IDP | ID Precision | ↑ |
| IDR | ID Recall | ↑ |
| HOTA | Detection·Association 공동 평가 | ↑ |
| IDs | ID Switch 횟수 | ↓ |
| MPJPE | 3D Joint 위치 오차 | ↓ |
| PA-MPJPE | Procrustes Alignment 후 Joint 오차 | ↓ |
| PVE / MVE | Mesh Vertex 위치 오차 | ↓ |
| FPS | 초당 처리 프레임 수 | ↑ |

3D 오차는 mm 단위이다.

## 5.2. PoseTrack21

Table 1은 **Original Evaluation**과 **Ignore Region Fix** 적용 평가를 분리하므로 섞어 비교하면 안 된다.

### Ignore Region Fix 적용 결과

| Method | MOTA ↑ | IDF1 ↑ | IDP ↑ | IDR ↑ | FPS ↑ |
|---|---:|---:|---:|---:|---:|
| 4DHumans | 56.7 | 70.9 | 87.1 | 59.7 | 0.51 |
| CoMotion | 71.4 | 79.5 | 87.1 | 73.0 | 5.68 |
| DETRAM | 73.6 | 80.6 | 85.0 | 75.1 | 10.87 |
| DETRAM + Prompt | 75.7 | 81.8 | 85.0 | 77.2 | 10.87 |

DETRAM은 CoMotion 대비 MOTA +2.2포인트, IDF1 +1.1포인트이며 IDP는 CoMotion(87.1)이 DETRAM(85.0)보다 높다.

| Method | FN ↓ |
|---|---:|
| 4DHumans | 50,652 |
| CoMotion | 30,394 |
| DETRAM | 26,620 |
| DETRAM + Prompt | 23,369 |

### Prompt 성능

\[
MOTA:73.6\rightarrow75.7,\qquad IDF1:80.6\rightarrow81.8
\]

False Negative 최초 발생 시 GT 유래 Bounding Box를 제공한 **Oracle Protocol**이다. 사용자 박스 오차가 있는 실제 Prompt 성능과 동일하게 해석하면 안 된다.

### Original Evaluation

| Method | MOTA ↑ | IDF1 ↑ |
|---|---:|---:|
| CoMotion | 67.6 | 77.9 |
| DETRAM | 71.0 | 78.3 |

**Figure 5(12페이지)**는 배경 인물 검출, 카메라 이동, 가림, 중복 Track 감소 등 CoMotion 대비 정성적 사례를 제시한다.

## 5.3. MuPoTS-3D

| Method | IDs ↓ | MOTA ↑ | IDF1 ↑ | HOTA ↑ |
|---|---:|---:|---:|---:|
| T3DP | 38 | 62.1 | 79.1 | 59.2 |
| PHALP | 22 | 66.2 | 81.4 | 59.4 |
| ByteTrack | 15 | 73.3 | 84.6 | 63.5 |
| TRACE | 0 | 86.9 | 93.4 | 65.3 |
| CoMotion | 0 | 92.1 | 95.9 | 69.2 |
| DETRAM | 0 | 95.9 | 97.4 | 71.3 |
| DETRAM + Prompt | 0 | 96.5 | 98.2 | 71.6 |

CoMotion 대비 MOTA +3.8포인트, IDF1 +1.5포인트, HOTA +2.1포인트이며 ID Switch 0회이다. Prompt는 MOTA·IDF1을 추가 향상시킨다.

## 5.4. BEDLAM

### Tracking

| Method | IDs ↓ | MOTA ↑ | IDF1 ↑ | HOTA ↑ |
|---|---:|---:|---:|---:|
| Multi-HMR + ByteTrack | 515 | 91.35 | 92.54 | 84.59 |
| MaQ | 128 | 95.15 | 97.01 | 97.73 |
| CoMotion | 885 | 95.22 | 97.39 | 70.94 |
| DETRAM | 37 | 99.18 | 99.37 | 95.90 |

DETRAM은 IDs 37회, MOTA 99.18, IDF1 99.37을 기록하나 HOTA는 MaQ(97.73)보다 낮다. 저자들은 2D Box 정의 차이를 원인으로 제시하지만 별도로 입증된 원인이라고 볼 수는 없다.

### Reconstruction (단위: mm)

| Method | MPJPE ↓ | PA-MPJPE ↓ | PVE ↓ |
|---|---:|---:|---:|
| Multi-HMR + ByteTrack | 81.58 | 42.93 | 88.88 |
| MaQ | 79.88 | 45.56 | 86.40 |
| CoMotion | 55.40 | 24.50 | 62.17 |
| DETRAM | 48.68 | 31.24 | 56.33 |

DETRAM은 MPJPE 48.68 mm, PVE 56.33 mm를 기록하나 PA-MPJPE는 CoMotion의 24.50 mm가 DETRAM의 31.24 mm보다 낮다. 저자들은 CoMotion의 추가 학습 데이터(DanceTrack, InstaVariety)가 영향을 미쳤을 가능성을 언급한다.

### Inference Time

| Method | Time ↓ |
|---|---:|
| MaQ | 0.027초 |
| CoMotion | 0.239초 |
| DETRAM | 0.069초 |

DETRAM은 CoMotion보다 짧지만 MaQ보다는 오래 걸렸다. **Figure 6(13페이지)**은 MaQ와의 Mesh 시각화 비교다. 저자들은 신체 비율·자세 복원 사례를 제시하나, MaQ의 추론 코드가 없어 MaQ 논문에서 시각화 결과를 가져왔다.

## 5.5. Dyna3DPW 및 3DPW

Tracking은 **Dyna3DPW**, Reconstruction은 **3DPW**에서 평가한다.

### Dyna3DPW Tracking

| Method | IDs ↓ | MOTA ↑ | IDF1 ↑ | HOTA ↑ |
|---|---:|---:|---:|---:|
| BEV + ByteTrack | 37 | 93.6 | 79.1 | 59.3 |
| TRACE | 1 | 99.3 | 99.7 | 74.7 |
| MaQ | 0 | 99.6 | 99.8 | 70.8 |
| CoMotion | 0 | 95.7 | 97.8 | 72.8 |
| DETRAM | 0 | 99.8 | 99.9 | 79.1 |

### 3DPW Reconstruction (mm)

| Method | PA-MPJPE ↓ | MPJPE ↓ | PVE ↓ |
|---|---:|---:|---:|
| BEV + ByteTrack | 46.9 | 78.5 | 92.3 |
| TRACE | 50.8 | 80.3 | 98.1 |
| MaQ | 44.7 | 72.6 | 84.9 |
| CoMotion | 37.3 | 60.6 | 71.2 |
| DETRAM | 38.3 | 60.0 | 70.1 |

DETRAM은 Dyna3DPW에서 IDs 0, MOTA 99.8, IDF1 99.9를 기록한다. 3DPW에서 MPJPE·PVE는 CoMotion보다 낮지만 PA-MPJPE는 CoMotion이 낮다.

## 5.6. Ablation Study

### 1) Tracking Query와 Memory Bank의 효과 — Table 4

| Tracking Query | Memory | PoseTrack MOTA ↑ | PoseTrack IDF1 ↑ | BEDLAM IDs ↓ | BEDLAM MPJPE ↓ |
|---|---|---:|---:|---:|---:|
| O | O | 73.6 | 80.6 | 37 | 48.68 |
| O | X | 73.3 | 80.3 | 32 | 47.33 |
| X | O | 68.9 | 78.1 | 37 | 48.27 |
| X | X | 68.3 | 77.1 | 58 | 46.91 |

Tracking Query만 추가하면 PoseTrack MOTA가 68.3에서 73.3으로, Memory까지 추가하면 73.6으로 향상된다. 그러나 BEDLAM MPJPE는 두 요소를 모두 쓰면 48.68 mm로 높아진다. 저자들은 더 많은 가림 인물의 복구에 따른 **Recall–Coverage Trade-off**로 설명하며, MuPoTS-3D 3DPCK@150이 Memory에 의해 95.66%에서 97.84%로 상승했다고 보고한다. Tracking Query만 쓰면 BEDLAM IDs 32회지만 Memory까지 추가하면 37회로, 오래된 Embedding의 Drift 가능성이 언급된다. Memory가 모든 지표를 개선하는 것은 아니다.

### 2) TMCA 및 Spatial Modulation — Table 6

| Variant | PoseTrack MOTA ↑ | IDF1 ↑ | BEDLAM IDs ↓ | MPJPE ↓ |
|---|---:|---:|---:|---:|
| Tracking Query 제거 | 68.3 | 77.1 | 58 | 46.91 |
| TMCA → Direct Propagation | 71.7 | 79.6 | 47 | 49.37 |
| Spatial MLP → Identity Mapping | 70.2 | 77.1 | 305 | 52.73 |
| DETRAM Full | 73.6 | 80.6 | 37 | 48.68 |

TMCA를 단순 전달로 대체하면 PoseTrack MOTA 73.6→71.7, Spatial Modulation을 제거하면 BEDLAM IDs 37→305이다. Tracking Query 보존뿐 아니라 현재 Detection 기반 갱신과 위치 정보 조절이 중요함을 보여준다.

### 3) 인원수에 따른 성능 — Table 5

| 인원수 | 시퀀스 | DETRAM IDF1 ↑ | DETRAM MOTA ↑ | CoMotion IDF1 ↑ | CoMotion MOTA ↑ |
|---|---:|---:|---:|---:|---:|
| 1–10 | 103 | 87.69 | 82.13 | 87.72 | 82.05 |
| 11–20 | 44 | 79.58 | 74.60 | 75.81 | 69.72 |
| 21명 이상 | 23 | 71.66 | 66.83 | 70.67 | 60.53 |

적은 인원에서는 두 방법이 유사하다. 21명 이상일 때 DETRAM은 CoMotion보다 MOTA가 6.30포인트 높지만, DETRAM 자체의 IDF1과 MOTA도 혼잡도가 높아질수록 낮아진다.

## 5.7. Supplementary Material 추가 실험

### 1) PoseTrack21 PCK 평가 — Figure D.1

Full-image 조건:

| Method | PCKn@0.05 ↑ | PCKn@0.1 ↑ |
|---|---:|---:|
| CoMotion | 0.88 | 0.96 |
| DETRAM | 0.84 | 0.96 |

PCKn@0.05에서는 CoMotion보다 낮다. 저자들은 SMPL Joint와 PoseTrack21 Keypoint의 정의·위치 기준 차이를 이유로 제시한다. Figure D.1(22페이지)에 Pelvis와 Head의 체계적인 위치 차이 사례가 있다. 수치 차이 전체가 이 원인 때문이라고 정량적으로 분리 입증된 것은 아니다.

### 2) Memory Slot 및 Injection 위치 — Table D.1

| Slot | 위치 | PoseTrack MOTA ↑ | IDF1 ↑ | BEDLAM IDs ↓ | MPJPE ↓ |
|---|---|---:|---:|---:|---:|
| 4 | Early | 73.6 | 80.6 | 37 | 48.68 |
| 4 | Late | 74.2 | 81.1 | 36 | 49.40 |
| 8 | Early | 73.0 | 79.4 | 61 | 49.82 |
| 8 | Late | 73.0 | 79.4 | 61 | 49.80 |

Early는 Decoder 이전, Late는 Decoder 이후이되 Prediction Head 이전에 Memory를 주입한다. PoseTrack에서는 Late가 Tracking 지표에서 앞서지만 BEDLAM의 MPJPE와 여러 결과를 고려해 Early를 기본 설정으로 선택했다. Slot 8은 4보다 Tracking 성능이 대체로 낮았다.

### 3) Memory Aggregation 방식 — Table D.2

| Method | PoseTrack MOTA ↑ | IDF1 ↑ | BEDLAM MPJPE ↓ |
|---|---:|---:|---:|
| Similarity-Based | 73.6 | 80.6 | 48.68 |
| Cross-Attention | 72.2 | 80.2 | 47.68 |

Similarity-Based는 PoseTrack Tracking 결과가 높지만 BEDLAM 3D 오차는 Cross-Attention이 낮다. 저자들은 데이터셋 간 결과를 고려하여 Similarity-Based를 기본으로 사용한다.

### 4) Identity Embedding Visualization — Figure E.1

BEDLAM, PoseTrack, Dyna3DPW에서 Tracking Query Embedding을 t-SNE로 시각화한다. 동일 GT ID가 시간에 따라 군집을 형성하고 ID별 군집이 분리되는 양상을 보여준다. 이는 정성적 근거이며 장기 추적 성능 자체를 보장하는 정량 지표는 아니다.

## 5.8. 한계점 및 향후 연구

1. **Prompt 종류 제한:** 직접 지원되는 것은 Bounding Box이다. Text, Point, 고수준 의미 명령은 향후 확장 과제다.
2. **Texture 미복원:** SMPL Mesh는 생성하지만 Texture를 복원하지 않아 의복 등의 세밀한 외관 묘사에 한계가 있다.
3. **SMPL 표현 한계:** 손, 얼굴 표정, 정교한 상호작용을 다루려면 SMPL-X 확장이 필요하다.
4. **시간적 일관성 추가 연구:** 명시적 Temporal Smoothness 평가와 Motion Prior는 향후 과제이며 오래된 Memory의 ID Drift 가능성이 있다.
5. **사회적 영향:** 장기 개인 추적 및 3D Mesh 복원은 개인정보 보호와 비동의 감시 문제를 수반하므로 데이터 수집·처리 안전장치와 데이터셋/모델 편향에 대한 고려가 필요하다.

---

## 핵심 결론

이 논문의 기여는 기존에 분리된 **Detection, Tracking, Human Mesh Recovery를 Query 기반 Transformer에 통합**한 것이다. 핵심 설계는 이전 프레임 인물 정보를 **Tracking Query**로 유지하면서 현재 프레임의 **Detection Query**를 통해 **TMCA**로 갱신하는 구조다.

별도의 학습 가능한 Tracking Model 없이 인물별 3D Mesh와 ID를 함께 예측하고 새 인물의 등장 및 사용자 Prompt에도 대응한다. 여러 벤치마크의 Tracking 성능은 높지만 일부 PA-MPJPE, HOTA, 2D PCK, 추론 시간에서는 비교 방법이 높은 성능을 보인다. 따라서 모든 지표의 일관된 개선보다는 **3D Human Reconstruction과 Identity Tracking을 하나의 온라인 프레임워크로 통합하면서 사용자 지정 추적까지 지원한다**는 점에 연구 의의가 있다.

---
