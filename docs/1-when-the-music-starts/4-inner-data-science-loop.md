## 데이터 사이언스 이너 루프 (Data Science Inner Loop)

전통적인 소프트웨어 개발과 마찬가지로, 여기서 말하는 **이너 루프(inner loop)** 는  
머신러닝 모델을 **구현 → 테스트 → 개선**하는 반복적인 과정을 의미합니다.

이 이너 루프는 데이터 사이언스에서 매우 중요합니다.  
모델을 지속적으로 개선하고 최적화할 수 있게 해주기 때문입니다.  
일반적으로 ML 모델을 구축할 때 거치는 주요 단계는 다음과 같습니다.

![inner-loop.png](./images/inner-loop.png)

- **데이터 준비 (Data Preparation)**:  
  모델 학습에 적합하도록 데이터를 수집, 정제, 변환하는 단계
- **모델 개발 (Model Development)**:  
  적절한 머신러닝 알고리즘을 선택하고 구현하는 단계
- **모델 학습 (Model Training)**:  
  준비된 데이터를 모델에 입력하고, 패턴과 관계를 학습하도록 파라미터를 조정하는 단계
- **모델 평가 (Model Evaluation)**:  
  정확도, 정밀도, 재현율, F1-score 등의 지표를 사용해 모델 성능을 평가하는 단계
- **모델 개선 (Model Refinement)**:  
  이전 단계를 반복 수행하며 모델의 정확도와 일반화 성능을 향상시키는 단계

---

1. 이제 워크벤치에 클론해 둔 노트북을 사용해 위 단계를 직접 경험해보겠습니다.  
   모든 데이터 사이언스 프로젝트의 첫 단계인 **데이터 탐색**부터 시작합니다.  
   아래 노트북을 먼저 열어주세요.  
   `jukebox/1-data_exploration/1-data_exploration.ipynb`

    ![jupyter_notebook.png](./images/jupyter_notebook.png)

    Jupyter Notebook에서 각 박스는 `cell` 이라고 부릅니다.  
    셀에는 다음과 같은 세 가지 종류가 있습니다.  
    코드 셀은 코드를 작성하고 실행하는 셀이며,  
    마크다운 셀은 설명이나 문서를 작성하는 셀입니다.  
    Raw 셀은 이번 실습에서는 신경 쓰지 않아도 됩니다.

    코드 셀을 실행하는 방법은 다음과 같습니다.

    1. 실행할 셀을 선택합니다.
    2. 상단 바의 ▶️ 버튼을 클릭하거나 Shift+Enter를 누릅니다.
    3. 셀이 실행되고, 결과가 바로 아래에 출력됩니다.  
       이 동작은 커서를 다음 셀로 이동시켜 자연스럽게 실습을 이어갈 수 있게 해줍니다.

> 셀을 실행하면 셀 앞에 `[*]` 표시가 나타나며, 이는 실행 중임을 의미합니다.  
> 실행이 완료되면 `[21]` 과 같은 숫자가 표시되는데,  
> 이는 해당 셀이 실행된 순서를 나타냅니다.

1. 이제 노트북을 실행할 시간입니다.  
   우리의 목표는 **댄서빌리티(danceability)**, **어쿠스틱니스(acousticness)** 와 같은  
   곡의 특성을 기반으로, 어떤 국가에서 해당 노래를 좋아할지 예측하는 모델을 만드는 것입니다.

    아래 노트북을 **순서대로 실행**하세요.

    1. `jukebox/1-data_exploration/1-data_exploration.ipynb`: 데이터셋 분석  
    2. `jukebox/2_dev_datascience/1-experiment-train.ipynb`: 모델 생성  
    3. `jukebox/2_dev_datascience/2-save_model.ipynb`: 학습된 모델 저장  

    시작하기 전에 다음 사항을 확인하세요.
    - 셀은 **반드시 순서대로, 하나씩 실행**합니다.  
    - 각 노트북의 마지막에 있는 안내 문구까지 **모두 주의 깊게 읽어주세요**.  
      다음 단계로 진행하기 위한 안내가 포함되어 있습니다.

    마지막 노트북까지 완료하면,  
    모델은 MinIO의 `models` 버킷에 업로드되어 있어야 합니다.  
    완료 후 다시 이 문서로 돌아와 다음 단계를 진행하세요! 😁

    이제 첫 번째 노트북부터 시작해봅시다:  
    `jukebox/1-data_exploration/1-data_exploration.ipynb` 🏃💨

    **....**

    **...**

    ![one-eternity-later-sponge-bob](./images/one-eternity-later-sponge-bob.png)

