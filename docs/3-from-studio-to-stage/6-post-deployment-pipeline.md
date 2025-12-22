# 배포 후 파이프라인

테스트 환경이 배포 및 검증되고, 새로운 버전이 예상대로 작동하며 회귀가 발생하지 않는 것을 확인한 후, 자동으로 생성된 풀 리퀘스트(PR)를 수락하여 프로덕션 배포를 진행할 수 있습니다.

`mlops-gitops` 저장소에 `prod/config.yaml`을 대상으로 하는 PR이 생성됩니다. 이를 검토하려면 Gitea UI > `mlops-gitops` 저장소 > Pull Requests로 이동하세요. 다음과 같은 화면이 보일 것입니다. 하지만 아직 승인하지 마세요!

> ⚠️ 이 시점에서 **PR을 승인하지 마세요** ⚠️  PR을 병합하기 전에 다른 파이프라인을 추가해야 합니다 😉

![prod-pr-1.png](./images/prod-pr-1.png)

만약 PR을 승인하면, 새로운 모델 버전이 `<USER_NAME>-prod` 환경에 롤아웃되기 시작합니다. 이후에는 모델 레지스트리 메타데이터를 업데이트해야 합니다. 현재 레지스트리는 테스트 환경을 반영하고 있습니다:

![mr-test.png](./images/mr-test.png)

모델 레지스트리는 각 환경에 어떤 모델 버전이 배포되었는지 추적하는 진실의 소스 역할을 합니다. 이를 수동으로 업데이트하는 대신, 배포 후 파이프라인을 통해 자동화할 것입니다.

---

## 배포 후 파이프라인 설치

![tekton-post-deployment-pipeline.jpg](./images/tekton-post-deployment-pipeline.jpg)

1. `toolings` 아래에 `post-deployment-pipeline`이라는 새 폴더를 만들고 `config.yaml` 파일을 추가합니다. 수동으로 하거나 다음 명령어를 실행할 수 있습니다:

    ```bash
    mkdir /opt/app-root/src/mlops-gitops/toolings/post-deployment-pipeline
    touch /opt/app-root/src/mlops-gitops/toolings/post-deployment-pipeline/config.yaml
    ```

2. `post-deployment-pipeline/config.yaml`을 열고 다음 내용을 추가합니다:

    ```yaml
    chart_path: charts/post-deployment-pipeline
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    ```

3. 변경사항을 커밋하고 푸시하여 Argo CD가 자동으로 동기화하도록 합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🪑 ADD - Post deployment pipeline 🪑"
    git push
    ```

    동기화가 완료되면 Argo CD에서 파이프라인이 나타나는 것을 확인할 수 있습니다:

    ![post-deployment-pipeline.png](./images/post-deployment-pipeline.png)

    **참고:** Persistent Volume Claims(PVC)이 Argo CD에서 *Progressing* 상태로 남아있다면, OpenShift가 첫 번째 파이프라인 실행을 기다리며 Persistent Volume을 프로비저닝 중이기 때문입니다. 초기 실행 후 상태가 정상(녹색)으로 변경됩니다.

    OpenShift 대시보드에서도 파이프라인 생성 여부를 확인할 수 있습니다:

    ![post-deployment-pipeline-2.png](./images/post-deployment-pipeline-2.png)

4. 다음으로, 웹훅 URL을 얻으려면 다음 명령어를 실행하세요:

    ```bash
    echo https://$(oc -n <USER_NAME>-toolings get route el-post-prod-deploy-listener --template='{{ .spec.host }}')
    ```

5. 생성된 URL을 복사하여 Gitea에 웹훅으로 추가합니다:

    - `mlops-gitops` 저장소 > `Settings` > `Webhooks`로 이동
    - `Gitea`를 선택하고 웹훅을 추가합니다:

    ![post-deployment-webhook.png](./images/post-deployment-webhook.png)

6. 이제 열린 PR을 수락하고 파이프라인이 자동으로 시작되는 것을 확인하세요. **Create Merge Commit** 버튼을 클릭하여 PR을 병합합니다:

    ![prod-pr-2.png](./images/prod-pr-2.png)

    ![prod-pr-3.png](./images/prod-pr-3.png)

7. PR 병합은 배포 후 파이프라인을 트리거합니다. OpenShift에서 실행 상태를 모니터링할 수 있습니다:

    ![post-deployment-pipeline-run.png](./images/post-deployment-pipeline-run.png)

8. 파이프라인이 완료되면 모델 레지스트리 메타데이터가 자동으로 업데이트됩니다. 동일한 모델 버전이 `test`와 `prod` 환경 모두에서 실행 중이므로 두 레이블이 나란히 표시됩니다:

    ![mr-prod.png](./images/mr-prod.png)

이 워크플로우를 계속 개선하면서 파이프라인에 추가 단계가 포함될 예정입니다—많은 기대 부탁드립니다! 😊