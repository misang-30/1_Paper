- SAM 3D Body: Robust Full-Body Human Mesh Recovery

# 사전 지식

> [!사전 지식] 
> - 토큰
> - 백본
> - MHR (Momentum Human Rig)
> - Token
> - Cross Attention
> - SMPL

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
# Training Datasets


- “SAM 3D Body를 어떤 종류의 데이터로 학습했는가”를 설명하는 장
- 핵심은 **한 종류의 데이터셋만 쓰지 않고, 서로 장단점이 다른 데이터들을 섞었다**는 것.

- 저자들은 학습 데이터를 크게 네 부류로 나눈다.

## 1. Single-view in-the-wild

- 실제 환경에서 찍힌 **일반 단일 이미지**들이다.
	- AI Challenger
	- MS COCO
	- MPII
	- 3DPW
	- SA-1B 일부
- 이 데이터들의 역할은 주로 아래를 모델이 경험하게 하는 것
	- 다양한 사람 외형
	- 다양한 자세
	- 다양한 배경
	- 다양한 촬영 조건
- 즉 **현실 세계 다양성 확보용**이야.

## 2. Multi-view consistent

- 여러 카메라가 같은 사람을 동시에 보는 **multi-view 데이터셋**이다.
- 사용한 데이터는:
	- Ego-Exo4D
	- Harmony4D
	- EgoHumans
	- InterHand2.6M
	- DexYCB
	- Goliath

- 여러 시점이 있으니까 “이 관절이 실제 3D 공간에서 어디에 있는가?”를 더 정확하게 알 수 있다.

- 그래서 **3D geometry의 신뢰도를 높이는 데이터**라고 보면 된다.

## 3. High-fidelity Synthetic

- 실제 이미지뿐 아니라 **합성 데이터**도 사용한다.

- 논문에서는 Goliath의 photorealistic synthetic extension을 사용하고, 수백만 프레임 규모의 synthetic 데이터를 활용한다고 설명한다. 
- 이 데이터에는 정확한 MHR ground truth가 있고, 다양한 사람, 옷, 상황이 포함된다. 
- 합성 데이터의 장점은 **정답 3D 값이 정확하다는 것**
- 실제 사진은 다양하지만 GT가 부정확할 수 있고, synthetic은 현실성은 조금 떨어질 수 있지만 정답은 정확하다.
- 그래서 둘을 섞는다.


## 4. Hand datasets

- 손 성능을 높이기 위해 별도의 **hand-centric dataset**도 사용한다.
- 논문의 Table 1에서 `*`가 붙은 데이터가 hand decoder 학습에도 사용된다.
- 대표적으로:
	- InterHand
	- Re:InterHand
	- Goliath
	- Synthetic
- hand decoder를 학습할 때는 **wrist-truncated hand sample**도 제공한다. 즉 손목 아래쪽 손 영역을 중심으로 따로 학습시키는 것.


## 5. Table 1

|Dataset|Images/Frames|특징|
|---|---|---|
|MPII|5K|single-view|
|MS COCO|24K|single-view|
|3DPW|17K|single-view|
|AI Challenger|172K|single-view|
|SA-1B|1.65M|대규모 in-the-wild|
|Ego-Exo4D|1.08M|multi-view|
|DexYCB|291K|multi-view|
|EgoHumans|272K|multi-view|
|Harmony4D|250K|multi-view|
|InterHand|1.09M|hand|
|Re:InterHand|1.50M|hand|
|Goliath|966K|대규모 multi-view|
|Synthetic|1.63M|synthetic GT|

## 6. 왜 이렇게 섞었는가

- 이 장의 핵심은 데이터 종류마다 서로 부족한 점을 보완한다는 것.

```
Single-view in-the-wild
→ 현실 환경의 다양성

Multi-view
→ 정확한 3D geometry

Synthetic
→ 정확한 ground truth

Hand dataset
→ 정밀한 손 자세
```

- 그래서 최종적으로는

> **일반 body pose + hand pose + interaction + in-the-wild 환경을 모두 커버하기 위해 여러 종류의 데이터셋을 함께 사용했다.**

- 라는 게 논문의 요지. 

---
# Evaluation
- **Evaluation**은 목차 기준으로 보면 크게 **“기본 성능 → 새로운 환경 일반화 → 손 성능 → 상황별 세부 분석 → 정성 평가 → 사람 선호도 평가”** 순서
- 평가에 앞서 논문은 기본 metric으로 **MPJPE, PA-MPJPE, PVE, PCK**를 사용한다. 
- SMPL 기반 데이터셋에서 평가할 때는 MHR mesh를 SMPL 형식으로 매핑한다. 
- 또 모델은 `3DB-H`와 `3DB-DINOv3` 두 버전을 평가하고, 입력 이미지는 512×512로 사용한다. 

