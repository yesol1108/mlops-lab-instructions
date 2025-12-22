## 데이터 사이언스 프로젝트

1. [OpenShift AI](https://rhods-dashboard-redhat-ods-applications.<CLUSTER_DOMAIN>)에 로그인합니다.  
   접속 링크와 계정 정보는 강사가 제공합니다. 로그인 후, 이미 두 개의 `Data Science Project`가 생성되어 있는 것을 확인할 수 있습니다.

![datascienceproject.png](./images/datascienceproject.png)

2. `<USER_NAME>-jukebox` 프로젝트를 클릭합니다.  
   이 프로젝트는 이후 모델을 실험하고, 학습시키고, 배포하는 공간이 됩니다.

![datascienceproject-2.png](./images/datascienceproject-2.png)

3. 이제 노트북을 생성해봅시다. `Create a Workbench`를 클릭하세요.  
   OpenShift AI Dashboard는 꽤 직관적이지 않나요? 🙂

   원하는 이름을 입력합니다. 예를 들어 `<USER_NAME>-hitmusic-wb` 같은 이름을 사용할 수 있습니다 🎺

    **Notebook Image:** 

    - Image selection: `Standard Data Science`
    - Version selection: `2025.1`
  
    **Deployment size**
    - Container size: `Small`

    **Environment variables**
    - 현재는 추가할 필요가 없습니다.

    **Cluster storage**
    - 최대 20 GiB로 그대로 둡니다.

    **Connections**
    - `Attach existing connections` 선택
    - 드롭다운 메뉴에서 `models`를 선택한 뒤 `Attach` 클릭
       
    설정을 모두 마쳤다면 `Create workbench`를 클릭합니다.

4. 새로 생성한 워크벤치의 상태가 `Running`으로 표시되면, 워크벤치 이름을 클릭해 접속합니다.

    ![create-a-workbench.png](./images/create-a-workbench.png)
<!-- 
   Jupyter Notebook UI가 열립니다. 다시 한 번 계정 정보로 로그인해야 합니다.  
   아래와 같은 화면이 보이면 `Allow selected permissions`를 클릭하세요.  
   그러면 Jupyter Notebook으로 이동합니다.

    ![create-a-workbench-4.png](./images/create-a-workbench-4.png) -->

5. 이미 Gitea 서버에 사용자 계정 기준으로 여러 Git 저장소가 준비되어 있습니다.  
   아래 링크를 통해 Gitea에 로그인하여 확인할 수 있습니다.

    ```bash
    https://<GIT_SERVER>
    ```

6. 동일한 계정 정보로 로그인한 뒤, 앞으로의 실습에 사용할 **4개의 저장소**가 준비되어 있는지 확인하세요.  
   스포일러 하나 하자면, GitOps 🦄🔥 를 주의 깊게 보셔야 합니다.

  ![gitrepositories.png](./images/gitrepositories.png)

7. 이제 모델 소스 코드가 포함된 `Jukebox` 저장소를 클론해보겠습니다.  
   Jupyter Notebook으로 돌아가 Git 아이콘을 클릭한 후, 아래 Git URL을 복사해 **clone** 합니다.

    ```bash
    https://<USER_NAME>:<PASSWORD>@<GIT_SERVER>/<USER_NAME>/jukebox.git
    ```

    ![notebook-clone-repo.png](./images/notebook-clone-repo.png)

    저장소를 클론한 뒤에는, 왼쪽 패널에 `jukebox` 폴더가 표시될 것입니다.

    ![jupyter-notebook-ui.png](./images/jupyter-notebook-ui.png)

8. 본격적으로 실습을 시작하기 전에, 다음 챕터에서 사용하게 될 스토리지 환경에 대해 먼저 알아보겠습니다. 🫡
