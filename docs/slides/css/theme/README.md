## Dependencies

테마는 Sass를 사용하여 작성되어 모듈화를 유지하고 파일 간 반복 선택자 사용을 줄입니다. 진행하기 전에 reveal.js 개발 환경이 설치되어 있는지 확인하세요: https://revealjs.com/installation/#full-setup

## Creating a Theme

자신만의 테마를 만들려면 [/css/theme/source](https://github.com/hakimel/reveal.js/blob/master/css/theme/source)에서 ```.scss``` 파일을 복제하는 것부터 시작하세요. `npm run build -- css-themes` 명령을 실행하면 Sass에서 CSS로 자동 컴파일됩니다([gulpfile](https://github.com/hakimel/reveal.js/blob/master/gulpfile.js) 참고).

각 테마 파일은 다음 순서로 네 가지 작업을 수행합니다:

1. **[/css/theme/template/mixins.scss](https://github.com/hakimel/reveal.js/blob/master/css/theme/template/mixins.scss) 포함**
공유 유틸리티 함수들입니다.

2. **[/css/theme/template/settings.scss](https://github.com/hakimel/reveal.js/blob/master/css/theme/template/settings.scss) 포함**
템플릿 파일(4단계)에서 기대하는 사용자 정의 변수 집합을 선언합니다. 3단계에서 재정의할 수 있습니다.

3. **재정의**
기본 테마를 재정의하는 부분입니다. 변수 지정([settings.scss](https://github.com/hakimel/reveal.js/blob/master/css/theme/template/settings.scss) 참고) 또는 원하는 선택자와 스타일을 추가할 수 있습니다.

4. **[/css/theme/template/theme.scss](https://github.com/hakimel/reveal.js/blob/master/css/theme/template/theme.scss) 포함**
현재 정의된 변수에 기반하여 최종 CSS 출력을 생성하는 템플릿 테마 파일입니다.