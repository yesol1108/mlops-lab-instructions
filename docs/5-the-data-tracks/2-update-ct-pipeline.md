## 데이터 버전 관리를 통한 지속적 학습 파이프라인 업데이트

데이터 버전 관리가 도입됨에 따라, 이제 `dvc` 버전 파일을 활용하도록 파이프라인을 개선할 수 있습니다. 이 버전 파일을 통합하면 파일이 업데이트될 때마다 파이프라인이 자동으로 실행됩니다. 이를 통해 새로운 데이터가 제공될 때마다 자동으로 새로운 모델이 빌드되어 모델 재학습 프로세스가 간소화되고 일관성이 유지됩니다.

먼저 모든 설정을 올바르게 하기 위해 약간의 준비 작업이 필요합니다.

### MinIO 설정

1. MLOps 환경에도 `data`와 `data-cache` 버킷이 필요합니다. 이전에는 내부 루프에서 이미 버킷이 존재했지만, MLOps 환경에서는 가능한 한 GitOps를 실천하기 위해 버킷 정보를 코드로 Git에 저장합니다. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)으로 돌아가서 `mlops-gitops/toolings/minio/config.yaml` 파일을 아래와 같이 업데이트하세요:

    ```bash
    chart_path: charts/minio
    buckets:
    - name: pipeline
    - name: models
    - name: data # 👈 add this
    - name: data-cache # 👈 add this
    ```

2. 변경 사항을 이전과 같이 리포지토리에 커밋합니다.

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🪣 data buckets added 🪣"
    git push
    ```

### CT 파이프라인 업데이트

1. Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)으로 돌아갑니다. 이제 DVC에 익숙해졌으니, 파이프라인을 업데이트하여 GitHub에서 모든 데이터를 가져오는 대신 `Jukebox` Git 리포지토리의 dvc 파일을 기반으로 노래 속성 데이터를 가져오도록 하겠습니다. 이를 위해 초기의 `fetch_data()` 함수를 주석 처리하고 dvc 명령어를 호출하는 새 함수를 도입해야 합니다.

    Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)에서 `jukebox/3-prod_datascience/prod_train_save_pipeline.py` 파일을 열고, 아래 줄 앞에 **＃**를 붙여 주석 처리하거나 해당 줄에 커서를 두고 CTRL (Command) + Shift를 누르세요.

    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-python">
    def training_pipeline(hyperparameters: dict, model_name: str, version: str, cluster_domain: str, model_storage_pvc: str, prod_flag: bool):
        ### 🐶 Fetches Data from GitHub
        fetch_task = fetch_data() # 👈 이 줄을 주석 처리하세요
    </code></pre></div>

2. `fetch_data()`를 주석 처리한 후, `### 🍇 Fetches data from DVC` 주석 바로 아래에 아래 함수를 붙여넣고 파일을 저장하세요!

    ```python
        ### 🍇 Fetches data from DVC
        fetch_task = fetch_data_from_dvc(
            cluster_domain = cluster_domain,
            git_version = version
        )
        kubernetes.use_field_path_as_env(
            fetch_task,
            env_name='namespace',
            field_path='metadata.namespace'
        )

        kubernetes.use_secret_as_env(
            fetch_task,
            secret_name='aws-connection-data',
            secret_key_to_env={
                'AWS_S3_ENDPOINT': 'AWS_S3_ENDPOINT',
                'AWS_ACCESS_KEY_ID': 'AWS_ACCESS_KEY_ID',
                'AWS_SECRET_ACCESS_KEY': 'AWS_SECRET_ACCESS_KEY',
                'AWS_S3_BUCKET': 'AWS_S3_BUCKET',
            },
        )
        kubernetes.use_secret_as_env(
            fetch_task,
            secret_name='git-auth',
            secret_key_to_env={
                'username': 'username',
                'password': 'password',
            },
        )
    ```

3. 변경 사항을 Git에 반영합시다. Jupyter Notebook에서 `Launcher`를 열고 `Terminal`을 선택하세요:

   ![open-terminal.png](./images/open-terminal.png)

   그리고 아래 명령어를 실행하세요.

    ```bash
    cd /opt/app-root/src/jukebox/
    git pull
    git add 3-prod_datascience/prod_train_save_pipeline.py
    git commit -m "🍇 fetch data via DVC 🍇"
    git push
    ```

4. 이 푸시는 학습 파이프라인을 트리거하지만, 파이프라인은 실패할 것입니다. 이유가 무엇일까요? 맞습니다! 이전에는 `Jukebox` 리포지토리에 dvc 설정 파일을 푸시하지 않았기 때문에 데이터 가져오기 단계에서 파이프라인이 실패합니다. 하지만 데이터가 변경될 때마다 dvc 파일을 수동으로 커밋하고 싶지 않습니다. 이를 자동화해야 하므로 또 다른 파이프라인을 도입해야 합니다.

### DVC 버전 관리를 활용한 데이터 파이프라인

1. Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)에서 `5-data-versioning/4-data_pipeline_with_dvc_versioning.py` 파일을 열고 ▶️ 버튼을 클릭하여 실행하세요.

    이번에는 파이프라인이 실행되는 대신 파이프라인 명세가 담긴 YAML 파일이 생성됩니다. `5-data-versioning/` 폴더에서 새로고침 버튼을 누르면 `song-properties-etl.yaml` 파일이 보일 것입니다.

    이 파일을 로컬로 다운로드하세요.

    ![data-pipeline-download.gif](./images/data-pipeline-download.gif)

2. OpenShift AI 대시보드에서 `Data science pipelines` > `Pipelines`로 이동한 후, 프로젝트를 `<USER_NAME>-toolings`으로 선택하고 `Import pipeline`을 클릭하세요.

    ![import-pipeline-1.png](./images/import-pipeline-1.png)

3. 파이프라인 이름을 `data-pipeline-with-dvc`로 지정하고, 방금 로컬에 다운로드한 YAML 파일을 `Upload` 버튼을 눌러 업로드한 후 `Import pipeline`을 클릭하세요.

    ![import-pipeline-2.png](./images/import-pipeline-2.png)

   이 파이프라인은 이전 섹션에서 내부 루프 동안 수행했던 작업들을 자동화합니다: 데이터 가져오기, DVC 구성, S3에 데이터 저장, 노래 속성 데이터 버전 관리, 그리고 한 가지 추가 단계가 있습니다. 바로 DVC 버전 파일을 `Jukebox` 리포지토리에 커밋하는 것입니다.

    ![data-pipeline-steps.png](./images/data-pipeline-steps.png)

    이 커밋은 무엇을 할까요? 맞습니다, 지속적 학습 파이프라인을 트리거합니다 🎉

4. 이상적으로는 이 데이터 파이프라인을 주기적으로 실행되도록 설정하는 것이 좋습니다. 이렇게 하면 새로운 데이터가 제공될 때마다 파이프라인이 데이터를 처리, 변환, 버전 관리하고 업데이트된 데이터를 사용해 모델을 재학습할 수 있습니다. 설정 방법을 살펴봅시다.

    `data-pipeline-with-dvc` 뷰에 있는 상태에서 오른쪽 상단의 `Action`을 클릭하고 `Create schedule`을 선택하세요.

    ![schedule-run-1.png](./images/schedule-run-1.png)

    **스케줄 세부사항:**

    - 이름: `data-pipeline-with-dvc-daily`
    - 트리거 유형: `Periodic`
    - 실행 주기: `1 Day`

    나머지는 기본값으로 두고 `Parameters` 섹션으로 이동하세요.

    **파라미터:**

    - dataset_url:

      ```
      https://github.com/rhoai-mlops/jukebox/raw/refs/heads/main/99-data_prep/song_properties.parquet
      ```

    - repo_url:

      ```
      https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox.git
      ```

    ..그리고 `Create schedule`을 클릭하세요.

5. 예약된 실행은 `Experiments` > `Experiment and runs` > `Default` > `Schedules`에서 확인할 수 있습니다.

    ![view-scheduled-runs.png](./images/view-scheduled-runs.png)

6. 하지만 실행을 기다리지 말고 즉시 실행할 수도 있습니다. `Data science pipelines` > `Pipelines` > `data-pipeline-with-dvc`로 돌아가 오른쪽 상단의 `Actions`를 다시 클릭하고 `Create run`을 선택하세요.

    ![create-run.png](./images/create-run.png)

    비슷한 정보를 입력합니다:

    - 이름: `data-pipeline-with-dvc-adhoc-run`
    - dataset_url:

      ```
      https://github.com/rhoai-mlops/jukebox/raw/refs/heads/main/99-data_prep/song_properties.parquet
      ```

    - repo_url:

      ```
      https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox.git
      ```

    ..그리고 `Create run`을 클릭하세요. 파이프라인이 즉시 실행됩니다.

    ![create-run-2.png](./images/create-run-2.png)

7. 파이프라인이 완료되면 `Jukebox` Git 리포지토리를 확인하세요. `.dvc/config` 폴더와 `song_properties.parquet.dvc` 파일이 생성되어 있을 것입니다.

    ![gitea-dvc.png](./images/gitea-dvc.png)

    DVC 파일이 데이터 파이프라인에 의해 푸시되었으므로, 지속적 학습 파이프라인이 트리거되었음이 확실합니다. OpenShift 파이프라인을 확인해봅시다.

    OpenShift 콘솔 > Pipelines > `ct-pipeline`으로 이동하세요.

    ![pipeline-run-dvc.png](./images/pipeline-run-dvc.png)

    이제 코드 변경 및 알림 외에도 새로운 데이터가 있을 때마다 파이프라인이 실행됩니다!

8. 모델 레지스트리를 확인하면, dvc 구성과 이 버전의 머신러닝 모델을 빌드하는 데 사용된 데이터 버전을 확인할 수 있습니다.

    ![dvc-model-registry.png](./images/dvc-model-registry.png)