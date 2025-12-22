## ML500 클러스터 설정

<p class="warn">
    ⛷️ <b>참고</b> ⛷️ - OpenShift 4.16+ 클러스터와 클러스터 관리자 권한이 필요합니다.
</p>

강의 전반에 걸쳐 실습하듯이, 클러스터 구성은 GitHub 저장소에 코드로 관리합니다: https://github.com/rhoai-mlops/deploy-lab

이 저장소는 세 부분으로 구성되어 있습니다:
- OpenShift AI 제품 설치를 위한 오퍼레이터 배포 Helm 차트
- 로깅 스택, 사용자 워크로드 모니터링 등을 구성하는 Helm 차트
- 마지막으로 학생 환경을 위한 Helm 차트


## 설치
첫 번째 단계는 기본 오퍼레이터를 설치하는 것입니다.

```bash
git clone https://github.com/rhoai-mlops/deploy-lab.git
cd deploy-lab/operators
helm dep up
helm upgrade --install ml500-base . --namespace ml500 --create-namespace
```

위 작업이 성공하면(최대 15분 정도 소요될 수 있음) 두 번째 설치 단계를 실행할 수 있습니다:

```bash
cd ../toolings
helm dep up
helm upgrade --install ml500-toolings . --namespace ml500 --create-namespace 
```
이 단계도 시간이 걸릴 수 있습니다 🙈

마지막으로 학생 콘텐츠를 배포합니다:

```bash
cd ../student-content
helm dep up
helm upgrade --install ml500-student-content . --namespace ml500 --create-namespace --set cluster_domain=<CLUSTER_DOMAIN> --set attendees=5 # number of users you want to create
```

클러스터 도메인을 모를 경우, 아래 한 줄 명령어로 확인할 수 있습니다:

```bash
oc get ingresscontroller default -n openshift-ingress-operator -o jsonpath='{.status.domain}'
```

## 설치 확인
UI를 통해 클러스터에 로그인하고 `htpasswd` 로그인으로 학생 사용자 이름과 비밀번호를 사용하세요. `<USER_NAME>` 및 `<USER_NAME>-toolings` 네임스페이스만 보여야 합니다.


## 필요한 링크 확인
OpenShift 콘솔, OpenShift AI 대시보드, Gitea 등의 필요한 링크는 이 페이지 우측 상단의 `Quick Links`에 포함되어 있습니다.

## Red Hat 제품 데모 시스템

현재 Red Hat 직원만 이용 가능합니다. [RHPDS](https://demo.redhat.com/catalog?search=ml500)에서 ML500 환경을 주문할 수 있습니다. 최신 OpenShift 및 ML500 워크숍 환경이 프로비저닝됩니다. 클러스터 크기, 사용자 수, 지역을 선택할 수 있습니다.

![images/tl500-order-rhpds.png](images/ml500-order-rhpds.png)