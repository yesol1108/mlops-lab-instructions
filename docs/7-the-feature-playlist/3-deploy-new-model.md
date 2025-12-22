# 추론을 위한 Feast 사용법  

학습을 위한 피처를 조회하는 것 외에도, Feast는 추론 시 피처를 가져오는 데 사용할 수 있습니다. 이는 학습과 서빙에서 동일한 피처를 사용하여 불일치 위험을 줄이고 모델 성능을 향상시키는 일관성을 보장합니다.  

Feast를 통합한 새로운 모델 서버를 배포하기 전에, 먼저 **Feast 서버**와 **Feast UI**를 설정해 봅시다.  

Feast 서버는 피처 조회를 위한 인터페이스 역할을 하며, UI는 사용 중인 피처를 더 잘 확인할 수 있게 하여 피처 소비를 더 간단하고 투명하게 만듭니다. 이는 Feast 내부 루프에서 보았던 동일한 Feast UI이며, 이제 MLOps 네임스페이스에 배포하여 프로덕션 피처에 대한 UI도 갖추게 됩니다.  

모든 것을 배포해 봅시다!  

## Feast 서버 및 UI

1. `<USER_NAME>-mlops-toolings` 작업 공간(code-server)에서 `mlops-gitops/toolings` 아래에 `feast-server` 폴더를 생성합니다.
   
  ```bash
  mkdir /opt/app-root/src/mlops-gitops/toolings/feast-server
  touch /opt/app-root/src/mlops-gitops/toolings/feast-server/config.yaml
  ```

2. 다음 설정을 `mlops-gitops/toolings/feast-server/config.yaml`에 복사합니다:

  ```yaml
  repo_url: https://github.com/feast-dev/feast.git
  target_revision: v0.40.1
  chart_path: infra/charts/feast-feature-server
  feature_store_yaml_base64: cHJvamVjdDogbXVzaWMKcHJvdmlkZXI6IGxvY2FsCnJlZ2lzdHJ5OgogICAgcmVnaXN0cnlfdHlwZTogc3FsCiAgICBwYXRoOiBwb3N0Z3Jlc3FsOi8vZmVhc3Q6ZmVhc3RAZmVhc3Q6NTQzMi9mZWFzdAogICAgY2FjaGVfdHRsX3NlY29uZHM6IDYwCiAgICBzcWxhbGNoZW15X2NvbmZpZ19rd2FyZ3M6CiAgICAgICAgZWNobzogZmFsc2UKICAgICAgICBwb29sX3ByZV9waW5nOiB0cnVlCm9ubGluZV9zdG9yZToKICAgIHR5cGU6IHBvc3RncmVzCiAgICBob3N0OiBmZWFzdAogICAgcG9ydDogNTQzMgogICAgZGF0YWJhc2U6IGZlYXN0CiAgICBkYl9zY2hlbWE6IGZlYXN0CiAgICB1c2VyOiBmZWFzdAogICAgcGFzc3dvcmQ6IGZlYXN0Cm9mZmxpbmVfc3RvcmU6CiAgICB0eXBlOiBmaWxlCmVudGl0eV9rZXlfc2VyaWFsaXphdGlvbl92ZXJzaW9uOiAyCg==
  ```

  base64 설정은 기본적으로 이전에 배포한 PostgreSQL 서버를 가리킵니다.

  <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
  <pre><code class="language-yaml">
  project: music
  provider: local
  registry:
      registry_type: sql
      path: postgresql://feast:feast@feast:5432/feast
      cache_ttl_seconds: 60
      sqlalchemy_config_kwargs:
          echo: false
          pool_pre_ping: true
  online_store:
      type: postgres
      host: feast
      port: 5432
      database: feast
      db_schema: feast
      user: feast
      password: feast
  offline_store:
      type: file
  entity_key_serialization_version: 2
  </code></pre></div>

3. UI용 폴더를 하나 더 생성합니다:
   
  ```bash
  mkdir /opt/app-root/src/mlops-gitops/toolings/feast-ui
  touch /opt/app-root/src/mlops-gitops/toolings/feast-ui/config.yaml
  ```

