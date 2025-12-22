# 고급 배포

머신러닝 모델은 독립적으로 작동하지 않으며, 종종 더 큰 시스템에 통합됩니다. 각 릴리스마다 기존 통합이 유지되고 시스템 성능이 유지되거나 향상되며 보안 및 컴플라이언스 요구사항이 충족되는지 확인하는 것이 중요합니다. 카나리 릴리스와 같은 고급 배포 관행은 회귀 발생 위험을 최소화하고 다운타임을 줄이며 새로운 버전의 자신 있는 롤아웃을 가능하게 합니다.

## 카나리 배포

카나리 배포 또는 A/B 배포는 일반적으로 테스트 또는 실험 목적으로 두 버전의 애플리케이션을 동시에 실행하는 것을 의미합니다. 처음에는 하나의 모델이 모든 요청을 처리합니다. 그런 다음 새 버전이 배포되고 일부 트래픽이 새 버전으로 라우팅됩니다. 새 버전이 예상대로 작동하면 점차 모든 트래픽을 새 버전으로 전환하고 원래 모델은 종료합니다. 그러나 새 모델에 문제가 발생하면 재배포나 추가 대기 없이 모든 트래픽을 원래 모델로 빠르게 되돌릴 수 있습니다. 본질적으로 이것은 <sup>[1](https://kserve.github.io/website/latest/modelserving/v1beta1/rollout/canary/)</sup>👇 

![canary-diagram.png](./images/canary-diagram.png)

KServe는 모델 엔드포인트로 들어오는 트래픽을 분배할 수 있습니다. 그렇다면 어떻게 할까요? KServe는 우리가 배포하는 각 모델 버전의 정보를 보유합니다. 기본적으로 최신 배포 버전으로 100% 트래픽을 보냅니다. `<USER_NAME>-mlops-toolings` 워크벤치(code-server) 터미널에서 다음 명령어를 실행하여 확인하세요.

  ```bash
  oc get isvc jukebox -n <USER_NAME>-test
  ```

  다음과 유사한 출력이 나타납니다:

  <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
  <pre><code class="language-bash">
    NAME      URL                                                                          READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION       AGE
    jukebox   https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>   True           100                              jukebox-predictor-00003   2d2h
    </code></pre>
    </div>

  이 출력은 트래픽의 100%가 최신 버전으로 향하고 있음을 알려줍니다.


1. `<USER_NAME>-mlops-toolings` 워크벤치(code-server)에서 `mlops-gitops/model-deployments/test/jukebox/config.yaml` 파일을 업데이트하여 테스트 환경에서 `InferenceService`에 카나리 배포를 활성화합시다.  
⚠️autoscaling 라인을 제거하는 점에 유의하세요. 이는 메트릭 해석을 좀 더 쉽게 하기 위함입니다⚠️

    ```bash
    ---
    chart_path: charts/model-deployment/music-transformer
    name: jukebox
    version: 4562a17c17
    image_repository: image-registry.openshift-image-registry.svc:5000
    image_namespace: <USER_NAME>-test
    canary:  # 👈 add this
      trafficPercent: 20 # 👈 add this
    ```

    이 작업은 아래 설정을 추가하여 `InferenceService`를 업데이트하고, 이전 버전 모델도 함께 실행하여 트래픽을 80%와 20%로 분할합니다.

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
      ---
      apiVersion: serving.kserve.io/v1beta1
      kind: InferenceService
      metadata:
        name: jukebox
        annotations:
          openshift.io/display-name: jukebox
          serving.knative.openshift.io/enablePassthrough: 'true'
          sidecar.istio.io/inject: 'true'
          sidecar.istio.io/rewriteAppHTTPProbers: 'true'
          autoscaling.knative.dev/target: "1"
        finalizers:
          - inferenceservice.finalizers
        labels:
          opendatahub.io/dashboard: 'true'
      spec:
        predictor:
          canaryTrafficPercent: 20 ### 👈 이 설정이 핵심입니다 🔮
       ...
    </code></pre></div>


2. 변경사항을 푸시합시다. 그러면 이전 버전이 자동으로 재배포되어 위에서 지정한 비율에 따라 트래픽을 분할합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🦜 UPDATE - canary deployment enabled 🦜"
    git push
    ```

3. Argo CD가 변경사항을 동기화한 후, 이전 명령어의 출력을 다시 확인하세요.
   
    ```bash
    oc get isvc -n <USER_NAME>-test
    ```

    다음과 같은 출력이 나타납니다:

    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-bash">
        NAME      URL                                                                           READY   PREV   LATEST   PREVROLLEDOUTREVISION     LATESTREADYREVISION       AGE
        jukebox   https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>   True    80     20       jukebox-predictor-00002   jukebox-predictor-00003   25h
   </code></pre></div>


4. 실제로 최신 버전으로 20% 트래픽이 전달되고 나머지 80%는 이전 버전이 처리하는지 확인합시다. 다시 `locust`를 사용해 트래픽을 생성할 수 있습니다. Jupyter Notebook `<USER_NAME>-hitmusic-wb` 워크벤치(Standard Data Science)로 돌아가 `jukebox/6-advanced_deployments/1-test_autoscale.ipynb`를 다시 실행하세요.

5. 요청이 두 모델 모두에 도달하며 80% 대 20% 비율로 분할되는지 확인하려면 메트릭을 확인할 수 있습니다. `OpenShift Dashboard`를 열고 `Developer` 뷰로 전환하세요. `<USER_NAME>-test` 네임스페이스에서 `Observe` > `Metrics`로 이동합니다. 아래 쿼리를 사용해 파드별 요청 수를 필터링하고 그룹화하세요. 결과 값은 약 80%-20% 분할을 나타내야 합니다.

  다음 쿼리를 실행하고 출력을 검토하세요:

  ```bash
  sum(rate(ovms_requests_success[5m])) by (pod) 
  ```

  ![canary-metrics.png](./images/canary-metrics.png)