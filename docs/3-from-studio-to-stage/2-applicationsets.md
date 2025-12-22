## ApplicationSets

# GitOps를 위한 Gitea 준비하기

> 이 실습에서는 Argo CD(우리의 GitOps 컨트롤러)를 Git 저장소에 연결하여 GitOps 워크플로우를 활성화합니다. `mlops-gitops` 저장소에 도구 및 모델 배포 정의를 저장하고 Argo CD가 해당 저장소를 인식하도록 설정할 것입니다.

Gitea는 팀이 저장소를 관리하고 이슈를 추적하며 효율적으로 코드 협업을 할 수 있도록 하는 경량의 자체 호스팅 Git 서버입니다. 오픈 소스이며 배포가 쉽고 다양한 버전 관리 작업을 지원합니다. Gitea는 본 워크숍에서 `mlops-gitops` 구성이 위치하는 중앙 저장소 역할을 하며 Argo CD와 원활하게 통합됩니다.

1. 자격 증명으로 Gitea에 로그인하세요. Gitea URL:

    ```bash
    https://<GIT_SERVER>
    ```

    이미 생성된 `mlops-gitops` 저장소를 볼 수 있습니다. 이 저장소는 <span style="color:purple;" >GIT</span>Ops 용도로 사용할 Git 저장소입니다. 도구 구성과 모델 배포 정의를 모두 포함하는 모노레포 역할을 합니다. 실제 환경에서는 이를 별도의 저장소로 분리할 수도 있습니다! 어쨌든 시작해봅시다!

    ![gitea-mlops-gitops.png](images/gitea-mlops-gitops.png)

2. `<USER_NAME>-mlops-toolings` 작업 공간(code-server) 터미널로 돌아가서 저장소를 클론합니다.

    ```bash
    cd /opt/app-root/src
    git clone https://<USER_NAME>:<PASSWORD>@<GIT_SERVER>/<USER_NAME>/mlops-gitops.git
    ```

   Git 프로젝트를 클론했으니 GitOps 여정을 시작해봅시다 🧙‍♀️🦄!

3. 이 `mlops-gitops` 저장소는 Argo CD `ApplicationSet` 정의를 포함하고 있어 여기서 정의한 모든 애플리케이션을 생성할 수 있습니다. `ApplicationSet`은 여러 Argo CD 애플리케이션을 동적으로 생성하고 관리할 수 있는 리소스입니다. 도구용, 테스트 환경용, 프로덕션 환경용 총 세 가지 `ApplicationSet` 정의가 있습니다.

  도구부터 시작해봅시다 - IDE에서 `mlops-gitops/appset-toolings.yaml` 파일을 열고 `CLUSTER_DOMAIN`과 `USER_NAME` 플레이스홀더를 본인 값으로 업데이트하세요. 그런 다음 `toolings/bootstrap/config.yaml` 파일도 동일하게 수정합니다. 또는 아래 명령어를 실행하여 자동으로 변경사항을 적용할 수 있습니다.

    ```bash
      sed -i -e 's/CLUSTER_DOMAIN/<CLUSTER_DOMAIN>/g' /opt/app-root/src/mlops-gitops/appset-toolings.yaml
      sed -i -e 's/USER_NAME/<USER_NAME>/g' /opt/app-root/src/mlops-gitops/appset-toolings.yaml
      sed -i -e 's/USER_NAME/<USER_NAME>/g' /opt/app-root/src/mlops-gitops/toolings/bootstrap/config.yaml
    ```

5. 이것이 바로 GITOPS입니다 - 먼저 변경사항을 커밋해야 합니다! 구성을 git에 반영해봅시다 👇

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git config --global user.email "<USER_NAME>@mlops-wizard.com"
    git config --global user.name "<USER_NAME>"
    git add .
    git commit -m  "🦆 ADD - ApplicationSet definition 🦆"
    git push
    ```

6. `appset-toolings.yaml` 파일은 `toolings` 폴더를 참조하며, 이 폴더에는 MinIO, Tekton 파이프라인, Feast 등 지속적 학습 파이프라인에 필요한 모든 정의가 포함되어 있습니다. 현재는 두 개의 애플리케이션만으로 시작합니다. `toolings` 폴더 내에는 클러스터 부트스트랩에 필요한 네임스페이스와 권한을 처리하는 `bootstrap` 하위 폴더와 저장소 환경을 정의하는 `minio` 하위 폴더가 있습니다. 이 구성은 저장소와 환경 정의를 모두 Git에 저장한다는 의미입니다. 앞서 설명했듯이, 이것이 GitOps이며 원하는 상태는 ✨Git✨에 저장되어야 합니다.

  우리가 해야 할 일은 ApplicationSet 객체를 생성하는 것뿐이며, 나머지는 Argo CD가 처리합니다.

    ```bash
      oc apply -f /opt/app-root/src/mlops-gitops/appset-toolings.yaml -n <USER_NAME>-toolings
    ```

6. 이제 Argo CD에서 ApplicationSet이 `toolings` 하위 폴더를 인식하고 애플리케이션을 배포했는지 확인하세요!

    ![argocd-bootstrap-tooling](./images/argocd-bootstrap-tooling.png)

8. Argo CD가 리소스를 동기화하면 클러스터에서도 확인할 수 있습니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)에서 다음 명령어를 실행하세요:

    ```bash
    oc get projects | grep <USER_NAME>
    ```

  모든 것이 정상이라면 다음과 같은 결과를 볼 수 있습니다:

  ![argocd-created-projects](./images/argocd-created-projects.png)

  또한 `<USER_NAME>-toolings` 네임스페이스에서 실행 중인 파드를 확인할 수 있습니다:

  ```bash
  oc get pods -n <USER_NAME>-toolings
  ```

  모든 것이 정상이라면 다음과 같은 결과를 볼 수 있습니다:

  ![deployed-apps-pods](./images/deployed-apps-pods.png)

🪄🪄 매직! 이제 `ApplicationSet`을 통해 도구와 프로젝트를 반복 가능하고 감사 가능한 방식(즉, git을 통해)으로 배포했습니다! 이제 git push만으로 도구를 확장하는 방법을 살펴봅시다! 🪄🪄