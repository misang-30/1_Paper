- SAM 3D Body: Robust Full-Body Human Mesh Recovery

# 사전 지식

> [!사전 지식] 
> - 토큰
> - 백본
> - MHR (Momentum Human Rig)
> - Token
> - Cross Attention

# 개요

## 1. 특징
- 단일 이미지 기반의 전신 인간 메쉬 복원(HMR)을 위한 프롬프트 기반 모델 SAM 3D Body(S3D)
- 3DB는 신체, 발, 손의 인간 포즈를 추정한다.
- Momentum Human Rig(MHR)를 사용하는 최초의 모델
- 인코더 - 디코더 아키텍처 채택.
- 2D 키포인트 및 마스크를 포함한 보조 프롬프트 지원. -> 사용자 가이드 추론 제공.
- 야외 이미지에서 적용될 때 강인성을 보인다는 것이 이 모델의 큰 특징.

## 2. HMR 모델 개발의 주요 과제
- 데이터셋
	- 고품질 메쉬 주석이 포함된 대규모의 다양한 인간 자세 데이터셋을 수집하는 것은 어렵다. 계산 비용도 많이 든다.
- 모델
	- 현재의 HMR 아키텍처는 신체 및 손 자세 추정에 필요한 서로 다른 최적화 메커니즘을 적절하게 다루지 못하며, 단안 이미지에서 발생하는 불확실성과 모호성을 처리하기 위한 효과적인 학습 전략을 통합하지 못하고 있습니다.
- 본 연구에서는 데이터 엔진을 통해 큐레이션된 대규모의 고품질 인간 자세 데이터를 기반으로 하는 강력한 전신 HMR 모델인 SAM 3D Body(3DB)를 제안합니다.

## 3. 논문의 기여
- 1.모델이 제어 가능한 자세 추정을 위해 선택적인 2D 키포인트, 마스크 또는 카메라 정보에 조건부로 반응할 수 있도록 하는 새로운 프롬프트 기반 인코더-디코더 아키텍처(Kirillov et al., 2023; Ravi et al., 2024)를 제안합니다.

- 이러한 프롬프트 기반 설계는 학습 중 모호하거나 어려운 시나리오에서 상호작용적인 안내를 자연스럽게 촉진하며, 손과 신체 예측을 통합하기 위한 일관된 접근 방식을 제공합니다.

- 2.우리 모델은 공유 이미지 인코더와 신체 및 손을 위한 두 개의 개별 디코더를 활용합니다.

- 이러한 양방향 디코더 설계는 입력 해상도, 카메라 추정 및 감독 목표의 차이로 인해 발생하는 신체 및 손 자세 추정 최적화 간의 충돌을 효과적으로 완화합니다.

- 3.SMPL(Loper et al., 2015) 인간 메쉬 모델에 의존하는 대부분의 이전 연구와 달리, 우리는 골격 자세와 신체 형태를 분리하여 전신 복원에 더 풍부한 제어력과 해석 가능성을 제공하는 새로운 파라메트릭 메쉬 표현인 MHR(Ferguson et al., 2025)을 기반으로 3DB를 구축합니다.

- 4.다양한 인간 자세 및 고품질 주석을 위한 데이터 엔진.

- HMR 방법론은 더 높은 성능을 위해 점점 더 대규모 학습 데이터에 의존하고 있습니다(Goel et al., 2023; Cai et al., 2023; Yin et al., 2025).
- 그러나 고품질 3D 감독 데이터는 여전히 부족하며, 기존의 실제 환경(in-the-wild) 데이터셋은 규모와 다양성 면에서 한계가 있습니다.

- 이를 위해 우리는 다음과 같은 특징을 가진 새로운 데이터 생성 파이프라인을 설계하였습니다: (i) 데이터 품질: 우리의 주석 파이프라인은 기하학적 제약 조건, 파라메트릭 사전 정보, 밀집 키포인트 회귀와 같은 구성 요소의 다양한 조합을 결합하여 고품질 3D 인간 메쉬 주석을 자동으로 생성합니다.

