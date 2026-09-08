# skin_lesion_classification
프로젝트 주제 : 피부 병변 분류 최적 모델  
프로젝트 기간 : 2023.9.1 ~ 2023.12.7  
프로젝트 목표 :  
1.	 주요 피부 병변을 분류하는 최적의 알고리즘을 개발하여 피부 질환 감별에 있어 중요한 피부 조직 검사를 시행하지 않은 상태에서도 보다 정확한 진단을 가능케 한다.
2.	 피부암의 조기 진단 및 분류를 자동화하고, 의료 전문가에게 정확하고 효과적인 의사결정을 지원한다. 의료 전문가 개인의 경험 및 예측에 의존적인 기존 진단에 비하여 진단 및 치료 결정의 정확성을 향상시킨다.
3.	 진료와 검사에 소요되는 시간과 노력을 축소해 환자의 편의성을 증대하고, 병원의 자원 절약을 유도하며 원격 의료로 피부 질환을 진단할 수 있는 가능성을 높인다.

  -	 해당 프로젝트는 피부암 진단의 정확성을 향상시키고 의료 전문가들에게 의사 결정 지원 도구를 제공한다. 피부 병변을 촬영한 이미지를 이용해 해당 병변을 분류하는 것이 목표이며, 피부 병변 이미지 데이터의 수집 및 전처리, 딥러닝 및 이미지 처리 알고리즘의 구현, 다양한 실험을 통한 최적의 분류 모델 탐구를 진행한다.  
## 프로젝트 과정
 Inputs은 Labelme를 이용해 병변 부분을 polygonal annotation한 후 얻어낸 마스크 이미지와 원래 이미지를 128*128 size와 256*256 size로 리사이징한 이미지이다. 해당 inputs으로 위 이미지에 보이는 모델을 학습시킨다. 최종적으로 test image를 넣었을 때 해당 병변이 어떤 피부 병변인지 classification하는 알고리즘을 학습시킨다.  
<img width="741" height="363" alt="image" src="https://github.com/user-attachments/assets/870b5438-f3b3-408d-954f-3ba6b82e81b9" />

 초기에는 진단명과 발병 위치에 대한 정보가 모두 포함된 이미지를 선별하고자 하였다. 또한 품질이 낮거나 특징적이지 못한 이미지를 감지하여 육안으로 분별하고, Anaconda를 이용하여 Labelme를 실행한 후 병변 부분을 segmentation하고, 파이썬 코드로 데이터를 사용 의도에 맞게 변환하고자 하였다. 실제로 이용하고자 하는 정보를 모두 가지고 있는 이미지를 선별하였고, Labelme를 이용해 annotation한 후 label2voc.py를 이용해 mask 이미지를 생성하였고 데이터셋을 구축하였다.
 그러나 해당 데이터셋을 사용해 1차적으로 알고리즘을 학습하는 중에 병변 별로 모인 데이터 양의 차이가 심하다는 것을 인지하였고, 이것이 모델 학습에 부정적인 영향을 끼칠 것을 우려해 병변의 발생 위치나 양성/악성 정보 존재 여부에 상관 없이 병변 별 데이터 균형에 초점을 맞추어 이미지를 재선별하여 새롭게 데이터셋을 구축하였다.
 초기 데이터셋과 새로운 데이터셋 모두 데이터 분할을 거쳤고 Colab 환경에서 Classification 모델을 학습시키는데 사용되었다. 또한 이 데이터셋들을 사용해 test accuracy를 기준으로 최적의 모델을 찾기 위해 여러 비교 분석을 진행하였다.  

## 실험 결과
### 1.	Segmentation 유무에 따른 test accuracy 비교
 동일 모델 기준으로 segmentation 과정을 거치지 않은 이미지와 segmentation한 후 생성된 mask 이미지를 사용해 모델을 학습시켰을 때 loss와 test accuracy는 아래 표에 기재된 것과 같다. Segmentation 과정을 거치지 않은 이미지인 경우에 test accuracy가 높은 것을 확인할 수 있다.  
<img width="799" height="190" alt="image" src="https://github.com/user-attachments/assets/0cb5fb40-fd9a-48f5-93b5-987c0b85034e" />

  
-	Segmentation O  
<img width="495" height="332" alt="image" src="https://github.com/user-attachments/assets/97f5766d-a3bb-4fa8-b5d6-1869972c9558" />

-	Segmentation X  
<img width="489" height="327" alt="image" src="https://github.com/user-attachments/assets/db6015d7-b1d5-4598-baf8-bdf7f80b577e" />

 추가적으로 segmentation한 후 생성된 mask와 segmentation을 과정을 거치지 않은 이미지를 혼합한 데이터셋을 만들어 모델을 학습시켜 보았는데, 결과는 다음과 같다.  
<img width="939" height="201" alt="image" src="https://github.com/user-attachments/assets/60bac98c-5ba9-4cba-b8d0-5fdf7dec9b24" />

-	Segmentation O + Segmentation X  
<img width="489" height="323" alt="image" src="https://github.com/user-attachments/assets/a4c8227b-f094-40b4-b1f6-9bcff2b0e459" />

