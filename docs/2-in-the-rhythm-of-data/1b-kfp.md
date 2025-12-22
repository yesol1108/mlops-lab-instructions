## Kubeflow Pipelines (KFP)

Kubeflow Pipelines(KFP)는 컨테이너를 사용해 **이식 가능하고 확장 가능한 머신러닝 파이프라인을 구축하고 배포**하기 위해 설계된 플랫폼입니다  
(아니요, 쿵푸 팬더 🐼 와는 관련 없습니다).  
KFP는 **버저닝, 메타데이터 추적, 리소스 관리**와 같은 고급 기능을 제공하며,  
이를 통해 팀이 파이프라인을 효율적으로 모니터링하고 최적화할 수 있습니다.  
이러한 기능들은 현재 Elyra에서는 제공되지 않는 부분이기도 합니다.

Elyra가 빠른 실험과 프로토타이핑에 적합했다면,  
KFP는 파이프라인을 **지속적으로 실행하기 위한 견고함**을 제공합니다.  
KFP를 사용하면 더 나은 로깅, 오류 처리, 재시도 로직 등  
프로덕션 수준의 기능들을 활용할 수 있습니다.  
즉, 이것이 우리의 **프로덕션 등급 트레이닝 파이프라인**이 됩니다.  

또한 소스 코드 변경, 새로운 데이터 유입,  
혹은 모델의 이상 동작을 알리는 알림에 따라  
이 파이프라인이 **자동으로 트리거**되도록 구성할 수 있습니다.  
(스포일러, 스포일러 🤭🤭)

파이프라인의 단계는 다음과 같습니다.  
![pipeline-steps.png](./images/pipeline-steps.png)

1. 먼저 모델 학습에 사용할 데이터를 수집합니다.
2. 이후 데이터를 **검증(validation)** 하고 **전처리(preprocessing)** 합니다.  
   이 두 단계는 병렬로 수행되지만, 검증 단계가 실패하면 파이프라인은 중단됩니다.  
   전처리 단계에서는 이너 루프에서 했던 것처럼  
   모든 데이터를 숫자로 변환하고 정규화합니다.
3. 데이터 준비가 완료되면 모델을 학습합니다.
4. 모델 학습이 끝난 후, 성능이 충분히 좋은지 평가합니다.  
   이와 동시에 모델을 ONNX 형식으로 변환합니다.  
   이는 모델 서빙에 사용할 형식이기 때문입니다.
5. ONNX 모델이 생성되면, 테스트 데이터를 사용해  
   원본 Keras 모델과 동일한 성능을 내는지 검증합니다.
6. 마지막으로, 모델 서버가 접근할 수 있는 위치에  
   모델을 저장함으로써 파이프라인을 마무리합니다. 🎉

이제 파이프라인이 어떤 역할을 하는지 이해했으니,  
직접 실행해 보겠습니다! 🏃‍♂️

1. 파이프라인 정의 파일은 `3-prod_datascience` 폴더에 있습니다.

    이 폴더의 파일들을 살펴보면,  
    이전에 노트북에서 수행했던 단계들과 대부분 동일하다는 것을 알 수 있습니다.  
    다만 각 단계가 개별 함수로 분리되고,  
    여러 파일로 구성되어 모듈성이 향상되었습니다.  
    이를 통해 파이프라인을 더 쉽게 수정하고 유지보수할 수 있습니다.

    ![kfp.png](./images/kfp.png)

2. 이너 루프에서 Model Registry URL을 설정했던 것과 마찬가지로,  
   파이프라인 정의 파일에 클러스터 도메인을 업데이트합니다.  
   `jukebox/3-prod_datascience/prod_train_save_pipeline.py` 파일을 열고  
   아래 코드에서 플레이스홀더를 찾아 교체하세요  
   (대략 120번째 줄 근처에 있습니다).  
   수정 후 반드시 파일을 저장하세요 👻

    ```bash
        metadata = {
        "hyperparameters": {
            "epochs": 2
        },
        "model_name": "jukebox",
        "version": "0.0.2",
        "cluster_domain": "<CLUSTER_DOMAIN>", # 👈 여기에 클러스터 도메인을 입력하세요
        "model_storage_pvc": "jukebox-model-pvc",
        "prod_flag": False
    }
    ```

2. 앞서 언급했듯이, 이 파이프라인은 원래 수동으로 실행하는 것이 목적은 아닙니다.  
   하지만 기능을 테스트하고 출력 결과를 확인하기 위해,  
   `prod_train_save_pipeline.py` 파일에서 ▶️ 버튼을 눌러 실행해 보겠습니다.

    ![kfp-run.png](./images/kfp-run.png)

    `Console Output`에서 아래와 같은 출력이 보이면  
    파이프라인이 성공적으로 시작된 것입니다 🐍

    ```bash
        Connecting to Data Science Pipelines: https://ds-pipeline-dspa.<USER_NAME>-jukebox.svc:8443
        /opt/app-root/lib64/python3.11/site-packages/kfp/dsl/component_decorator.py:121: FutureWarning: The default base_image used by the @dsl.component decorator will switch from 'python:3.8' to 'python:3.9' on Oct 1, 2024. To ensure your existing components work with versions of the KFP SDK released after that date, you should provide an explicit base_image argument and ensure your component works as intended on Python 3.9.
        return component_factory.create_component_from_func(
        /opt/app-root/lib64/python3.11/site-packages/kfp/client/client.py:159: FutureWarning: This client only works with Kubeflow Pipeline v2.0.0-beta.2 and later versions.
        warnings.warn(
        <IPython.core.display.HTML object><IPython.core.display.HTML object>
    ```

3. OpenShift AI Dashboard로 이동합니다.  
   왼쪽 메뉴에서 `Experiments`를 선택한 뒤  
   `Experiments and runs`로 이동하세요.  
   상태가 `Running`인 실행 하나가 보일 것입니다.  
   이를 클릭해 파이프라인 실행 상세 정보를 확인합니다.

    ![experiments.png](./images/experiments.png)

    여기서 다음과 같은 다양한 정보를 확인할 수 있습니다.

    - 각 단계 간의 관계
    - 각 단계의 출력 결과
    - 생성된 아티팩트
    - 메트릭 및 그래프

    ![experiments-2.png](./images/experiments-2.png)

    이 화면을 충분히 살펴보며 기능에 익숙해지세요.  
    첫 실행에서는 파이프라인이 완료되기까지 다소 시간이 걸릴 수 있지만,  
    끝날 때까지 기다릴 필요는 없습니다.  
    다음 단계로 바로 넘어가셔도 됩니다!

4. 다음 단계에서는 MLOps 환경을 설정하여,  
   이 파이프라인이 자동으로 실행되도록 하고  
   다양한 MLOps 실천 방법을 적용할 수 있도록 합니다.  
   이제 본격적인 **MLOps 여정**이 시작됩니다! 🙌