| Section                | 뭘 평가하나                    |
| ---------------------- | ------------------------- |
| **1 Common Datasets**  | 기존 표준 benchmark 성능        |
| **2 New Datasets**     | 처음 보는 환경에서 generalization |
| **3 Hand Pose**        | 손 자세 추정 성능                |
| **4 2D Categorical**   | 상황별 2D keypoint 성능        |
| **5 3D Categorical**   | 상황별 3D mesh/pose 성능       |
| **6 Qualitative**      | 눈으로 mesh 품질 비교            |
| **7 Human Preference** | 사람이 보기에도 더 좋은지 평가         |
### 1. Evaluating Performance on Common Datasets

- 기존 HMR 논문들이 많이 쓰는 **표준 benchmark에서 다른 모델들과 비교**한다.

- 사용 데이터셋은:
	- 3DPW
	- EMDB
	- RICH
	- COCO
	- LSPET

- 비교 대상에는 HMR2.0b, CameraHMR, PromptHMR, SMPLer-X, NLF와 WHAM, TRAM, GENMO 같은 video 기반 모델도 포함된다. 
- 논문의 목적은 여기서 **기존 표준 benchmark에서도 SAM 3D Body가 잘 동작하는가**를 확인하는 것이다. 
### 2. Evaluating Performance on New Datasets


- 기존 benchmark에서 잘하는 것만으로는 **새로운 환경에서도 잘하는지 알 수 없기 때문에**, 학습 때 보지 않은 새로운 domain에서 generalization을 평가한다.

- 사용한 데이터는:
	- Ego-Exo4D
	- Harmony4D
	- Goliath
	- Synthetic
	- SA1B-Hard



- 여기서 **leave-one-out** 방식도 사용한다.

- 즉 특정 데이터셋에서 평가할 때. 그 데이터셋은 학습에서 빼고 → 평가해서 정말 처음 보는 domain에서도 잘하는지 본다. 
- Full dataset으로 학습했을 때 결과도 같이 보여줘서 비교한다.

## 3. Evaluating Hand Pose Estimation Performance

- SAM 3D Body의 특징 중 하나가 **손까지 잘 추정하는 full-body 모델**이니까 손 성능을 별도로 평가한다.

- 사용 benchmark는 **FreiHand**다.

- 여기서는 full-body 출력 전체가 아니라 **Hand Decoder의 출력**을 사용해서 hand-only 모델들과 비교한다.

- 평가지표는:
	- PA-MPVPE
	- PA-MPJPE
	- F@5
	- F@15

- 핵심 질문은:
> **몸 전체를 추정하는 모델인데도 손 전용 모델 수준의 손 정확도를 낼 수 있는가?**

## 4. Evaluating 2D Categorical Performance

- 여기서는 단순 평균 성능 말고,**어떤 종류의 이미지에서 잘하고 못하는가?** 를 분석한다.

- SA1B-Hard를 **24개 category**로 나눈다.

- 큰 그룹은:
	- Body Shape
	- Camera View
	- Hand
	- Multi-person
	- Pose
	- Visibility
- 예를 들면 아래 같은 상황 별로 따로 본다.
	- 옆/뒤에서 본 사람
	- 아래에서 올려다본 사람
	- 손가락이 겹침
	- 물체를 잡고 있음
	- 사람이 서로 겹침
	- 몸이 뒤집힌 자세
	- 다리를 벌린 자세
	- 손/발이 가려짐
	- 몸 일부가 잘림

- 평가는 **PCK / Avg-PCK**를 사용한다.

## 5. Evaluating 3D Categorical Performance

- 여기는 **3D mesh 기준 상황별 분석**.
- 3D 평가는 single-view pseudo-GT가 부정확할 수 있어서, **multi-view와 synthetic 데이터 기반으로 고품질 평가셋**을 별도로 구성했다. 
- 총 **28개 category**로 나눈다.
- 예:
	- depth ambiguity
	- orientation ambiguity
	- scale ambiguity
	- FOV
	- close interaction
	- hard / very hard pose
	- BMI / body shape
	- truncation
	- bottom-up / top-down viewpoint


- 여기서는 아래 지표를 사용해서 상황별 성능을 비교한다.
	- PVE
	- MPJPE
	- PA-MPJPE

