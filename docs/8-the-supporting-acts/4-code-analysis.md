# Sonarqube

> Sonarqube는 정적 코드 분석을 수행하는 도구입니다. 코딩상의 함정을 찾아내어 보고합니다. 취약점을 잡아내기에 훌륭한 도구입니다!

## GitOps를 사용하여 Sonarqube 배포하기

1. `mlops-gitops/toolings` 폴더 아래에 `sonarqube` 폴더를 생성합니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/sonarqube
    touch /opt/app-root/src/mlops-gitops/toolings/sonarqube/config.yaml
    ```

2. `sonarqube/config.yaml` 파일을 열고 아래 yaml을 `config.yaml`에 붙여넣습니다. 이 파일에는 Argo CD가 SonarQube의 helm 차트를 어디서 찾을 수 있는지, 그리고 이 helm 차트에 제공할 값들이 포함되어 있습니다.

    ```yaml
    repo_url: https://github.com/redhat-cop/helm-charts.git
    chart_path: charts/sonarqube
    fullnameOverride: sonarqube
    account:
      username: admin
      password: <PASSWORD>Strong123_
      currentAdminPassword: admin
    plugins:
      install:
        - https://github.com/checkstyle/sonar-checkstyle/releases/download/10.9.3/checkstyle-sonar-plugin-10.9.3.jar
        - https://github.com/dependency-check/dependency-check-sonar-plugin/releases/download/3.1.0/sonar-dependency-check-plugin-3.1.0.jar
    ```

> ⚠️ 비밀번호를 더 안전하게 만드는 멋진 방법! 🍬 달콤한 숫자와 🧂 짭짤한 특수문자를 추가하세요... 네, 우리는 SonarQube의 기본 admin 비밀번호를 업데이트하고 싶으며, 여기서 자격 증명을 평문으로 저장하는 것이 바람직하지 않다는 것도 알고 있지만 GitOps와 함께하는 시크릿 관리에 대해 아직 논의하지 않았으니 조금만 기다려 주세요!🫣

3. 변경사항을 푸시하고 Argo CD가 SonarQube를 배포하도록 합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🦇 ADD - sonarqube 🦇"
    git push 
    ```

4. [SonarQube UI](https://sonarqube-<USER_NAME>-toolings.<CLUSTER_DOMAIN>/)에 접속하여 설치가 성공했는지 확인합니다 (사용자 이름 `admin` & 비밀번호 `<PASSWORD>Strong123_`).

    _SonarQube 설정에는 몇 분 정도 소요될 수 있습니다._

    ```bash
    echo https://$(oc get route sonarqube --template='{{ .spec.host }}' -n <USER_NAME>-toolings)
    ```

    _클러스터에서 로그아웃한 경우, 아래 명령어로 다시 로그인하세요._

    ```bash
    oc login --server=https://api.<TRIMMED_CLUSTER_DOMAIN>:6443 -u <USER_NAME> -p <PASSWORD>
    ```

5. SonarQube를 사용해 파이프라인을 확장하기 전에 IDE에서 코드 품질 검사를 실행할 수 있습니다. 먼저 `pysonar` 라이브러리를 설치합니다.

    ```bash
    pip install pysonar-scanner
    ```

    API 토큰을 가져옵니다:

    ```bash
    SONARQUBE_TOKEN=$(curl -s -u admin:<PASSWORD>Strong123_ -XPOST https://$(oc get route sonarqube --template='{{ .spec.host }}' -n <USER_NAME>-toolings)/api/user_tokens/generate -d "name=scan&type=GLOBAL_ANALYSIS_TOKEN" | jq -r .token )
    ```

    그리고 스캔을 실행합니다:

    ```bash
    cd /opt/app-root/src
    pysonar-scanner -Dsonar.host.url=http://sonarqube.<USER_NAME>-toolings.svc.cluster.local:9000 -Dsonar.projectKey=jukebox -Dsonar.token=$SONARQUBE_TOKEN
    ```

6. 분석이 완료되면 [SonarQube UI](https://sonarqube-<USER_NAME>-toolings.<CLUSTER_DOMAIN>/)로 돌아가 페이지를 새로고침하고 `jukebox`가 `Projects` 아래에 있는지 확인합니다.

    ![sonarqube-1.png](./images/sonarqube-1.png)

    `jukebox` 프로젝트를 클릭하여 분석 결과를 확인합니다.

    ![sonarqube-2.png](./images/sonarqube-2.png)

    SonarQube가 식별한 문제들을 자세히 살펴볼 수 있습니다.

    ![sonarqube-3.png](./images/sonarqube-3.png)

## 파이프라인 확장하기

1. 이제 SonarQube가 어떻게 작동하는지 확인했으니, 파이프라인을 확장하여 매번 정적 코드 분석 검사를 수행하도록 하겠습니다. 다시 `mlops-gitops/toolings/ct-pipeline/config.yaml`을 열고 `static_code_analysis: true` 플래그를 추가하여 [작업](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/tasks/static-code-analysis.yaml)을 도입합니다.

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true
    linting: true 
    static_code_analysis: true # 👈 add this
    ```

2. 변경사항을 저장소에 커밋합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🧦 ADD - static code analysis step 🧦"
    git push
    ```

3. OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 Pipelines로 이동하여 작업이 파이프라인에 포함되었는지 확인합니다.

    ![sonarqube-task.png](./images/sonarqube-task.png)

4. 원한다면 빈 커밋을 만들어 파이프라인의 변화를 확인할 수 있습니다. 하지만 더 흥미로운 도구들과 함께 계속 확장해 나가도 좋습니다!

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🐹 trigger pipeline for SonarQube scan 🐹"
    git push
    ```

5. 이번에는 파이프라인이 `static-code-analysis` 단계를 통과하는 것을 확인하세요 🥳

    ![sonarqube-task-success.png](./images/sonarqube-task-success.png)