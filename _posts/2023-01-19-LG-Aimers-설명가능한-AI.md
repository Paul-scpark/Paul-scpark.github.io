---
title: LG Aimers 2기 Explainable AI (서울대학교 문태섭 교수님)
date: 2023-01-19 00:00:00 +0900
categories: [Education, LG Aimers 2기]
tags: [AI, Deep learning, Machine learning]
description: LG Aimers AI 전문가 과정 강의 기록
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true

---

이번 글에서는 LG Aimers의 AI 전문가 과정에서 설명가능한 AI에 대하여 학습합니다. 머신러닝은 크고 복잡한 데이터를 이해하고, 이들 간의 관계성을 살펴보는 기술이지만 본질적으로 해석 가능성을 제한하는 블랙박스인 경우가 많습니다. 현실에서 이것을 사용할 때는 그에 따른 해석이 요구되는데, 따라서 그 한계를 보완하기 위해 Explainable AI 즉, 설명가능한 AI 기술에 대해 학습하여 머신러닝 모델과 그 의미에 대하여 학습할 것입니다.

---

## 1. Explainable AI 1
- 최근 딥러닝은 그 성능이 매우 빠르게 발전하고 있음
- 하지만 대용량 학습 데이터로부터 학습하는 모델 구조는 점점 더 복잡해지고, 이해하기 어려워진다는 한계점이 있음
- 즉, 입력을 주게 되면, 그에 따른 결과가 나오는 하나의 블랙박스 형태의 결과가 보여진다는 것
    - 이러한 예측 결과가 사람에게 직접 영향을 미치게 되는 경우에는 이 한계점은 매우 심각하게 될 것
    - 자율주행, 의학적 진단, 대출 승인 등을 비롯한 AI 편향성 문제
- Reliability & Robustness (Pascal VOC 2007 Classification)
    - 말 이미지가 주어졌을 때, 말의 어느 부분을 보고 해당 사진이 말이라는 것을 알게 됐는지 특정 부분을 하이라이트
    - XAI 기법은 이미지에서 말에 해당하는 부분이 아닌, 아랫쪽에 주로 하이라이트가 되어 있었음
    - 데이터를 확인해보니, 말 사진에는 텍스트 워터마크가 있었고, 이것으로 말이라고 예측하고 있었다는 것
    - 이처럼 XAI 기법을 통해서 모델이나 데이터셋의 오류를 색출하고, 편향성을 확인 할 수 있을 것
- 결국, 왜 알고리즘이 그런 예측 결과를 냈는지 설명하여 신뢰 여부를 결정할 수 있어야 함
- XAI에서 설명가능하다는 것은 모델을 사용할 때, 그 동작을 이해하고 신뢰할 수 있게 해주는 기계학습 기술으로 정의

### XAI의 대비되는 종류
- Local vs Global
    - Local: Describes an individual prediction (개별적인 예측 결과를 설명)
    - Global: Describes entire model behavior (전반적인 행동을 설명)
- White-box vs Black-box
    - White-box: Explainer can access the inside of model (모델 내부 구조를 알고 설명)
    - Black-box: Explainer can access only the output (모델 구조를 모르고, 출력만으로 설명)
- Intrinsic vs Post-hoc
    - Intrinsic: Restricts the model complexity before training
    - Post-hoc: Applies after the ML model is trained
- Model-specific vs Model-agnostic
    - Model-specific: Some methods restricted to specific model classes
    - Model-agnostic: Some methods can be used for any model

-> Linear model, Simple Decision Tree: Global, White-box, Intrinsic, Model-specific <br/>
-> Grad-CAM: Local, White-box, Post-hoc, Model-agnostic

### Simple Gradient method
- Simply use the gradient as the explanation (딥러닝의 Back-propagation)
- Strength: Easy to compute (via Back-propagation)
- Weakness: Becomes noisy (due to shattering gradient problem)
    - 즉, 똑같은 예측 결과를 갖는 조금씩 변하는 이미지들에 대해 각 이미지에 대한 설명이 많이 다를 수도 있음

### SmoothGrad
- Simple method to address the noisy gradients
- 이미지가 주어졌을 때, 작은 노이즈인 엡실론을 섞어준 뒤, 노이즈가 섞인 입력 이미지에 Gradient를 구하는 과정을 여러 번 수행하여 그 Gradient의 평균으로 설명하는 것 (대략 50번 정도)
- Strength: Clearer interpretation via simple averaging, Applicable to most sensitive maps
- Weakness: Computationally expensive

