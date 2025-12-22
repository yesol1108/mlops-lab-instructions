## 🐙 ArgoCD - GitOps 컨트롤러  
GitOps부터 시작해봅시다! GitOps는 MLOps에서 매우 중요한데, 머신러닝 워크플로우와 모델 배포를 일관되고 자동화된 방식으로 관리하여 모든 것이 버전 관리되고 추적 가능하며 재현 가능하도록 보장하기 때문입니다. Git을 단일 진실 소스로 사용함으로써 팀은 변경 사항을 쉽게 추적하고, 구성을 관리하며, 모델과 애플리케이션이 항상 올바른 상태로 배포되도록 할 수 있습니다.

GitOps를 실천하기 위해 Argo CD를 GitOps 엔진으로 사용하겠습니다.

### Argo CD 애플리케이션  
Argo CD는 가장 인기 있는 GitOps 도구 중 하나입니다. OpenShift 애플리케이션의 상태를 Git 저장소와 동기화 상태로 유지합니다. 이는 Git 저장소에 저장된 상태(원하는 상태)와 클러스터에서 실제로 실행 중인 상태(실제 상태)를 조정하는 컨트롤러입니다.

MLOps 맥락에서 Argo CD를 활용하여 도구와 모델을 반복 가능하고 재현 가능한 방식으로 배포할 것입니다. 구성 정의를 Git에 저장함으로써 Argo CD가 자동으로 해당 정의를 적용하여 배포 과정을 더 효율적이고 일관되게 만듭니다. 즉, YAML 파일을 다루게 되므로 다른 작업 환경인 `code-server`로 전환할 시간입니다. 솔직히 말해, Jupyter Notebook은 YAML 파일과 커맨드라인 유틸리티 작업에 최적이 아니죠🥲.

기존 Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 환경(Standard Data Science) 옆에 새로운 작업 환경을 `<USER_NAME>-jukebox` 프로젝트 내에 생성하고 시작해봅시다!

1. `OpenShift AI` > `Data Science Projects` > `<USER_NAME>-jukebox` > `Workbenches`로 이동하여 `Create workbench`를 클릭합니다.

   원하는 이름을 선택하세요. 예를 들어 `<USER_NAME>-mlops-toolings` 같은 이름이 좋습니다.

   노트북 이미지 선택:

   - 이미지 선택: `code-server`

   나머지는 기본값으로 두고 `Create workbench`를 클릭합니다.

   실행 상태가 되면 열고 자격 증명을 사용해 접속하세요.

   ![codeserver-wb.png](./images/codeserver-wb.png)

   저자를 신뢰하는지 묻는 메시지가 나오면 'Yes'를 선택하세요 :) 결국 저희를 신뢰하시잖아요… 맞죠? 💚

2. 왼쪽 상단 햄버거 메뉴를 클릭한 후 `Terminal` > `New Terminal`을 선택해 새 터미널을 엽니다.

   ![code-server-terminal.png](./images/code-server-terminal.png)

3. `<USER_NAME>-toolings` 환경에 이미 Argo CD 인스턴스가 설치되어 있습니다. 실행 중인지 확인하고 Argo CD UI에 로그인해봅시다.

   자격 증명을 사용해 OpenShift에 로그인하세요 (<PASSWORD>를 실제 비밀번호로 교체).

   ```bash
    oc login --server=https://api.<TRIMMED_CLUSTER_DOMAIN>:6443 -u <USER_NAME> -p <PASSWORD>
  ```

   Argo CD 파드가 실행 중인지 확인합니다:

   ```bash
  oc get pods -n <USER_NAME>-toolings
  ```

   ![argocd-running.png](./images/argocd-running.png)

4. 모든 파드가 실행 중이면 [여기](https://argocd-server-<USER_NAME>-toolings.<CLUSTER_DOMAIN>)를 클릭해 Argo CD UI에 로그인할 수 있습니다.  
   
   또는 아래 명령어로 URL을 확인한 후 새 브라우저 탭에서 열어도 됩니다.

   ```bash
  echo https://$(oc get route argocd-server --template='{{ .spec.host }}' -n <USER_NAME>-toolings)
  ```

5. `Log in via OpenShift`를 클릭하고 OpenShift 자격 증명을 사용해 Argo CD에 로그인합니다.

   ![argocd-login.png](./images/argocd-login.png)

6. 초기 로그인 시 `Allow selected permissions`를 선택합니다.

7. 이제 Argo CD에 로그인했습니다 👏👏👏! UI를 통해 샘플 애플리케이션을 배포해보겠습니다. MLOps 용도로 사용하기 전에 Argo CD의 마법을 맛보기 위한 것입니다. Argo CD에서 `CREATE APPLICATION`을 클릭하세요. 빈 폼이 나타납니다. 다음과 같이 작성합니다:
   * "GENERAL" 박스
      * Application Name: `todolist`
      * Project Name: `default`
      * Sync Policy: `Automatic`
   * "SOURCE" 박스
      * Repository URL: `https://rht-labs.com/todolist/`
      * 오른쪽 GIT/HELM 드롭다운 메뉴에서 `Helm` 선택
      * Chart: `todolist`
      * Version: `1.1.0`
   * "DESTINATION" 박스
      * Cluster URL: `https://kubernetes.default.svc`
      * Namespace: `<USER_NAME>-toolings`
   * "HELM" 박스
      * Values Files: `values.yaml`

    폼은 다음과 같아야 합니다:  
    ![argocd-create-application](images/argocd-create-application.png)

8. 생성 버튼을 누르면 `todolist` 애플리케이션이 생성되고 `<USER_NAME>-toolings` 네임스페이스에 배포가 시작됩니다.

   ![argocd-todolist-1.png](./images/argocd-todolist-1.png)

9. 애플리케이션을 자세히 보면 Argo CD가 생성한 모든 k8s 리소스를 멋지게 보여줍니다. 이 리소스들은 선택한 Helm 차트에 정의되어 있습니다.

   ![argocd-todolist-2.png](./images/argocd-todolist-2.png)

10. todolist 애플리케이션이 정상적으로 실행 중인지 확인하려면 앱 URL로 접속하세요. `<USER_NAME>-mlops-toolings` 작업 환경(code-server)으로 돌아가 터미널에서 다음 명령어를 실행합니다:

    ```bash
    echo https://$(oc get route/todolist -n <USER_NAME>-toolings --template='{{.spec.host}}')
    ```

  _URL을 CMD/CTRL + 클릭하면 새 브라우저 탭에서 열립니다._

🪄🪄 마법처럼! 이제 GitOps 컨트롤러인 Argo CD를 갖게 되었고, 수동으로 애플리케이션을 배포해보았습니다. 다음으로 Argo CD를 활용해 더 많은 GitOps 작업을 진행해봅시다 🪄🪄