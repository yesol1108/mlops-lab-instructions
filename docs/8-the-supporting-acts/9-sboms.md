# SBOM 생성 및 증명

> SBOM은 Software Bill of Materials(소프트웨어 자재 명세서)를 의미합니다. SBOM은 특정 빌드에 포함된 내용을 반영합니다. 이는 소프트웨어 조직이 시장에 내놓고 사용하는 구성 요소에 대한 투명성과 가시성을 제공합니다. 기본적으로 소프트웨어가 포함하는 구성 요소 목록입니다. 이를 통해 애플리케이션 각 구성 요소의 보안 취약점을 추적하여 모든 것이 최신 상태이고 안전한지 확인할 수 있습니다.

SBOM은 수신한 코드가 실제로 릴리스된 코드임을 증명하거나 보증하는 증명이 없으면 큰 가치가 없습니다. 증명(attestation)은 프레디케이트(predicate)로 알려진 이벤트 또는 아티팩트의 무결성을 검증하는 데 사용되는 암호화 서명된 메타데이터입니다. 이 경우 SBOM이 프레디케이트이고, 증명은 SBOM 내 코드를 검증하는 메타데이터입니다. SBOM과 함께하는 증명은 빌드 프로세스의 일부로 생성되어야 하며, 이미지에 첨부되기 전에 SBOM이 변조되지 않았음을 보장합니다. <a href="https://next.redhat.com/2022/10/27/establishing-a-secure-pipeline/"><sup>[1]</sup></a>

이번 실습에서는 [Syft](https://github.com/anchore/syft)를 사용해 SBOM을 생성합니다. 그 후 `cosign`을 사용해 생성된 SBOM 문서를 이미지 메타데이터에 첨부하고, 서명과 인증서를 공개 [Rekor Server](https://rekor.sigstore.dev) 투명성 로그에 저장합니다. _개인 정보를 사용하지 마세요🫣_


## 시작 전에 키 생성하기


1. SBOM이 무엇인지 확인해봅시다:

    ```bash
    syft quay.io/rhoai-mlops/jukebox:latest
    ```
    다음과 같이 긴 목록 출력이 나와야 합니다:
    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    $ syft quay.io/rhoai-mlops/jukebox:latest
    ✔ Loaded image  quay.io/rhoai-mlops/jukebox:latest
    ✔ Parsed image  sha256:da60a37da67b030e5b67613c2f2085563e34176a6ce38ebd921ce9600d12f862
    ✔ Cataloged contents   9baf2da7474a4fca9ae4d48093ca39e46f7e9b709dc27b85a5de8b36b5213dab
    ├── ✔ Packages                        [106 packages]  
    ├── ✔ File digests                    [1,221 files]  
    ├── ✔ File metadata                   [1,221 locations]  
    └── ✔ Executables                     [262 executables]  
    NAME                    VERSION                        TYPE   
    alternatives            1.24-1.el9                     rpm     
    audit-libs              3.1.2-2.el9                    rpm     
    basesystem              11-13.el9                      rpm     
    bash                    5.1.8-9.el9                    rpm     
    bzip2-libs              1.0.8-8.el9                    rpm     
    ca-certificates         2024.2.69_v8.0.303-91.4.el9_4  rpm   
    ....
    </code></pre></div>

3. `cosign`은 이미지의 알려진 보안 관련 아티팩트를 나열하는 올인원 명령어를 제공합니다. 이 이미지를 확인하려면:

    ```bash
    cosign tree quay.io/rhoai-mlops/jukebox:latest
    ```

    이 이미지는 보안 관련 아티팩트가 첨부되어 있지 않음을 확인할 수 있습니다:
    <div class="slider" style="background: #f7f7f7">
    <pre><code class="slide">
    <pre><code class="language-bash">
    📦 Supply Chain Security Related artifacts for an image: quay.io/rhoai-mlops/jukebox:latest
    No Supply Chain Security Related Artifacts artifacts found for image quay.io/rhoai-mlops/jukebox:latest
    </pre></code>
    </code></pre></div>

이제 파이프라인에 SBOM 생성 및 증명 단계를 추가해 보겠습니다.

_이 단계는 외부 이미지 레지스트리를 사용하거나 클러스터 간 또는 공개적으로 이미지를 공유할 때 더 의미가 있습니다._

4. `mlops-gitops/toolings/ct-pipeline/config.yaml` 파일을 열고 `generate_sboms: true` 플래그를 추가하여 [해당 작업](https://<GIT_SERVER>/<USER_NAME>/mlops-helmcharts/src/branch/main/charts/pipelines/templates/tasks/generate-sboms.yaml)을 도입합니다.

    ```yaml
    chart_path: charts/pipelines
    USER_NAME: <USER_NAME>
    cluster_domain: <CLUSTER_DOMAIN>
    git_server: <GIT_SERVER> 
    alert_trigger: true 
    apply_feature_changes: true
    unit_tests: true
    linting: true 
    static_code_analysis: true
    static_code_analysis_secret: sonarqube-auth
    model_scanning: true
    image_scan: true
    image_signing: true
    generate_sboms: true # 👈 add this
    ```

5. 변경 사항을 저장소에 커밋합니다:

    ```bash
    cd /opt/app-root/src/mlops-gitops
    git pull
    git add .
    git commit -m "🦤 ADD - generate SBOMs step 🦤"
    git push
    ```
6. OpenShift 콘솔 > `<USER_NAME>-toolings` 네임스페이스의 Pipelines로 이동하여 작업이 파이프라인에 포함되었는지 확인합니다.

    ![sboms.png](./images/sboms.png)

7. 빈 커밋으로 파이프라인을 실행하여 변경 사항을 확인합니다:

    ```bash
    cd /opt/app-root/src/jukebox
    git commit --allow-empty -m "🦖 trigger pipeline for SBOM generation 🦖"
    git push
    ```

8. 작업이 성공적으로 완료되면 `OpenShift UI` > `Builds` > `ImageStreams` > `jukebox`로 이동합니다. `.sbom` 및 `.att`로 끝나는 태그가 보이며, 이는 SBOM 프레디케이트에 대한 증명이 첨부되었음을 나타냅니다. 이를 통해 SBOM은 증명 내에 서명되어(따라서 변조 방지) 소비자가 진위를 검증할 수 있습니다.

    ![sbomatt.png](./images/sbomatt.png)

9. 공개 키로 서명된 이미지를 검증해 봅시다. 이미지의 올바른 `VERSION`을 사용했는지 확인하세요. (이 경우 `c6575637d8`)

    ```bash
    export REGISTRY_AUTH_FILE=~/.docker/auth.json
    oc registry login
    cosign tree default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:c6575637d8 
    ```

    출력 예시는 다음과 같습니다:

    <div class="slider" style="background: #f7f7f7">
    <pre><code class="slide">
    <pre><code class="language-bash">
    📦 Supply Chain Security Related artifacts for an image: default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:c6575637d8
    └── 💾 Attestations for an image tag: default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:sha256-52dbdc446fa22b95e4ed13989a968b8672f39d0aab2b7a94670814461eeb1681.att
    └── 🍒 sha256:786d4e2aaa457b8218f68988d95d3ebb6d01d8bac160b7d98b733e73ec0b14e4
    └── 📦 SBOMs for an image tag: default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<USER_NAME>-test/jukebox:sha256-52dbdc446fa22b95e4ed13989a968b8672f39d0aab2b7a94670814461eeb1681.sbom
    └── 🍒 sha256:a43e16448b9568717051d2da079e7f9e2ce9d0d6af0afb181fda27793f1d459e
    </pre></code>
    </code></pre></div>