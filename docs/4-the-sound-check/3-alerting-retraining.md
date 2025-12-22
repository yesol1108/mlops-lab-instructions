## 알림 및 트리거링 학습 파이프라인

### 알림 구성

OpenShift의 모니터링 스택에는 특정 임계값을 초과하거나 미만일 때 알림을 트리거할 수 있는 Alert Manager가 있습니다. 예를 들어, 데이터 드리프트 감지를 위해 TrustyAI를 구성했으므로 데이터가 드리프트하기 시작할 때 알림을 받고 싶을 것입니다. 이를 위해 `PrometheusRule`을 생성해야 합니다.

1. `model-deployments/test`와 `model-deployments/prod` 아래에 `alerting` 폴더를 생성합니다. 두 환경 모두 모니터링하기 위함입니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/model-deployments/test/alerting
    touch /opt/app-root/src/mlops-gitops/model-deployments/test/alerting/config.yaml
    mkdir /opt/app-root/src/mlops-gitops/model-deployments/prod/alerting
    touch /opt/app-root/src/mlops-gitops/model-deployments/prod/alerting/config.yaml
    ```

2. 아래 코드 스니펫을 **test/alerting/config.yaml** 및 **prod/alerting/config.yaml** 파일 모두에 붙여넣어 Argo CD에 배포할 차트를 알립니다.

    ```yaml
    chart_path: charts/alerting
    name: jukebox
    user: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    ```

    이는 아래와 같이 `PrometheusRule`을 생성하여 `test` 및 `prod` 환경에서 `danceability` 특성의 변화를 모니터링합니다.

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
        ---
        apiVersion: monitoring.coreos.com/v1
        kind: PrometheusRule
        metadata:
        name: jukebox-alerts
        spec:
        groups:
        - name: jukebox.rules
            rules:
            - alert: jukebox-datadrift-alert
            annotations:
                message: 'jukebox meanshift p-value has dropped below 0.05 for danceability, 
                          indicating a drift in data over the last 5000 samples compared to the training data.'
            expr: trustyai_meanshift{namespace="{{ .Release.Namespace }}", subcategory="danceability"}<0.05
            for: 10m
            labels:
                severity: "critical"
    </code></pre></div>
    

3. 이전과 같이 변경 사항을 저장소에 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🚨 Alert definition added 🚨"
    git push
    ```

5. OpenShift 콘솔의 Developer 뷰로 이동하여 `Observe` > `Alerts`에서 방금 생성한 `<USER_NAME>-test` 프로젝트의 알림을 확인합니다. 잠시 후 `Firing` 상태가 됩니다. 아직 `Firing` 상태가 보이지 않는다면 이전 챕터 "TrustyAI"에서 드리프트를 유발하는 노트북을 실행했는지 확인하세요.

    ![alert-1.png](./images/alert-1.png)


### 알림 기반 재학습 파이프라인 트리거

데이터 드리프트나 편향과 같은 중요한 이벤트를 모니터링하고 알림을 받는 것은 중요합니다. 하지만 알림만으로는 충분하지 않습니다. 모델이 신뢰성을 유지하고 기대한 성능을 발휘하도록 하려면 신속하고 효과적으로 대응해야 합니다.

드리프트나 기타 이상이 감지되면 자동 재학습 파이프라인을 트리거하여 문제를 해결할 수 있습니다. 이는 새로 들어오는 데이터로 학습할 수 있는 경우 매우 효과적인 전략입니다. Alert Manager를 구성하여 파이프라인을 트리거해 보겠습니다.

1. Tekton 파이프라인의 웹훅 URL을 아는 `Alertmanager Config`를 생성합니다. `test/alerting/config.yaml`과 `prod/alerting/config.yaml`을 아래와 같이 업데이트하여 이 구성을 활성화합니다.

    ```yaml
    chart_path: charts/alerting
    name: jukebox
    user: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    alert_manager: true # 👈 add this
    ```

    이는 Tekton 파이프라인의 웹훅을 가리키는 AlertManager 구성을 생성합니다.

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
        ---
        apiVersion: monitoring.coreos.com/v1beta1
        kind: AlertmanagerConfig
        metadata:
        name: jukebox-alerting
        spec:
          route:
            receiver: default
        receivers:
        - name: default
            webhookConfigs:
            - url: >-
                https://el-ct-listener-<USER_NAME>-toolings.<CLUSTER_DOMAIN>/
    </code></pre></div>


2. Alertmanager의 트리거는 Git 저장소의 트리거와 매우 다릅니다. 서로 다른 유형의 페이로드와 정보를 웹훅에 전송하기 때문입니다. 따라서 학습 파이프라인에도 일부 변경이 필요합니다. `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열어 다음과 같이 업데이트합니다.

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> # 👈 add this
    alert_trigger: true # 👈 add this
    ```

3. 변경 사항을 저장소에 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git add .
    git commit -m "🔔 Alertmanager Config added 🔔"
    git push
    ```

4. `OpenShift Console` > `<USER_NAME>-toolings` > `Pipelines`로 이동하여 알림 파이프라인이 트리거되었는지 확인합니다.

    ![alert-pipeline.png](./images/alert-pipeline.png)

    파이프라인 실행이 완료되면 데이터 드리프트로 인해 모델 레지스트리에 새 버전이 등록된 것을 확인할 수 있습니다.

    ![alert-model-registry.png](./images/alert-model-registry.png)