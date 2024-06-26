# AI


## 1. Environment/Usage
- Python 3.8.18
- Pytorch 1.12.1
Environment
<pre><code>conda install pytorch torchvision torchaudio cpuonly -c pytorch
conda install -c conda-forge opencv
pip install albumentations
pip install timm
pip install Pillow
pip install albumentations
pip install torch torchvision
pip install pydantic
pip install fastapi
pip install opencv-python-headless    </code></pre>

Usage
<pre><code>python classification_DDalki-train.py
python classification_OE-train.py
...
</code></pre>

## 2. Dataset

![added](https://github.com/mukkbo/Crop-disease-classification/assets/133736337/7436df31-ccfc-48fe-92fc-48e97c00ec54)

### 포도 질병 예시

<div align="center">
  <img src="https://github.com/mukkbo/Crop-disease-classification/assets/133736337/62b15472-c531-40f7-9a65-7dde9bd3632a" width="224" height="224"/>
  <p align="center">노균병</p>
</div>
<div align="center">
  <img src="https://github.com/mukkbo/Crop-disease-classification/assets/133736337/3989ef26-0e54-406f-b659-89e850137f24" width="224" height="224"/>
   <p align="center">탄저병</p>
</div>
<div align="center">
  <img src="https://github.com/mukkbo/Crop-disease-classification/assets/133736337/ff0773d1-b6ec-4c1d-80d7-1454a75733f7" width="224" height="224"/>
   <p align="center">축과병</p>
</div align="center">
<div align="center">
  <img src="https://github.com/mukkbo/Crop-disease-classification/assets/133736337/e38f617d-852d-44bd-9056-bce1c54d26bc"  width="224" height="224"/>
   <p align="center">일소피해</p>
</div>

## 3. Models
  #### a. ResNet50
  #### b. Simple-ViT
  #### c. MAE-ViT

앞서 소개한 Dataset은 포도 외 5가지 작물이 추가적으로 있습니다.

포도를 예시로 설명을 진행합니다.
각각의 이미지는 탄저병, 노균병, 축과병, 일소피해, 정상 이렇게 5가지 class로 분류됩니다.
이 5가지 class는 fine grained task로 단순하게, 고양이, 비행기, 강아지 같은 간단한 분류와는 다르게 구분하기 어려운 것을 분류해내야 하는 task입니다.
그렇기에 높은 정확도를 보이기는 어려웠습니다. 하지만 여러 모델을 사용하여 가장 정확도가 높은 모델을 찾는 것을 목표로 실험을 진행하였습니다.

ResNet50에서는 65%의 정확도를 보였습니다. 
confusion matrix를 분석한 결과, 노균병과 축과병은 눈으로 식별하기에도 차이가 크지 않았기에 모델이 분류에 어려움을 보였습니다.

그를 극복하기 위해 전체적인 부분에 대해 학습하는 supervised learning을 사용하는 대신 Masked Autoencoder를 통한 학습방식이 이미지의 질감과 같은 복잡한 패턴을 잘 학습할 수 있다고 판단하여, MAE로 학습을 진행하도록 하였습니다. 
하지만 여기서 짚고 넘어가야하는 점은 ViT 자체가 ResNet50보다 성능이 좋다는 것은 이미 자명한 사실이기에 이것의 성능자체가 MAE를 통해 classification 해서 성능이 우수해진 것인지 아니면, ViT를 사용해서 성능이 우수해진 것인지 자체에 대한 판단이 불가능했습니다. 그렇기 때문에 이를 위해 단순한 simple ViT를 통한 classification을 하여 MAE ViT와 성능을 비교하였습니다.

## 4. Experiments

실험결과를 비교하기 위해, 포도 작물에 대해 아래 4가지 모델의 성능을 비교하였습니다.

모델은 다음과 같습니다. 
1. ResNet50
2. ViT w/o fine-tuned
3. ViT w/ fine-tuned
4. MAE-ViT w/o fine-tuned

이때, MAE-ViT같은 경우는 fine-tuning을 위한 데이터가 부족하여 오히려 MAE-ViT w/ fine-tuned모델의 성능이 급격하게 떨어지는 것을 확인하였습니다. 그렇기에 비교를 위해서 MAE-ViT w/ fine-tuned 모델만 사용하였습니다.

포도 모델의 성능은 각각 다음과 같습니다.

![image](https://github.com/mukkbo/Crop-disease-classification/assets/133736337/46f44caa-06f4-4c60-ad98-04904e40ca7d)

여기서 말하는 w/o fine-tuned는 Encoder를 frozen한 상태로 fc layer에서만 학습을 진행한 결과를 뜻 합니다.

이 결과로 인해 MAE의 학습방식이 모델을 효과적으로 분류할 수 있다는 사실은 유추하는 것을 실패했습니다. 다른 작물에서의 결과비교를 위해 다른 작물들에 대해서도 같은 실험을 반복하였습니다.

![image](https://github.com/mukkbo/Crop-disease-classification/assets/133736337/71e1f413-5a12-4986-8e06-fb42f32c6822)

MAE-ViT를 이용해 특정 작물의 feature vector에 대해 더 잘 학습하려 하였지만, 이 시도는 위 테이블의 결과에 따르면 포도를 제외한 5가지 작물 전부에서 ResNet50 보다 낮은 성능을 보이면서 유추에 실패하였습니다.

또한 일부 작물에서는 ViT보다 ResNet50의 성능이 더 우수하다고 알려져 있음에도 데이터셋에 따라 성능은 달라질 수 있다는 점을 알 수 있었습니다. 또한 적합한 모델을 선택하는 것이 중요하다는 점을 확인할 수 있었습니다.

이에 따라 최종 모델은 딸기, 토마토, 오이, 고추의 경우는 ResNet50 모델을 사용하고, 파프리카, 포도 작물은 ViT w/ fine-tuned model을 사용하였습니다