- 즉, 평균값 하나만 보지 말고, **어려운 자세나 시점에서도 실제로 강한가**를 보는 장이다.

## 6. Qualitative Results

- 여기부터는 숫자 대신 **눈으로 직접 결과를 비교하는 정성 평가**다.

- SA1B-Hard의 어려운 이미지들에 대해 SAM 3D Body와 여러 SOTA 모델의 mesh 결과를 나란히 보여준다.

- 특히, 아래 같은 부분의 시각적 복원 품질을 비교한다. 
	- 복잡한 pose
	- 다양한 body shape
	- occlusion
	- 팔/다리
	- 손

-  hand crop만 있는 경우에는 Hand Decoder가 만든 mesh 결과도 별도로 보여준다.

## 7. Human Preference Study

- 마지막은 **사람이 직접 어느 결과가 더 좋아 보이는지 고르는 평가**다.

- 왜 이걸 하냐면, MPJPE 같은 숫자가 낮다고 해서 사람이 보기에도 항상 더 자연스럽다고 할 수는 없기 때문



- 총 **7,800명**이 참여했고, 6개의 baseline과 SAM 3D Body를 pairwise comparison했다. 참가자는 원본 이미지와 두 모델의 3D reconstruction을 보고 "어느 3D 모델이 원본 사람을 더 잘 표현했는가?”를 선택한다. 

- 평가는 아래 지표 사용

	- Win Rate
	- Vote Share


---
# Conclusion

## 1. SAM 3D Body는 무엇인가

- 저자들은 **몸과 손을 함께 다루는 robust HMR 모델인 3DB**를 제안했다고 정리한다.

- 핵심 구성은:
	- **Momentum Human Rig(MHR)** 기반의 parametric body model
	- 유연한 **encoder–decoder architecture**
	- **2D keypoint / mask prompt** 지원



- 즉 단순히 사람 몸을 복원하는 모델이 아니라,
 **사람 전체 + 손까지 복원하면서, prompt로 추론을 유도할 수 있는 HMR 모델**이라는 것.


## 2. 가장 중요한 발전점은 supervision pipeline

- Conclusion에서 저자들이 특히 강조하는 건 **모델 구조만큼이나 학습 데이터와 supervision 방식이 중요했다는 점**.

- 기존처럼 noisy한 monocular pseudo-ground-truth에만 의존하지 않고, 아래와 같은 방식을 사용.
	- multi-view capture
	- synthetic data
	- scalable data engine
	- 어려운 샘플을 적극적으로 mining하고 annotation

- 즉 저자들 주장은 **좋은 모델 구조 + 더 깨끗하고 다양한 supervision**이 같이 있어야 robust한 HMR이 된다는 거야.

### 3. Generalization을 중요하게 봄

- 이렇게 만든 데이터와 학습 파이프라인 덕분에 curated benchmark 안에서만 잘하는 게 아니라, **새로운 환경에서도 더 잘 일반화할 수 있었다**고 정리한다. 
- “우리는 그냥 benchmark 점수를 올린 게 아니라, unseen domain에서도 강한 모델을 만들려고 했다.”는 메시지를 강조.

### 4. Hand Decoder의 의미

- 또 별도의 **Hand Decoder**를 둬서 hand crop을 입력으로 사용하고, 그 결과 **손 전용 SoTA 모델과 비교할 만한 hand pose estimation 성능**을 냈다고 정리한다. 
- 이게 SAM 3D Body의 특징 중 하나이다.
- 보통 full-body 모델은 손에서 약한데, 이 논문은
- **full-body를 유지하면서도 손 성능을 높이기 위해 hand decoder를 따로 둠** 이라는 전략을 썼다.

---
# Author Contributions

## 1. Model

- **Xitong Yang**이 전체 **model lead**를 맡았다.그 외 모델 쪽 역할은 다음처럼 나뉜다.

|연구자|담당|
|---|---|
|Xitong Yang|Model Lead|
|Jinkun Cao|Hand pose, 모델 개선|
|Jinhyung Park|MHR integration, 모델 개선|
|Nicolas Ugrinovic|Multi-person interaction|
|Jiawei Liu|SAM 3D unification|

- 즉 우리가 앞에서 봤던 **SAM 3D Body architecture, MHR 적용, hand pose, multi-person 처리** 같은 부분을 여러 사람이 나눠 개발. 

## 2. Data

- 데이터 파이프라인도 세부적으로 담당자가 나뉘어 있다.

