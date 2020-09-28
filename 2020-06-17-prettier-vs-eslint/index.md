---
title: Pretter vs ESLint
author: langley
---

많은 프로젝트들이 일관성을 유지하기 위해서 스타일 가이드를 사용한다. [ESLint]와 [Prettier]는 개발자들이 일과성 있는 코딩 스타일을 유지하도록 도와주는 툴이다.이들은 기능면에서 접점이 있지만, 동시에 사용한다고 해도 충분히 좋은 코드를 만들수 있고, 도움이 된다. 

# [Prettier]

Prettier 는 어떻게 보면 고집스러운 코드 포매터이다. 장점은 코드 포맷팅에 대해서 두번 다시 신경쓰고 싶지 않을수있다. 이렇게 하고 싶다면 Prettier 를 에디터에 설치해서 포매팅을 처리하도록 하자. Pretter 는 코드를 [abstract syntax tree](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%EA%B5%AC%EB%AC%B8_%ED%8A%B8%EB%A6%AC)로 변환해서 주어진 포맷에 맞추어서 새로 코드를 기술한다. 이게 Prettier 가 하는 전부이다.

Prettier의 목적은 코드 스타일에 대해서 논쟁을 종결하는 것이다. 개발자들은 코드를 기술하는 동안에 스타일에 대해서 생각하지 않아도 되며, "실수" 를 찾기위해서 일관성없는 코드를 훑어볼 필요가 없다. 개발자는 코드를 작성하고, Prettier 가 포매팅을 해준다

# [ESLint]

ESLint 는 가장유명한 Javascript 린터이다, 린터는 코드오류를 분석해주는데, 스타일뿐만 아니라 버그가 될 수 있는 코드들도 같이 분석한다. Prettier 가 코드오류에 대해서는 할 수 있는 것이 거의 없는 것에 비해, ESLint는 이쪽으로는 매우 큰 도움이 된다.

예를 들어, ESLint는 선언없이 변수를 사용하면 경고를 한다. 혹은 개발자가 필요없는 할당을 해도 경고를 낸다. Prettier 에서는 당신이 작성한 포맷팅은 완전히 사라진다(Prettier 는 당신이 원하는 코딩 스타일을 반영하지 않는다). ESLint 는 자신이 생각하는 문제에 대해서 알려주고, -fix 옵션으로 수정작업을 진행하게 된다. 하지만 이과정은 Prettier 처럼 설정해놓으면 전혀 신경안써도 되는 그런것은 아니다.

# Using Prettier with ESLint

Prettier 와 ESLint 는 서로 보완적이지만, 스타일에 대해서 서로 일치되지 않으면 충돌이 발생한다. 나는 둘을 모두 사용하기를 강하게 추천하는데, 이러한 충돌을 없앨 수 있는 [사용가이드] (https://prettier.io/docs/en/integrating-with-linters.html)를 Prettier 에서 제안하고 있다 


*(Prettier 기준의 가이드 내용 추가)*
# Integrating with Linters

Prettier 는 기존의 린팅툴들과 같이 통합이 가능하다. Prettier 를 사용해서 코드 포맷팅을 하는동안, 다른 린터들은 코드 품질을 관리하도록 하는 것이다
(두가지 린팅에 대해서 [참고 자료](https://prettier.io/docs/en/comparison.html) 첨부)

만약 통합하고자 하는 린팅도구가 있다면 일반적으로 과정은 비슷하다. 우선 Prettier 가 워하는 코드 포맷과 충돌이 되지 않도록 해당 도구의 포맷팅 기능을 중지시켜야 한다. 혹은 Prettier 와 같이 코드  포맷팅을 하도록 할 수 도 있는데, Prettier 가 실행된 이후에 별도의 프로세스로 포맷팅을 하는 등의 방법이 있다.

## ESLint 와 같이 사용하기

### 포맷팅 룰 무효화
`[eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)` 는 Prettier 와 충돌화는 규칙들을 무효화시켜주는 설정파일ㅇ이다. 이것을 devDependencies 에 추가하고 `.eslintrc` 설정에 추가하면 된다. 

``` bash
npm install eslint-config-prettier --save-dev
```

`.eslint.json` 에 추가
``` json
{
    "extends": ["prettier"]
}
```
### Prettier 를 ESLint 에서 사용하기

`[eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier)` 는 Prettier 의 포맷팅을 ESlint 에서 쓸수 있도록 해주는 플러그인이다. 

``` bash
npm install eslint-plugin-prettier --save-dev
```

`.eslint.json` 에 추가
``` json
{
    "plugins": ["prettier"],
    "rules": {
        "prettier/prettier": "error"
    }
}
```

### 통합설정
`eslint-plugin-prettier` 에는 `eslint-config-prettier` 의 기능을 포함하고 있기 때문에 한번에 설정이 가능하다

우선 모듈을 설치한다
``` bash
npm install eslint-config-prettier eslint-plugin-prettier --save-dev
```

`.eslint.json` 에 추가
``` json
{
    "extends": ["plugin:prettier/recommended"]
}
```




[ESLint]: https://eslint.org/
[Prettier]: https://github.com/prettier/prettier