- (ii) 데이터 양: 우리는 대규모 라이선스 스톡 사진 저장소, 다중 뷰 캡처 데이터셋 및 합성 데이터로부터 데이터를 큐레이션합니다.우리는 고품질 주석이 포함된 700만 개의 대규모 이미지를 생성합니다.

- (iii) 데이터 다양성: 우리의 데이터는 실제 환경의 어려운 이미지를 발굴하여 주석 작업을 수행하도록 라우팅하는 VLM 기반 데이터 엔진을 사용하여 다양화됩니다. 이는 희귀한 자세, 어려운 시점 및 다양한 외형을 포괄하여 감독을 위한 보다 다양한 데이터셋을 제공합니다.

---
# 관련 연구
## 1. Human Mesh Model
  

- 가장 널리 사용되는 인간 메쉬 모델은 인간의 신체를 자세와 형태로 파라미터화하는 SMPL(Loper et al., 2015)입니다.

- SMPL-X(Pavlakos et al., 2019)는 여기서 더 나아가 손(MANO, Romero et al., 2022)과 얼굴(FLAME, Li et al., 2017)을 포함합니다.

- SMPL 모델은 형태 공간 내에서 골격 구조와 연조직 질량을 서로 얽히게 하는데, 이는 해석 가능성(예: 파라미터가 항상 뼈 길이와 직접적으로 매핑되지 않음)과 제어 가능성을 제한할 수 있습니다.

- 대안으로, ATLAS(Park et al., 2025)의 개선 버전인 Momentum Human Rig(Ferguson et al., 2025)는 골격 구조와 신체 형태를 명시적으로 분리하며, 우리는 이를 인간 신체 표현으로 채택합니다.


## 2. Human Mesh Recovery (HMR)
- HMR 2.0 Goel et al. (2023)과 같은 초기 HMR 방법들은 관절이 있는 손이나 발을 제외하고 신체만을 예측하는 신체 전용 방법들이었습니다 Kolotouros et al. (2019); Li et al. (2022); Dwivedi et al. (2024).

- 대신, 3DB는 신체, 손, 발을 모두 추정하는 최근의 전신(full-body) 방법론 패러다임을 따릅니다 Baradel et al. (2024); Choutas et al. (2020); Rong et al. (2021); Cai et al. (2023); Wang et al. (2025c).

- 또한 손의 자세와 형태만을 추정하는 부위별 손 메시 복원 방법들도 존재하며 Pavlakos et al. (2024); Potamias et al. (2025), 이는 일반적으로 전신 방법들에 비해 더 정확한 성능을 보입니다.

- 반면, 3DB는 손과 전신 추정 모두에서 강력한 성능을 보여줍니다.  
  

- 프롬프트 기반 추론(Promptable Inference): SAM 제품군 Kirillov et al. (2023); Ravi et al. (2024)에 의해 대중화된 프롬프트 기반 추론은 사용자나 시스템이 제공하는 프롬프트(예: 2D 키포인트 또는 마스크)를 통해 모델 예측을 유도할 수 있게 합니다.

- Wang et al. (2025c)과 유사하게, 본 연구의 접근 방식은 2D 키포인트와 마스크를 포함한 다양한 프롬프트 유형을 지원하며, 프롬프트 토큰을 트랜스포머(transformer) 아키텍처에 직접 통합함으로써 사용자 유도 메시 복원을 가능하게 합니다.  
  

## 3. 데이터 품질 및 주석 파이프라인
- HMR의 주요 병목 현상은 학습 데이터의 품질입니다.

- 많은 데이터셋은 단안 피팅(monocular fitting)을 통해 얻은 의사 정답(pseudo-ground-truth, pGT) 메시에 의존하며 Kolotouros et al. (2019); Kanazawa et al. (2018), 이는 종종 자세, 형태 및 카메라 파라미터에서 체계적인 오류를 포함합니다 Patel and Black (2025).

