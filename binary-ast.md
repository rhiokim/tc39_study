# 🔥 Binary AST

* status: stage1
* slide https://docs.google.com/presentation/d/12A0w7XuazDyhvSxqPcfXZCqV17iFJF6LOaNP-gtzifM/edit#slide=id.g3973b301fe_0_56

## Binary AST 는 무엇

> Normally parsing source text is fundamentally slow, but we could ship
> a pre-parsed AST (compliant to 262 syntax grammar) to make things fundamentally faster.

컴파일 언어와 마찬가지로 자바스크립트 엔진도 Text 기반의 소스가 동작하기 위한 몇 단계의 과정을 거칩니다.

```
url/main.js -> (http) -> main.js -> Parse -> AST -> Compilant -> Browser
```

아래 작은 `sum` 함수를 호출하는 코드는 Parse 되고 AST 로 나온 결과입니다.

```js
sum(1, 2)
```

Abstract Syntax Tree

```
ExpressionStatement {
  - expression: CallExpression {
    + callee: IdentifierExpression
    - arguments: [
      - BinaryEpxression {
        left: LiteralNumericExpression
        operator: "+"
        right: LiteralNumericExpression
      }
    ]
  }
}
```

## Parsing 자바스크립트

자바스크립트 파싱이라는 것은 흔히 인터프리팅 언어의 동작원리와 같이 캐릭터 단위로 읽어가면서 전체의 소스를
훑어야 한다는 것입니다.

오늘날의 Babel, Markdown 의 동작원리도 유사합니다.
Babel 의 경우 자바스크립트 를 이용해 최신 스펙의 자바스크립트를 구 버젼의 자바스크립트로 전환하거나
Markdown 문법의 텍스트를 HTML 로 변환하는 과정에서도 Text 전체를 탐색하며 AST 를 생성하고 이를
다시 표준화된 HTML 태그로 맵핑하여 변환하는 과정이 존재합니다.

아무튼 어떤 최척화된 방법을 고안하던 간에 전체 텍스트를 한번은 캐릭터 단위로 읽어야 한다는 것입니다.

그리고 이 역할은 브라우저가 모두 매번 처리해야 하는 비용입니다.

그래서 BinaryAST 는 브라우저가 소스를 파싱하는 단계의 작업량과 복잡도를 줄이기 위한 목적을 갖습니다.

```js
자바스크립트 -> Parse -> BinAST  ---> BinAST -> Compilant -> Browser
```

서버에서 BinAST 를 직접 내려받는다면 브라우저에서 AST 를 만들기 위한 Parsing 과정이 0를 수렴하게 되고
BinAST 는 자바스크립트 빌드 과정에서 브라우저에서 매번 들였던 비용을 해소시켜 줍니다.

Binary AST 의 또 하나의 이점은 애플리케이션의 구동 타이밍에 사용되지 않는 bit 는 패스하고 꼭 필요한
코드들만 파싱하게 할 수 있습니다. 이건 앱의 구동속도를 극적으로 끌어 올려줍니다.

## 속도 저해요소

웹 애플리케이션 성능 향상에 대한 도전과 시도는 꽤 다양한 파트에서 진행되고 있습니다.

로우레벨 디코딩, 레이턴시를 낮추기 위한 병렬 처리, 데이터 표시를 위한 최적의 새로운 포맷 그리고
네트웍 레이어에서의 향상을 위한 프로토콜의 발명과 향상 등 Binary AST 마찬가지로 이런한 일환중에 하나로
자바스크립트 애플리케이션이 서버에서 클라이언트로 딜리버리되고 브라우저에서 애플리케이션으로 구동되기 까지의
일련의 과정속에서 텍스트 타입의 코드가 v8 엔진에 의해서 해석되는 과정에 포커스하고 있습니다.

근데 자바스크립트 파서가 무한한 성능 향상을 얻는데 매우 많은 토큰의 모호함이 존재합니다.

> * 많은 인코딩 방법들, 유니코드 바이트 순서표시나 기타 사항 다루기
> * `/` 문자가 나눗셈 연산자인지, 주석의 시작인지, 정규식 표현인지를 알아내기
> * `(` 문자가 표현식의 시작인지, 함수 호출의 인자 리스트인지, 화살표 함수의 인자인지 알아내기
> * 문자열(각각 문자열 템플릿, 배열, 함수)이 어디서 멈추는지 알아내야 하는 모호성 문제
> * `let a` 선언이 유효한지, 소스 코드 뒷부분에 나올 수 있는 다른 `let a`, `var a` 혹은 `const a` 선언과 충돌나지 않는지 알아내기
> * `eval`을 만났을 때 `eval`의 4가지 의미에 대해서 결정하기
> * 지역변수의 실제 로컬범위 결정짖기
>
> https://ui.toast.com/weekly-pick/ko_20170911/ 발췌

안타깝게 이런점들은 처리의 동시성 측면에서 성능 향상에 대한 기회가 많은 반면에 구문 오류가 발생해야 하는 시점에
지연 구문분석을 통한 성능향상에 대한 기회는 상당히 제한을 받게 됩니다.

## 정확히 어떻게 동작?

## 정확히 우리가 얻을 수 있는거?

[HTTPArchive](https://httparchive.org/reports/state-of-javascript#bytesJs) 데이터에 따르면
웹 애플리케이션의 페이지당 자바스크립트 전체 평균 용량은 350KB 를 넘었고 페이지의 10%가 1MB 를 초과하고 있습니다.

특히나 인기있는 웹 사이트들은 빈번한 배포와 관련해 최초 실행시간은 매우 중요합니다.

* 웹 애플리케이션의 구동 시간의 최적화

## Benchmarking


* https://gist.github.com/Yoric/1d41cdf3715815d39032f0dbce31ed42

## 개인적인 생각

Binary AST 가 브라우저에 안정적으로 되면 브라우저는 서버로부터 필요한 리소스를 Binary 형태의 리소스를
HTTP 로 전달받게 된다.

거기에 더불어 웹 애플리케이션의 변화 주기는 매우 빠르고 빈번하다. 그때마다 리소스를 계속 받는게 아니라
변경된 diff 만 배포하는 전략이 필요하지 않을까?

로컬에서 Hot Module Replacement 전략을 취하듯이 HTTP 에서도 변경에 대한 Git 전략과 비슷한 형태가
될 수 있지 않을까?

물론 꼭 Binary AST 와 무관하게 현재에도 고민할 것 같은데

## 참고

* https://github.com/rwaldron/tc39-notes/blob/master/meetings/2018-05/may-24.md
* https://yoric.github.io/post/binary-ast-newsletter-1/
  * (ko) https://ui.toast.com/weekly-pick/ko_20170911/
* https://blog.cloudflare.com/binary-ast/