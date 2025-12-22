# 전처리 및 후처리

지금까지 우리는 `Jukebox`에 모델이 이해할 수 있는 형식으로 요청을 보내고, 노래가 유명할 수 있는 국가 목록을 받아왔습니다. 그러나 국가 목록은 사람이 읽기 쉬운 형태가 아니라, 기계가 처리하기 위한 정규화된 데이터입니다. 예를 들어, 시스템은 영국을 나타내기 위해 `12`를 출력할 수 있는데, 이는 모델에게는 명확하지만 우리에게는 그렇지 않습니다.

실제 애플리케이션에서는 모델과 상호작용하는 시스템이나 사용자가 모델이 요구하는 정확한 형식으로 데이터를 제공하지 않을 수 있습니다. 마찬가지로, 모델이 생성하는 원시 출력값을 이해하지 못할 수도 있습니다. 이때 전처리와 후처리가 필요합니다.

- 전처리는 원시 입력을 모델이 이해할 수 있는 형식으로 변환합니다.
- 후처리는 모델의 출력을 실행 가능한, 사람이 이해할 수 있는 인사이트로 변환합니다.

이 두 단계는 실제 데이터와 모델이 제공하는 가치 있는 인사이트 사이의 간극을 메워줍니다.

이번 실습에서는 KServe에서 `Transformers`를 사용하여 전처리 및 후처리를 구현하는 방법을 살펴보겠습니다. KServe의 Transformers는 동일한 배포 내에서 입력 요청의 전처리와 모델 출력의 후처리를 위한 Python 코드를 통합할 수 있게 하여, 서비스 과정을 향상시킵니다. 이 접근법은 복잡성을 줄이고 유지보수성을 개선하며, 배포 전반에 걸쳐 일관된 데이터 변환을 보장합니다.

### Transformers 활성화

1. 먼저 `<USER_NAME>-mlops-toolings` 작업 공간(code-server)으로 이동합니다.

2. Transformer는 모델 배포 내에서 사이드카 컨테이너로 작동합니다. 먼저 어떻게 생겼는지 살펴보겠습니다. 전처리 및 후처리를 담당하는 Python 코드는 `mlops-helmcharts/charts/model-deployment/music-transformer/music_transformer/music_transformer.py` 파일에서 확인할 수 있으며, Transformer 이미지를 빌드하는 [Containerfile](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/model-deployment/music-transformer/Containerfile)도 함께 있습니다.

3. 요청을 받을 때마다 이 Transformer를 사용하도록 모델 배포를 업데이트해야 합니다. 즉, `InferenceService`의 helm-chart 템플릿을 변경해야 합니다. 변경 사항은 [여기](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/model-deployment/music-transformer/templates/inferenceservice.yaml#L49-L63)에서 확인할 수 있습니다.

    실제로 업데이트를 진행하기 전에, 파이프라인에서 자동으로 수행된 모든 변경 사항을 가져오기 위해 GitOps 저장소에서 `git pull`을 실행합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git config pull.rebase false
    git pull
    ```

    이제 이 helm 차트를 사용하도록 GitOps 구성을 업데이트합니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)에서 `mlops-gitops/model-deployments/test/jukebox/config.yaml` 파일을 열고 `chart_path`를 다음과 같이 업데이트합니다:

    ```yaml
    ---
    chart_path: charts/model-deployment/music-transformer # 👈 update this
    name: jukebox
    version: 4562a17c17 # this value can be different for you
    image_repository: image-registry.openshift-image-registry.svc:5000
    image_namespace: <USER_NAME>-test
    ```

    **prod** 환경도 동일하게 진행합니다. `mlops-gitops/model-deployments/prod/jukebox/config.yaml` 파일을 열고 위와 같이 `chart_path`를 업데이트합니다.

4. Transformers는 요청 전송 방식과 모델 예측 처리 방식을 변경합니다. 따라서 프론트엔드 측에서도 변경이 필요하며, 우리에게 더 의미 있는 값을 전송하도록 해야 합니다. 프론트엔드 업데이트 역시 GitOps 구성을 변경하여 필요한 변경 사항이 반영된 새로운 버전의 `jukebox-ui` 이미지를 가리키도록 해야 합니다.

    `mlops-gitops/model-deployments/test/jukebox-ui/config.yaml` 파일을 열고 `image`를 다음과 같이 업데이트합니다:

    ```yaml
    ---
    repo_url: https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox-ui
    chart_path: chart
    model_endpoint: https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>
    model_name: jukebox
    image: quay.io/rhoai-mlops/jukebox-ui:transformer-1.6 # 👈 update this
    ```

    **prod** 환경도 동일하게 진행합니다. `mlops-gitops/model-deployments/prod/jukebox-ui/config.yaml` 파일을 열고 위와 같이 `image`를 업데이트합니다.

5. 변경 사항을 커밋하고 푸시합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🚝 UPDATE - helm chart with transformers 🚝"
    git push
    ```

6. 다음 명령어로 모델 배포와 프론트엔드의 새 버전이 롤아웃되고 있는지 확인할 수 있습니다:

    ```bash
    oc get po -n <USER_NAME>-test -w
    ```

    새 파드가 생성되고 이전 파드가 종료되는 것을 확인할 수 있습니다.

    _‘watch’ 모드를 종료하려면 Control+C를 누르세요._

7. [Jukebox UI](https://jukebox-ui-<USER_NAME>-test.<CLUSTER_DOMAIN>/)로 이동하여 변경 사항을 확인합니다.

    ```bash
    https://jukebox-ui-<USER_NAME>-test.<CLUSTER_DOMAIN>/
    ```

    이제 값과 국가명이 훨씬 더 이해하기 쉬워졌을 것입니다 (그렇길 바랍니다 🤭)

    ![jukebox-ui.png](./images/jukebox-ui.png)