- 최근 연구 Dwivedi et al. (2024); Wang et al. (2025b)는 주석 노이즈가 보고된 지표와 일반화에 미치는 영향을 강조합니다.

- 이를 해결하기 위해 본 연구에서는 더 높은 충실도의 감독(supervision)을 제공하고자 다중 뷰 데이터셋 Martinez et al. (2024); Khirodkar et al. (2024); Moon et al. (2020)과 합성 데이터를 사용하였습니다.

- 본 연구의 방법은 이러한 통찰을 바탕으로 비전-언어 모델을 사용하여 까다로운 사례를 추출하는 확장 가능한 데이터 엔진을 채택하고, 밀집 키포인트 탐지, 강력한 파라메트릭 사전 정보, 그리고 견고한 최적화를 결합한 다단계 주석 파이프라인을 활용합니다.

---
# SAM 3D Body Model 아키텍처
- 본 연구의 목표는 단일 이미지로부터 3D 인간 메시(즉, MHR 파라미터)를 정확하고 견고하며 상호작용적으로 복원하는 것입니다.

- 이를 위해, 본 연구에서는 풍부한 프롬프트 토큰 세트를 갖춘 프롬프트 기반 인코더-디코더 아키텍처(그림 2 참조)로 3DB를 설계합니다.

- 3DB는 2D 키포인트나 마스크를 입력받을 수 있어 사용자나 하위 시스템이 추론을 유도할 수 있도록 상호작용적으로 설계되었습니다.

``` text
- 1. 사람 이미지 입력
- 2. Image Encoder
- 3. Image Feature
- 4. Prompt + 여러 Token
- 5. Body Decoder / Hand Decoder
- 6. Pose / Shape / Skeleton / Camera
- 7. MHR 3D Human Mesh
  
```
![](SAM3DBody.png)
## 1. 사람 이미지 입력
- 입력은 사람 전체 사진이 아니라 **사람 영역을 crop한 이미지**
- 필요하면 손만 따로 crop한 이미지도 넣을 수 있다.

## 2. Image Encoder
- 역할 : 이미지를 숫자 특징으로 바꾼다
- 사진을 보고 feature를 뽑는다.

## 3. Image Feature
- Image Encoder 에서 나온 것.
- F = ImgEncoder(I) 로 표현한다. (F:특징, I:이미지)
## 4. Prompt + 여러 Token
### 1). Prompt ( = 사용자 입력)
- 아래 2개를 추가적으로 줄 수 있다.
	- 2D KeyPoint (  예를 들어 왼쪽 손목의 위치를 알려주는 것.)  
		- => (x,y,label)  ( ex : (320,180,왼쪽 손목))
	- Segmentation Mask (예를 들어 사람 영역은 여기라고 알려주는 것.)
- 2D Keypoint Prompt : Decoder에서 합쳐진다. (2D keypoint Prompt Token)
- Mask Prompt : Convolution을 거쳐서 Image Feature에서 합쳐진다.

### 2). 여러 Token (= Decoder 입력)
- 아래 4가지 종류가 있다.
	- MHR + Camera Token
	- 2D keypoint Prompt Token
	- Auxiliary 2D / 3D Keypoint Token
	- Hand Position Token
- 논문에서는 이 토큰을 전부 합친다.
- T = [T_{pose}, T_{prompt}, T_{keypoint2D}, T_{keypoint3D}, T_{hand}]
- 1.MHR + Camera Token
	- "사람의 3D mesh와 camera 정보를 예측해라" 라는 역할
	- 아래 4개 출력
		- Pose
		- Shape
		- Skeleton
		- Camera
- 2.2D Keypoint Prompt Token
	- 아까 사용자가 넣은 2D keypoint가 있다면 token으로 바꿔서 Decoder에 넣는다
