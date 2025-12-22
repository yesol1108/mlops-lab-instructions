# 연습 1 - 음악이 시작될 때
> 실험을 시작해 봅시다!

## 👨‍🍳 연습 소개

이번 연습에서는 `데이터 사이언스 프로젝트`를 단계별로 진행하며, 우리가 구축하려는 모델에 필요한 도구들을 익힐 것입니다. 작업 환경을 만들고, 데이터셋을 탐색하며 실험을 시작합니다.

## 🖼️ 큰 그림

![empty-big-picture-empty](images/big-picture-empty.jpg)

## 🔮 학습 목표

- OpenShift AI와 그 구성 요소에 익숙해지기
- 작업 환경을 설정하고 실험 시작하기

## 🔨 이번 연습에서 사용하는 도구
* <span style="color:blue;">[Jupyter Notebook](https://jupyter.org/)</span> - 노트북, 코드, 데이터를 위한 웹 기반 인터랙티브 개발 환경
* <span style="color:blue;">[Minio](https://min.io/)</span> - 오픈 소스 객체 저장 시스템
* <span style="color:blue;">[Kubeflow Model Registry](https://www.kubeflow.org/docs/components/model-registry/)</span> - 머신러닝 모델 메타데이터를 위한 중앙 인덱스 제공
* <span style="color:blue;">[KServe](https://kserve.github.io/website/master/)</span> - OpenShift 위에 구축된 모델 추론 플랫폼