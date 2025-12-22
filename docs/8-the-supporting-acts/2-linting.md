## 린터(Linters)

> 린팅은 오류를 줄이고 코드의 전반적인 품질을 향상시키는 데 중요합니다. 린트 도구를 사용하면 개발 속도를 높이고 오류를 조기에 발견하여 비용을 절감할 수 있습니다.

### 코드 린팅

코드 린팅을 위해 여러 Python 라이브러리가 제공되며, 코드 품질을 보장할 수 있습니다: 포맷팅을 위한 `black`, import 정렬을 위한 `isort`, 구문 오류나 정의되지 않은 이름을 감지하는 `flake8` 등이 있습니다. 이러한 도구들은 Continuous Training 파이프라인에 통합하여 코드 품질 검사를 자동으로 시행하고 일관성을 유지할 수 있습니다. 좋은 코딩 관행을 무시하거나 깨끗한 코드를 작성하지 않으면 시간이 지남에 따라 유지보수에 큰 어려움이 발생할 수 있으므로 처음부터 품질을 우선시하는 것이 중요합니다.

파이프라인에 추가하기 전에 수동으로 하나를 사용해 봅시다:

1. 코드 서버 워크벤치에서 터미널을 열고 아래 코드를 실행하세요.

    ```bash
    cd /opt/app-root/src/jukebox/3-prod_datascience
    pip install black
    black . --check --diff
    ```

2. 다음과 유사한 결과를 얻을 수 있습니다:
   
   ![black-output.png](./images/black-output.png)

3. `flake8` 출력도 확인해 봅시다:

    ```bash
    cd /opt/app-root/src/jukebox/3-prod_datascience
    pip install flake8
    flake8 . --show-source
    ```

4. 다음과 같은 출력이 나올 수 있습니다:

    ![flake8.png](./images/flake8.png)

개선할 부분이 있음을 알 수 있습니다 :) 이러한 검사를 파이프라인에 추가하기만 해도 높은 코드 품질 기준을 유지할 수 있습니다.

린터를 추가하기 전에 다른 도구를 소개하겠습니다.

### 쿠브 린팅(Kube Linting)

KubeLinter는 Kubernetes YAML 파일과 Helm 차트를 분석하여 다양한 모범 사례에 따라 검사하는 오픈 소스 도구로, 특히 프로덕션 준비 상태와 보안에 중점을 둡니다. 우리는 `kubelinter`를 사용하여 모델 배포 Helm 차트를 검사할 수 있습니다.

1. KubeLinter는 린트를 수행할 때 검사할 수 있는 많은 내장 모범 사례를 가지고 있습니다. 이를 나열할 수 있습니다.

    ```bash
    kube-linter checks list | grep Name:
    ```

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
    Name: cluster-admin-role-binding
    Name: dangling-service
    Name: default-service-account
    Name: deprecated-service-account-field
    Name: docker-sock
    Name: drop-net-raw-capability
    Name: env-var-secret
    Name: exposed-services
    Name: host-ipc
    Name: host-network
    Name: host-pid
    Name: mismatching-selector
    Name: no-anti-affinity
    Name: no-extensions-v1beta
    Name: no-liveness-probe
    Name: no-read-only-root-fs
    Name: no-readiness-probe
    Name: non-existent-service-account
    Name: privilege-escalation-container
    Name: privileged-container
    Name: privileged-ports
    Name: required-annotation-email
    Name: required-label-owner
    Name: run-as-non-root
    Name: sensitive-host-mounts
    Name: ssh-port
    Name: unsafe-proc-mount
    Name: unsafe-sysctls
    Name: unset-cpu-requirements
    Name: unset-memory-requirements
    Name: writable-host-mount
    </code></pre></div>

    그러나 이들은 기본적으로 `Deployment`나 `Service` 같은 일반 Kubernetes 리소스에만 유효합니다. 우리의 모델 배포는 `InferenceService`와 같은 커스텀 리소스를 사용합니다. 따라서 kube-linter는 기본적으로 이를 검사할 수 없지만, Jukebox UI Helm 차트에서 `kube-linter`를 실행하여 내장 기능을 확인할 수 있습니다.

