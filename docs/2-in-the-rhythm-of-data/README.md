# Exercise 2 - 데이터의 리듬에 맞춰
> Elyra 파이프라인과 Kubeflow Pipelines SDK (KFP)를 사용하여 학습 파이프라인 생성하기

## 👨‍🍳 연습 소개

이번 연습에서는 Elyra 파이프라인을 시작으로 이전 단계를 자동화하고, 이후에는 Kubeflow Pipelines (KFP)로 전환하여 자동화를 프로덕션 환경에 적용하는 방법을 다룹니다.

## 🖼️ 큰 그림

![big-picture](./images/big-picture.jpg)

## 🔮 학습 목표

- [ ] Elyra 파이프라인 생성하기
- [ ] KFP 및 데이터 사이언스 파이프라인 익히기
- [ ] 워크벤치에서 파이프라인 실행하기

## 🔨 이번 연습에 사용되는 도구
* <span style="color:blue;">[Elyra](https://elyra.readthedocs.io/en/latest/getting_started/overview.html)</span> - 노트북, 파이썬 스크립트, R 스크립트로 AI 파이프라인을 구축할 수 있는 파이프라인 시각 편집기 제공
* <span style="color:blue;">[Kubeflow Pipelines](https://www.kubeflow.org/docs/components/pipelines/overview/)</span> - 컨테이너를 사용하여 이식 가능하고 확장 가능한 머신러닝(ML) 워크플로우를 구축하고 배포하는 플랫폼