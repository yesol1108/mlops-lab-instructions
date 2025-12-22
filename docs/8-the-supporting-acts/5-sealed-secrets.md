## Sealed Secrets

GitOps라고 하면 우리는 _"Git에 없으면, 존재하지 않는 것"_ 이라고 말합니다. 그런데 많은 사람들이 접근할 수 있는 Git 저장소에 자격 증명과 같은 민감한 데이터를 어떻게 저장할 수 있을까요?! 물론 Kubernetes는 시크릿을 관리하는 방법을 제공하지만, 문제는 민감한 정보를 base64 문자열로 저장한다는 점입니다 - 누구나 base64 문자열을 디코딩할 수 있습니다! 따라서 `Secret` 매니페스트 파일을 공개적으로 저장할 수 없습니다. 이 문제를 해결하기 위해 Sealed Secrets라는 오픈소스 도구를 사용합니다.

Sealed Secrets는 `kubeseal`이라는 유틸리티를 사용하여 Kubernetes 시크릿을 _봉인(seal)_ 할 수 있게 해줍니다. `SealedSecrets`는 컨트롤러만 복호화할 수 있는 암호화된 `Secret` 객체를 포함하는 Kubernetes 리소스입니다. 따라서 `SealedSecret`은 공개 저장소에 저장해도 안전합니다.

### Sealed Secrets 실습

1. 이전 실습에서 SonarQube용 시크릿을 생성하고 그냥 Git에 추가한 것을 눈치채신 분들도 있을 겁니다...😳 이제 이를 수정하고 SonarQube 자격 증명을 봉인하여 안전하게 저장소에 커밋할 수 있도록 해봅시다. (네, git 커밋 기록은 알지만, 요점을 설명하기 위함이니 양해 부탁드립니다 🤣)

    먼저, 임시 디렉터리에 시크릿을 생성하겠습니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)으로 이동하여 터미널에서 아래 코드를 실행하세요.

    ```bash
    cat << EOF > /tmp/sonarqube.yaml
    apiVersion: v1
    data:
      username: "$(echo -n admin | base64 -w0)"
      password: "$(echo -n <PASSWORD>Strong123_ | base64 -w0)"
      currentAdminPassword: "$(echo -n admin | base64 -w0)"
    kind: Secret
    metadata:
      name: sonarqube-auth
    EOF
    ```

3. `kubeseal` 명령어를 사용하여 시크릿 정의를 봉인합니다. 이 명령은 클러스터 내에서 실행 중인 컨트롤러에 저장된 인증서를 사용해 암호화합니다. 클러스터당 하나의 인스턴스만 존재할 수 있으므로 이미 배포되어 있습니다.

    <p class="warn">
        ⛷️ <b>참고</b> ⛷️ - Kubeseal 명령 실행 시 "Error: cannot get sealed secret service: Unauthorized" 오류가 발생하면, OpenShift에 다시 로그인한 후 명령을 다시 실행하세요.
    </p>

    ```bash
    oc login --server=https://api.<TRIMMED_CLUSTER_DOMAIN>:6443 -u <USER_NAME> -p thisisthepassword

    ```

    ```bash
    kubeseal < /tmp/sonarqube.yaml > /tmp/sealed-sonarqube.yaml \
    -n <USER_NAME>-toolings \
    --controller-namespace sealed-secrets \
    --controller-name sealed-secrets \
    -o yaml
    ```

4. 시크릿이 봉인되었는지 확인합니다:

    ```bash
    cat /tmp/sealed-sonarqube.yaml
    ```

    이제 시크릿이 봉인되어 저장소에 안전하게 저장할 수 있음을 확인할 수 있습니다. 출력은 아래와 비슷하지만, 비밀번호와 사용자 이름은 더 길게 표시됩니다.

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
    apiVersion: bitnami.com/v1alpha1
    kind: SealedSecret
    metadata:
      creationTimestamp: null
      name: sonarqube-auth
      namespace: <USER_NAME>-toolings
    spec:
      encryptedData:
        username: AgAj3JQj+EP23pnzu...
        password: AgAtnYz8U0AqIIaqYrj...
        currentAdminPassword: AgAtnYz8U0AqIIaqYrj...
    ...
    </code></pre></div>