2. Jukebox UI 차트 폴더에서 **kube-linter**를 실행해 봅시다.
   
    ```bash
    cd /opt/app-root/src/
    git clone https://<USER_NAME>:<PASSWORD>@gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox-ui.git
    cd /opt/app-root/src/jukebox-ui
    kube-linter lint chart
    ```

    다음과 같은 보고서를 출력합니다:
    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-bash">
    /opt/app-root/src/jukebox-ui/chart/templates/deployment.yaml: (object: <no namespace>/jukebox-ui apps/v1, Kind=Deployment) The container "jukebox-ui" is using an invalid container image, "quay.io/rhoai-mlops/jukebox-ui:latest". Please use images that are not blocked by the `BlockList` criteria : [".*:(latest)$" "^[^:]*$" "(.*/[^:]+)$"] (check: latest-tag, remediation: Use a container image with a specific tag other than latest.)
    /opt/app-root/src/jukebox-ui/chart/templates/deployment.yaml: (object: <no namespace>/jukebox-ui apps/v1, Kind=Deployment) container "jukebox-ui" does not have a read-only root file system (check: no-read-only-root-fs, remediation: Set readOnlyRootFilesystem to true in the container securityContext.)
    Error: found 2 lint errors
    </code></pre>
    </div>

    _첫 번째 오류는 GitOps 저장소에서 `latest`가 아닌 태그를 지정하여 해결할 수 있습니다. 두 번째 오류는 다행히 OpenShift 사용자의 경우 기본적으로 컨테이너의 루트 파일 시스템을 읽기 전용으로 마운트하므로 배포 시 별도로 지정할 필요가 없어 문제가 되지 않습니다 🎉_

    공통 Kubernetes 객체에 대한 kube-linter 검사를 확장하거나 `InferenceService` 같은 커스텀 리소스에 대한 검사를 추가하고 싶다면 [제품 문서](https://docs.kubelinter.io/#/configuring-kubelinter?id=run-custom-checks)를 참고하세요.

### 헬름 린팅(Helm Linting)

`helm lint`를 사용하여 차트의 잠재적 문제를 검사할 수도 있습니다. 헬름 린터가 설치 실패를 유발할 문제를 발견하면 오류 메시지를 출력하고, 관례나 권장사항에 어긋나는 문제는 경고로 알려줍니다.

1. 파이프라인에 추가하기 전에 모델 배포 차트 폴더에서 **helm linter**를 실행해 봅시다.

    ```bash
    cd /opt/app-root/src/
    cd /opt/app-root/src/mlops-helmcharts/charts/model-deployment/
    helm lint music-transformer-with-feast
    ```

    훌륭합니다! 실패한 차트가 없습니다 👏

    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-bash">
    $ helm lint music-transformer-with-feast 
    ==> Linting music-transformer-with-feast
    [INFO] Chart.yaml: icon is recommended

    1 chart(s) linted, 0 chart(s) failed
    </code></pre>
    </div>

## 파이프라인 확장

데이터 사이언스 파이프라인을 트리거하기 전에 린팅을 수행하는 새로운 Task를 Tekton 파이프라인에 추가합니다. 이를 통해 코드나 배포 파일의 문제를 조기에 발견하여, 코드가 요구되는 기준을 충족하지 못할 때 불필요한 학습 실행을 방지함으로써 시간과 자원을 절약할 수 있습니다.

1. `mlops-gitops/toolings/ct-pipeline/config.yaml`을 열고 `linting: true` 플래그를 추가하여 [linting task](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/tasks/linting.yaml)를 도입하세요.

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true
    linting: true # 👈 add this
    ```

2. 변경 사항을 저장소에 커밋합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "☀️ linting task is added ☀️"
    git push
    ```

    OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 Pipelines에서 `linting` 작업이 파이프라인에 포함되었는지 확인하세요:

    ![linting-task.png](./images/linting-task.png)

3. 빈 커밋으로 파이프라인을 실행하여 변경 사항을 확인하세요:

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🩴 trigger pipeline for linting 🩴"
    git push
    ```

4. 이전에 보았던 동일한 오류로 `Pipeline Run`이 실패하는 것을 확인할 수 있습니다.

    ![linting-fail.png](./images/linting-fail.png)

5. 오류를 수정하고 린팅 단계를 통과하도록 파이프라인을 통과시켜 봅시다. `<USER_NAME>-mlops-toolings` 워크벤치(코드 서버) 터미널에서 다음 명령어를 실행하세요:
   
    ```bash
    cd /opt/app-root/src/jukebox/3-prod_datascience
    black .
    ```

6. 원한다면 `black`이 포맷팅을 수정한 후 변경 사항을 푸시하고 새 파이프라인을 실행하세요:

    ```bash
    cd /opt/app-root/src/jukebox
    git add .
    git commit -m "🪄 black format fixes 🪄"
    git push
    ```

7. 이번에는 파이프라인이 `linting` 단계를 통과하는 것을 확인하세요 🥳

    ![linting-success.png](./images/linting-success.png)