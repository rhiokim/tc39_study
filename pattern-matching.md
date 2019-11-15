# Pattern Matching

## Motiviation Example

```js
const res = await fetch(jsonService)
case (res) {
  when {status: 200, headers: {'Content-Length': s}} ->
    console.log(`size is ${s}`),
  when {status: 404} ->
    console.log('JSON not found'),
  when {status} if (status >= 400) -> {
    throw new RequestError(res)
  },
}
```

## Introduction

* [Destructuring binding patterns](https://tc39.es/ecma262/#sec-destructuring-binding-patterns) 를 기반으로 함