4. 설정을 `mlops-gitops/toolings/feast-ui/config.yaml`에 복사합니다:

  ```yaml
  chart_path: charts/feast-ui
  feast-feature-server:
      feature_store_yaml_base64: cHJvamVjdDogbXVzaWMKcHJvdmlkZXI6IGxvY2FsCnJlZ2lzdHJ5OgogICAgcmVnaXN0cnlfdHlwZTogc3FsCiAgICBwYXRoOiBwb3N0Z3Jlc3FsOi8vZmVhc3Q6ZmVhc3RAZmVhc3Q6NTQzMi9mZWFzdAogICAgY2FjaGVfdHRsX3NlY29uZHM6IDYwCiAgICBzcWxhbGNoZW15X2NvbmZpZ19rd2FyZ3M6CiAgICAgICAgZWNobzogZmFsc2UKICAgICAgICBwb29sX3ByZV9waW5nOiB0cnVlCm9ubGluZV9zdG9yZToKICAgIHR5cGU6IHBvc3RncmVzCiAgICBob3N0OiBmZWFzdAogICAgcG9ydDogNTQzMgogICAgZGF0YWJhc2U6IGZlYXN0CiAgICBkYl9zY2hlbWE6IGZlYXN0CiAgICB1c2VyOiBmZWFzdAogICAgcGFzc3dvcmQ6IGZlYXN0Cm9mZmxpbmVfc3RvcmU6CiAgICB0eXBlOiBmaWxlCmVudGl0eV9rZXlfc2VyaWFsaXphdGlvbl92ZXJzaW9uOiAyCg==
      feast_mode: ui
  ```

  동일한 설정으로 PostgreSQL을 가리켜 레지스트리에 저장된 피처를 시각화합니다.

5. 변경 사항을 커밋하고 푸시합니다:
 
  ```bash
  cd /opt/app-root/src/mlops-gitops
  git pull
  git add .
  git commit -m  "🎁 ADD - Feast Server and UI 🎁"
  git push
  ```

