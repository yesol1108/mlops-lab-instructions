## 새로운 워크벤치 이미지

앞으로의 실습에서는 여러 명령줄 인터페이스, 즉 CLI(_스포일러 없음!🤫_)를 사용하여 보안 관련 작업을 로컬에서 수행한 후 파이프라인에 통합할 예정입니다. 이러한 CLI는 OpenShift AI에 기본으로 제공되는 워크벤치 이미지에 포함되어 있지 않습니다. 이를 해결하기 위해 커스텀 워크벤치 이미지를 제작했습니다!

기본 이미지로 `<USER_NAME>-mlops-toolings` 워크벤치(code-server) 이미지를 사용하고, 필요한 도구들을 설치한 후 OpenShift AI에서 모두가 사용할 수 있도록 배포했습니다.**

이제 현재 워크벤치를 이 새로운 이미지로 데이터 손실 없이 원활하게 전환할 수 있습니다.

**_Containerfile은 [여기](https://github.com/rhoai-mlops/deploy-lab/blob/main/Containerfile)에서 확인할 수 있습니다._


1. `<USER_NAME>-jukebox` 프로젝트의 Workbenches에서 `<USER_NAME>-mlops-toolings` 워크벤치(code-server)의 `Edit Workbench`를 선택합니다.
    ![codeserver-notebook-1.png](./images/codeserver-notebook-1.png)

2. `Notebook image` > `Image selection`으로 이동하여 `ml500-code-server`를 선택한 후 `Update workbench`를 클릭합니다.

    ![codeserver-notebook-1.png](./images/codeserver-notebook-2.png)

3. 선택한 이미지가 아래와 같이 Workbenches 목록에 표시되는 것을 확인할 수 있습니다.

    ![codeserver-notebook-1.png](./images/codeserver-notebook-3.png)

4. 새로운 워크벤치 이미지로 시작하기 위해 워크벤치가 재시작됩니다. 상태가 `Running`이 되면 열어서 몇 가지 CLI를 실행해 이미지를 확인해보세요.

    ```bash
    helm version
    ```
    ```bash
    roxctl version
    ```
    ```bash
    kube-linter version
    ```
다음과 같은 출력 결과를 확인할 수 있을 것입니다:

    ![cli-ouput.png](./images/cli-output.png)


이제 이 명령어들을 사용해 봅시다! 🏃💨