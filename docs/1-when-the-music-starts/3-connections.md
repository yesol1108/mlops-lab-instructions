## 오브젝트 스토리지

오브젝트 스토리지는 대규모 비정형 데이터를 효율적으로 저장할 수 있는 유연하고 확장성 높은 방식으로,  
클라우드 스토리지나 콘텐츠 배포에서 널리 사용되며 **이번 실습에서도 핵심적인 역할**을 합니다.  
우리는 오브젝트 스토리지를 사용해 **데이터셋과 모델 아티팩트**를 저장할 예정입니다.

### 오브젝트 스토리지 살펴보기

> MinIO는 가장 널리 사용되는 오브젝트 스토리지 중 하나입니다.  
> 클라우드 네이티브 환경에 최적화되어 있어 실험용 인스턴스를 빠르게 구성할 수 있으며,  
> Amazon S3 API와 호환되고 RESTful HTTP API로 접근 가능해  
> 클라우드 네이티브 애플리케이션 및 자동화와의 연동도 매우 수월합니다.

1. 실습의 편의를 위해, 이미 개발 환경에 MinIO 인스턴스가 설치되어 있습니다.  
   UI를 통해 MinIO에 접속하면, **이미 4개의 버킷이 생성되어 있는 것**을 확인할 수 있습니다.  
   아래는 MinIO UI 접속 링크이며,  
   로그인 정보는 `<USER_NAME>` (username), `<PASSWORD>` (password) 를 사용합니다.

MinIO UI:  
[https://minio-ui-<USER_NAME>-jukebox.<CLUSTER_DOMAIN>](https://minio-ui-<USER_NAME>-jukebox.<CLUSTER_DOMAIN>)

![minio-ui.png](./images/minio-ui.png)

`models` 버킷은 **모델을 저장**하는 용도이며,  
`pipeline` 버킷은 **데이터 사이언스 파이프라인 아티팩트**를 저장하는 데 사용됩니다.  
나머지 두 개의 버킷(`data`, `data-cache`)은 조금 뒤에 활용할 예정이니, 잠시만 기다려주세요 🙂

---

## Connections

1. OpenShift AI Dashboard로 돌아가서  
   `Data Science Projects` > `<USER_NAME>-jukebox`로 이동하면  
   `Connections`라는 섹션이 보일 것입니다.

2. 이미 **4개의 Connection**이 생성되어 있는 것을 확인할 수 있습니다.  
   이 Connection들은 MinIO의 **엔드포인트와 버킷 정보를 담고 있는 객체**입니다.  
   각 Connection의 오른쪽에 있는 점 세 개(`⋮`)를 클릭한 뒤 `Edit`를 선택하면  
   상세 정보를 확인할 수 있습니다.

> Connection은 버킷 정보를 **환경 변수 형태로 워크벤치에 노출**해주기 때문에,  
> 코드에 직접 값을 하드코딩하지 않고도 손쉽게 접근할 수 있도록 도와줍니다.

![data-connections.png](./images/data-connections.png)

워크벤치를 생성할 때 `models` Connection을 선택했는데,  
이는 실험 단계에서 해당 버킷과 직접 상호작용할 예정이기 때문입니다.

---

🪄🪄🪄  
이제 여정을 시작하기 위한 필수 도구들이 모두 준비되었습니다.  
다음으로, **데이터셋 속으로 본격적으로 들어가 봅시다!**  
🪄🪄🪄