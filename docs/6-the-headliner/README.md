# Exercise 6 - The Headliner
> 머신러닝 시스템의 신뢰성과 확장성을 향상시키는 기법들.

## 👨‍🍳 Exercise Intro
이번 연습에서는 데이터와 예측에 대한 전처리 및 후처리를 다루고, 부하를 처리하기 위한 자동 확장 기능을 탐구하며, 안전하고 원활한 모델 롤아웃을 보장하는 카나리 및 블루-그린 배포와 같은 고급 배포 패턴을 소개합니다.

## 🖼️ Big Picture

![big-picture-advanced-deployment.jpg](./images/big-picture-advanced-deployment.jpg)

## 🔮 Learning Outcomes
- [ ] 카나리 배포 방식을 통해 두 가지 다른 버전 간에 트래픽을 점진적으로 전환하기
- [ ] 챔피언 모델을 프로덕션에 배포하기
- [ ] 라이브 요청에서 새로운 버전을 모니터링하기 위해 트래픽 미러링하기

## 🔨 Tools used in this exercise
* <span style="color:blue;">[KServe](https://kserve.github.io/website/docs/intro)</span> - OpenShift 위에 구축된 모델 추론 플랫폼