## 2. Explainable AI 2

### Class Activaiton Map (CAM)
- 어떤 이미지가 CNN 모델의 입력으로 주어졌을 때, 이미지에 해당하는 Activation들이 최종 이미지에 최종 Activation으로 나타나게 됨. 이러한 각 Activation map은 Global Average Pooling 별로 학습된 w_1부터 w_n을 활용하여 Activation map을 결합하게 됨. 최종 결합된 이미지는 하이라이트 되어 표현됨
- 어떤 Activation map에 Activation이 크게 된다는 것은 그 Map이 주어진 입력과 관련이 많다는 뜻이고, 그것을 결합하는 w가 크다는 것도 최종 분류에 큰 영향을 주는 Activation이라는 것
- CAM은 Object detection 이나, Semantic segmentation 등 더 복잡한 응용 분야에도 적용 가능
- Strength: It clearly shows what objects the model is looking at
- Weakness  
    - Model-specific: It can be applied only to models with limited architecture
    - It can only be obtained at the last convolutional layer and this makes the interpretation resolution coarse (마지막 Convolutional layer의 Activation에서 얻을 수 있으므로 해상도가 좋지 않음)

### Grad-CAM
- CAM 방식을 보완한 방식으로, Global Average Pooling layer가 없는 모델도 적용할 수 있음
- 특정 모델 구조를 가지고 학습된 w를 사용하는 것이 아닌, Activation map의 Gradient를 구한 다음에 그것의 Global Average Pooling 값으로 w를 적용한다는 것
- Strength: Model-agnostic 하여 It can be applied to various output models
- Weakness: Average gradient sometimes is not accurate

### Perturbation-based
- 입력 데이터를 조금씩 바꾸면서 그에 대한 출력을 보고, 그 변화에 기반하여 설명하는 접근법
- Local Interpretable Model-agnostic Explanations (LIME)
    - 어떤 분류기가 딥러닝 모델처럼 복잡한 비선형적 특징을 가지더라도, 주어진 데이터 포인트들에 대해서는 아주 Local 하게는 다 선형적인 모델로 근사화가 가능하다는 관찰에서 출발
    - 그래서 주어진 데이터를 조금씩 교란해 가면서, 교란된 입력 데이터를 모델에 여러 번 통과시켜 나오는 출력을 보고, 나오는 입출력 Pair들을 간단한 선형 모델로 근사하여 설명을 얻어내는 방식
    - Strength: Black-box interpretation (딥러닝 모델 뿐 아니라, 입력과 출력을 얻을 수 있다면 모두 적용 가능)
    - Weakness: Computationally expensive, Hard to apply to certain kind of models, When the underlying model is still locally non-linear
- Randomized Input Sampling for Explanation (RISE)
    - LIME 방식과 비슷하게, 여러 번 입력을 Perturb 해서 설명을 구하는 방식
    - 랜덤한 Mask를 만들어서 그 Mask를 씌운 입력이 모델을 통과 했을 때, 해당 클래스에 대한 예측 확률이 얼마나 떨어지는 확인하여 설명력을 예측하게 됨. 즉, 여러 개의 랜덤 마스킹이 되어 있는 입력에 대해 출력 스코어를 구하고, 그 확률을 통해 이 마스크들을 가중치를 두어 평균을 냈을 때 나오는 것이 설명 Map
    - Strength: Much clear saliency-map
    - Weakness: High computational complexity, Noisy due to sampling
- Different approach for XAI
    - Identify most influential training data point for the given prediction
    - Influence function
        - Measure the effect of removing a training sample on the test loss value
        - 특정 Training 이미지 없이 모델을 훈련시켰을 때, 해당 모델의 성능이 얼마만큼 변할지 근사화 하는 함수
        - 이 함수를 가지고, 각 Training 이미지마다 영향력을 계산하고, 그 값이 가장 큰 이미지를 설명으로 제공

## 3. Explainable AI 3

### XAI 방법을 비교 평가하는 방법
- 사람들이 직접 XAI 방법들이 만들어낸 설명을 보고 비교 및 평가하는 것
    - AMT (Amazon Mechanical Turk) Test
    - Guided Backprop, Guided Grad-CAM 등 확인 가능
    - Weakness: Obtaining human assessment is very expensive
