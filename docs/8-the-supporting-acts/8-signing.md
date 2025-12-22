# 이미지 서명

> 클러스터에 배포하는 컨테이너 이미지가 변조되지 않았고 유효한 출처에서 왔음을 확인하는 것이 중요합니다. 이는 일반적으로 이미지를 빌드한 후 서명하고 배포 전에 서명을 검증하는 방식으로 이루어집니다. 이 실습에서는 컨테이너 이미지 서명을 생성, 저장 및 검증하기 위해 `cosign`을 사용합니다.

## 시작하기 전에, 키를 생성하세요

1. 이미지 서명에 사용할 키 쌍을 생성합니다. 개인 키의 비밀번호를 입력하라는 메시지가 나타납니다. 원하는 비밀번호를 자유롭게 선택하세요 :)

    ```bash
    cd /tmp
    cosign generate-key-pair k8s://<USER_NAME>-toolings/<USER_NAME>-cosign 
    ```

    다음과 같은 출력이 나와야 합니다:
    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    $ cosign generate-key-pair k8s://<USER_NAME>-toolings/<USER_NAME>-cosign
    Enter password for private key:
    Enter again:
    Successfully created secret cosign in namespace <USER_NAME>-toolings
    Public key written to cosign.pub
    </code></pre></div>

    이제 두 개의 키(개인 키 1개, 공개 키 1개)를 생성했습니다. 개인 키는 이미지를 서명하는 데 사용되며, 선택한 비밀번호와 함께 `toolings` 네임스페이스에 시크릿으로 자동 저장됩니다. 공개 키는 서명된 이미지를 검증하는 데 사용됩니다. 공개 키는 다른 사람이 이미지를 검증할 수 있도록 공유할 수 있지만, 개인 키는 공유해서는 안 되며 공개 저장 전에 반드시 봉인(Sealed)해야 합니다.

    <p class="tip">
    🐌 이것은 GitOps가 아닙니다 - 생성된 개인 키는 Kubernetes 시크릿으로 <USER_NAME>-toolings 프로젝트에 저장됩니다. 이를 Sealed Secret으로 추출하고 저장하는 작업은 독자에게 맡기겠습니다! 🐎
    </p>

    <p class="tip">
    😱 만약 <i>cosign</i> 명령어가 오류를 반환한다면, 클러스터에서 로그아웃된 상태일 수 있으니 아래 명령어를 실행한 후 다시 cosign 명령어를 실행하세요.
    </p>

    ```bash
    oc login --server=https://api.<TRIMMED_CLUSTER_DOMAIN>:6443 -u <USER_NAME> -p <PASSWORD>
    ```

이제 이미지 서명 단계를 포함하도록 파이프라인을 확장해 보겠습니다.

_이 단계는 외부 이미지 레지스트리를 사용하거나 클러스터 간 또는 공개적으로 이미지를 공유할 때 더욱 의미가 있습니다._

2. `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열고 `image_signing: true` 플래그를 추가하여 [해당 작업](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/tasks/image-signing.yaml)을 도입합니다.

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true
    linting: true 
    static_code_analysis: true
    static_code_analysis_secret: sonarqube-auth
    model_scanning: true
    image_scan: true
    image_signing: true # 👈 add this
    ```

5. 변경 사항을 저장소에 커밋합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🐦‍⬛ ADD - image signing step 🐦‍⬛"
    git push
    ```

6. OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 파이프라인 > 작업이 파이프라인에 포함되었는지 확인합니다.

    ![image-signing-pipeline.png](./images/image-signing-pipeline.png)

7. 빈 커밋으로 파이프라인을 실행하여 변경 사항을 확인합니다:

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🐒 trigger pipeline for image signing 🐒"
    git push
    ```

8. 작업이 성공적으로 완료되면 `Administrator` 뷰에서 `OpenShift UI` > `Builds` > `ImageStreams`로 이동하여 `jukebox`를 선택합니다. `.sig`로 끝나는 태그가 표시되며, 이는 이미지가 서명되었음을 나타냅니다.

    ![cosign-image-signing](images/cosign-image-signing.png)

9. 공개 키로 서명된 이미지를 검증해 보겠습니다. 이미지에 맞는 `VERSION`을 사용했는지 확인하세요. (이 경우 `c6575637d8`)

    ```bash
    export REGISTRY_AUTH_FILE=~/.docker/auth.json
    oc registry login
    cosign tree default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:c6575637d8 
    cosign verify --key k8s://<USER_NAME>-toolings/<USER_NAME>-cosign default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:c6575637d8 --allow-insecure-registry --insecure-ignore-tlog
    ```

    출력은 다음과 같아야 합니다:

    ```bash
    Verification for default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:c6575637d8 --
    The following checks were performed on each of these signatures:
      - The cosign claims were validated
      - The signatures were verified against the specified public key
      - Any certificates were verified against the Fulcio roots.
    {"critical":{"identity":{"docker-reference":"default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox"},"image":{"docker-manifest-digest":"sha256:1545e1d2cf0afe5df99fe5f1d39eef8429a2018c3734dd3bdfcac5a068189e39"},"type":"cosign container image signature"},"optional":null}
    ```