6. Argo CD가 변경 사항을 동기화한 후, 다음 명령어로 Feast UI 경로를 확인하거나, [여기](https://feast-ui-<USER_NAME>-toolings.<CLUSTER_DOMAIN>)를 클릭하세요 :)  

  ```bash
  echo https://$(oc get route feast-ui --template='{{ .spec.host }}' -n <USER_NAME>-toolings)
  ```

 ![feast-ui.png](./images/feast-ui.png)

## 새로운 모델 서버 배포  

이제 피처 스토어를 사용해 학습했으니, 동일한 피처 스토어를 사용해 모델을 서빙할 차례입니다. 이전 장과 마찬가지로, 추론 시 Feast에서 피처를 조회하기 위해 **KServe 트랜스포머**를 사용합니다.  

새 모델을 배포하기 위해, 테스트 모델이 추론 시 피처 조회를 위해 **Feast 트랜스포머**를 가리키도록 구성할 것입니다. 기본 모델은 방금 학습한 것과 동일하므로 변경할 필요가 없습니다.  

서빙 파이프라인에서 Feast 트랜스포머를 사용함으로써, 추론 시 사용되는 피처가 학습 시 사용된 피처와 정확히 일치하게 됩니다. 배포를 시작해 봅시다!  

1. 먼저 최신 변경 사항을 가져옵니다. 모델 버전이 **mlops-gitops** 저장소에서 업데이트되었을 가능성이 높습니다.  
   
  ```bash
  cd /opt/app-root/src/mlops-gitops
  git pull
  ```

2. 그런 다음 테스트 모델의 설정 파일(`mlops-gitops/model-deployments/test/jukebox/config.yaml`)을 다음과 같이 업데이트합니다: 
   
  ```bash
  sed -i 's|chart_path: charts/model-deployment/music-transformer|chart_path: charts/model-deployment/music-transformer-with-feast|' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox/config.yaml
  sed -i '$a feast_server_url: http://feast-server-feast-feature-server.<USER_NAME>-toolings.svc.cluster.local:80' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox/config.yaml
  sed -i '$a feature_service: serving_fs' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox/config.yaml
  sed -i '$a entity_id_name: spotify_id' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox/config.yaml
  ```

  최종적으로 다음과 같은 내용이 되어야 합니다:

  <div class="highlight" style="background: #f7f7f7; overflow-x: auto; padding: 10px;">
  <pre><code class="language-yaml">
  chart_path: charts/model-deployment/music-transformer-with-feast # 👈 업데이트됨
  name: jukebox
  version: 68b0c7cf1b
  image_repository: image-registry.openshift-image-registry.svc:5000
  image_namespace: <USER_NAME>-test
  autoscaling: true
  canary:
    trafficPercent: 0
  feast_server_url: http://feast-server-feast-feature-server.<USER_NAME>-toolings.svc.cluster.local:80 # 👈 새 항목
  feature_service: serving_fs # 👈 새 항목
  entity_id_name: spotify_id # 👈 새 항목
  </code></pre></div>

  모델 서버를 Feast에서 온라인 피처를 사전 처리 단계로 가져오는 새로운 트랜스포머를 사용하도록 변경했습니다. 이로써 모델 입력은 값 목록이 아닌 ID가 됩니다. 이 흐름에서는 모든 데이터가 Feast를 통해 전달되어 온라인 스토어가 항상 최신 노래를 보유하도록 하는 것이 중요합니다.  
  또한 온라인 피처 요청을 보낼 수 있도록 Feast 피처 서버 엔드포인트를 지정했습니다.  
  마지막으로 다음을 추가했습니다:
  - 사용하려는 특정 Feature Service. Feature Service는 여러 피처를 그룹화하여 한 번에 반환합니다.
  - Entity ID. 이는 피처 값을 조회할 때 사용할 ID를 의미하며, 여기서는 노래의 spotify ID입니다.

3. 변경 사항을 git에 커밋합니다:

  ```bash
  cd /opt/app-root/src/mlops-gitops
  git add .
  git commit -m  "🦒 ADD - Use Feast with the served model 🦒"
  git push
  ```


## 새로운 Jukebox UI 배포 및 테스트

지금까지는 UI에서 노래 특성을 수동으로 입력하고 예측을 요청했습니다. 그러나 Feast가 통합되면서 이 과정을 단순화할 수 있습니다. 모든 피처를 입력하는 대신 노래 하나만 선택하면 필요한 피처가 자동으로 Feature Store에서 조회됩니다. 

모델 서버는 이제 개별 노래 피처가 아닌 노래 자체를 입력으로 기대하므로, UI를 업데이트하여 올바른 요청 데이터(`song ID`)를 보내 Feast가 선택한 노래에 적합한 피처를 조회할 수 있도록 해야 합니다.

1. 테스트 네임스페이스의 UI를 업데이트하려면 `mlops-gitops/model-deployments/test/jukebox-ui/config.yaml`을 열고 `image`를 다음과 같이 수정합니다:

    ```yaml
    ---
    repo_url: https://gitea-gitea.<CLUSTER_DOMAIN>/<USER_NAME>/jukebox-ui
    chart_path: chart
    model_endpoint: https://jukebox-<USER_NAME>-test.<CLUSTER_DOMAIN>
    model_name: jukebox
    image: quay.io/rhoai-mlops/jukebox-ui:feast-1.4 # 👈 update this
    search_as_default: true # 🧸 add this
    ```

  또는 (귀찮은 분들을 위해 ;)) 터미널에서 다음 명령어를 실행할 수 있습니다:

  ```bash
  sed -i 's|image: quay.io/rhoai-mlops/jukebox-ui:transformer-1.6|image: quay.io/rhoai-mlops/jukebox-ui:feast-1.4|' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox-ui/config.yaml
  sed -i '$a search_as_default: true' /opt/app-root/src/mlops-gitops/model-deployments/test/jukebox-ui/config.yaml
  ```

2. 변경 사항을 git에 커밋합니다:
  
  ```bash
  cd /opt/app-root/src/mlops-gitops
  git add .
  git commit -m  "🧪 UPDATE - new UI 🧪"
  git push
  ```

3. Argo CD가 변경 사항을 동기화한 후 UI로 이동합니다. URL을 잊었다면 다음 명령어를 참고하세요. 이미 사이트가 열려 있다면 최신 코드 변경 사항을 반영하기 위해 페이지를 새로고침하세요:  
  
  ```bash
  https://jukebox-ui-<USER_NAME>-test.<CLUSTER_DOMAIN>
  ```

4. 이제 무작위 피처 기반 예측 대신, 오른쪽 상단의 **Search** 버튼을 클릭합니다:

![UI-search.png](./images/UI-search.png)


5. 인기 있는 노래(예: ABBA의 `Gimme! Gimme! Gimme!`)를 검색하고 선택합니다.

![search-song-prediction.png](./images/search-song-prediction.png)

사전 처리는 온라인 데이터베이스에서 해당 노래의 최신 피처 값을 조회하여 위치 예측에 사용합니다.  

이렇게 하면 언제든지 조회할 수 있는 최신 중요 피처 저장소를 유지할 수 있습니다.  