- 3.Auxiliary 2D / 3D Keypoint Token
	- 관절 정보를 추론하기 위한 token
	- 관절의 위치 추론. 
	- T_keypoint2D, T_keypoint3D로 표현
- 4.Hand Position Token
	- 왼손과 오른손이 **이미지 어디에 있는지 찾는 token**이다.


## 5. Body Decoder / Hand Decoder
### 1). Body Decoder 
- 이미지 특징 F, 모델이 알고 싶은 정보 T
- 출력 O = Decoder (T, F) ( O는 Output Token)
- 여기서 Cross - Attention이 사용된다.
	- Token : "왼쪽 손목 어디 있지?" 가 Cross Attention을 통해
	- Image Feature에서 왼쪽 손목과 관계 있는 부분을 찾아낸다.
- Token이 물어보면 Decoder가 Feature 참고해서 답을 만들어 낸다.
- 첫번째 Output Token인 O를 MLP에 넣는다. 이때 나온 파라미터 값을 MHR Parameter라 한다.
	- MHR 파라미터 : Human Mesh를 만들기 위한 숫자 집합
	- MHR Parameter : theta = MLP(O_0)
	- theta = {P,S,C,Sk}
		- P = Pose , 관절이 어떻게 꺾여 있는가
		- S = Shape, 몸의 표면/체형이 어떻게 생겼는가
		- C = Camera, 카메라와 사람의 관계
		- Sk = Skeleton, 사람의 뼈대 구조

- MHR Parameter(Theta)를 MHR(Moment Human Rig)에 넣어서 3D Human Mesh 복원.
### 2). Hand Decoder
- Body Decoder 만으로 Full-Body Mesh를 만들 수 있으나,
- 손은 작고 복잡하므로(손은 아주 작은 영역), 손을 따로 Crop해서 Image Encoder에 넣고 Hand Decoder에 넣어서 더 정밀한 Hand Pose를 한 번 더 구한다.
- Body Decoder와 Hand Decoder를 분리한 것도 body와 hand가 입력 해상도나 supervision 등의 특성이 다르기 때문이다.

## 6. Body + Hand 결과 합친다.
- Hand Decoder에서 나온 Hand Pose를 Body Decoder가 만든 결과에 붙이면, 손목 위치가 바뀌면서 **팔꿈치 같은 주변 관절이 이상해질 수 있었다.**
- 그래서 연구진은 아래 2개를 Prompt로 하여 Body Decoder에 다시 입력시켜서 Refined Full-body Pose 를 얻는다.
	- 1). Hand Decoder가 예측한 Wrist(손목) 위치
	- 2). Body Decoder가 예측한 Elbow 위치




---
# 모델 학습 및 추론
## 1. Model Training

- SAM 3D Body는 여러 목표를 동시에 학습하는 **multi-task loss**를 사용한다.

-  하나의 loss만 쓰는 게 아니라 **관절 위치, MHR parameter, 손 위치 등 여러 항목의 loss를 합쳐서 학습**한다. 
- 일부 loss, 특히 3D keypoint loss는 처음부터 강하게 적용하지 않고 학습이 진행되면서 weight를 점차 키우는 warm-up 방식도 사용한다. 
- 또한 promptable 모델이기 때문에 학습 중에도 keypoint나 mask prompt를 무작위로 제공하면서 interactive inference 상황을 모사한다.


### 1). 2D / 3D Keypoint Loss

- 모델이 예측한 관절 위치와 정답 관절 위치의 차이를 학습한다.
	- 2D keypoint: 이미지 상의 관절 위치
	- 3D keypoint: 3차원 공간상의 관절 위치
	- Loss: L1 loss
	- Body 3D keypoint는 pelvis 기준으로 정규화
	- Hand 3D keypoint는 wrist 기준으로 정규화