|연구자|담당|
|---|---|
|Devansh Kukreja|Data engine 및 infrastructure|
|Don Pinkus|Manual annotation tool|
|Taosha Fan|Multi-view mesh fitting|
|Soyong Shin|Single-view mesh fitting, dense keypoint detector|
|Jinhyun Park|MHR mesh fitting|
|Jinkun Cao|Hand 및 whole-body data|


- 즉 **5~7장의 데이터 관련 내용 자체가 여러 연구자의 세부 프로젝트를 합친 것**이라고 보면 된다.

## 3. Evaluation

- 평가도 역할이 분리되어 있다.

|연구자|담당|
|---|---|
|Xitong Yang|Internal / External Benchmark|
|Jinkun Cao|Hand Pose Evaluation|
|Jiawei Liu|Human Preference Study, Visualization|
|Nicolas Ugrinovic|Multi-person Evaluation|

### 4. Leadership and XFN

- 프로젝트 리더십 역할은 다음 연구자들이 담당했다고 적혀 있다.

- **Kris Kitani, Anushka Sagar, Piotr Dollar, Matt Feiszli, Jitendra Malik** 
- 여기서 논문은 `XFN`을 별도로 풀어서 정의하지는 않는다.
- 일반적인 연구·기업 문맥에서는 **cross-functional**, 즉 여러 팀·직군 사이의 협업을 가리킬 때 자주 사용하는 약어.


---
# Evaluating 3DB Prompt Following

> **“SAM 3D Body에 prompt를 주면 실제로 그 prompt를 잘 따라가고, 성능도 좋아지는가?”** 를 확인하는 부분.

- 논문에서는 크게 **2D Keypoint Prompt**와 **Mask Prompt** 두 가지를 평가한다. 

## 1. 2D Keypoint Prompt

- 먼저 사용자가 이미지 위에서 특정 관절 위치를 알려주는 경우다.

- 예를 들어 모델이 손목을 잘못 찾았다면... 

```
"왼쪽 손목은 여기야"
        ↓
2D Keypoint Prompt
        ↓
SAM 3D Body
        ↓
Pose 다시 추정 
```



- 논문에서는 **현재 예측에서 오차가 가장 큰 keypoint를 골라 prompt로 제공**하고, prompt 개수를 늘렸을 때 성능이 어떻게 변하는지 확인했다. 
- 결과는 꽤 명확하다.

|Prompt 개수|COCO PCK ↑|EMDB MPJPE ↓|
|---|---|---|
|0|86.7|63.3|
|1|90.2|60.1|
|2|93.0|58.9|

- 즉 **정확한 keypoint prompt가 많아질수록 2D와 3D 성능이 모두 좋아졌다.** 
- 특히 흥미로운 점은 prompt 자체는 `(x,y)`라는 **2D 정보**인데도, 그 정보를 이용해서 **3D pose도 더 정확하게 추정했다는 것**이다. 
## 2. Prompt가 조금 틀려도 괜찮은가?

- 실제 사용자가 찍어주는 keypoint나 detector 결과는 완벽하지 않을 수 있다.

- 그래서 저자들은 keypoint 위치에 일부러 noise를 넣었다.

- 여기서 noise scale은 **사람 bounding box 크기에 대한 상대적인 오차**다.

- 결과는:
	- 작은 오차(`noise < 0.05`)에는 비교적 robust
	- 오차가 커질수록 성능 저하
	- 잘못된 prompt가 너무 심하면 모델이 **그 잘못된 위치를 따라가기 때문에** 오히려 성능이 떨어짐


- Table 7을 보면 EMDB MPJPE가 악화된다.

```
정확한 Prompt       → 60.1
noise 0.03          → 61.5
noise 0.05          → 63.3
noise 0.10          → 67.8
```

- 즉, **Prompt는 도움이 되지만, 틀린 prompt를 주면 모델도 그 잘못된 정보를 따라간다.**


## 3. Hand Pose에서도 Keypoint Prompt를 사용

- 앞의 Model Training and Inference에서 봤던 내용과 연결된다.
- Hand Decoder가 손을 더 정확하게 추정한 다음 아래 정보를 prompt로 다시 Body Decoder에 넣어 전체 자세를 보정했었다.
	- Hand Decoder → Wrist 위치
	- Body Decoder → Elbow 위치


- Figure 9에서는 이 전략이 실제로 효과가 있는지를 보여준다. 

- 비교는:

```
Image Crop

① Keypoint Prompt 없음

② Body/Hand Decoder 통합 없음

③ Default Inference
   = Prompt + Hand Decoder 통합
```


- 논문에서는 keypoint prompt가 없으면 **손목과 손 관절의 2D alignment가 나빠지고**, 반대로 Hand Decoder를 사용하지 않으면 **wrist rotation과 finger alignment가 나빠진다**고 설명한다. 

