# GitOps를 통한 모델 배포

우리는 `jukebox` 모델을 실험 환경에 수동으로 배포했지만, 상위 환경에서는 정의를 Git에 저장하고 Argo CD를 통해 모델을 배포하여 GitOps가 제공하는 모든 이점을 누려야 합니다.

## 🚗 ModelCars와 GitOps 방식의 배포

학습 파이프라인에서 새로운 모델 버전이 생성되면, 코드 변경이나 모니터링 알림에 의해 트리거되든 간에, 더 나은 성능을 보인다면 자동으로 배포되도록 하는 것이 목표입니다(걱정 마세요, 최고의 모델만 배포되도록 할 테니 조금만 기다려 주세요 😉). 자동화와 함께 배포 과정에서 추적 가능성, 불변성, 재현성을 우선시합니다.

이 요구사항을 충족하기 위해, 우리는 KServe의 **ModelCars** 기능을 사용합니다. ModelCars는 모델을 Open Container Initiative(OCI) 이미지로 컨테이너화하여 모델 배포를 단순화합니다. 본질적으로 ModelCars는 모델 서버를 위한 모델 저장소 역할을 합니다. ModelCars를 사용하면 각 모델 버전이 컨테이너 이미지로 패키징되어 버전 관리가 가능하며, 다양한 모델 버전을 추적하고 관리할 수 있습니다.

파이프라인이 Git 푸시 이벤트를 감지하면, ModelCar에 Git 커밋 ID를 태그로 붙입니다. 이 ID는 모델을 소스 코드 변경 사항과 연결하여 모델이 왜, 어떻게 만들어졌는지에 대한 맥락을 제공합니다.

모델 아티팩트와 버전 정보가 생성되면, 파이프라인은 업데이트를 `mlops-gitops/model-deployment/` 저장소에 푸시합니다. Argo CD는 변경 사항을 감지하고 OpenShift의 `InferenceService` 구성을 업데이트합니다. 이는 새로운 모델 버전의 자동 롤아웃을 트리거하여 프로덕션에 원활하게 통합됩니다.

## Jukebox 배포하기

1. 툴링과 마찬가지로, 모델 배포를 위한 `ApplicationSet` 정의를 생성해야 합니다. `test` 환경과 `prod` 환경 각각에 대해 두 개의 별도 `ApplicationSet` 정의가 있습니다. 간편한 적용을 위해 동일한 저장소에 보관하지만, 실제 환경에서는 `prod` 정의를 보호된 `main` 브랜치에서 Pull Request를 통해서만 변경하는 별도의 저장소에 두는 것이 좋습니다. `ApplicationSet` 정의를 분리해 두면 나중에 `prod` 정의를 다른 곳으로 옮기기 쉽습니다 :)

    이전과 같이 `CLUSTER_DOMAIN`과 `USER_NAME` 정의로 `ApplicationSet` 정의를 업데이트합시다. `mlops-gitops/appset-test.yaml`과 `mlops-gitops/appset-prod.yaml` 파일을 열어 값을 교체하세요. 게으른 분들을 위해 명령어도 준비했습니다:

  ```bash
    sed -i -e 's/CLUSTER_DOMAIN/<CLUSTER_DOMAIN>/g' /opt/app-root/src/mlops-gitops/appset-test.yaml
    sed -i -e 's/USER_NAME/<USER_NAME>/g' /opt/app-root/src/mlops-gitops/appset-test.yaml
    sed -i -e 's/CLUSTER_DOMAIN/<CLUSTER_DOMAIN>/g' /opt/app-root/src/mlops-gitops/appset-prod.yaml
    sed -i -e 's/USER_NAME/<USER_NAME>/g' /opt/app-root/src/mlops-gitops/appset-prod.yaml
  ```

2. `jukebox` 모델 정의를 추가합시다. `test`와 `prod` 두 환경이 있으므로 두 파일을 업데이트해야 합니다. `model-deployments/test/jukebox/config.yaml`과 `model-deployments/prod/jukebox/config.yaml` 파일을 다음과 같이 업데이트하세요.

    이는 헬름 차트 저장소에서 모델 배포 헬름 차트를 가져와 이미지 버전 등의 추가 구성을 적용합니다.

    ```yaml
    chart_path: charts/model-deployment/simple
    name: jukebox
    version: latest
    image_repository: quay.io
    image_namespace: rhoai-mlops
    ```