- 또 모델이 각 관절 예측의 불확실성도 학습해서, 확신이 낮은 관절은 loss에 다르게 반영할 수 있도록 한다. 사용자가 직접 keypoint prompt를 제공한 경우에는 그 위치를 더 잘 맞추도록 loss weight를 높인다. 
### 2). Parameter Loss

- SAM 3D Body가 예측하는 MHR parameter 중 "Pose, Shape"를 정답 값과 비교해서 학습한다.
- 이때 **L2 regression loss**를 사용한다.
- 추가로 사람이 실제로 취하기 어려운 관절 자세가 나오지 않도록 **joint limit penalty**도 사용한다.
- 예를 들어 팔꿈치가 비정상적인 방향으로 꺾이는 자세를 억제하는 역할이라고 보면 돼. 

### 3). Hand Detection Loss

- SAM 3D Body는 손 위치도 자체적으로 찾는다.
- 그래서 hand bounding box를 예측하고, 이를 학습하기 위해 아래 2개 사용.
	- GIoU Loss
	- L1 Loss
- 또 손 bounding box 예측에 대한 uncertainty도 예측한다. 손이 가려져 있는 sample에서는 추론 시 hand decoder를 끌 수도 있다.


## 2 Full-body Inference

실제 추론에서는 먼저 **Body Decoder 결과를 기본 full-body 결과**로 사용한다.

```
Image
  ↓
Encoder
  ↓
Body Decoder
  ↓
Full-body MHR
```

손이 검출되면 Hand Decoder 결과를 추가로 사용해서 손 자세를 더 정밀하게 만든다. 

### 1) Body Decoder + Hand Decoder 결합

처음에는 Hand Decoder 결과를 Body Decoder의 full-body 결과에 바로 합치는 방식을 생각할 수 있다.

그런데 논문에서는 이렇게 단순히 합치면 **손목 주변의 kinematic chain이 깨질 수 있고, 특히 elbow 위치가 부정확해질 수 있다**고 설명한다. 

그래서 한 번 더 보정한다.

### 2) Wrist / Elbow Prompt를 이용한 Refinement

- Hand Decoder에서 얻은 wrist 위치와 Body Decoder에서 얻은 elbow 위치 를 다시 prompt로 사용한다.
- 이렇게 해서 손 결과를 억지로 붙이는 대신, **Body Decoder가 손목과 팔꿈치 위치를 고려해서 전체 자세를 다시 계산하게 한다.**

### 3) Final Full-body Configuration

마지막에는 예측된 local MHR parameter들을 MHR의 **kinematic tree** 구조에 따라 결합해서 최종 full-body mesh를 만든다.


---
# Data Engine for Diversity

## 1. 왜 데이터 엔진이 필요한가

- HMR 모델을 잘 만들려면 **정확한 3D mesh annotation이 붙은 다양한 사람 이미지**가 많이 필요하다.
- 그런데 문제가 있다.
	- 정확한 3D mesh annotation을 만드는 것은 비용이 많이 든다.
	- 비디오에서 많은 프레임을 뽑아 학습 데이터를 늘릴 수는 있지만, 아래 것들이 반복될 가능성이 높다.
	    - 비슷한 사람
	    - 비슷한 자세
	    - 비슷한 배경
	    - 비슷한 촬영 조건

- 즉, **데이터 양은 많아져도 다양성은 부족할 수 있다.** 그래서 SAM 3D Body는 단순히 이미지를 많이 모으는 게 아니라, **모델이 어려워할 만한 이미지를 골라내는 데이터 엔진**을 만든다.


## 2. 핵심 아이디어: VLM 기반 Data Mining

- 이 데이터 엔진의 핵심은 **VLM(Vision-Language Model)**이다.

- 무작위로 이미지를 뽑는 것이 아니라, “어떤 이미지가 현재 모델에게 어렵고, 학습 가치가 높은가?”를 VLM을 이용해 찾아낸다.

```
많은 이미지
   ↓
VLM
   ↓
어려운 이미지 선별
   ↓
Annotation
   ↓
학습 데이터에 추가
```


