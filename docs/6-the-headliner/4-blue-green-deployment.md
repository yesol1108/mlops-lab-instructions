# 고급 배포

## 블루/그린 배포

블루/그린 배포는 간단합니다: 모든 트래픽을 한 환경에서 다른 환경으로 한 번에 전환합니다. 이 접근법은 단순성을 우선시하고 Canary 또는 A/B 배포에서 요구되는 트래픽 분할 관리를 피하고자 할 때 이상적입니다. 장기간 모니터링 없이 깔끔한 전환이 필요한 배포에 더 적합합니다.

Canary 또는 A/B 배포는 일반적으로 사용자 상호작용을 기반으로 다양한 모델의 효과를 측정하는 실험에 사용됩니다. 실험이 목적이 아니라 기존 모델을 통제된 방식으로 새 모델로 교체하는 것이 목표라면 블루-그린 배포가 더 적합합니다.

그러나 구현 관점에서 보면, KServe에서는 Canary 배포와 매우 유사합니다. 단지 트래픽의 100%를 새 모델 리비전으로 전환하는 것입니다. KServe는 각 리비전 정의를 유지하여 쉽게 롤백할 수 있는 옵션을 제공합니다.

1. `trafficPercent` 값을 `100`으로 업데이트하면 모든 트래픽이 최신 버전으로 이동합니다. `<USER_NAME>-mlops-toolings` 워크벤치(code-server)에서 `mlops-gitops/model-deployments/test/jukebox/config.yaml`을 업데이트하세요.

    ```bash
    ---
    chart_path: charts/model-deployment/music-transformer
    name: jukebox
    version: 4562a17c17
    image_repository: image-registry.openshift-image-registry.svc:5000
    image_namespace: <USER_NAME>-test
    canary:
      trafficPercent: 100 # 👈 update this
    ```

2. 변경 사항을 푸시합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🐳 UPDATE - blue green deployment 🍏"
    git push
    ```

3. 현재 하나의 버전만 실행 중인지 확인합니다:

    ```bash
    oc get isvc jukebox -n <USER_NAME>-test
    ```

    ```bash                                                                                               
    NAME      URL                                                                          READY   PREV   LATEST   PREVROLLEDOUTREVISION     LATESTREADYREVISION       AGE
    jukebox   https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>   True    0     100       jukebox-predictor-00023   jukebox-predictor-00024   38h
    ```

4. 동일한 방법으로 최신(그린) 버전으로만 트래픽이 전송되는지 확인해 봅시다. 다시 Jupyter Notebook으로 돌아가 `jukebox/6-advanced_deployments/1-test_autoscale.ipynb`를 실행하세요. 그리고 `OpenShift Dashboard`에서 `<USER_NAME>-test` 네임스페이스의 `Observe` > `Metrics`로 이동합니다. 아래 쿼리를 사용하세요.

  최신 리비전만 트래픽을 받고 있는 것을 확인할 수 있습니다.

  ```bash
  sum(rate(ovms_requests_success[5m])) by (pod) 
  ```

  ![bluegreen-metrics.png](./images/bluegreen-metrics.png)

5. 이전 버전으로 롤백하려면 `trafficPercent` 값을 `0`으로 업데이트하세요.

    ```bash
    ---
    chart_path: charts/model-deployment/music-transformer
    name: jukebox
    version: 4562a17c17
    image_repository: image-registry.openshift-image-registry.svc:5000
    image_namespace: <USER_NAME>-test
    canary:
      trafficPercent: 0 # 👈 update this
    ```

6. 그리고 변경 사항을 푸시합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🍏 UPDATE - blue green deployment 🐳"
    git push
    ```

7. `locust` 명령어를 실행하여 현재 트래픽이 이전 버전만 받고 있는지 관찰하세요:

  ```bash
  oc get isvc jukebox -n <USER_NAME>-test
  ```

    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-bash">                                                                                                  
    NAME      URL                                                                          READY   PREV   LATEST   PREVROLLEDOUTREVISION     LATESTREADYREVISION       AGE
    jukebox   https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>   True    100     0       jukebox-predictor-00023   jukebox-predictor-00024   38h
    </code></pre>
    </div>

8. 다시 `OpenShift Dashboard`에서 `<USER_NAME>-test` 네임스페이스의 `Observe` > `Metrics`로 이동하여 아래 쿼리를 사용해 메트릭을 확인하세요.

  ```bash
  sum(rate(ovms_requests_success[5m])) by (pod) 
  ```

  ![greenblue-metrics.png](./images/greenblue-metrics.png)

9. 블루-그린 배포에서는 두 개의 모델 복제본이 실행 중입니다. 여기서의 트레이드오프는 블루-그린이 중복 환경을 유지해야 하므로 리소스 소모가 클 수 있다는 점입니다. `<USER_NAME>-mlops-toolings` 워크벤치(code-server) 터미널에서 아래 명령어를 실행하여 확인할 수 있습니다.

    ```bash
    oc get po -l component=predictor -n <USER_NAME>-test
    ```
    
    ```bash
    NAME                                                  READY   STATUS    RESTARTS        AGE
    jukebox-predictor-00023-deployment-7469ddd454-jjsww   6/6     Running   1 (8m46s ago)   8m52s
    jukebox-predictor-00024-deployment-7f8f5fbdff-tp8vh   6/6     Running   1 (25m ago)     25m
    ```