# GitOpsifying Feast

이제 워크플로우의 **외부 루프(outer loop)**에서 사용할 Feast를 설정해 보겠습니다. 간단한 PostgreSQL 데이터베이스를 구성하여 두 가지 용도로 사용합니다:  

- **레지스트리(Registry)**: 피처에 대한 정의와 저장 위치 등 메타데이터를 저장합니다.  
- **온라인 스토어(Online Store)**: 추론 시 실시간으로 피처를 조회할 수 있도록 합니다.  

시작해 보겠습니다!  

1. `<USER_NAME>-toolings` 네임스페이스에 Feast 데이터베이스를 생성하는 것부터 시작합니다. `<USER_NAME>-mlops-toolings` 워크벤치(code-server)로 전환한 후 `mlops-gitops/toolings` 아래에 `feast-database` 폴더를 만듭니다.

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/feast-database
    touch /opt/app-root/src/mlops-gitops/toolings/feast-database/config.yaml
    ```

2. `feast-database/config.yaml`에 아래 설정을 추가하여 Argo CD가 Helm 차트를 어디서 찾을지 지정합니다.

    ```yaml
    chart_path: charts/feast-database
    USER_NAME: <USER_NAME>
    git_server: <GIT_SERVER>
    ```

3. 변경 사항을 커밋하고 푸시하세요. Git에 없으면..😉

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m  "🍕 ADD - Feast Database 🍕"
    git push
    ```

## 학습 파이프라인에서 Feast 활용하기

이제 Feast를 학습 파이프라인에 통합해 보겠습니다! 이를 통해 모델 학습에 필요한 특정 피처를 Feast에서 직접 요청할 수 있습니다.

Feast와 DVC가 모두 포함된 전체 흐름은 다음과 같습니다:

![feast-dvc-diagram.png](./images/feast-dvc-diagram.png)

DVC는 데이터를 추적하고 버전 관리를 하지만, Feast는 피처 정의를 생성, 관리, 사용하기 위해 데이터를 가져오는 역할을 합니다.

이렇게 하면 학습 과정에서 DVC를 통해 데이터를 가져오는 대신 Feast에서 피처를 조회하도록 전환할 수 있습니다. 시작해 봅시다!  

설정 방법:  

1. Jupyter Notebook 워크벤치를 엽니다.
2. `3-prod_datascience/prod_train_save_pipeline.py` 파일로 이동합니다.  
3. `### 🍇 Fetches data from DVC` 라벨이 붙은 섹션을 찾아 해당 섹션 아래 모든 줄을 주석 처리합니다. 
   
   > 🪄 팁: 주석 처리할 줄을 모두 선택한 후 CTRL/Command + `/`를 누르세요.

   다음과 같이 보일 것입니다:

   <!-- ## ADD GIF HERE MAYBE? ## -->
    <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
    <pre><code class="language-python">
        ### 🍇 Fetches data from DVC
        # fetch_task = fetch_data_from_dvc(
        #     cluster_domain = cluster_domain,
        #     git_version = version
        # )
        # kubernetes.use_field_path_as_env(
        #     fetch_task,
        #     env_name='namespace',
        #     field_path='metadata.namespace'
        # )
        #
        # kubernetes.use_secret_as_env(
        #     fetch_task,
        #     secret_name='aws-connection-data',
        #     secret_key_to_env={
        #         'AWS_S3_ENDPOINT': 'AWS_S3_ENDPOINT',
        #         'AWS_ACCESS_KEY_ID': 'AWS_ACCESS_KEY_ID',
        #         'AWS_SECRET_ACCESS_KEY': 'AWS_SECRET_ACCESS_KEY',
        #         'AWS_S3_BUCKET': 'AWS_S3_BUCKET',
        #     },
        # )
        # kubernetes.use_secret_as_env(
        #     fetch_task,
        #     secret_name='git-auth',
        #     secret_key_to_env={
        #         'username': 'username',
        #         'password': 'password',
        #     },
        # )
    </code></pre></div>
    
4. 이제 `### 🛍️ Fetch Data from Feast` 아래에 다음 줄을 추가합니다:
   
    ```python
        ### 🛍️ Fetch Data from Feast
        fetch_task = fetch_data_from_feast(version=version)
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

    ```

    파일을 저장하는 것을 잊지 마세요!

5. 변경 사항을 Git에 반영합시다. 터미널을 열거나 `Launcher`에서 `Terminal`을 선택해 새 터미널을 엽니다:

   ![open-terminal.png](./images/open-terminal.png)

   그리고 아래 명령어를 실행하세요.

    ```bash
    cd /opt/app-root/src/jukebox/
    git pull
    git add 3-prod_datascience/prod_train_save_pipeline.py
    git commit -m "🛍️ fetch data via Feast 🛍️"
    git push
    ```

6. `Jukebox` 저장소에 푸시가 발생하면 파이프라인이 트리거됩니다. 이번에는 Feast를 사용하므로 `fetch-data` 작업 로그를 확인하여 제대로 작동하는지 검증할 수 있습니다.

   ![fetch-data-from-feast-pipeline.png](./images/fetch-data-from-feast-pipeline.png)