## 3. 어떤 이미지를 어려운 이미지로 보는가

- VLM은 다음과 같은 조건의 이미지를 찾는다.
	- **Occlusion**
	    - 사람 일부가 물체나 다른 사람에게 가려진 경우
	- **Unusual poses**
	    - 곡예, 춤처럼 드물고 복잡한 자세
	- **Interaction**
	    - 사람-물체 상호작용
	    - 사람-사람 상호작용
	- **Extreme scale**
	    - 사람이 카메라에서 너무 크거나 너무 작게 보이는 경우
	- **Low visibility**
	    - 저조도
	    - motion blur
	    - 일부만 보이는 경우
	- **Hand-body coordination**
	    - 손과 몸의 움직임이 강하게 연결된 경우
	    - 예: 수어, 스포츠
## 4. 단순한 한 번짜리 선별이 아니다

- 이 데이터 엔진은 **반복적으로 개선되는 구조**다.

- 논문에서는 현재 버전의 SAM 3D Body를 먼저 어려운 이미지에 적용한 뒤, **어떤 이미지에서 모델이 실패하는지 분석**한다. 
- 즉 **모델이 못하는 사례를 다시 찾아서 학습시키는 반복 구조**다.

```
현재 SAM 3D Body
      ↓
어려운 이미지에서 평가
      ↓
실패 사례 확인
      ↓
사람이 실패 유형을 몇 단어로 설명
      ↓
그 설명을 VLM prompt로 사용
      ↓
비슷한 어려운 이미지 추가 탐색
      ↓
Annotation
      ↓
학습 데이터에 추가
```


## 5. Failure Analysis

- 논문에서는 failure analysis를 **semi-manual**하게 한다.

	- 현재 모델을 annotation된 이미지에 적용한 뒤,
	- keypoint 위치 오류를 이용해서 어려운 사례를 찾고
	- 사람이 그 이미지를 직접 보고
	- 몇 개의 단어로 실패 상황을 설명한다.

- 그 단어와 이미지를 이용해 **VLM용 text prompt를 만든다.** 

- 예를 들어 모델이 뒤집힌 자세를 자주 틀린다고 하면,

```
"inverted body"
"unusual athletic pose"
"severe self-occlusion"
```

- 같은 식의 개념이 다음 데이터 탐색 기준이 되는 구조라고 이해하면 된다.


## 6. 최종 목적

- 이 데이터 엔진의 목적은 단순히 데이터 수를 늘리는 게 아니다.

- 핵심은 **수천만 장의 이미지 중에서 모델 학습에 가치가 높은 어려운 사례를 효율적으로 찾아내는 것**이다.

- 그래서 annotation 비용을 아무 이미지에나 쓰는 것이 아니라, **모델이 아직 잘 못하는 다양한 사례에 집중해서 사용**한다. 


---
# Data Annotation and Mesh Fitting
- SAM 3D Body를 학습시킬 정답 3D 사람 mesh를 어떻게 만들어냈는가?
- 저자들은 실제 모든 이미지에 사람이 직접 3D mesh를 그릴 수 없으니까, 여러 방법을 조합해 **pseudo-ground truth(pGT)**를 만든다.
- 전체 흐름
```
학습에 쓸 사람 이미지
        ↓
사람이 2D 관절 확인/수정
        ↓
더 촘촘한 2D Keypoint 생성
        ↓
초기 MHR Mesh 예측
        ↓
이미지의 관절과 Mesh가 잘 맞도록 최적화
        ↓
고품질 MHR Mesh
        ↓
SAM 3D Body 학습용 정답(pseudo-GT)
```

- 그리고 데이터 종류에 따라서 방법을 두 가지로 나눈다.
```
Single Image → Single-Image Mesh Fitting
Multi-view   → Multi-View Mesh Fitting
```

## 1. Manual Annotation

