# 소량의 의료데이터 학습을 위한 도메인내 종류가 다른 데이터를 활용한 자기 지도학습



### 요약

---

- 유방암 데이터의 갯수를 제한하고 소량의 데이터로 지도학습 모델과 자기 지도학습(Self-supervised learning)모델인 RotNet과 SimCLR 모델을 학습
- 자기 지도학습의 성능 개선을 위해 유방암 병리 영상 외, 신장(Kidneys) 병리 영상을 추가로 활용
- 각 모델들의 유방암 병리 이미지를 분류하는 성능 비교
- 분류 범주 : 정상(Benign), 비정형 유관 증식증(ADH), 유방 상피내암(DCIS)

---



### 개요



#### 데이터 셋

---

학습에 사용된 유방암 병리 조직영상은 정상(Benign), 비정형 유관 증식증(Atypical Ductal Hyperplasia, ADH), 유방 상피내암(Ductal Carcinoma In-Situ, DCIS)으로 구분된 Whole Slide Image(WSI)입니다.

WSI는 전용 스캐너를 이용해 유리 슬라이드를 고해상도 디지털 데이터로 변환한 파일입니다.



![WSI](https://github.com/user-attachments/assets/f97565db-bfda-4aa1-a1f1-a3f53959d183)

<center>WSI 이미지</center>



- 모델 학습을 위해 WSI를 x40배율의 패치 이미지로 분할하고 (224, 224, 3)의 shape으로 변환
- 학습에 불필요한 배경 이미지 제거
- imbalance 데이터를 방지하기 위해 각 클래스 별 동일한 학습 데이터 갯수를 사용
- 데이터 비율 별로 학습 성능을 평가하기 위해 학습에 사용될 데이터의 비율을 5%, 25%, 50%로 분할



| 데이터 비율     | Benign | ADH  | DCIS | Total |
| --------------- | ------ | ---- | ---- | ----- |
| 전체            | 3080   | 3080 | 3080 | 9240  |
| 훈련 데이터 5%  | 154    | 154  | 154  | 462   |
| 훈련 데이터 25% | 770    | 770  | 770  | 2310  |
| 훈련 데이터 50% | 1540   | 1540 | 1540 | 4620  |
| 테스트 데이터   | 1136   | 1876 | 770  | 3782  |



#### 자기지도 학습 모델

---

자기지도 학습은 데이터의 레이블 없이 데이터의 representation을 사전에 학습하는 방법입니다.



**실험에서 사용한 모델**

1. **RotNet**

   RotNet은 데이터의 특징을 학습하기 위해 이미지를 회전시킨 뒤, 이미지의 회전 각도를 예측하도록 학습하는데 본 연구에서는 4개의 회전 각도 [0°, 90°, 180°, 270°]로 사전 학습했습니다.

   ![RotNet_deg1](https://github.com/user-attachments/assets/4389bb1a-79da-4f5b-849b-55df84f5ed7e)

   <center>4가지 각도로 회전된 이미지</center>

   또한 위의 그림과 동일하게 이미지를 회전시킨 뒤 이미지에 색상 왜곡을 적용한 데이터의 회전 각도를 예측하도록 하는 실험을 진행하였습니다.

   ![RotNet_deg2](https://github.com/user-attachments/assets/830c240e-1016-4fd1-ae76-d1d2bdfa94e7)

   <center>색상 왜곡이 적용된 이미지</center>

2. **SimCLR**

   하나의 데이터에 두 가지의 데이터 증강을 적용해 2개의 증강 데이터를 Positive Pair라 정의하고 나머지 데이터에도 동일하게 데이터 증강을 적용시켜 Negative Pair를 생성한 뒤, 대조 학습(Contrastive learning)을 통해 데이터의 representation을 학습합니다.

   ![Data_Augmentation](https://github.com/user-attachments/assets/f8b38fc6-5542-4259-82f7-4c9f01c4b70c)

<center>데이터 증강 적용</center>

---



### 결과

---

데이터 비율을 제외하고 동일한 조건으로 실험을 진행했을 때, 유방암과 관련 없는 **신장 병리영상으로 사전학습**을 진행한 뒤 전이 학습(Transfer learning)을 통해 유방암 병리 영상을 학습한 **SimCLR의 암 진단 정확도가 모든 데이터 비율에서 가장 높았음**.

특히 **유방암 병리영상 데이터를 25%으로 제한해 학습**을 진행했을 때, **모든 데이터를 사용해 학습한 지도 학습 모델의 정확도가 동일**

| 데이터 비율 | 모델                | Loss   | Accuracy |
| :---------: | :-------------------: | :------: | :--------: |
| 5%          | Supervised learning | 1.595  | 67.10%   |
|             | RotNet              | 0.7344 | 64.11%   |
|             | RotNet+Jitter       | 0.776  | 79.32%   |
|             | SimCLR              | 0.5545 | 75.59%   |
|             | **SimCLR+kidneys data** | **0.4881** | **80.65%** |
| 25%         | Supervised learning | 0.5947 | 83.15%   |
|             | RotNet              | 0.3881 | 84.51%   |
|             | RotNet+Jitter       | 0.3466 | 86.64%   |
|             | SimCLR              | 0.3569 | 84.8%    |
|             | **SimCLR+kidneys data** | **0.2785** | **88.96%** |
| 50%         | Supervised learning | 0.5538 | 88.7%    |
|             | RotNet              | 0.3864 | 85.76%   |
|             | RotNet+Jitter       | 0.2897 | 89.4%    |
|             | SimCLR              | 0.2767 | 88.79%   |
|             | **SimCLR+kidneys data** | **0.2229** | **91.5%** |
| 100%    | Supervised learning | 0.3376 | 88.96% |

