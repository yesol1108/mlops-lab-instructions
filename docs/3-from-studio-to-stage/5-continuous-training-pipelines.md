# 연속 학습 파이프라인

이 실습에서는 OpenShift Pipelines (Tekton)를 설정하여 `jukebox` 저장소에 푸시가 발생할 때마다 Kubeflow Pipeline이 자동으로 트리거되도록 합니다. Kubeflow Pipeline은 모델 학습을 처리하며, 모델이 준비되면 테스트 환경에 배포되어 검증을 진행합니다. 추적 가능성을 보장하기 위해 Tekton 파이프라인은 Git의 `mlops-gitops/model-deployments/test/jukebox` 파일에 모델 버전 정보를 업데이트합니다. 이를 통해 Argo CD가 변경 사항을 감지하고 배포 업데이트를 자동으로 관리할 수 있습니다.


## 연속 학습 파이프라인 배포

1. 먼저, Tekton 파이프라인 정의를 저장한 Git 저장소를 클론합니다.

    ```bash
    cd /opt/app-root/src
    git clone https://<USER_NAME>:<PASSWORD>@<GIT_SERVER>/<USER_NAME>/mlops-helmcharts.git
    ```

    저장소를 클론한 후, 왼쪽 탐색기 메뉴에서 `mlops-helmcharts/charts/pipelines` 폴더로 이동합니다. 내부에는 이전 장에서 수동으로 실행했던 Kubeflow Pipeline을 `templates/tasks/execute-ds-pipeline.yaml` 파일에서 호출하는 것을 확인할 수 있습니다.

    또한, 파이프라인에서 실행해야 할 여러 다른 단계들도 포함되어 있습니다:

    ![tekton-pipeline-overview.png](./images/tekton-pipeline-overview.png)

2. 이 Tekton 파이프라인 정의를 `<USER_NAME>-toolings` 환경에 적용해야 합니다. 이렇게 하면 웹훅 URL이 제공되며, 이를 `Jukebox` 저장소의 트리거로 추가할 것입니다. 이 설정을 통해 모델 소스 코드에 변경이 있을 때마다 파이프라인이 실행되도록 할 수 있습니다 (다른 업데이트에도 작동할 수 있지만, 그건 일단 비밀로 해두죠 🤭).

    `mlops-gitops/toolings/` 아래에 `ct-pipeline` 폴더를 만들고, 새로 생성한 폴더 안에 `config.yaml` 파일을 생성하세요. 또는 아래 명령어를 실행해도 됩니다.  
    여기서 `ct`는 Continuous Training의 약자입니다 :)

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/ct-pipeline
    touch /opt/app-root/src/mlops-gitops/toolings/ct-pipeline/config.yaml
    ```

3. `ct-pipeline/config.yaml` 파일을 열고 아래 yaml 내용을 붙여넣으세요. 내용은 익숙하실 겁니다:

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    ```