### 2.	이미지 해상도에 따른 test accuracy 비교
 모델과 불러온 데이터 사이즈가 동일한 경우로, 코드 내에서 이미지 사이즈를 변화시킨 후 모델에 적용한 후 loss과 test accuracy를 비교하였다. 이때 불러온 데이터 사이즈는 128*128이었는데, 이미 resizing을 거친 이미지이기 때문에 코드 내에서 따로 해상도가 변경되지 않았을 때 가장 test accuracy가 높았던 것으로 추측한다.  
<img width="820" height="272" alt="image" src="https://github.com/user-attachments/assets/24c6e34b-7798-441f-a9fc-1400f0cee8a0" />

### 3.	class간의 data 균형에 따른 test accuracy 비교
-	초기 데이터셋의 병변 별 이미지 수  
<img width="525" height="327" alt="image" src="https://github.com/user-attachments/assets/3ce92908-e8d6-4e98-8e1c-e8030d9c15ce" />

-	새로운 데이터셋의 병변 별 이미지 수  
<img width="517" height="315" alt="image" src="https://github.com/user-attachments/assets/80e64093-c222-42bf-9e74-8054d4e7dcaf" />

 초기에 구성한 데이터셋은 병변 별 데이터 수가 불균형했다. 데이터가 많은 병변은 몇백 장에 달했지만, 적은 병변은 열두 장에 불과했다. 이러한 불균형이 모델 학습에 부정적인 영향을 미칠 것이라 생각하여 추가적으로 데이터셋을 구성했다. 이전에 진행했던 비교 분석에서 segmentation 과정을 거치지 않은 이미지의 test accuracy가 높은 것을 확인했기 때문에 새롭게 구성한 데이터셋은 선별 후 리사이징만 시행한 이미지로 구성되어 있다. 
 이렇게 균형이 맞춰지지 않은 초기 데이터셋과 균형을 맞춘 데이터셋을 각각 epochs 10, batch 2, learning rate 0.000007로 학습 진행한 후 loss와 test accuracy를 비교해보았다. 
<img width="682" height="222" alt="image" src="https://github.com/user-attachments/assets/aeae9b6c-e470-4ec7-83ec-a2c647bea483" />

 
-	Balanced X  
<img width="466" height="310" alt="image" src="https://github.com/user-attachments/assets/dde9b60d-f353-46cd-b249-d8f8d1ca368d" />

-	Balanced O  
<img width="467" height="302" alt="image" src="https://github.com/user-attachments/assets/bc97bcda-e7d1-4e0c-8ed7-3a7007b59aed" />

 그 결과 불균형한 초기 데이터셋의 test accuracy가 더 높게 나왔다. 초기에 구성했던 데이터셋은 nevus의 데이터 양이 압도적으로 많았고, 학습과 평가 또한 편중된 nevus를 중심으로 이루어졌을 것이다. 때문에 다른 병변들은 충분히 학습되지 않았더라도 nevus의 학습 정도만 좋다면 더 높은 결과값이 나올 것이다. 따라서 좋은 학습을 위해서는 병변 간 데이터 균형을 고려한 새로운 데이터셋을 이용하는 것이 적합할 것이라고 판단하였다.
### 4.	input data의 size에 따른 test accuracy 비교
 input data size는 128*128픽셀, 256*256픽셀이고 epochs 200, batch 8, learning rate 0.000007로 각각 학습을 진행하였다. input data size가 256*256일 때 test accuracy가 더 높게 나온 것으로 확인되었다.  
<img width="683" height="162" alt="image" src="https://github.com/user-attachments/assets/720740bc-cd2c-4a51-83ad-d0f4e56edcbc" />

-	128*128  
<img width="493" height="330" alt="image" src="https://github.com/user-attachments/assets/e14dd7ea-5026-4952-905a-e2921295e3a9" />

-	256*256  
<img width="483" height="322" alt="image" src="https://github.com/user-attachments/assets/a0ae0c12-ff95-4244-b0e3-d42fb5ae96d2" />

이후에 두 input data size에 대해 learning rate는 0.000007로 고정하고 epohcs와 batch size를 변경해가며 모델을 학습시키고 test accuracy 값을 확인했는데, 이 경우에도 전반적으로 input size가 256*256일 때 test accuracy가 높게 나온 것을 확인하였다.  
<img width="667" height="441" alt="image" src="https://github.com/user-attachments/assets/50cd5536-91c6-4bcd-b5e5-52d4aa346aa2" />

<img width="689" height="434" alt="image" src="https://github.com/user-attachments/assets/d037f859-fd53-492e-8133-c8beda7ccc89" />

### 5.	batch size(hyper parameter)에 따른 test accuracy 비교
input data size는 4.에서 test accuracy가 상대적으로 높았던 256*256으로 설정하였고,  epochs 200, learning rate 0.000007으로 고정하고 batch size를 2^1에서 2^6까지 변화시켜가며 실험을 진행했다. 본 비교 분석에서는 batch size가 8일 때 test accuracy가 가장 높은 것으로 나타났다.  
<img width="879" height="275" alt="image" src="https://github.com/user-attachments/assets/94f91c02-6132-411a-b34d-77efb35976e8" />