- 즉 둘 다 필요하다는 것.


## 4. Mask Prompt


- 이 기능은 특히 **사람이 여러 명 있을 때** 중요하다.

- 예를 들어:

```
┌───────────────────┐
│ 사람 A   사람 B   │
│   서로 겹쳐 있음   │
└───────────────────┘
```

- bounding box만 주면 "A를 복원해야 해? B를 복원해야 해?"가 애매할 수 있다.
- 그래서 사람 A의 segmentation mask를 주면 명확하게 지정할 수 있다

```
"이 픽셀 영역의 사람을 복원해"
             ↓
        Mask Prompt
```

## 5. Mask Prompt 평가

- 저자들은 multi-person 데이터에서 3DB without mask Vs 3DB with mask를 비교했다.

- 특히 **Hi4D, Harmony4D**는 두 사람이 가까이 붙어 있고 서로 심하게 가리는 장면이 포함돼 있다. 

- 대표적인 Hi4D 결과는:

||PVE ↓|MPJPE ↓|
|---|--:|--:|
|Mask 없음|91.4|76.4|
|Mask 있음|**58.3**|**47.0**|

- 즉 mask를 추가하니:
	
	- PVE: **33.1 감소**
	- MPJPE: **29.4 감소**

- 이유는 ... 
> segmentation mask가 **“여러 사람 중 누구를 복원해야 하는지”** 정확하게 알려주기 때문.

- SA1B에서도 전체 데이터 성능 향상은 +0.9%였지만, **Multi-person subset에서는 +4.4%** 향상되어 multi-person 상황에서 mask가 특히 중요하다는 걸 보여준다. 


---
# Limitations


- AM 3D Body의 한계를 딱 **세 가지**로 정리.
> **SAM 3D Body는 사람 개개인의 full-body mesh 복원에는 강하지만, 사람과 주변 환경의 물리적 상호작용을 직접 모델링하지 않고, 손 전문 모델 수준의 정밀도와 모든 연령대의 체형 표현에도 아직 한계가 있다.** 

|한계|의미|
|---|---|
|**Interaction 부족**|사람을 개별적으로 복원하며 사람-사람/물체/환경 관계를 직접 이해하지 않음|
|**Hand accuracy 한계**|Full-body 모델치고 강하지만 전문 hand-only 모델보다 뛰어나지는 않음|
|**Age diversity 한계**|MHR이 모든 연령의 체형을 충분히 표현하지 못하며 특히 어린이에 약할 수 있음|
## 1. Multi-person / Human-Object Interaction을 직접 모델링하지 않음

- SAM 3D Body는 이미지에 여러 사람이 있어도 **각 사람을 개별적으로 처리**한다.

- 즉,
```
사람 A → 따로 3D 복원
사람 B → 따로 3D 복원
```

- 하지만, 아래와 같은 상호작용 자체를 이해하는 모델은 아니다.
```
사람 A ↔ 사람 B
사람 ↔ 물체
사람 ↔ 주변 환경
```


- 따라서 사람끼리의 상대적인 위치나 실제 물리적 접촉을 정확하게 해석하는 데 한계가 있다. 
- 저자들은 향후에는 사람-사람, 사람-물체, 사람-환경 interaction을 학습 과정에 포함하는 것이 자연스러운 발전 방향이라고 말한다.

### 2. Hand Pose는 좋아졌지만 Hand-only 모델보다 뛰어나지는 않음

- SAM 3D Body는 full-body HMR 중에서는 손 성능을 많이 개선했지만, **손만 전문적으로 추정하는 모델보다 정확도가 더 높지는 않다.**

- 또한 **Body Decoder 하나만 사용했을 때의 손 추정 성능도 충분하지 않다.**

- 논문에서는 그 원인 중 하나로 **고품질 full-body 학습 데이터가 부족한 점**을 지적한다. 그래서 별도의 Hand Decoder가 중요한 역할을 한다.

### 3. 모든 연령대의 Body Shape를 충분히 표현하지 못함

- 세 번째 한계는 **사람의 나이에 따른 체형 차이**다.

- SAM 3D Body뿐 아니라 기반 human mesh model인 **MHR 자체도 모든 연령대의 사람 체형을 완전히 모델링하지 못한다.**

- 특히 논문에서는 **children, 즉 어린이**를 명시적으로 언급한다.

- 그 결과 어린이에서는 아래 성능이 충분하지 않을 수 있다.
	- Pose estimation
	- Body shape modeling


