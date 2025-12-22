# GitOps로 툴링 확장하기

지속적인 학습 파이프라인을 성공적으로 운영하려면, MLOps 툴셋에 두 가지 애플리케이션을 추가해야 합니다: KubeFlow Registry와 Data Science Pipelines Application(DSPA)입니다.

이 두 애플리케이션은 이미 개발 환경에 설치되어 있습니다. 이제 GitOps로 이들을 관리할 차례입니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server) 터미널로 이동하세요.

1. `mlops-gitops` 저장소에서 `toolings` 디렉토리 아래에 `model-registry` 폴더를 생성합니다. 그리고 `model-registry` 폴더 안에 `config.yaml` 파일을 만듭니다. 또는 아래 명령어를 실행하세요:

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/model-registry
    touch /opt/app-root/src/mlops-gitops/toolings/model-registry/config.yaml
    ```

2. `toolings/model-registry/config.yaml` 파일을 열고 아래 yaml 내용을 붙여넣으세요. 이 파일에는 Argo CD가 모델 레지스트리 헬름 차트를 어디서 찾을지, 그리고 헬름 차트에 제공할 값들이 포함되어 있습니다.

    ```yaml
  chart_path: charts/model-registry
  name: <USER_NAME>
  cluster_domain: <CLUSTER_DOMAIN>
    ```

3. 이제 Data Science Pipeline Application 정의를 위해 `toolings` 아래에 `dspa` 폴더를 만들고, `dspa` 폴더 안에 `config.yaml` 파일을 생성합니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/dspa
    touch /opt/app-root/src/mlops-gitops/toolings/dspa/config.yaml
    ```

4. `dspa/config.yaml` 파일을 열고 아래 yaml 내용을 붙여넣으세요. Data Science Pipeline Application은 별도의 값을 제공할 필요가 없으므로 간단한 설정 파일입니다.

    ```yaml
  chart_path: charts/dspa
    ```

5. 변경사항을 푸시하기 전에, 변경사항이 Git 저장소에 반영되는 즉시 동기화되도록 설정하고 싶습니다! 하지만 Argo CD는 기본적으로 약 3분 주기로 동기화되므로 너무 느립니다. 이를 해결하기 위해 Argo CD와 GitOps 저장소를 연결하는 웹훅을 추가하겠습니다. 다음 명령어로 Argo CD URL을 확인하세요:

    ```bash
    echo https://$(oc get route argocd-server --template='{{ .spec.host }}'/api/webhook  -n <USER_NAME>-toolings)
    ```
    > 참고: 이 웹훅 URL은 Argo CD와 GitOps 저장소를 연결하기 위한 것이며, 브라우저에서 열지 마세요 :)

6. Gitea로 이동하여 `mlops-gitops` 저장소의 오른쪽 상단 `Settings`로 들어갑니다. `Settings` 페이지에서 `Webhooks`를 클릭하고, `Gitea` 타입의 새 웹훅을 추가합니다.

![gitea-argocd-webhook.png](./images/gitea-argocd-webhook.png)

7. 이전 명령어로 얻은 Argo CD URL을 `Target URL`에 복사하고 `Add Webhook`을 클릭합니다.

![gitea-argocd-webhook-2.png](./images/gitea-argocd-webhook-2.png)

8. 이제 변경사항을 GitOps 저장소에 푸시합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git add .
    git commit -m  "😻 ADD - Model Registry and DSPA 😻"
    git push
    ```

9. Argo CD에서 배포된 애플리케이션을 확인하세요 :)  
OpenShift에 접속하면 `<USER_NAME>-toolings` 네임스페이스에서 이 툴들이 배포된 것을 확인할 수 있습니다. 자유롭게 살펴보세요!

![model-registry-dspa.png](./images/model-registry-dspa.png)

다음 단계는 GitOps를 통해 모델 배포를 위한 Argo CD 설정입니다!