- 먼저 **사람이 직접 확인하는 단계**다.
- 하지만 처음부터 사람이 모든 관절을 찍는 건 아니다.

### 1). 현재 SAM 3D Body가 먼저 관절을 예측

### 2). 사람이 확인하고 틀린 곳을 수정

- 전문 annotator가 모델의 결과를 보고  "손목 위치가 조금 틀렸네." 하면 직접 고쳐준다.

### 3). Visibility도 표시
- 각 관절이 이미지에서 제대로 보이는지도 표시한다.

- 예를 들어
	- 정상적으로 보임 → visible
	- 심하게 가려짐 → not visible
	- motion blur로 위치 판단 불가능 → not visible

- 논문에서는 약 50% 이상 가려지는 등의 경우 정확한 위치를 잡기 어렵다면 not visible로 처리한다고 설명한다. 

## 2. Single-Image Mesh Fitting

- 이제 **사진 한 장만 가지고 3D mesh 정답을 만드는 과정**이야.


### 1). 초기 MHR Mesh를 만든다

- 먼저 현재 SAM 3D Body를 이용해서 "이 사람은 아마 이런 3D 몸일 것이다." 라는 초기 MHR parameter를 얻는다.

- 동시에 별도의 고성능 keypoint detector로 **595개의 dense 2D keypoint**도 얻는다. 

```
Image
 ├→ SAM 3D Body → 초기 MHR Mesh
 │
 └→ Keypoint Detector → 595개 2D Keypoint
```

### 2). 왜 무려 595개 Keypoint인가?

- 우리가 흔히 pose estimation에서 보는 건 수십 개 관절이다.

- 그런데 여기서는 몸의 **표면 형태와 손 자세까지 정확하게 맞추기 위해 595개의 촘촘한 지점**을 사용한다.

- 논문에서는 다양한 body shape와 hand pose를 표현하기 위한 dense keypoint 구성이라고 설명한다. 

### 3). Mesh Fitting



- 초기 3D mesh를 이미지에 투영해봤는데, 안 맞으면  MHR의 Pose, Shape, Skeleton 등parameter를 조금씩 바꿔서 **3D mesh를 이미지 속 사람에게 맞춘다.**

- 이게 **Mesh Fitting**.
- 즉, **3D 인체 모델의 parameter를 조정해서 관측된 사람과 최대한 일치하게 만드는 과정**

### 4). 무엇을 기준으로 맞추나?

- 여러 Loss를 사용한다.
#### (1). 2D Keypoint Loss
- 3D Mesh의 관절을 다시 사진 위로 투영한다.
- 그리고 detector가 찾은 실제 2D keypoint와 비교한다.
- 둘의 거리를 줄인다.
- 논문에서는 **L2 distance**를 사용한다. 


#### (2). Initialization-Anchored Regularization

- 그런데 keypoint만 맞추다 보면 mesh가 이상하게 변할 수 있다.
- 예를 들어, Keypoint만 억지로 맞추다 보니 팔이 비정상적으로 길어질 수 있다.
- 그래서, **처음 SAM 3D Body가 예측한 결과에서 너무 멀리 가지 마라** 라는 제약을 건다.
- MHR parameter와 3D keypoint가 초기 예측에서 지나치게 벗어나면 penalty를 준다.
#### (3). Pose & Shape Prior

- 또 사람 몸이 실제 인간답게 생겨야 한다.

- 예를 들어 아래 같은 건 keypoint를 잘 맞추더라도 좋은 mesh가 아니다.
```
팔꿈치가 뒤로 180° 꺾임
다리가 비정상적으로 길어짐
몸통이 이상한 형태
```

- 그래서 , "사람의 Pose와 Shape는 대체로 이런 범위 안에 있어야 한다." 라는 **prior(사전 지식)**를 적용한다.
- 논문에서는 learned Gaussian Mixture prior와 L2 regularization을 사용한다.

### 5). Dense Keypoint Detector는 어떻게 만드나?


