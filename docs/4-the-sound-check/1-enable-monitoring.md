## 사용자 작업 부하 모니터링

> OpenShift에는 모니터링 기능이 내장되어 있습니다. Prometheus 스택을 배포하고 OpenShift 콘솔과 통합하여 클러스터 메트릭을 소비할 수 있습니다.

1. 기본 제공 대시보드를 살펴보겠습니다. OpenShift AI Dashboard > `<USER_NAME>-test` > Models > Model deployments 뷰로 이동하여 `jukebox` 모델을 클릭하세요.

    ![test-model-serving.png](./images/test-model-serving.png)

    들어오는 요청, CPU 및 메모리 사용량에 대한 그래프를 확인하여 모델 활용도를 파악할 수 있습니다.

    ![model-metrics-dashboard.png](./images/model-metrics-dashboard.png)


2. 런타임에서 노출하고 OpenShift가 기본적으로 수집하는 다른 메트릭도 있습니다. Prometheus용 쿼리 언어인 `promql`을 사용하여 기존 메트릭에 대한 쿼리를 쉽게 실행할 수 있으며, 시각화할 추가 메트릭이 있는지 결정할 수 있습니다. `OpenShift Console` > `Developer view`에서 `Observe`로 이동하면 `OpenShift AI Dashboard`와 유사한 기본 상태 지표가 표시됩니다.

    ![model-metrics-dashboard-2.png](./images/model-metrics-dashboard-2.png)

    이미 배포된 `jukebox` 모델을 관찰할 수 있는 여러 대시보드가 준비되어 있습니다. Jukebox UI를 사용하여 트래픽을 생성하고 여기에서 확인하세요!


3. 또한 promql 쿼리 언어를 사용하여 테스트 환경에서 `jukebox`에 대한 성공적인 요청 정보를 Prometheus에 쿼리할 수 있습니다.

    `Metrics` 탭으로 이동하여 아래 쿼리를 붙여넣고 `Enter`를 누르세요.

    ```bash
    ovms_requests_success{namespace="<USER_NAME>-test", interface="REST"}
    ```

    ![query-metrics.png](./images/query-metrics.png)

    _참고: "Access restricted" 경고는 단순한 외관상의 오류이며 메트릭 쿼리에 영향을 주지 않으므로 무시하세요._

### Grafana 배포

> 모델에 대한 구체적인 정보를 담은 대시보드를 더 만들어 보겠습니다!

1. `mlops` 환경에 Grafana 인스턴스를 배포할 수 있습니다. Jukebox의 엔드 투 엔드 여정을 지원하는 또 다른 도구입니다. 따라서 `mlops-gitops/toolings/`를 통해 설치해야 합니다.

    `toolings` 아래에 `grafana` 폴더를 생성하세요. 그리고 `grafana` 폴더 내에 `config.yaml` 파일을 생성합니다. 또는 아래 명령어를 실행하세요:

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/grafana
    touch /opt/app-root/src/mlops-gitops/toolings/grafana/config.yaml
    ```

2. `grafana/config.yaml` 파일을 열고 아래 내용을 붙여넣어 Argo CD에 배포할 차트를 알려줍니다.

    ```yaml
    chart_path: charts/grafana
    ```

3. 이전과 같이 변경 사항을 리포지토리에 커밋하세요.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "📈 Grafana added 📈"
    git push
    ```

4. 이 변경 사항이 동기화되면(Argo CD에서 확인 가능), [여기](https://jukebox-grafana-route-<USER_NAME>-toolings.<CLUSTER_DOMAIN>)를 클릭하여 Grafana에 로그인하고 Jukebox용 사전 정의된 대시보드를 확인하세요. 또는 `<USER_NAME>-mlops-toolings` 워크벤치(code-server) 터미널에서 아래 명령어를 실행할 수 있습니다:

    ```bash
    # get the route and open it in your browser
    echo https://$(oc get route jukebox-grafana-route --template='{{ .spec.host }}' -n <USER_NAME>-toolings)

    ```

    OpenShift 자격 증명을 사용하고 `Allow selected permissions`를 클릭하여 로그인하세요.

5. 대시보드를 보려면 `Dashboards` > `grafana <USER_NAME>-toolings Dashboards` > `OpenVINO Model Server - Model Metrics`로 이동하세요.

    _참고: 대시보드 구성이 동기화되는 데 시간이 걸릴 수 있습니다. 처음에 보이지 않으면 페이지를 강력 새로고침하세요._

    ![grafana-dashboard-1.png](./images/grafana-dashboard-1.png)
    
    다음과 같은 대시보드를 볼 수 있습니다:

    ![grafana-dashboard-2.png](./images/grafana-dashboard-2.png)

    대시보드가 코드로 정의되어 있으므로 여기서 하는 모든 변경 사항은 일시적이며 유지되지 않습니다—진정한 GitOps의 실천입니다!👻


이제 또 다른 흥미로운 도구인 TrustyAI 🔦🏡로 넘어가겠습니다.