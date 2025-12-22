# TrustyAI를 활용한 설명 가능성

앞선 섹션에서는 TrustyAI를 사용하여 드리프트와 편향을 감지하고 이를 모니터링하도록 설정했습니다. TrustyAI는 또한 로컬 및 글로벌 모델 설명과 같은 다양한 책임 있는 AI 워크플로우를 위한 도구를 제공합니다. 이를 통해 특정 입력에 대해 특정 출력이 나오는 이유를 이해할 수 있습니다.

우리는 반사실 분석(counterfactual analysis)과 SHAP 값과 같은 다양한 방법을 적용하여 이러한 인사이트를 얻을 수 있습니다.

반사실 분석은 우리가 받은 답변과 다른, 원하는 답변을 얻기 위해 어떤 값을 입력해야 하는지 알려줍니다.

SHAP(SHapley Additive exPlanations) 값은 출력 결과에 가장 크게 기여한 입력 특징이 무엇인지 알려줍니다.

시작해 보겠습니다.

1. 필요한 모든 라이브러리와 구성이 이미 포함된 TrustyAI 워크벤치를 사용하겠습니다. 이렇게 하면 작업이 훨씬 수월해집니다. `OpenShift AI Dashboard` > `Data Science Projects` > `<USER_NAME>-jukebox` > `Workbenches`로 이동하여 `Create a Workbench`를 클릭하세요.

  원하는 이름을 선택하세요. 예를 들어 `trustyai-wb`와 같이 지정할 수 있습니다.

  Notebook Image 설정:

  - 이미지 선택: `TrustyAI`

  - 버전 선택: `2025.1`

  - 컨테이너 크기: `Small`

  - 클러스터 스토리지: 크기 `20 GB`로 새 영구 스토리지 생성

  - 연결 설정에서 `Attach existing connections` 선택
    
    드롭다운 메뉴에서 `models` 선택

  나머지는 그대로 두고 `Create workbench`를 클릭하세요.


2. 워크벤치가 준비되면 열고 Jukebox를 다시 클론하세요.

  ```bash
    https://<USER_NAME>:<PASSWORD>@<GIT_SERVER>/<USER_NAME>/jukebox.git
  ```

3. `jukebox/4-metrics/3-download-artifacts.ipynb` 노트북으로 이동하여 지침에 따라 학습 파이프라인 실행에서 필요한 아티팩트를 다운로드하세요. 이렇게 하면 우리가 학습하고 프로덕션에 배포한 모델에 대해 설명 가능성 분석을 수행할 수 있습니다.

4. 다음으로 `jukebox/4-metrics/4-counterfactuals.ipynb` 노트북으로 이동하여 원하는 출력을 얻기 위해 어떤 입력이 필요한지 확인하는 지침을 따르세요.

5. 그 후, `jukebox/4-metrics/5-SHAP.ipynb` 노트북을 열어 Jukebox 모델의 출력을 설명할 수 있습니다 :)