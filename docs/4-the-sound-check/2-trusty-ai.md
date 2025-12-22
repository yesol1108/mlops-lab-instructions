# TrustyAI

전통적인 소프트웨어에서는 주로 이전 섹션에서 살펴본 대기 시간(latency)과 처리량(throughput) 같은 시스템의 운영 기대치에 관심을 가집니다. 머신러닝 시스템에서는 운영 지표와 모델 성능 지표 모두에 관심을 둡니다. 이를 위해 TrustyAI가 있습니다.

TrustyAI는 모델 설명 가능성, 모델 모니터링, 책임 있는 모델 서빙과 관련된 프로젝트를 유지하는 책임 있는 AI 개발 및 배포를 위한 다양한 툴킷을 제공하는 오픈 소스 커뮤니티입니다. TrustyAI를 사용하여 데이터와 모델의 드리프트를 감지하여 모델이 예상대로 작동하는지 확인할 것입니다.

## TrustyAI 설치

1. TrustyAI는 모델과 동일한 환경(네임스페이스)에서 실행되어야 합니다. `model-deployments/test`와 `model-deployments/prod` 아래에 `trustyai` 폴더를 생성합니다. 두 환경 모두 모니터링할 것이기 때문입니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/model-deployments/test/trustyai
    touch /opt/app-root/src/mlops-gitops/model-deployments/test/trustyai/config.yaml
    mkdir /opt/app-root/src/mlops-gitops/model-deployments/prod/trustyai
    touch /opt/app-root/src/mlops-gitops/model-deployments/prod/trustyai/config.yaml
    ```

2. `test/trustyai/config.yaml`와 `prod/trustyai/config.yaml` 파일을 열고 아래 줄을 붙여넣어 Argo CD에 어떤 차트를 배포할지 알립니다.

    ```yaml
    chart_path: charts/trustyai
    ```

3. 이전과 같이 변경 사항을 저장소에 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🔦🏡 TrustyAI added 🔦🏡"
    git push
    ```

4. `test`와 `prod` 환경에 TrustyAI가 배포되었는지 확인합니다. `Model deployments`로 이동하여 `<USER_NAME>-test` 네임스페이스에서 `jukebox`를 클릭하면 이제 `Model bias`라는 새 탭이 있는 것을 확인할 수 있습니다.

    ![trustyai-model-bias.png](./images/trustyai-model-bias.png)

    또는 다음 명령어로 파드를 확인할 수도 있습니다:

     ```bash
    oc get pod  -l app=trustyai-service -n <USER_NAME>-test
    ```

    ![trustyai-cli.png](./images/trustyai-cli.png)


## 데이터 드리프트를 위한 TrustyAI 구성

대부분의 머신러닝 모델은 입력 데이터의 분포에 매우 민감합니다. 즉, 들어오는 데이터의 다양한 특성 값들이 학습 시에 본 값들의 범위와 어떻게 비교되는지가 중요합니다. 모델은 학습 데이터와 분포가 다른 데이터에 대해 성능이 저하되는 경우가 많습니다.

예를 들어, 각 나라에서 인기 있는 음악을 기반으로 히트곡을 만들려고 하는 작곡가라고 상상해보세요. 각 나라의 과거 히트곡(학습 데이터)을 연구하고 그 나라 청중에게 어필할 곡을 썼습니다. 그런데 어느 날 한 나라가 갑자기 선호하는 장르를 팝에서 일렉트로닉 댄스 음악으로 바꿨다고 가정해봅시다. 이 예상치 못한 변화(데이터 드리프트) 때문에 당신의 곡은 그 나라에서 성공할 가능성이 줄어듭니다. 이는 곡이 나쁘기 때문이 아니라 그 나라의 선호도에 대한 정보가 오래되었기 때문입니다.

이 맥락에서 데이터 드리프트는 음악 트렌드가 예상보다 빠르게 변하는 동안 오래된 트렌드를 기반으로 히트곡을 만들려고 하는 것과 같습니다.

1. `<USER_NAME>-jukebox` 네임스페이스의 Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)으로 돌아가 TrustyAI 서비스를 구성하여 모델 학습에 사용한 데이터와 요청 시 받은 데이터 간에 드리프트가 있는지 확인합니다. 마찬가지로 출력 예측에도 드리프트가 있는지 TrustyAI에 확인하도록 요청할 것입니다. Jupyter Notebook 작업 공간에서 `jukebox/4-metrics/1-trustyai_setup.ipynb`를 열고 지침을 따르세요.

설정이 완료되면 `jukebox/4-metrics/2-introducing_drift.ipynb` 노트북을 사용해 드리프트를 도입합니다. 이 노트북을 실행하세요!

    드리프트를 도입한 후, 여기로 돌아와 Prometheus에 쿼리하여 메트릭을 관찰하고 Grafana에 새 대시보드를 생성할 것입니다!📈📉

