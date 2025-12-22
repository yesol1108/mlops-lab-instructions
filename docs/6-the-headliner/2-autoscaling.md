# 자동 확장

자동 확장은 수요가 많은 시기에 추가 리소스를 프로비저닝하여 최적의 성능을 보장하고, 활동이 적을 때는 축소하여 비용을 최소화합니다. 이는 모델 학습 및 추론과 같은 머신러닝 작업이 자원 요구량이 변동하는 경우가 많기 때문에 중요합니다.

다행히도, KServe와 OpenShift를 사용하면 들어오는 요청 부하에 기반한 자동 확장이 매우 쉽습니다.

1. `<USER_NAME>-mlops-toolings` 워크벤치(code-server)에서 `mlops-gitops/model-deployments/test/jukebox/config.yaml` 파일을 업데이트하여 테스트 환경의 `InferenceService`에 자동 확장을 활성화합시다. 설정 파일에 `autoscaling: true`를 추가하세요.

    ```bash
    ---
    chart_path: charts/model-deployment/music-transformer
    name: jukebox
    version: 4562a17c17 # this value can be different for you
    image_repository: image-registry.openshift-image-registry.svc:5000
    image_namespace: <USER_NAME>-test
    autoscaling: true # 👈 add this
    ```

    이렇게 하면 아래 주석이 추가되어 `InferenceService`가 업데이트되고 자동 확장 기능을 갖춘 새로운 모델 배포가 트리거됩니다!

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
    ---
    apiVersion: serving.kserve.io/v1beta1
    kind: InferenceService
    metadata:
    annotations:
      openshift.io/display-name: jukebox
      serving.knative.openshift.io/enablePassthrough: 'true'
      sidecar.istio.io/inject: 'true'
      sidecar.istio.io/rewriteAppHTTPProbers: 'true'
      autoscaling.knative.dev/target: "1" ### 👈 이 부분이 핵심입니다 🔮
    </code></pre></div>

2. 변경 사항을 푸시하세요:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "💸 UPDATE - autoscaling enabled 💸"
    git push
    ```
    
3. 모델이 재배포될 때까지 기다리세요. 이전과 같이 `oc get po -n <USER_NAME>-test -w` 명령어로 파드를 모니터링할 수 있습니다. (취소하려면 Ctrl+C)

4. 부하를 생성하여 자동 확장을 테스트해봅시다. `<USER_NAME>-hitmusic-wb` Jupyter Notebook 워크벤치(Standard Data Science)로 이동하여 `jukebox/6-advanced_deployment/1-test_autoscale.ipynb` 노트북을 실행하세요.

> ⚠️ 추론 엔드포인트를 입력할 때는 `<USER_NAME>-test`에서 서비스되는 모델의 엔드포인트를 반드시 입력하세요.

5. OpenShift 대시보드에서 관리자 뷰 > `<USER_NAME>-test` 프로젝트 > `Workloads` > `Pods`로 이동하여 새로운 파드가 생성되는 것을 확인하세요.

    ![autoscaling-1.png](./images/autoscaling-1.png)

6. 자동 확장이므로, 추가된 리소스는 더 이상 필요하지 않을 때 자동으로 제거됩니다. 즉, 동시 요청이 없으면 잠시 후 동일한 파드가 종료되는 것을 확인할 수 있습니다.

    ![autoscaling-2.png](./images/autoscaling-2.png)