2. 다시 오신 것을 환영합니다! 👋  
   이제 MinIO에 학습된 모델이 저장되어 있으며,  
   OpenShift AI를 통해 서빙할 준비가 완료되었습니다.

    ![model_in_bucket.png](./images/model_in_bucket.png)

모델을 배포하고 테스트하기 전에, Model Registry에 대해 먼저 알아보겠습니다.

## Model Registry

OpenShift AI Dashboard에서 등록된 모델을 확인할 수 있으며,  
여기서 바로 모델을 배포할 수도 있습니다.

1. `Models` > `Model registry`로 이동한 뒤,  
   `<USER_NAME>-registry`를 보고 있는지 확인합니다.

![model-registry-1.png](./images/model-registry-1.png)

2. `jukebox`를 클릭하여 사용 가능한 버전을 확인합니다.  
   현재는 `0.0.1` 버전만 존재합니다.

![model-registry-2.png](./images/model-registry-2.png)

여기에서는 모델이 저장된 위치, 버전, 모델 세부 정보 등을 확인할 수 있습니다.  
현재는 실험 단계이기 때문에 정보가 많지 않지만,  
아우터 루프 단계로 넘어가면 다음과 같은 메타데이터가 점점 추가됩니다.

- 어떤 학습 데이터로 생성되었는지  
- 모델의 정확도 수준  
- 어떤 파이프라인 실행 결과로 생성되었는지  

즉, Model Registry는 모델에 대한 ✨**canonical 메타데이터 소스**✨로 활용됩니다.  
이제 모델을 배포하고, 컨테이너 안에서 정상적으로 동작하며  
예측 결과를 반환하는지 확인해보겠습니다.

## Model Serving

모델 아티팩트가 버킷에 저장되었으므로,  
이제 데이터 사이언스 프로젝트에 모델을 배포할 수 있습니다.

OpenShift AI와 그 기반 기술인 KServe의 장점은  
모델의 컨테이너화나 런타임을 직접 신경 쓰지 않아도 된다는 점입니다.  
이러한 복잡한 부분은 모두 추상화되어 있으며,  
우리는 적절한 런타임을 선택하고 모델 위치만 지정하면 됩니다.

직접 해보겠습니다.

1. Model Registry로 이동해 방금 등록한 모델을 찾습니다.  
   `Deploy`를 클릭하고, 배포 대상 프로젝트로 `<USER_NAME>-jukebox`를 선택합니다.
   
    ![deploy-from-registry.png](./images/deploy-from-registry.png)

2. 아래 정보로 폼을 수정합니다.

- Model deployment name: `jukebox`
- Serving runtime: `OpenVino Model Server`
- Model framework (name - version): `onnx - 1`
- Deployment mode: `Advanced` 
- Model server replicas: `1`
- Model server size: `Small`
- Model route:
  - `Make deployed models available through an external route` 선택
  - `Require token authentication`은 **체크 해제**

    나머지는 그대로 두고 `Deploy`를 클릭합니다.

    ![jukebox.png](./images/jukebox.png)
    ![jukebox-2.png](./images/jukebox-2.png)

1. OpenShift AI가 백그라운드에서  
   런타임 이미지 pull, 모델 다운로드, 파일 복사, 런타임 기동 등의 작업을 수행하기 때문에  
   다소 시간이 걸릴 수 있습니다.  
   완료되면 모델과 상호작용할 수 있는 엔드포인트가 생성됩니다.  
   시간이 오래 걸리면 페이지를 새로고침해도 됩니다.

    ![jukebox-deployed.png](./images/jukebox-deployed.png)

2. **External URL**을 복사한 뒤 워크벤치로 돌아갑니다.  
   `jukebox/2-dev_datascience/3-request_model.ipynb` 노트북을 열고  
   안내에 따라 예측 요청을 보내보세요 🎶

    ![jukebox-deployed-2.png](./images/jukebox-deployed-2.png)
