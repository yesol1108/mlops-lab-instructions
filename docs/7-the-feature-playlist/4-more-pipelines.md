# 파이프라인에서 Apply 및 Materialize 적용하기

기능(feature)이 항상 최신 상태로 유지되고 즉시 사용할 수 있도록 하기 위해, 우리는 기능을 자동으로 **materialize** 해야 합니다. Materialization은 실시간 추론 또는 서빙을 위해 온라인 스토어에 기능 데이터를 제공 가능하도록 만드는 과정을 의미합니다.

기능을 업데이트할 때마다 수동으로 변경 사항을 적용하고 materialization을 트리거하는 것은 원하지 않습니다. 대신, 파이프라인 내에서 이 과정을 자동화하여 기능 변경 사항이 자동으로 적용되고 materialize되어 수동 개입 없이 바로 사용할 수 있도록 할 것입니다.

네, 맞습니다. _또 다른 파이프라인_ 이라는 뜻입니다. 😅

## 새로운 데이터 자동 Materialize  

새로운 데이터 포인트가 시스템에 도착하면, **온라인 기능 스토어**가 최신 기능 값으로 업데이트되어야 합니다. 이를 위해, 타임스탬프를 기준으로 새로운 데이터만 점진적으로 materialize하는 방식을 사용하여 모든 데이터를 다시 처리하지 않고도 materialize를 수행할 것입니다.

이를 구현하기 위해, 데이터가 **S3**에 업로드된 후 materialization 단계를 포함하도록 **ETL 파이프라인**을 업데이트할 것입니다.

가장 최신 데이터만 materialize함으로써, 불필요한 데이터 부하를 줄이고 기능 스토어를 지속적으로 최신 상태로 유지하며 최소한의 처리 오버헤드로 실시간 추론에 대비할 수 있습니다.

1. UI로 이동하여 `1D2SVYXZdtNfJCg8wZWvVz` (이 ID는 `Live Forever from Oasis`에 해당) 를 검색하세요. 아직 이 노래를 온라인 기능 스토어에 추가하지 않았으므로 아무 결과도 나오지 않아야 합니다.
   
    ![search-in-ui.png](./images/search-in-ui.png)

2. Jupyter Notebook `<USER_NAME>-hitmusic-wb` 작업 공간(Standard Data Science)으로 이동하여 `jukebox/7-feature_store/5-data_pipeline_with_materialize.py` 파일을 열고 실행 버튼을 누르세요.
    ![run-etl-pipeline.png](./images/run-etl-pipeline.png)

3. 그러면 `song-properties-etl.yaml`이라는 새 파일이 생성됩니다. 이 파일을 다운로드하세요 🗃️
   
4. `OpenShift AI` 대시보드에서 `Data science pipelines` > `Pipelines`로 이동한 후 `<USER_NAME>-toolings` 프로젝트로 들어갑니다. 그리고 `data-pipeline-with-dvc` 파이프라인을 클릭한 후 `Actions` -> `Upload new version`을 선택하세요.  
다운로드한 `song-properties-etl.yaml` 파일을 업로드합니다.
   
    ![import-new-version.png](./images/import-new-version.png)

5. 예약된 실행을 기다리지 않고 즉시 파이프라인 실행을 시작하려면 (Actions -> Create Run) 다음 설정을 사용하세요:

    - 이름: `data-pipeline-with-feast-adhoc-run`
    - repo_url:
    ```
    https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox.git
    ```
    - dataset_url:
    ```
    https://github.com/rhoai-mlops/jukebox/raw/refs/heads/main/99-data_prep/fav_new_song_properties.parquet
    ```

    ![run-etl-pipeline-2.png](./images/run-etl-pipeline-2.png)

6. 다시 UI로 돌아가서 `1D2SVYXZdtNfJCg8wZWvVz`를 검색하세요. 프론트엔드가 온라인 기능 스토어와 동기화되어 있지 않아 드롭다운은 나타나지 않지만, 이제 해당 노래에 대한 예측 결과를 확인할 수 있습니다!
   
    ![song-id-prediction.png](./images/song-id-prediction.png)

## 새로운 변경 사항 자동 적용

기능 스토어에 새로운 변경 사항을 적용하기 위해, Continuous Training 파이프라인에 기능 변경 여부를 확인하는 단계를 추가할 수 있습니다. 변경 사항이 있을 경우에만 학습을 시작하도록 하는 것입니다.

1. `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열어 다음과 같이 업데이트하세요:

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER>
    alert_trigger: true
    apply_feature_changes: true # 👈 add this
    ```

2. 변경 사항을 저장하고 리포지토리에 커밋하세요:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "✅ Features automatically applied ✅"
    git push
    ```

3. 테스트를 위해 한 가지 기능을 제거하고, UI를 통한 학습부터 추론까지 모든 과정이 정상 작동하는지 확인해봅니다.
     
    Jupyter Notebook에서 `jukebox/7-feature_store/feature_repo/feature_service.py` 파일로 이동하세요. 데이터 탐색 과정에서 `loudness` 기능이 다른 기능(`energy`)과 높은 상관관계를 보여 둘 다 필요 없다는 것을 확인했으므로 14번째 줄의 `loudness` 기능을 제거합니다. 변경 후 저장하는 것을 잊지 마세요!

    ```python
    from feast import FeatureService

    from features import song_properties

    song_properties_fs = FeatureService(
        name="serving_fs",
        features=[
            song_properties[[
                "is_explicit",
                "duration_ms",
                "danceability",
                "energy",
                "key",
                # "loudness", # 👈 change this to a comment
                "mode",
                "speechiness",
                "acousticness",
                "instrumentalness",
                "liveness",
                "valence",
                "tempo"
                ]]
        ]
    ) 
    
    ```
    > 참고: 새 버전을 생성할 수도 있지만, 이 경우에는 이렇게 수정하는 것이 더 간단하며 git이 자동으로 버전을 관리합니다.

4. 파일을 저장한 후 터미널에서 다음 명령어로 변경 사항을 git에 커밋하세요:
   
    ```bash
    cd /opt/app-root/src/jukebox/
    git pull
    git add 7-feature_store/feature_repo/feature_service.py
    git commit -m "🎺 Remove feature loudness 🎺"
    git push
    ```

5. 이제 Continuous Training 파이프라인이 완료될 때까지 기다리면 됩니다.  
   
    OpenShift 콘솔에서 `Pipelines` -> `Pipeline Runs`로 이동하여 파이프라인 진행 상황을 확인할 수 있습니다.  

    또한 OpenShift AI의 `Experiments` -> `Experiment and Runs` -> `training`에서 Kubeflow 파이프라인 상태도 확인할 수 있습니다.  

    OpenShift 파이프라인이 완료되면 새로운 모델이 배포되어 테스트할 준비가 된 상태입니다.

    ![apply-features-ct.png](./images/apply-features-ct.png)

6. Jukebox UI (https://jukebox-ui-<USER_NAME>-test.<CLUSTER_DOMAIN>)로 이동하여 노래를 **검색**해보세요. UI를 전혀 변경하지 않았음에도 불구하고 기능 변경 사항이 반영되어 이전과 다른 결과가 나오는 것을 확인할 수 있습니다. Feast가 기능 변경을 자동으로 처리해주기 때문입니다 👏👏