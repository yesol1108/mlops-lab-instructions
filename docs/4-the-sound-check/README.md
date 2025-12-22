# 연습 4 - 사운드 체크
> 모델이 오늘 어떤 것을 정확하게 예측한다고 해서 항상 정확하게 예측할 수 있다는 의미는 아닙니다. 머신러닝 모델은 데이터 패턴의 변화, 사용자 행동의 변화, 외부 환경의 변화 등 다양한 요인의 영향을 받을 수 있습니다. 지속적인 모니터링을 구현함으로써 이러한 변화를 사전에 식별하고, 모델 정확도에 미치는 영향을 평가하며, 최적의 성능을 유지하기 위해 필요한 조정을 할 수 있습니다.

## 👨‍🍳 연습 소개
이번 연습에서는 OpenShift의 모니터링 스택을 활용하여 배포된 모델에서 메트릭을 수집합니다. 여기에는 리소스 사용량, 요청 성공률과 같은 일반적인 메트릭뿐만 아니라 데이터 및 모델 드리프트와 같은 머신러닝 특화 메트릭도 포함됩니다. Grafana를 사용해 이 메트릭들을 시각화하고, 합리적인 임계값을 기반으로 알림을 설정하여 학습 파이프라인을 트리거함으로써 프로덕션 모델이 일관되게 기대 성능을 발휘하도록 합니다. 또한 모델에서 로그를 수집하여 Loki에 저장하고, OpenShift 로깅 컴포넌트를 활용해 시각화합니다.

## 🖼️ 큰 그림

![big-picture-monitoring.jpg](./images/big-picture-monitoring.jpg)

## 🔮 학습 목표

- [ ] 모델을 모니터링하고 알림을 생성하기 위한 필요한 도구 배포
- [ ] Prometheus 쿼리를 통해 모델과 TrustyAI의 메트릭 확인
- [ ] Grafana 설치 및 대시보드 생성하여 메트릭 시각화
- [ ] OpenShift 로깅 스택에 검색 인덱스 생성하여 로그 저장

## 🔨 이번 연습에서 사용하는 도구
* <span style="color:blue;">[Prometheus](https://prometheus.io/)</span> - 메트릭 저장 및 알림에 사용
* <span style="color:blue;">[Grafana](https://grafana.com/)</span> - 메트릭 시각화에 사용
* <span style="color:blue;">[Loki](https://grafana.com/oss/loki/)</span> - 확장 가능하고 고가용성의 멀티 테넌트 로그 집계 시스템
* <span style="color:blue;">[TrustyAI](https://trustyai-explainability.github.io/trustyai-site/main/main.html)</span> - AI 설명 가능성 도구