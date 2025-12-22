# 모델 보안

AI 모델은 강력한 도구이지만, 보안 위험도 함께 수반합니다. 다른 소프트웨어와 마찬가지로, 데이터 탈취, 결과 조작, 무단 접근을 노리는 공격자들의 표적이 될 수 있습니다. 그중 하나의 일반적인 공격 방법은 **모델 직렬화 공격(Model Serialization Attacks)** 입니다.

그게 무엇일까요? 기본적으로 모델이 저장되고 로드될 때 직렬화(serialization)라는 과정을 거칩니다. 이 과정이 제대로 처리되지 않으면 악성 코드를 삽입할 수 있습니다. 이러한 공격은 다음과 같은 결과를 초래할 수 있습니다:

- 자격 증명 탈취 – 공격자가 모델에 저장된 클라우드 자격 증명을 추출하여 다른 시스템에 접근할 수 있습니다.
- 데이터 탈취 – 모델에 들어오는 요청에는 종종 중요한 정보가 포함되어 있습니다. 공격자가 이를 가로챌 경우, 개인 정보나 민감한 데이터를 탈취할 수 있습니다.
- 데이터 중독(Data Poisoning) – 공격자가 모델에 도달하기 전 입력 데이터를 조작하여 오도하거나 해로운 결과를 생성하게 할 수 있습니다. 이는 자동화된 의사결정 시스템에서 특히 위험합니다.
- 모델 중독(Model Poisoning) – 단순히 데이터를 훼손하는 대신, 공격자가 모델 자체를 변경하여 시간이 지남에 따라 편향되거나 부정확하며 악의적인 출력을 내도록 할 수 있습니다.

이러한 공격을 방어하기 위해, 모델을 사용하기 전에 보안 취약점을 스캔할 수 있습니다.

## 모델 스캔 탐색하기

1. 오픈 소스 도구인 `modelscan`을 사용하여 모델에 안전하지 않은 코드가 포함되어 있는지 스캔해 보겠습니다. 먼저 Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)으로 이동하여 도구에 익숙해집니다. 노트북 `8-securing_ai/1-modelscan.ipynb`를 열고 셀을 차례대로 실행해 보세요.

## 파이프라인에 모델 스캔 포함하기

1. 이제 모델이 생성된 후, `modelcar` 아티팩트를 생성하기 전에 파이프라인에 작업을 추가해 보겠습니다. 모델에 취약점이 있다면, 배포하지 않을 것이므로 OCI 레지스트리에 저장할 필요가 없습니다.

    이를 위해 업데이트가 필요합니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)으로 이동하여 `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일에 아래 줄을 추가하세요. 이 작업은 노트북에서 실행한 것과 동일한 명령을 실행하는 Tekton 작업을 활성화합니다:

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true
    linting: true 
    static_code_analysis: true
    static_code_analysis_secret: sonarqube-auth
    model_scanning: true # 👈 add this
    ```

2. 변경 사항을 커밋하고 git에 푸시합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🦠 Model scanning added 🦠"
    git push
    ```

    이제 파이프라인은 다음과 같이 보일 것입니다:  
    ![model-scan-task.png](./images/model-scan-task.png)

3. _선택 사항_: 이제 빈 커밋을 생성하여 파이프라인을 트리거하고 스캔을 수행할 수 있습니다. 물론 더 흥미로운 도구들을 추가하며 파이프라인을 확장할 수도 있습니다!

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "☢️ trigger pipeline for model scanning ☢️"
    git push
    ```

    파이프라인 실행 후, 다음과 같은 출력 결과를 확인할 수 있습니다 🎉

    ![model-scan-output.png](./images/model-scan-output.png)