4. 다시 말하지만, 이것은 GITOPS입니다 - 변경 사항을 적용하려면 이제 커밋을 해야 합니다! Argo CD가 변경 사항을 동기화하도록 하기 전에 구성을 git에 반영합시다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git add .
    git commit -m  "🥁 ADD - Continuous training pipeline 🥁"
    git push
    ```

    Argo CD에서 확인하면 파이프라인이 이미 나타난 것을 볼 수 있습니다!

    ![ct-pipeline.png](./images/ct-pipeline.png)

    _참고: Argo CD에서 PVC가 아직 Progressing 상태로 표시된다면, 이는 OpenShift 클러스터가 첫 번째 소비자, 즉 첫 번째 파이프라인 실행이 퍼시스턴트 볼륨을 생성하기를 기다리고 있기 때문입니다. 첫 실행 후 동기화 상태가 정상으로 바뀝니다._

    ![pvc-progressing.png](./images/pvc-progressing.png)

5. 이제 웹훅을 가져와 Jukebox 저장소에 추가합시다. 아래 명령어를 실행하여 웹훅 URL을 복사하세요:

    ```bash
    echo https://$(oc -n <USER_NAME>-toolings get route el-ct-listener --template='{{ .spec.host }}')
    ```

6. URL을 복사한 후, Gitea에서 `jukebox` 저장소 > `Settings` > `Webhooks`로 이동하여 `Gitea`를 선택해 웹훅을 추가합니다:

    ![add-webhook.png](./images/add-webhook.png)

    Jukebox 저장소에 커밋을 생성하여 웹훅을 트리거할 수 있습니다. 간단히 시뮬레이션해봅시다!  
    `<> Code`를 클릭해 Jukebox 파일로 돌아갑니다.

    ![jukebox-gitea.png](./images/jukebox-gitea.png)

    `README.md` 파일을 열고 ✏ 아이콘을 클릭해 편집합니다.

    ![jukebox-edit-readme.png](./images/jukebox-edit-readme.png)

    파일에 새 줄을 추가하고 `Commit changes`를 클릭하세요:

    ![jukebox-empty-commit.png](./images/jukebox-empty-commit.png)

8. 이 커밋이 파이프라인을 트리거합니다! OpenShift 콘솔의 `PipelineRuns` 뷰와 OpenShift AI의 `Data Science Pipeline` > `Runs` 뷰에서 파이프라인 진행 상황을 모니터링할 수 있습니다. (파이프라인이 너무 많네요!🙈)

    먼저 `OpenShift Console` > `Pipelines` > `PipelineRuns`로 이동하여 `컬러 바`를 클릭해 로그를 확인하세요.

    `<USER_NAME>-toolings` 프로젝트에 있는지 확인하세요.

    ![openshift-pipeline.png](./images/openshift-pipeline.png)

    또는 아래 링크를 사용할 수 있습니다:

    ```bash
    https://console-openshift-console.<CLUSTER_DOMAIN>/pipelines/ns/<USER_NAME>-toolings/pipeline-runs
    ```

    Tekton 파이프라인 실행 로그에서 두 번째 단계에서 Kubeflow Pipeline이 트리거되는 것을 확인할 수 있습니다.

    ![pipeline-running-state.png](./images/pipeline-running-state.png)

    Kubeflow 파이프라인이 트리거되면 `OpenShift AI Dashboard` > `Experiments` > `Experiments and Runs`로 이동하여 현재 실행 중인 작업을 클릭해 세부 정보를 확인할 수 있습니다.

    ![openshift-ai-pipeline.png](./images/openshift-ai-pipeline.png)

    파이프라인은 모델을 빌드하고, 컨테이너화하며, 배포하고, Kubeflow Registry에 정보를 저장합니다. 이는 Data Science 내부 루프에서 수동으로 했던 작업과 동일합니다!!

    이 파이프라인의 첫 실행은 완료하는 데 시간이 걸립니다. 하지만 이후 실행부터는 Kubeflow Pipeline의 캐싱 기능을 활용하여 입력이 변경되지 않은 이전 단계의 결과를 재사용합니다. 이를 통해 처리 시간이 크게 단축되고 파이프라인 속도가 빨라집니다 🧚‍♂️🧚‍♂️

9. 파이프라인이 완료되면 다음과 같은 화면을 볼 수 있습니다:

    ![pipeline-done.png](./images/pipeline-done.png)

    파이프라인에서 모델에 추가한 메타데이터는 모델 레지스트리에서 확인할 수 있습니다.  
    Models -> Model Registry -> **<USER_NAME>-prod-registry 선택** -> jukebox -> <모델 버전 링크>로 이동하세요.

    ![Model Metadata](./images/model-metadata-info.png)

    또한 `<USER_NAME>-test` 네임스페이스에 모델이 포함된 새 배포가 생성되고, `mlops-gitops` 저장소에 PR이 생성됩니다 👏 PR에 대한 자세한 내용은 다음 섹션에서 다룹니다.  
    ⚠️ 아직 PR을 승인하지 마세요 ⚠️