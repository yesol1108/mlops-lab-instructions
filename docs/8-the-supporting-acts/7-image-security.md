# 이미지 보안 (StackRox)

> 우리는 모델카 🚗를 빌드하기 위해 일부 준비된 컨테이너 이미지를 사용합니다. 이들을 베이스로 사용하고 그 위에 모델 아티팩트를 추가합니다. 공개 레지스트리 이미지의 오류 및 취약점, 또는 오래된 패키지와 라이브러리로부터 컨테이너를 보호해야 합니다. 파이프라인의 이미지 보안 단계는 이미지를 프로덕션으로 이동하기 전에 이를 발견하는 데 도움을 줍니다.


## StackRox 접근 설정

StackRox (Advanced Cluster Security, 또는 ACS)는 클러스터 레벨에 배포되어 여러 클러스터를 모니터링할 수 있습니다. 이 환경에서는 ACS / StackRox 오퍼레이터가 이미 배포 및 구성되어 있습니다.

1. ACS WebUI에 접속합니다:

    ```bash
    https://central-rhacs-operator.<CLUSTER_DOMAIN>
    ```

    자격 증명을 사용하여 로그인합니다.
    ![acs-login.png](./images/acs-login.png)
    ![acs-dashboard.png](images/acs-dashboard.png)

2. 설치 구성의 일부로 API 토큰이 생성되었습니다. 다음 명령어로 토큰을 가져올 수 있습니다:

    토큰을 환경 변수로 내보내기:

    ```bash
    export ROX_API_TOKEN=$(oc -n <USER_NAME>-toolings get secret rox-auth-ml500 -o go-template='{{index .data "password" | base64decode}}')
    ```
    _클러스터에서 로그아웃한 경우, 아래 명령어로 다시 로그인하세요._

    ```bash
    oc login --server=https://api.<TRIMMED_CLUSTER_DOMAIN>:6443 -u <USER_NAME> -p <PASSWORD>
    ```

    StackRox 엔드포인트 내보내기:

    ```bash
    export ROX_ENDPOINT=central-rhacs-operator.<CLUSTER_DOMAIN>
    ```

3. **roxctl**을 실행하여 토큰을 검증합니다.

    ```bash
    roxctl central whoami --insecure-skip-tls-verify -e $ROX_ENDPOINT:443
    ```

    다음과 같은 출력이 나와야 합니다:

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    UserID:
        auth-token:40ads4db7-e7ad-49b2-aa24-5e11afrwe7372
    User name:
        anonymous bearer token "ml500" with roles
    Roles:
        - Admin
    Access:
        rw Access
        rw Administration
        rw Alert
        rw CVE
        rw Cluster
        rw Compliance
        rw Deployment
        rw DeploymentExtension
        rw Detection
        rw Image
        rw Integration
        rw K8sRole
        rw K8sRoleBinding
        rw K8sSubject
        rw Namespace
        rw NetworkGraph
        rw NetworkPolicy
        rw Node
        rw Secret
        rw ServiceAccount
        rw VulnerabilityManagementApprovals
        rw VulnerabilityManagementRequests
        rw WatchedImage
        rw WorkflowAdministration
        </code></pre></div>

4. 이 API 토큰은 파이프라인에서 사용됩니다. 이를 위해 Sealed Secret 정의를 생성합시다.

    ```bash
    cat << EOF > /tmp/rox-auth.yaml
    apiVersion: v1
    data:
      password: "$(echo -n ${ROX_API_TOKEN} | base64 -w0)"
      username: "$(echo -n ${ROX_ENDPOINT} | base64 -w0)"
    kind: Secret
    metadata:
      name: rox-auth
    EOF
    ```

    `kubeseal` 커맨드라인을 사용하여 시크릿 정의를 봉인(seal)합니다.

    ```bash
    kubeseal < /tmp/rox-auth.yaml > /tmp/sealed-rox-auth.yaml \
        -n <USER_NAME>-toolings \
        --controller-namespace sealed-secrets \
        --controller-name sealed-secrets \
        -o yaml
    ```

    다시, 이 봉인 작업의 결과, 특히 `encryptedData`를 가져와야 합니다. GitOps이므로 Git 저장소에 저장할 예정입니다 :)

    ```bash
    cat /tmp/sealed-rox-auth.yaml | grep -E 'username|password'
    ```

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
        username: AgAj3JQj+EP23pnzu...
        password: AgAtnYz8U0AqIIaqYrj...
    </code></pre></div>

    `mlops-gitops/toolings/sealed-secrets/config.yaml` 파일을 열어 Sealed Secrets 항목을 확장합니다. 이전 명령어의 `username`과 `password` 출력을 복사하여 값을 업데이트하세요. 데이터 들여쓰기가 올바른지 확인하세요.

    ```yaml
      - name: rox-auth
        type: kubernetes.io/basic-auth
        data:
          username: AgAj3JQj+EP23pnzu...
          password: AgAtnYz8U0AqIIaqYrj...
    ```

    변경사항을 git에 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🔒 ADD - stackrox sealed secret 🔒"
    git push
    ```

    Argo CD에서 봉인된 시크릿이 실제 OpenShift 시크릿으로 생성된 것을 확인할 수 있습니다.

    ![rox-auth.png](./images/rox-auth.png)

이제 ACS를 사용하여 연속 학습 파이프라인에서 보안을 **LEFT**로 이동시킬 수 있습니다.

## 이미지 스캔

1. 이미지 스캔을 위해 파이프라인을 확장합시다. 이를 위해 `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열고 `image_scan: true` 플래그를 추가하여 [스캔 작업](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/tasks/image-scan.yaml)을 도입합니다.
   
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
    image_scan: true # 👈 add this
    ```

2. 변경사항을 저장소에 커밋합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🤳 image scan task is added 🤳"
    git push
    ```

    OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 Pipelines에서 `image_scan` 작업이 파이프라인에 포함되었는지 확인하세요:

    ![image-scan-task.png](./images/image-scan-task.png)


3. 빈 커밋으로 파이프라인을 실행하여 변경사항을 확인합니다:

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🏃 trigger pipeline for image scanning 🏃"
    git push
    ```

4. 파이프라인 로그에서 이미지 스캔이 수행된 것을 확인할 수 있습니다:

    ![image-scan-pipeline.png](./images/image-scan-pipeline.png)

5. 또는 ACS UI에서 [여기](https://central-rhacs-operator.<CLUSTER_DOMAIN>/main/vulnerabilities/all-images?entityTab=Image&vulnerabilityState=OBSERVED&observedCveMode=WITH_CVES&sortOption[field]=Image%20scan%20time&sortOption[direction]=desc&s[SEVERITY][0]=Critical&s[SEVERITY][1]=Important&s[FIXABLE][0]=Fixable&s[Image][0]=<USER_NAME>-test/jukebox)를 클릭하여 보고서를 볼 수 있습니다. `Scan time`으로 정렬하면 최신 이미지 스캔을 자세히 확인할 수 있습니다.

    ![image-scan-acs.png](./images/image-scan-acs.png)

    스캔 결과:

    ![image-scan-result.png](./images/image-scan-result.png)