5. 이 봉인 작업의 결과, 특히 `encryptedData` 값을 가져와 Git에 추가해야 합니다. 이를 위해 반복 가능하게 클러스터에 봉인된 시크릿을 추가할 수 있는 <span style="color:blue;">[헬퍼 헬름 차트](https://github.com/redhat-cop/helm-charts/tree/master/charts/helper-sealed-secrets)</span>를 이미 작성해 두었습니다. 다음 단계에서 이 차트에 `encryptedData` 값을 제공할 것입니다.

    ```bash
    cat /tmp/sealed-sonarqube.yaml| grep -E 'username|password|currentAdminPassword'
    ```

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
        username: AgAj3JQj+EP23pnzu...
        password: AgAtnYz8U0AqIIaqYrj...
        currentAdminPassword: AgAtnYz8U0AqIIaqYrj...
    </code></pre></div>

4. `mlops-gitops/toolings` 디렉터리를 열고 `sealed-secrets` 폴더를 생성한 뒤, 그 안에 `config.yaml` 파일을 만듭니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/sealed-secrets
    touch /opt/app-root/src/mlops-gitops/toolings/sealed-secrets/config.yaml
    ```

5. `sealed-secrets/config.yaml` 파일을 열고 아래 YAML을 붙여넣으세요. 반복 가능하게 클러스터에 봉인된 시크릿을 추가할 수 있는 <span style="color:blue;">[헬퍼 헬름 차트](https://github.com/redhat-cop/helm-charts/tree/master/charts/helper-sealed-secrets)</span>를 이미 작성해 두었습니다. 다음 단계에서 이 차트에 `encryptedData` 값을 제공할 것입니다.

    먼저, 아래 내용을 복사하세요:

    ```yaml
    repo_url: https://github.com/redhat-cop/helm-charts.git
    chart_path: charts/helper-sealed-secrets
    ```

    그리고 이전에 얻은 암호화된 비밀번호로 `config.yaml` 파일을 확장합니다:

    ```yaml
    repo_url: https://github.com/redhat-cop/helm-charts.git
    chart_path: charts/helper-sealed-secrets
    # ⬇️ extend by adding sealed secrets below
    secrets:
      # Additional secrets will be added to this list when necessary
      - name: sonarqube-auth
        type: Opaque
        data:
          username: AgAj3JQj+EP23pnzu...
          password: AgAtnYz8U0AqIIaqYrj...
          currentAdminPassword: AgCHCphbYpeLYMPK...
    ```

6. `sonarqube/config.yaml` 파일을 열어 비밀번호 정보를 제거하고 봉인된 시크릿 정의를 참조하도록 아래와 같이 수정합니다:

    ```yaml
    repo_url: https://github.com/redhat-cop/helm-charts.git
    chart_path: charts/sonarqube
    account:
      existingSecret: sonarqube-auth # 👈 this is the change
    plugins:
      install:
        - https://github.com/checkstyle/sonar-checkstyle/releases/download/10.9.3/checkstyle-sonar-plugin-10.9.3.jar
        - https://github.com/dependency-check/dependency-check-sonar-plugin/releases/download/3.1.0/sonar-dependency-check-plugin-3.1.0.jar
    ```

7. 파이프라인을 수정하여 봉인된 시크릿으로 생성한 시크릿을 사용해 정적 코드 분석을 수행하도록 합니다. 다시 `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열고 `static_code_analysis_secret: sonarqube-auth` 플래그를 추가하여 파이프라인 [트리거](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/triggers/gitea-trigger-template.yaml)를 수정하세요.

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
    static_code_analysis_secret: sonarqube-auth # 👈 this is the change
    ```

8. 변경 사항을 Git에 추가하고 커밋한 후 푸시합니다 (GITOPS WOOOO 🪄🪄).

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🤫 ADD - sealed secrets 🤫"
    git push 
    ```

9. 🪄 🪄 Argo CD에 로그인하면 이제 Argo CD UI에서 Sealed Secret 애플리케이션을 볼 수 있습니다. 이는 일반 k8s 시크릿으로 복호화되어 표시됩니다 🪄 🪄

    `SealedSecret`을 자세히 보면 `sonarqube` 시크릿이 자동으로 동기화된 것을 확인할 수 있습니다:

    ![argocd-sonar-auth-synced.png](images/argocd-sonar-auth-synced.png)