-	2  
<img width="467" height="312" alt="image" src="https://github.com/user-attachments/assets/08ff8ae2-811c-4166-8989-882f77322a9b" />

-	4  
<img width="472" height="313" alt="image" src="https://github.com/user-attachments/assets/d5d6864f-57f4-4fc2-b2cd-d98101fb3394" />

-	8  
<img width="477" height="321" alt="image" src="https://github.com/user-attachments/assets/b42f9f0b-424a-4609-8fe3-ceef6f156a66" />

-	16  
<img width="475" height="317" alt="image" src="https://github.com/user-attachments/assets/1f6a0609-a080-44b7-b9f9-df695912fe50" />

-	32  
<img width="473" height="318" alt="image" src="https://github.com/user-attachments/assets/096f03dc-a4c6-4982-b72f-c46053f363e7" />

-	64  
<img width="472" height="321" alt="image" src="https://github.com/user-attachments/assets/ba613c14-487b-4729-8f70-b985efd79064" />

### 6.	learning rate(hyper parameter)에 따른 test accuracy 비교
input data size는 4.에서 test accuracy가 상대적으로 높았던 256*256, batch size는 5.에서 test accuracy가 상대적으로 높았던 8로 설정하였고,  epochs 10으로 고정하고 learning rate 0.001, 0.0001, 0.00007, 0.000007에 대하여 비교 분석을 실시하였다. 해당 비교 분석에서는 learning rate가 0.000007일 때 가장 높은 test accuracy를 보였다.  
<img width="857" height="217" alt="image" src="https://github.com/user-attachments/assets/edd6a0fe-0dde-4c33-8b2e-8e0a093aabb7" />

-	0.001  
<img width="437" height="296" alt="image" src="https://github.com/user-attachments/assets/6bff6e77-a0a4-4e19-ab0f-c1bbdc2d3b6f" />

-	0.0001  
<img width="438" height="290" alt="image" src="https://github.com/user-attachments/assets/60793801-158b-429a-bc9a-81a9d544e725" />

-	0.00007  
<img width="442" height="292" alt="image" src="https://github.com/user-attachments/assets/cd49d98d-22b0-43cf-b274-bc235b2be5cb" />

-	0.000007  
<img width="455" height="302" alt="image" src="https://github.com/user-attachments/assets/d0e125bd-f221-4050-b55f-20f647a73c33" />

### 7.	epochs(hyper parameter)에 따른 test accuracy 
input data size는 4.에서 test accuracy가 상대적으로 높았던 256*256, batch size는 5.에서 test accuracy가 상대적으로 높았던 8, learning rate는 6.에서 상대적으로 높았던 0.000007로 설정하고, epochs 2, 100, 200, 300에 대하여 비교 분석을 실시하였다. 그 결과 epochs이 200일 때 test accuracy가 가장 높은 것으로 나타났다.  
<img width="829" height="224" alt="image" src="https://github.com/user-attachments/assets/dd10d399-297c-457a-aab7-80128a500cb3" />

  
-	10  
<img width="473" height="314" alt="image" src="https://github.com/user-attachments/assets/d4b89f90-59b8-4d3b-a961-5d9f54bdc042" />

-	100  
<img width="456" height="305" alt="image" src="https://github.com/user-attachments/assets/c8c758ab-af1b-4eb7-877f-a8f3815e5867" />

-	200  
<img width="454" height="294" alt="image" src="https://github.com/user-attachments/assets/7f19229d-0d0c-4622-8587-e14987bd0993" />

-	300  
<img width="446" height="306" alt="image" src="https://github.com/user-attachments/assets/8e66631a-a532-4f52-b4af-0597a97f7b12" />

## Conclusion
-	 Input data size와 여러 hyper parameters 등을 변화시켜 학습한 결과 input data size 256*256, epochs 10, batch size 8, learning rate 0.000007에서 가장 높은 test accuracy 값을 보였다. 그러나 병변 진단에 있어서는 성공적인 결과가 도출되었다고 할 수는 없다. 더 좋은 성능을 위해서는 여러 요인을 변화시켜서 학습했던 결과를 바탕으로 model layer를 추가적으로 변경해야 할 것이다.

## Reference
-	 Mahbod, A., Schaefer, G., Wang, C., Dorffner, G., Ecker, R. & Ellinger, I. (2020). Transfer learning using a multi-scale and multi-network ensemble for skin lesion classification. https://doi.org/10.1016/j.cmpb.2020.105475.
-	GitHub - j05t/lesion-analysis: Skin Lesion Analysis Towards Melanoma Detection
-	GitHub - dasoto/CNN to identify malign moles on skin
-	Song Hyun Han, Soon Heum Kim, Cheol Keun Kim, Dong In Jo (2020). Multiple nonmelanocytic skin cancers in multiple regions. https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7349131/ 