3. 이제 배포를 진행합시다 — Git에 있어야 진짜입니다!

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git add .
    git commit -m  "🐰 ADD - ApplicationSets and jukebox to deploy 🐰"
    git push 
    ```

4. Git에 `jukebox` 값이 저장되었으니, 이제 Argo CD가 이 환경들의 변경 사항을 감지하도록 알려야 합니다. 이를 위해 간단히 ApplicationSets를 생성하면 됩니다:

    ```bash
    oc apply -f /opt/app-root/src/mlops-gitops/appset-test.yaml -n <USER_NAME>-toolings
    oc apply -f /opt/app-root/src/mlops-gitops/appset-prod.yaml -n <USER_NAME>-toolings
    ```

5. Argo CD에서 `test`와 `prod` 각각의 Jukebox 애플리케이션 두 개가 배포된 것을 확인할 수 있습니다.

    ![argocd-jukebox-deployed](./images/argocd-jukebox-deployed.png)

6. OpenShift AI 대시보드에서 `Models` 아래 `Model Deployments`를 선택하고, 두 프로젝트 중 하나를 선택하면 배포된 모델을 확인할 수 있습니다. 내부 루프에서 했던 것과 동일합니다.

    ![rhoai-deployed-models](./images/rhoai-deployed-models.png)

## Jukebox UI 사용하기

Argo CD에서 Jukebox 모델용 UI 애플리케이션도 `test`와 `prod`에 생성된 것을 볼 수 있습니다. 하지만 내부를 보면 비어 있습니다.

이제 UI를 배포해 봅시다! 📺

1. Jukebox 모델과 마찬가지로, 현재 비어 있는 `model-deployments/test/jukebox-ui/config.yaml`과 `model-deployments/prod/jukebox-ui/config.yaml`을 다음과 같이 업데이트합니다:

    TEST:

    ```yaml
    repo_url: https://<GIT_SERVER>/<USER_NAME>/jukebox-ui
    chart_path: chart
    model_endpoint: https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>
    model_name: jukebox
    image: quay.io/rhoai-mlops/jukebox-ui:simple-1.2
    ```

    PROD:

    ```yaml
    repo_url: https://<GIT_SERVER>/<USER_NAME>/jukebox-ui
    chart_path: chart
    model_endpoint: https://jukebox-<USER_NAME>-prod.<CLUSTER_DOMAIN>
    model_name: jukebox
    image: quay.io/rhoai-mlops/jukebox-ui:simple-1.2
    ```

2. Git에 푸시하면 Argo CD에서 업데이트되는 것을 볼 수 있습니다 🧙‍♂️

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git add .
    git commit -m  "📺 UPDATE - jukebox-ui configs for deployment 📺"
    git push 
    ```

3. Jukebox UI가 정상적으로 실행 중인지 확인하세요:

    ```bash
    oc get po -l app.kubernetes.io/name=jukebox-ui -n <USER_NAME>-test
    ```

4. 새로 배포된 UI를 보려면 다음 경로로 접속하세요:

    ```bash
    https://jukebox-ui-<USER_NAME>-test.<CLUSTER_DOMAIN>
    ```

5. 슬라이더를 조작하며 새로운 위치를 예측해 보세요! 🗺️ 또한 모든 멋진 사람들이 하는 것처럼 다크 모드로 전환할 수도 있습니다! 😎

![jukebox-ui](./images/jukebox-ui.png)

6. 🤓 개발자용 정보 ⌗ API 요청과 응답을 UI에서 모델로 주고받는 내용을 정말 궁금하다면, 개발자 도구에서 확인할 수 있습니다. 크롬에서 개발자 도구(CMD + OPTION + i on 🍎)를 열고, 세 점 메뉴 > 추가 도구 > 개발자 도구를 선택하세요. 네트워크 탭을 클릭하고 녹화 버튼이 켜져 있는지 확인합니다. UI에서 `predict` 버튼을 다시 누르면 네트워크 탭에 HTTP 요청 정보가 표시되어 탐색할 수 있습니다.

![dev-tools](./images/dev-tools.png)