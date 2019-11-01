# Nullish Coalescing Operator

status: stage 3
spec: https://github.com/tc39/proposal-nullish-coalescing

## What?

일종의 조건 연산자로 이미 많은 프로그래밍 언어가지고 있는 구문으로 삼항 연산자의 Sugar Syntax 로
사용되어지고 있습니다.

```
possiblyNullValue ?? valueIfNull
```

보통 이런 경우는 넘어오는 인자값(함수의 호출이나 서버의 응답값 중 일부의 필드)이 `null` 이나 `undefined` 일때
초기값을 설정하기 위해 사용됩니다.

Nullish Coalesching Operator 가 없을 때는 `||` 연산자를 통해서 다음과 같이 사용해 왔습니다.

```js
var possiblyNullValue = possiblyNullValue || defaultValue
```

## Practical

```js
var obj = {
  foo: "",
  bar: null,
  zoo: 0,
  fal: false
}
```

`||` 연산자의 경우 자바스크립트에서 거짓 값들을 의미하는 `0`, `undefined`, `""`, `false` 와 `null` 까지
모두 다음과 같은 결과를 얻게 됩니다.

```js
obj.foo || 'foo' // -> "foo"
obj.bar || 'bar' // -> "bar"
obj.zoo || 'zoo' // -> "zoo"
obj.fal || 'fal' // -> "fal"
obj.gar || 'gar' // -> "gar"
```

반면 Nullish Coalesing Operator 를 사용할 경우 다음과 같은 결과를 기대할 수 있습니다.

```js
obj.foo ?? 'foo' // -> ""
obj.bar ?? 'bar' // -> "bar"
obj.zoo ?? 'zoo' // -> 0
obj.fal ?? 'fal' // -> false
obj.gar ?? 'gar' // -> "gar"
```

즉 `null` 과 `undefined` 값의 경우에만 Coalesing 처리됩니다.

## References

* https://en.wikipedia.org/wiki/Null_coalescing_operator