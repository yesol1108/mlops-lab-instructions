# 연습문제 8 - 지원 역할
> 연속 배포는 빠르고 신뢰할 수 있는 피드백이 필요합니다. 연속 테스트에 투자하는 것은 가치 있는 활동입니다.


## 👨‍🍳 연습 소개
이 연습에서는 필수 품질 보증 관행을 통합하여 연속 학습 파이프라인의 신뢰성과 보안을 향상시킬 것입니다. 코드 기능을 검증하는 단위 테스트를 추가하고, 코드 품질 유지를 위한 린팅 및 정적 코드 분석을 도입하며, 학습된 모델과 해당 컨테이너 이미지를 알려진 취약점에 대해 스캔할 것입니다.


## 🖼️ 큰 그림

![big-picture-quality-gate.jpg](./images/big-picture-quality-gate.jpg)

## 🔮 학습 목표
- [ ] 파이프라인에 보안 게이트 추가
- [ ] 파이프라인에 테스트 게이트 추가
- [ ] 파이프라인에 정적 코드 분석 게이트 추가
- [ ] Git에 비밀 정보를 안전하게 저장
- [ ] modelcar 이미지 스캔
- [ ] 파이프라인에 이미지 서명 추가
- [ ] SBOM 생성 및 저장

## 🔨 이 연습에서 사용하는 도구
 * <span style="color:blue;">[Sonarqube](https://www.sonarqube.org/)</span> - 파이프라인에 정적 코드 분석 추가.
* 코드 린팅 - <span style="color:blue;">[black](https://github.com/psf/black)</span>, <span style="color:blue;">[flake8](https://flake8.pycqa.org/en/latest/)</span>, <span style="color:blue;">[pylint](https://pypi.org/project/pylint/)</span> - 정적 코드 린터.
* 쿠버네티스 린팅 - <span style="color:blue;">[kube-linter](https://docs.kubelinter.io/#/)</span>, <span style="color:blue;">[helm lint](https://helm.sh/docs/helm/helm_lint/)</span> - K8s YAML을 모범 사례에 맞게 검증.
* 이미지 보안 - <span style="color:blue;">[Stackrox](https://www.redhat.com/en/technologies/cloud-computing/openshift/advanced-cluster-security-kubernetes)</span> - StackRox를 사용하여 이미지 및 호스트 내 취약점 탐지.
* ModelScan - <span style="color:blue;">[modelscan](https://github.com/protectai/modelscan)</span> - 모델에 안전하지 않은 코드가 포함되어 있는지 스캔.
* SealedSecrets - <span style="color:blue;">[sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)</span> - Secret을 안전하게 저장 가능한 SealedSecret으로 암호화.
* 이미지 서명 - <span style="color:blue;">[sigstore](https://www.sigstore.dev/)</span> - cosign으로 이미지 서명.
* SBOM - <span style="color:blue;">[Syft](https://github.com/anchore/syft)</span> - 컨테이너 이미지에서 소프트웨어 자재 명세서(SBOM) 생성.