---
layout:     post
title:      SlowFast Networks for Video Recognition
author:     Youngsan
tags: 		FAIR Action_Recognition Kaiming_He
subtitle:  	2018 Action Classification winner
category:  dlpr
---
#### Key concept
* Fast pathway - high frame rate, motion
* Slow pathway - low frame rate, spatial semantic
<br/><br/><br/>

## Introduction
----------------
network의 motivation
* 영상에서 categorical spatial semantic은 '천천히' 변함
  * 'waving hands'에서 손(categorical semantic)이라는 identity는 변하지 않음
  * 따라서 categorical semantic을 파악할 때는 low frame rate로 처리해도 됨<br/><br/>
* 반면 motion은 빠르게 변할 수 있음
  * high frame rate 필요

$$ \Rightarrow $$ two-pathway SlowFast model을 제안

참고사항)<br/>
* 이 모델은 영장류 시각 시스템의 retinal ganglion cell에서도 부분적으로 영감을 얻음.
* 이 cell은 ~80%의 Parvocellular(P-cell), ~15~20%의 Magnocellular(M-cell)로 구성
* M-cell은 spatial detail과 색깔에는 둔감, temporal change에는 민감(high temporal frequency로 작동) (P-cell은 그 반대)
<br/><br/><br/>

## SlowFast Networks
--------------------
![Figure1](https://user-images.githubusercontent.com/22654096/52966598-0eca9780-33eb-11e9-8b7d-d26c9c80d8cd.PNG)

* Slow pathway
* Fast pathway

### Slow pathway
* video clip을 처리할 수 있는 모든 conv model 사용 가능 e.g., C3D, I3D
* large temporal stride $$ \tau(=16) $$
* $$ T $$ : frames sampled by the Slow pathway
<br/><br/>

### Fast pathway
#### High frame rate
* small temporal stride $$ \tau/\alpha $$ $$ (\alpha=8) $$
<br/><br/>

#### High temporal resolution features
* temporal downsampling layer 없음 $$ \Rightarrow $$ 항상 $$ \alpha T $$ 차원
  * temporal fidelity 유지
<br/><br/>

#### Low channel capacity
* Slow pathway의 channel 수 대비 $$ \beta $$배 $$ (\beta=1/8) $$ $$ \Rightarrow $$ lightweight
  * Fast pathway은 전체 계산량의 ~20% 차지(M-cell의 비중과 유사)<br/><br/>
* 적은 channel 수로 인해 spatial semantic을 표현하는 능력이 약함.
  * spatial modeling ability와 temporal modeling ability 사이의 tradeoff<br/><br/>
* spatial capacity를 약화시키는 방향의 추가적인 실험
  * input spatial resolution 축소
  * 색깔 정보 제거
<br/><br/>

### Lateral connections
* 각 pathway에서 학습한 정보 공유
* 매 stage마다 connection, ResNet을 예로 들면 pool1, res2, res3, res4 이후에..
* 두 pathway가 처리하는 temporal dimension이 다르기 때문에 일치시켜주는 작업 필요
  * Instantiations 절에서 자세히
* connection은 unidirectional (fast $$ \rightarrow $$ slow)
  * bidirectional도 실험해봤지만 성능면에서 차이 미미
<br/><br/>

## Instantiations
![Table1](https://user-images.githubusercontent.com/22654096/52969842-23139200-33f5-11e9-8081-658d7039b8bb.PNG)

### Slow pathway
(위 그림 예)
* 3D ResNet을 사용, $$ T=4, \tau=16 $$
* non-degenerate temporal conv(temporal kernel size>1 인 conv layer, <U>밑줄</U>)를 res4, res5에만 사용
  * 초기 layer에서의 temporal conv는 정확도에 악영향
  * __object의 움직임이 빠르고 temporal stride가 크면, spatial receptive field가 충분히 크지 않으면 temporal receptive field 사이에는 correlation이 거의 없다고 판단__
<br/><br/>

### Fast pathway
* $$ \alpha=8, \beta=1/8 $$  <br/>
* higher temporal resolution(green), lower channel capacity(orange)
* non-degenerate conv를 모든 block에 사용
<br/><br/>

### Lateral connections
* 각 pathway의 feature shape
  * Slow pathway : $$ {\{T, S^2, C\}} $$
  * Fast pathway : $$ {\{ \alpha T, S^2, \beta C \}} $$
  <br/><br/>
* 세 가지 fusion 방법
  * Time-to-channel : reshape and transpose
  $$ {\{ \alpha T, S^2, \beta C \}} \rightarrow {\{T, S^2, \alpha \beta C\}}$$
  * Time-strided sampling : sample one out of every $$ \alpha $$ frames
  $$ {\{ \alpha T, S^2, \beta C \}} \rightarrow {\{T, S^2, \beta C\}}$$
  * Time-strided convolution : 3D conv of $$ 5 \times 1^2 $$ kernel with stride= $$ \alpha $$
  $$ {\{ \alpha T, S^2, \beta C \}} \rightarrow {\{T, S^2, 2 \beta C\}}$$
  <br/><br/>

## Experiments: Kinetics Action Classification
#### Datasets
* Kinetics-400
* 평가: top-1, top-5 classification accuracy(%), FLOPs
  <br/><br/>

#### Training
* 상세 내용은 논문에..ㅜ
  <br/><br/>

#### Inference
* 상세 내용은 논문에..ㅜ
  <br/><br/>

### Results and Analysis
![Table2](https://user-images.githubusercontent.com/22654096/52984526-36455280-3433-11e9-8f9a-4a5ff3db5786.PNG)

#### Training from scratch
* [\[50\]](https://arxiv.org/pdf/1711.07971.pdf)
* our recipe : large-scale SGD
<br/><br/>

#### Individual pathways
* no temporal downsampling = temporal reduction factor(t-reduce) of 1
* individual pathway의 성능은 그닥, 다만 GFLOPs를 보면 매우 light함을 알 수 있음
* 두 pathway는 합쳐질 때 각자의 special expertise를 보임.(다음 실험)
<br/><br/>

#### SlowFast fusion
* SlowFast의 첫번째 row는 naive fusion 사용. 단순 concat
* SlowFast model은 Slow-only 보다 모두 좋은 성능을 보임
* T-conv가 젤 좋음. 따라서 default lateral connection으로 설정
<br/><br/>

#### Channel capacity of Fast pathway
* $$ \beta $$ 값에 상관 없이 모두 성능 향상.
* best는 $$ \beta=1/6 $$,  $$ \beta=1/8 $$(default)
* $$ \beta=1/32 $$의 경우 GFLOPs가 1.0밖에 안 늘어났지만 1.6%의 성능 향상을 보임
<br/><br/>

#### Weaker spatial inputs to Fast pathway
* time difference : 현재 frame - 이전 frame
* 모두 Slow-only baseline보다 좋음
* 특히 gray-scale의 경우 RGB만큼 좋은 성능을 보이는 동시에 FLOPs도 ~5% 가량 적음
  * 이 역시 색깔에 둔감한 M-cell과 비슷
<br/><br/>

#### vs. Slow+Slow
* 2-Slow ens. : 따로 training된 두 개의 Slow-only 모델을 앙상블<br/>
$$ \rightarrow $$ 0.5% 성능 향상, 계산량 x2
* SlowSlow : Fast pathway를 Slow pathway로 교체<br/>
$$ \rightarrow $$ overfitting으로 고통받음..
<br/><br/>

#### Various SlowFast Instantiations
* $$ T \times \tau $$<br/>
  * $$ 4 \times 16 $$ <br/>
  * $$ 8 \times 8 $$ <br/>
  * 두 경우 모두 약간의 FLOPs 증가로 성능 향상
* 나머지는 정확도와 FLOPs 사이의 tradeoff 양상을 보임
* 이 tradeoff는 더 최적화될 여지가 남아있으며, 최적의 tradeoff는 application-dependent함
<br/><br/>

#### Advanced backbones
* ResNet-101 모델과 non-local(NL) version에 대해 추가 실험
  * NL은 Slow pathway에만 추가
  * R101+NL에서 NL은 res4에만 추가
* advanced backbones은 SlowFast concept과는 독립적(orthogonal)
  * $$ \rightarrow $$ 추가 성능 향상
<br/><br/>

#### Comparison with state-of-the-art results
![Table3](https://user-images.githubusercontent.com/22654096/53137729-1e9ed300-35c7-11e9-8144-8ba51cf461f8.png)

* 각 논문마다 inference strategy가 다르기 때문에 inference time에 "view"(temporal clip with spatial crop) 개념 도입
  * 본 논문 예) 10 clips each with 3 spatial crops $$ \rightarrow $$ 30 views

* 기존 모델(w/wo ImageNet pre-training)보다 성능 좋음. 동시에 inference-time cost도 낮음.
  * cost 측면에서 기존 모델들의 clip은 extremely dense sampling됨.<br/>
  $$ \rightarrow $$ 100 이상의 views
<br/><br/>

#### Kinetics-600
![Table4](https://user-images.githubusercontent.com/22654096/53139533-42651780-35cd-11e9-83d8-6a865f22dc7c.png)

* Kinetics-600은 비교적 새로운 데이터이기 때문에 비교 대상이 제한적임. 따라서 이 결과의 주된 목적은 future reference를 제공함에 있음
* Kinetics-600의 validation set이 Kinetics-400의 training set과 겹치기 때문에 400으로 pre-training 하지 않음
* ActivityNet Challenge 2018의 우승 모델(79%), SlowFast, R101 + NL(81.1%)

## Experiments: AVA Action Detection
#### Dataset
* human action에 대한 spatiotemporal localization에 초점
* 437개의 영화로부터 데이터 수집
* 초당 1프레임에 대해 spatiotemporal label 제공
  * 모든 사람이 bounding box와 action으로 annotation됨
* training - 211k, validation - 57k
* metric : mAP on 60 classes, using a frame-level IoU threshold of 0.5
<br/><br/>

#### Detection architecture
* Faster R-CNN(영상에 적합하게 최소한의 modification 적용)
* SlowFast network를 backbone으로 사용
<br/><br/>

#### Training
* 상세 내용은 논문에..ㅜ
  <br/><br/>

#### Inference
* 상세 내용은 논문에..ㅜ
  <br/><br/>

### Results and Analysis
![Table5](https://user-images.githubusercontent.com/22654096/53144240-cf649c80-35de-11e9-8529-5d02cf55dccb.png)
* Slow-only baseline vs. SlowFast
* Fast pathway를 추가함으로써 5.2mAP 상승 효과<br/><br/>
* per-category AP
![Figure3](https://user-images.githubusercontent.com/22654096/53144453-ab558b00-35df-11e9-9971-c9dc939517d1.png)
  * 60개 중 57개의 category에 대해 성능 향상
  * 성능이 떨어진 3개에 대해, 떨어진 폭이 다른 category의 상승에 비해 작음<br/><br/>
* SlowFast의 다양한 instantiations에 대한 결과
![Table6](https://user-images.githubusercontent.com/22654096/53144745-d7bdd700-35e0-11e9-9260-e84735b30e79.png)
  * AVA 데이터에 대해 일관된 성능을 보임 $$ \rightarrow $$ robust
  <br/><br/>

#### Comparison with state-of-the-art results
![Table7](https://user-images.githubusercontent.com/22654096/53144852-43a03f80-35e1-11e9-9d91-a941b1ac103a.png)
* optical flow 사용에 대한 potential benefit을 주목할만 함
* 반면 Fast pathway로부터의 성능 향상은 +5.2mAP
  * optical flow를 사용하는 two-stream method는 computational cost를 두배로 증가시킴
  * Fast pathway는 lightweight
<br/><br/>

* ground-truth region proposal 분야에서는 35.1 mAP 달성
  * proposal generation의 성능 향상이 기대됨<br/><br/>

* validation mAP가 26.8인 모델을 train+val 으로 학습한 후 공식 test server에 제출 결과 26.6mAP 달성
$$ \rightarrow $$ consistency
<br/><br/>

#### Visualization
![Figure4](https://user-images.githubusercontent.com/22654096/53145415-79debe80-35e3-11e9-9695-745a344a77c2.png)
