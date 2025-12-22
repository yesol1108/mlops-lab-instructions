## 집계 로깅

> OpenShift의 내장 로깅은 LokiStack을 사용하는 오퍼레이터로 배포됩니다. 기본적으로 시스템 출력으로 로깅하는 모든 컨테이너의 출력을 수집합니다. 즉, 애플리케이션에서 별도의 로깅 구성이 필요하지 않습니다. 로그는 각 노드에서 실행되는 수집기를 통해 수집된 후 LokiStack으로 전달되어 JSON 형식의 시계열로 인덱싱됩니다. OpenShift에는 내장된 시각화 UI가 있지만, 외부 Grafana를 사용할 수도 있습니다.

1. OpenShift의 마법 같은 기능은 서비스 전반에 걸쳐 로그를 수집하는 훌륭한 방법을 제공합니다. STDOUT 또는 STDERR로 출력되는 모든 내용이 수집되어 LokiStack에 추가됩니다. 이를 통해 로그 인덱싱과 쿼리가 매우 쉬워집니다. 이제 OpenShift 로그 UI를 살펴보겠습니다.

    ![openshift-logging](./images/openshift-logging.png)

2. 정보를 필터링해 보겠습니다. 쿼리 바에 다음을 추가하여 test 네임스페이스에서 실행 중인 jukebox 앱의 로그만 찾습니다. `Show Query`를 클릭하고 아래 내용을 붙여넣은 후 `Run Query`를 클릭하세요.

    ```bash
    { log_type="application", kubernetes_pod_name=~"jukebox-ui.*", kubernetes_namespace_name="<USER_NAME>-test" }
    ```

    ![openshift-logging-2.png](./images/openshift-logging-2.png)

3. 컨테이너 로그는 일시적(ephemeral)입니다. 즉, 컨테이너가 중지되면 로그가 사라지므로 별도로 집계 및 저장하지 않으면 로그를 잃게 됩니다. 메시지를 생성하고 UI에서 쿼리해 보겠습니다. rsh를 통해 UI 파드에 접속하여 로그를 생성합니다.

    ```bash
    oc project <USER_NAME>-test
    oc rsh `oc get po -l app.kubernetes.io/name=jukebox-ui -o name -n <USER_NAME>-test`
    ```

    그런 다음 방금 원격으로 접속한 컨테이너 내부에서 로그에 임의의 메시지를 추가합니다:

    ```bash
    echo "🦄🦄🦄🦄" >> /tmp/custom.log
    tail -f /tmp/custom.log > /proc/1/fd/1 &
    echo "🦄🦄🦄🦄" >> /tmp/custom.log
    echo "🦄🦄🦄🦄" >> /tmp/custom.log
    echo "🦄🦄🦄🦄" >> /tmp/custom.log
    echo "🦄🦄🦄🦄" >> /tmp/custom.log
    exit
    ```

4. OpenShift 콘솔 > 관찰 > 로그로 돌아갑니다. 다음 쿼리를 사용해 이 메시지들을 필터링하고 찾을 수 있습니다:

    ```bash
    { log_type="application", kubernetes_pod_name=~"jukebox-ui.*", kubernetes_namespace_name="<USER_NAME>-test" } |= `🦄🦄🦄🦄` | json
    ```

    ![openshift-logging-3.png](./images/openshift-logging-3.png)