- Human annotation을 활용하는 것
    - Some metrics employ human annotations (localization and semantic segmentation) as a ground truth, and compare them with interpretation
    - Pointing game: Bounding box를 활용하여 평가하는 방법
    - Weakly supervised semantic segmentation: 어떤 이미지에 대해 Label만 주어졌을 때, 그것을 활용하여 픽셀 별로 객체의 Label을 예측하는 방법 (Weakly supervised인 이유는, 픽셀 별로 정답 Label이 없기 떄문)
    - IoU (Intersection over Union): 정답 Map과 이렇게 만들어낸 Segmentation map이 얼마나 겹치는지 평가
    - Weakness: Hard to make the human annotations, Such localization and segmentation labels are not a ground truth of interpretation
- Pixel Perturbation 즉, 픽셀을 교란하여 모델의 출력값의 변화를 테스트
    - 성이 있는지 분류하는 모델이 있을 때, 성 부분을 가려서 모델을 통과시키면 그 값이 크게 떨어질 것임. 만약 성이 아닌 다른 부분을 가리게 된다면, 분류 스코어 값은 크게 변동이 없을 것임
    - AOPC (Area Over the MoRF Perturbation Curve): 주어진 이미지에 대해 각 XAI 기법이 설명을 제공하면, 그 설명의 중요도 순서대로 각 픽셀을 정렬할 수 있고, 그 순서대로 픽셀을 교란했을 때, 원래 예측한 분류 스코어 값이 얼마나 빨리 바뀌는지를 측정하는 것
    - Insertion: 중요한 순서대로, 백지 상태의 이미지에서 중요한 픽셀의 순서대로 하나씩 추가해가면서 분류기의 출력 스코어가 어떻게 변화하는지 확인하는 것. 따라서 이 값이 클수록 좋은 결과
    - Deletion: AOPC 방법과 같이 XAI 기법이 제공한 중요도 순서대로 픽셀을 하나씩 지워가며, 분류 확률 값이 떨어지는 확인하는 것으로, AOPC와는 반대로 커브의 아래 쪽의 면적을 구함. 따라서 이 값이 낮을수록 좋은 결과
    - Weakness: 데이터를 지우거나, 추가하는 과정이 머신러닝의 주요한 가정에 위반하는 경우가 있음. 즉, 어떤 픽셀을 지우고, 모델의 입력으로 넣었을 때, 해당 이미지는 모델을 학습시킨 Training 이미지들의 분포와 다르기 때문에 정확하지 않다는 것
- ROAR (RemOve And Retrain) 
    - XAI 기법이 발견한 중요한 픽셀을 지우고 나서, 지운 데이터를 통해 모델을 재학습하고, 정확도가 얼마나 떨어지는지 평가하는 방법 (재학습한 후에 나오는 모델의 성능이 떨어지는 경우에는 좋은 설명 방법이라는 것)
    - 앞선 방법들에 비해 조금 더 객관적이고, 정확한 평가를 할 수 있지만, 계산 복잡도가 크다는 단점이 있음

### XAI 방법의 신뢰성에 관한 연구
- Sanity Checks 
    - Model Randomization
        - Model Randomization Test
        - 분류 모델에 위쪽 Layer부터 모델의 계수들을 순차적으로 Randomized 한 후에 얻어지는 설명들을 구함
        - 계속 Randomized 되었으므로, 시간이 지날수록 성능이 떨어지지 않은 모델들은 학습이 잘못됨을 추론
    - Adversarial Attack
        - 어떤 입력 이미지에 대한 픽셀을 아주 약간 바꿨을 떄, 분류기의 예측 결과를 완전히 다르게 만든다는 것
        - 많은 설명 방법들이 Gradient와 연관되는 값을 사용하는데, Decision boundary가 불연속적으로 나오게 된다면, Gradient의 방향이 급격하게 변할 수 있기 때문에 조금만 입력이 바뀌어도 Gradient가 아주 많이 바뀔 수가 있다는 것 
        - 이에 조금 더 강건하게 반응하기 위해 ReLU 대신, Softplus를 사용하라
    - Adversarial Model Manipulation
        - 모델이 편향되었다는 것을 알았을 때, 해당 모델을 다시 고쳐서 재학습 시키는 것이 아니라, 모델 계수를 조금씩 조작하여 모델의 정확도는 차이가 없지만 XAI의 결과가 공정한 것처럼 보일 수도 있다는 것