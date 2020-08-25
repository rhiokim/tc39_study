# Pipeline Operator

status: stage 3
spec: https://github.com/tc39/proposal-pipeline-operator

## What?

파이프 오퍼레이터 `|>` 는 유닉스의 [Pipe](https://en.wikipedia.org/wiki/Pipeline_(Unix)) 와 같이 이미 F#, OCaml, Elixir, Elm, Julia, Hack, LiveScripts 와 같은 언어들에서도 지원하고 있습니다.

그래서 TC39 제안에도 몇가지의 다른 스타일의 파이프 오퍼레이터가 제언되었고 현재 상호 경쟁중에 있습니다. [참고](https://github.com/tc39/proposal-pipeline-operator/wiki)

본질적으로 파이프 오퍼레이터는 하나의 인자를 갖는 함수호출 구문을 간결하게 표한 목적이 큽니다.

달리 말하면 `sqrt(64)` 는 `64 |> sqrt` 와 같습니다.

## Introduction

보통의 프로그래밍에서는 단순히 하나의 인자를 처리하는 함수가 여러개의 체이닝을 이루는 경우가 많습니다.
이런 경우 보통 코드의 가독성이 낮아지기 마련인데 아래의 예시처럼 파이프 오퍼레이터를 활용한다면 좀더 나은
가독성을 유지할 수 있습니다.

```js
function doubleSay (str) {
  return str + ", " + str;
}
function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}
function exclaim (str) {
  return str + '!';
}
```

파이프 오퍼레이터를 사용하면


```js
let result = exclaim(capitalize(doubleSay("hello")));
result //=> "Hello, hello!"

let result = "hello"
  |> doubleSay
  |> capitalize
  |> exclaim;

result //=> "Hello, hello!"
```

## Function with Multiple Arguments

인자가 여러개인 함수더라도 자바스크립트에서는 특별한 규칙없이 파이프 오퍼레이터를 이용할 수 있습니다.

```js
function double (x) { return x + x; }
function add (x, y) { return x + y; }

function boundScore (min, max, score) {
  return Math.max(min, Math.min(max, score));
}
```

```js
let person = { score: 25 };

let newScore = person.score
  |> double
  |> (_ => add(7, _))
  |> (_ => boundScore(0, 100, _));

newScore //=> 57

// As opposed to: let newScore = boundScore( 0, 100, add(7, double(person.score)) )
```

### Use of `await`

현재까지 TC39 의 최소한의 제안에는 파이프라인 내에서 `await` 를 지원하지 않기 떄문에
`|> await function() {}` 는 에러를 만듭니다. 각각의 제안에서 이 문제에 대한 솔루션을 제공하고 있어
곧 `await` 를 지원될 예정입니다.

### Usage with `?` partial application syntax

[partial appplication]() 스펙이 TC39 승인에 받아들여진다면 파이프 오퍼레이터는 더 간결하게
작성할 수 있게 됩니다. 이전 코드를 partial application 으로 작성하면 다음과 같습니다.

> partial application 은 함수의 인자를 고정하기 위한 사양으로 자세한 사항은 링크를 통해 스펙을 확인하세요.

```js
let person = { score: 25 };

let newScore = person.score
  |> double
  |> add(7, ?)
  |> boundScore(0, 100, ?);

newScore //=> 57
```

## Practical & Motivation

Iterables, AsyncIterables, Observable, Stream 프로그래밍에서는 흐르는 데이터를 컨슈밍하면서
순회 가능하고 스트림 형태의 객체로 변환하기 위한 구문없습니다.

```js
function* numbers() {
  yield 1
  yield 2
  yield 3
}
```

그런데 파이프 오퍼레이터를 사용하면 Iterable/Stream-like 객체를 읽기 쉬운 다단계 파이프라인으로 처리할 수 있습니다.

아래의 예시를 보시죠.

```js
const filter = predicate => function* (iterable) {
  for (const value of iterable) {
    if (predicate(value)) {
      yield value
    }
  }
}

const map = project => function* (iterable) {
  for (const value of iterable) {
    yield project(value)
  }
}

numbers()
  |> filter(x => x % 2 === 1)
  |> map(x => x + x)
```

## References

* https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Pipeline_operator
* https://github.com/babel/babel/tree/master/packages/babel-plugin-proposal-pipeline-operator
* https://docs.google.com/presentation/d/1eFFRK1wLIazIuK0F6fY974OIDvvWXS890XAMB59PUBA/edit#slide=id.p