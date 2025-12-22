# 좋은 참고자료 목록

이 목록은 워크숍 내내 참고할 수 있는 좋은 참고자료와 읽을거리를 모아놓은 것입니다.  
모르는 단어가 있을 경우를 대비해 용어집도 함께 제공합니다.  

## 참고자료
- [ai-on-openshift](https://ai-on-openshift.io/) - Red Hat의 AI 스위트를 활용하여 OpenShift에서 AI 애플리케이션을 배포하고 관리하는 훌륭한 자료입니다.
- [ai-on-openshift/gitops](https://ai-on-openshift.io/odh-rhoai/gitops/) - Kubernetes yaml 파일을 통해 OpenShift AI 리소스를 생성하는 방법입니다.
- [Data Science Tutorial](https://www.w3schools.com/datascience/default.asp) - 실행 가능한 예제와 함께 Python을 사용한 데이터 과학 방법에 대한 코드와 가이드입니다.
- [How Neural Networks Work Playlist](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi) - 신경망의 기본 개념을 다루는 훌륭한 동영상 가이드입니다.
- [Tensorflow Playgrounds](https://playground.tensorflow.org/) - 작은 신경망에 대해 다양한 설정과 데이터를 실험해보며 작동 방식을 이해할 수 있는 Tensorflow Playground입니다.
- [Made With ML](https://madewithml.com/) - 머신러닝과 그 라이프사이클에 대한 광범위한 가이드입니다.
- [Google MLOps Blog](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) - MLOps 성숙도의 다양한 단계, 단계 간 전환 방법 및 각 단계에 포함된 내용을 다룬 블로그입니다.
- [Cookie Cutter Project Structure](https://cookiecutter-data-science.drivendata.org/) - 좋은 프로젝트 구조를 갖춘 데이터 과학 프로젝트를 초기화하는 도구입니다.
- [CNN Explainer](https://poloclub.github.io/cnn-explainer/) - 합성곱 신경망(이미지 등에 특화된 네트워크 유형)이 어떻게 작동하는지 이해할 수 있습니다.

## 용어집
- **MLOps** - AI 모델을 신뢰성 있고 효율적으로 구축, 배포 및 유지 관리하기 위한 실천 방법, 문화 및 도구입니다.
- **ETL (Extract, Transform, Load)** - 데이터를 수집(추출), 정제/변환(변형), 저장(적재)하는 과정입니다.
- **EDA (Exploratory Data Analysis)** - 데이터의 패턴과 문제를 이해하기 위한 분석입니다.
- **Data Feature** - 모델에 사용되는 개별적이고 측정 가능한 데이터 속성(예: 나이, 온도, 거래 금액)입니다.
- **Feature Store** - 데이터 피처를 중앙에서 관리하고 제공하는 시스템입니다.
- **Feature Engineering** - 모델 성능 향상을 위해 데이터 피처를 생성하고 수정하는 작업입니다.
- **Neural Network** - 여러 층의 "뉴런"으로 구성되어 복잡한 문제를 처리할 수 있는 머신러닝 모델입니다.
- **Hyperparameter Tuning** - 층 수, 학습률 등 모델의 일반 설정을 조정하여 성능을 개선하는 과정입니다.
- **Pipeline** - 데이터 처리, 모델 학습 및 배포를 자동화하는 일련의 단계입니다.
- **Training** - 데이터를 사용하여 머신러닝 모델을 학습시키는 과정입니다.
- **Inference** - 학습된 모델을 사용하여 예측을 수행하는 과정입니다.
- **Kubeflow** - Kubernetes에서 ML 워크플로우를 관리하기 위한 도구 모음입니다.
- **Argo CD** - 배포 자동화를 위한 GitOps 도구입니다.
- **Model Registry** - 모델의 다양한 버전을 추적하고 관리하는 시스템입니다.
- **Canary Deployment** - 전체 롤아웃 전에 소규모 그룹에 새 모델을 배포하는 방식입니다.
- **Shadow Deployment** - 사용자에게 영향을 주지 않고 새 모델을 병렬로 실행하는 방식입니다.
- **Data Drift** - 예측에 사용하는 데이터가 학습 데이터와 현저히 달라지는 현상입니다.
- **Bias Detection** - 모델 내 불공정하거나 의도치 않은 편향을 식별하는 과정입니다.
- **SHAP (Shapley Additive Explanations)** - 각 피처가 예측에 얼마나 기여했는지 설명하는 방법입니다.
- **Counterfactuals** - 입력값에 다양한 "가정" 변화를 적용해 원하는 방식으로 출력/예측이 바뀌는지 테스트하는 방법입니다.

## 사용 도구 문서
### AI 도구
- [OpenShift AI](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2-latest)
- [Kubeflow Pipelines](https://www.kubeflow.org/docs/components/pipelines/)
    - [Kubeflow Pipelines SDK](https://kubeflow-pipelines.readthedocs.io/en/sdk-2.12.0/)
    - [Kubeflow Pipelines Kubernetes SDK](https://kfp-kubernetes.readthedocs.io/en/kfp-kubernetes-1.4.0/)
- [TrustyAI](https://github.com/trustyai-explainability)
- [DVC](https://dvc.org/doc)
- [ModelScan](https://github.com/protectai/modelscan)
- [KServe](https://kserve.github.io/website/master/modelserving/control_plane/)
    - [OpenVINO](https://docs.openvino.ai/2025/index.html)
- [Feast](https://docs.feast.dev/)
### GitOps/DevOps 도구
- [Argo CD](https://argo-cd.readthedocs.io/en/stable/)
- [Tekton](https://tekton.dev/)
- [PyTest](https://docs.pytest.org/en/stable/)
- 린팅
    - [Black](https://black.readthedocs.io/en/stable/)
    - [Flake8](https://flake8.pycqa.org/en/latest/)
    - [KubeLinter](https://docs.kubelinter.io/#/)
    - [Helm Lint](https://helm.sh/docs/helm/helm_lint/)
- [Sonarqube](https://docs.sonarsource.com/sonarqube-server/latest/)
- [Gitea](https://docs.gitea.com/)
- [Grafana](https://grafana.com/docs/grafana/latest/)