- Dense keypoint detector는 Transformer encoder-decoder 구조인데, 단순히 이미지 픽셀만 보는 게 아니라 **앞서 사람이 수정해놓은 sparse keypoint도 힌트로 사용한다.**

- 처음에는 정확한 3D 데이터셋으로 학습한 뒤:

```
3D Dataset
 ↓
Dense Keypoint Detector 학습
 ↓
일반 이미지에 적용
 ↓
MHR Mesh Fitting
 ↓
Mesh를 다시 2D Dense Keypoint로 투영
 ↓
Dense Detector 다시 학습
```

- 이 과정을 반복한다.
- 논문에서는 이 iterative training을 **두 번 수행**했다고 한다. 

## 3. Multi-View Mesh Fitting

- Single image에는 근본적인 문제가 있다.
- 예를 들어 사진 한 장만 보면, 팔이 앞으로 나온 건지 뒤로 나온 건지 애매할 수 있다.
- 이걸 **depth ambiguity**라고 한다.
- 가려져 있는 신체도 한 장으로는 볼 수 없다. 그래서 multi-view 데이터가 있는 경우에는 여러 카메라를 함께 사용한다. 



### 1). 여러 카메라에서 2D Keypoint를 얻는다

예:

```
Camera 1 ──→ 사람 ←── Camera 2
                 ↑
              Camera 3
```

각 카메라에서:

```
Camera 1 → 2D Keypoints
Camera 2 → 2D Keypoints
Camera 3 → 2D Keypoints
```

를 구한다.


### 2). Triangulation으로 3D Keypoint를 만든다

- 여러 카메라에서 같은 관절을 보고 있기 때문에, 실제 3D 손목 위치를 계산할 수 있다.

```
Camera A의 손목 위치
       +
Camera B의 손목 위치
       +
Camera C의 손목 위치
       ↓
   Triangulation
       ↓
실제 3D 손목 위치
```


- 논문에서는 synchronized 2D keypoint들을 triangulation해서 **sparse 3D keypoint**를 얻는다. 

### 3). 그 3D Keypoint에 MHR Mesh를 맞춘다

- Single image에서는 주로 2D Keypoint ↔ Mesh Projection 을 비교했다면,
- Multi-view에서는 추가로 실제 계산된 3D Keypoint, MHR Mesh의 3D Joint 를 직접 비교할 수 있다.
- 그래서 훨씬 정확하다.
- **3D Keypoint Loss**는 이 둘의 L2 distance다. 


### 4). Temporal Smoothness도 사용

- Multi-view 데이터가 영상이면 앞뒤 프레임도 있다.

- 예를 들어: 아래 처럼 자연스럽게 변해야 한다.

```
Frame 1   Frame 2   Frame 3
   팔       팔       팔
  30°      31°      32°
```


- 그런데 추정 결과가 30° → 95° → 31°처럼 갑자기 튄다면 이상하다.

- 그래서 **Temporal Smoothness Loss**를 사용해서 갑작스러운 pose 변화를 억제한다. 

- 또 shape와 skeleton처럼 한 사람에게 거의 고정되어야 하는 parameter는 여러 frame을 함께 사용해서 최적화한다. 

### 4. 결국 왜 이렇게 복잡하게 하느냐?

- **SAM 3D Body 자체를 학습하려면 정답 3D mesh가 필요한데, 인터넷에 있는 일반 사람 사진에는 정답 3D mesh가 없다.**

- 그래서 저자들이 아래를 조합해서 **최대한 정답에 가까운 3D MHR mesh를 만들어 학습용 label로 사용한 것**이야. 

> 사람의 2D 관절 annotation + 기존 3DB 예측 + dense keypoint + 최적화 + multi-view geometry


---
# 학습 데이터셋


---
# 평가


---
# 결론


---
# Author Contributions


---
# Evaluating 3DB Prompt Following



---
# Limitations