2. `OpenShift Console`에서 `Developer view` > `Observe` > `Metrics`로 이동합니다. 상단에서 `<USER_NAME>-test` 프로젝트를 선택하고 아래 쿼리를 실행하여 메트릭을 시각화합니다:

    ```bash
    trustyai_meanshift{subcategory=~"danceability|acousticness"}
    ```

    ![trusty-meanshift-metrics.png](./images/trusty-meanshift-metrics.png)


## 모델 편향을 위한 TrustyAI 구성

모델이 공정하고 편향되지 않았음을 보장하는 것은 사용자들 사이에서 모델에 대한 신뢰를 구축하는 데 매우 중요합니다. 공정성은 모델 학습 중에 탐색할 수 있지만, 실제 배포 시에야 모델이 외부 세계에 노출됩니다. 학습 데이터에서는 편향이 없더라도 실제 데이터에서 위험한 편향이 있다면 의미가 없습니다. 따라서 실제 배포 중에 모델의 공정성을 모니터링하는 것이 절대적으로 중요합니다.

우리의 경우, 데이터의 한 특성(`is_explicit`)을 사용하여 노래가 선정적일 때 모델이 특정 국가(예: `France`)에 편향되어 있는지 확인할 것입니다.

1. 이 설정은 OpenShift AI UI 또는 노트북에서 할 수 있습니다. 이번에는 UI에서 설정해보겠습니다. `OpenShift AI Dashboard` > `Models` > `Model deployments`로 이동합니다. `<USER_NAME>-test` 프로젝트를 선택합니다. `jukebox`로 가서 `Model bias`를 클릭한 후 `Configure`를 누릅니다.

    ![bias-monitoring.png](./images/bias-monitoring.png)

2. 아래와 같이 폼을 작성합니다:

    - Metric name: `fairness`
    - Metric type: `Statistical Parity Difference (SPD)`
    - Protected attribute: `is_explicit`
    - Privileged value: `1.0`
    - Unprivileged value: `0.0`
    - Output: `output-13`
    - Output value: `0.5`
    - Violation threshold: `0,1`
    - Metric batch size: `1000`

    ![bias-monitoring-2.png](./images/bias-monitoring-2.png)

3. `View Metrics`를 클릭하면 다음과 같은 화면을 볼 수 있습니다:

    ![bias-monitoring-3.png](./images/bias-monitoring-3.png)

    또는 노트북 `jukebox/4-metrics/1-trustyai_setup.ipynb`에 새 셀을 만들고 아래 코드를 추가한 후 실행하여 동일한 결과를 얻을 수 있습니다.

    이미 UI에서 완료했다면 실행할 필요 없습니다. 파이썬을 선호하는 분들을 위한 대안입니다 :)

    ```python
    # Get bias for a specific field-couple
    endpoint = "/metrics/group/fairness/spd"
    url = urljoin(base_url, endpoint)

    payload = {
        "modelId": model_name,
        "protectedAttribute": "is_explicit",
        "privilegedAttribute": 1.0,
        "unprivilegedAttribute": 0.0,
        "outcomeName": "output-13",
        "favorableOutcome": 0.5,
        "batchSize": 1000
    }

    response = requests.post(url, headers=headers, json=payload)
    print(response.text)
    ```

## Grafana에 새 대시보드 업데이트

운영 지표와 모델 성능 관련 지표를 같은 대시보드에서 보고 싶을 수 있습니다. 이를 위해 이전 Grafana 대시보드를 확장하여 모델 상태를 한눈에 볼 수 있습니다.

1. 모든 것을 코드로 정의합니다. 대시보드의 JSON 정의는 Gitea에서 [여기](https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/grafana/templates/grafana-dashboard-ml.yaml)에서 확인할 수 있습니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server) 편집기에서 `mlops-gitops/toolings/grafana/config.yaml` 파일을 열고 아래와 같이 업데이트하세요:

    ```yaml
    chart_path: charts/grafana
    include_trusty: true  # 👈 add this
    ```

2. 변경 사항을 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🔦 TrustyAI metrics visualization added 🏡"
    git push
    ```

3. Grafana로 돌아가 Jukebox에 대한 업데이트된 대시보드를 확인합니다;

    ```bash
    # get the route and open it in your browser
    echo https://$(oc get route jukebox-grafana-route --template='{{ .spec.host }}' -n <USER_NAME>-toolings)
    ```

    `Log in with OpenShift`를 사용해 로그인하고 대시보드를 표시하세요. `Dashboards` > `grafana <USER_NAME>-toolings Dashboards` > `OpenVINO Model Server - Model Metrics - Trustyai`로 이동합니다.

    ![grafana-with-trusty.png](./images/grafana-with-trusty.png)