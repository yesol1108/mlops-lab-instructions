# 단위 테스트

단위 테스트는 소프트웨어 개발에서 일반적으로 사용되는 방법으로, 코드의 각 구성 요소를 개별적으로 테스트하여 각각이 예상대로 작동하는지 확인할 수 있습니다.  
놀랍지 않게도, AI 개발에서도 모델을 생성하고 평가하는 모든 구성 요소가 제대로 작동하는지 확인하기 위해 이 작업을 수행하고자 합니다.

우리는 이미 파이프라인 구성 요소 중 하나에 대한 단위 테스트를 설정했습니다. 한번 실행해 봅시다!

1. 코드 서버 작업 공간에서 터미널을 열고 아래 코드를 실행하세요:

    ```bash
    cd /opt/app-root/src/
    git clone https://<USER_NAME>:<PASSWORD>@gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox.git
    cd /opt/app-root/src/jukebox/3-prod_datascience
    pip install -r tests/requirements.txt
    PYTHONPATH=$(pwd) PYTHONDONTWRITEBYTECODE=1 pytest tests/test_fetch_data.py -p no:cacheprovider
    ```  
    코드를 살펴보고 싶다면 `jukebox/3-prod_datascience/tests/test_fetch_data.py`에서 확인할 수 있습니다.  
    여기서는 로드한 데이터가 예상한 수와 순서의 컬럼을 가지고 있는지 테스트하고 있습니다.  
2. 잠시 후(몇 분 정도) 다음과 유사한 출력이 나타납니다:  
    ![pytest-output](./images/pytest-output.png)  
    테스트를 통과한 것 같습니다!

## 자동 단위 테스트

이제 연속 학습 파이프라인을 테스트할 수 있게 되었으니, 코드가 변경될 때마다 테스트가 자동으로 실행되도록 설정해 봅시다(좋은 소프트웨어 개발자처럼 🧑‍💻).  
이를 위해, 단위 테스트를 학습 파이프라인에 추가하면 코드 변경 시마다 실행됩니다.

1. `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열고 `unit_tests: true`를 추가하여 자동 테스트를 활성화하세요:

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true # 👈 add this
    ```

2. 변경 사항을 git에 푸시합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🧪 unit tests added 🧪"
    git push
    ```

    OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 Pipelines로 이동하면 파이프라인에 `unit-tests`라는 작업이 추가된 것을 확인할 수 있습니다:

    ![unit-test-task.png](./images/unit-test-task.png)

3. 원한다면, 아래와 같이 Jukebox 저장소에 빈 커밋을 푸시하여 파이프라인 실행을 시작하고 단위 테스트가 어떻게 실행되는지 확인할 수 있습니다. 하지만 본격적으로 파이프라인에 더 흥미로운 테스트와 검증을 추가해 봅시다!

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🤞 trigger pipeline for unit-testing 🤞"
    git push
    ```

4. 이번에는 파이프라인이 `unit-tests` 단계를 통과하는 것을 확인하세요 🥳

    ![unit-test-task.png](./images/unit-test-task-completed.png)