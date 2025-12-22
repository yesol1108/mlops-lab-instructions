# Exercise 3 - From Studio to Stage
> MLOps 소개: 머신러닝 워크플로우와 배포를 자동화하고 단순화하는 일련의 실천 방법입니다.

## 👨‍🍳 Exercise Intro
이번 실습에서는 지속적인 학습 파이프라인과 지원 도구들이 실행될 MLOps 환경을 구축합니다.

## 🖼️ Big Picture

![big-picture-pipeline.jpg](./images/big-picture-pipeline.jpg)

## 🔮 Learning Outcomes

- [ ] MLOps 개념에 익숙해지기
- [ ] 모델을 자동으로 빌드 및 배포하기 위한 필수 도구 배포하기
- [ ] 중요한 메타데이터 추적 이해하기

## 🔨 Tools used in this exercise
* <span style="color:blue;">[OpenShift GitOps](https://argoproj.github.io/argo-cd/)</span> - 애플리케이션을 지속적으로 모니터링하고 현재 상태를 원하는 상태와 비교하는 컨트롤러
* <span style="color:blue;">[OpenShift Pipelines](https://tekton.dev/)</span> - 클라우드 네이티브 CI/CD 도구로, 어디서든 빌드, 테스트, 배포 가능
* <span style="color:blue;">[Kubeflow Model Registry](https://www.kubeflow.org/docs/components/model-registry/)</span> - 머신러닝 모델 메타데이터를 위한 중앙 인덱스 제공