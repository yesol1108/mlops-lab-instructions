# 🎸 🎶 MLOps 실습 (AI500)

## 슬라이드 자료
슬라이드 자료는 실습 지침과 함께 제공됩니다.

👨‍🏫 👉 [발행된 슬라이드는 여기에서 확인하세요](https://rhoai-mlops.github.io/lab-instructions/slides/ai500-index.html) 👈 🧑‍💻

## 🪄 지침 맞춤 설정
페이지 상단의 박스에서 이 교육의 지침 내 값을 대체하여 렌더링할 수 있습니다. 페이지 상단의 박스에 팀 이름과 클러스터 도메인을 입력한 후 `save`를 누르기만 하면 됩니다. 이 값들은 사이트의 로컬 스토리지에 저장되어, 실수로 잘못 입력했을 경우 `clear`를 눌러 초기화할 수 있습니다.

* 팀 이름이 `biscuits`라면 첫 번째 박스에 해당 값을 입력하세요. 이 값은 네임스페이스 등 일부 항목에 접두사로 사용됩니다.
* 클러스터 도메인의 경우 OpenShift 도메인에서 `apps.*` 부분을 입력하세요. 예를 들어, 콘솔 주소가 <code class="language-yaml">https://console-openshift-console.apps.hivec.sandbox1243.opentlc.com/</code> 라면, `apps.hivec.sandbox1243.opentlc.com`을 입력하여 실습에 맞는 주소를 생성합니다.
* Git 서버는 선호하는 접근 가능한 Git 서버(GitHub, GitLab 등)를 사용할 수 있습니다. 강사가 제공할 수도 있습니다.
예를 들어, Git 서버 주소가 <code class="language-yaml">https://gitea.apps.hivec.sandbox1243.opentlc.com/</code> 라면, `gitea-gitea.apps.hivec.sandbox1243.opentlc.com`을 입력하여 실습에 맞는 주소를 생성합니다.

## 🦆 규칙
실습 진행 시 교체가 필요한 부분을 명확히 표시했습니다. 주요 교체 대상은 `<>` 안에 있는 내용이며 반드시 변경해야 합니다. 예를 들어, 팀 이름이 `biscuits`라면 지침 내 `<USER_NAME>`을 다음과 같이 바꿔야 합니다:
    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-bash">
    name: &lt;USER_NAME&gt;
    # ^ 이렇게 변경
    name: biscuits
    </code></pre></div>

복사 및 붙여넣기용 코드 블록이 많이 있습니다. 코드 블록 오른쪽에 마우스를 올리면 ✂️ 아이콘이 나타납니다.  
```bash
echo "like this one :)"
```  
복사 ✂️ 아이콘이 없는 블록은 복사하지 마세요. 해당 블록은 출력 결과나 yaml 파일을 검증하는 용도입니다.