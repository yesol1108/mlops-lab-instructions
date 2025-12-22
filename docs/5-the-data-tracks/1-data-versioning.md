# 데이터 버전 관리

데이터는 자주 업데이트되거나 수정되며, 모델을 학습하거나 평가할 때 어떤 버전을 사용했는지 알아야 재현성을 보장할 수 있습니다.

버전 관리가 없으면 문제를 디버깅하거나 결과를 일관되게 비교하기 어렵습니다. 이는 문서의 초안을 저장해 두었다가 변경된 내용을 확인하거나 필요 시 이전 버전을 복원하는 것과 같습니다.

이를 위해 DVC(Data Version Control)라는 오픈 소스 도구를 사용할 것입니다.  
DVC는 대규모 프로덕션 환경에는 적합하지 않지만, 시작하고 개념을 이해하는 데는 훌륭합니다.

먼저 내부 루프에서 DVC를 사용한 데이터 버전 관리를 살펴보겠습니다. 이후 파이프라인을 수정하여 데이터 버전을 자동으로 반영하는 외부 루프로 넘어갈 것입니다.

## (작은) ETL

데이터 버전 관리를 시작하기 전에, 데이터를 우리가 제어할 수 있는 곳에 저장해야 합니다. GitHub는 최대 저장 용량 제한 때문에 데이터 저장에 최적의 장소가 아니며, 클러스터 내에 저장하는 것을 원할 수도 있습니다.

이를 위해 데이터를 저장할 수 있는 S3 버킷을 만들고, 해당 버킷 내에서 데이터를 버전 관리할 것입니다.

먼저 데이터를 버킷으로 이동해 봅시다.

1. `OpenShift AI Dashboard` > `Data Science Projects` > `<USER_NAME>-jukebox` > `Workbenches`로 이동하여, `Standard Data Science` 노트북 이미지를 실행 중인 첫 번째 Jupyter 노트북 워크벤치에 연결합니다.

    `jukebox/5-data-versioning/1-data_pipeline_url_to_s3.py` 파일을 엽니다. 이 파이프라인은 현재 GitHub에 있는 노래 속성 데이터를 S3 버킷 `data`로 이동합니다.
    
    이후 파이프라인이 데이터를 자동으로 버전 관리하도록 할 예정이지만, 지금은 데이터를 옮기는 것만 진행합니다.

    ▶️ 실행 버튼을 클릭하여 파이프라인을 시작합니다.

    ![data-pipeline.png](./images/data-pipeline.png)

    이 작업은 `<USER_NAME>-jukebox` 프로젝트의 `OpenShift AI Dashboard` > `Experiments and runs`에서 확인할 수 있는 파이프라인을 트리거합니다.

    ![etl-pipeline.png](./images/etl-pipeline.png)

    성공적으로 완료될 때까지 기다립니다.

    ![etl-pipeline-2.png](./images/etl-pipeline-2.png)


## 데이터 버전 관리

이제 데이터가 S3 버킷에 저장되었으니, DVC를 사용하여 데이터셋을 추적하고 버전 관리하는 방법을 살펴보겠습니다. 제공된 Jupyter 노트북을 따라 아래 단계를 진행하세요:

1. S3 저장소 인터페이스로 이동합니다. 노트북에서 명령을 실행하면서 파일이 어떻게 추가되고 변경되는지 `data` 및 `data-cache` 버킷에서 확인할 수 있습니다: https://minio-ui-<USER_NAME>-jukebox.<CLUSTER_DOMAIN>

2. `jukebox/5-data-versioning/2-dvc-s3-track-remote-data.ipynb` 노트북을 엽니다. 지침에 따라 DVC 구성을 초기화하고 S3 버킷에 연결합니다.

3. 노트북 단계를 완료하면 `song_properties.parquet.dvc`라는 DVC 파일이 생성된 것을 확인할 수 있습니다. 이 파일은 데이터셋의 메타데이터를 추적하고 원격 저장소와 연결합니다.

4. 다음으로, 데이터 변경을 시뮬레이션하고 새 버전을 생성해 봅니다. `jukebox/5-data-versioning/3-dvc-s3-see-change.ipynb` 노트북을 열고 실행하여 데이터셋에 변경을 가하고 버전을 업데이트합니다.

완료하면 외부 루프에서 이 과정이 어떻게 진행되는지 계속 살펴보겠습니다 ⛷️