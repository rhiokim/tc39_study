# New Set methods

status: stage 2
spec: https://github.com/tc39/proposal-set-methods

## What

현재까지 반영된 Set 객체에는 요소의 추가, 삭제와 같이 Map 이 가지고 있는 기본적인 조작만 가능한 수준입니다.

이 스펙 제안은 Set 객체의 집합을 조작하는 몇가지 method 가 추가하는 것입니다.

```js
const a = new Set(1, 2)
```

* 다른 언어: https://github.com/tc39/proposal-set-methods/blob/master/other-languages.md

### Set.prototype.intersection()

집합 A 와 집합 B 의 교집합을 반환합니다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([3, 4, 5])
console.log(setA.intersection(setB))
// -> Set {3, 4}
```

### Set.prototype.union()

집합 A 와 집합 B 의 합집합을 반환합니다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([3, 4, 5])
console.log(setA.union(setB))
// -> Set {1, 2, 3, 4, 5}
```

### Set.prototype.difference()

집합 A 와 집합 B 의 차집합을 반환합니다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([3, 4, 5])
console.log(setA.difference(setB))
// -> Set {1, 2}
```

### Set.prototype.symmetricDifference()

집합 A 와 집합 B 의 [대칭 차집합](https://ko.wikipedia.org/wiki/%EB%8C%80%EC%B9%AD%EC%B0%A8)을 반환합니다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([3, 4, 5])
console.log(setA.symmetricDifference(setB))
// -> Set {1, 2, 5}
```

### Set.prototype.isSubsetOf()

부분 집합의 관계의 진위 여부를 반환한다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([1, 2, 3, 4, 5])
const setC = new Set([2, 3])
const setD = new Set([4, 5, 6])
const setE = new Set([6, 7, 8])
const setF = new Set([1, 2, 3, 4])
const setG = new Set()
console.log(setA.isSubsetOf(setB))
// -> true
console.log(setA.isSubsetOf(setC))
// -> false
console.log(setA.isSubsetOf(setD))
// -> false
console.log(setA.isSubsetOf(setE))
// -> false
console.log(setA.isSubsetOf(setF))
// -> true
console.log(setA.isSubsetOf(setG))
// -> false
```

### Set.prototype.isDisjointFrom()

[서로소 집합](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%A1%9C%EC%86%8C_%EC%A7%91%ED%95%A9)의 진위 여부를 반환한다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([1, 2, 3, 4, 5])
const setC = new Set([2, 3])
const setD = new Set([4, 5, 6])
const setE = new Set([6, 7, 8])
const setF = new Set([1, 2, 3, 4])
const setG = new Set()
console.log(setA.isDisjointOf(setB))
// -> false
console.log(setA.isDisjointOf(setC))
// -> false
console.log(setA.isDisjointOf(setD))
// -> false
console.log(setA.isDisjointOf(setE))
// -> true
console.log(setA.isDisjointOf(setF))
// -> false
console.log(setA.isDisjointOf(setG))
// -> true
```

### Set.prototype.isSupersetOf()

`isSubsetOf` 메소드와 반대의 기능으로 부분 집합의 진위 여부를 반환한다.

```js
const setA = new Set([1, 2, 3, 4])
const setB = new Set([1, 2, 3, 4, 5])
const setC = new Set([2, 3])
const setD = new Set([4, 5, 6])
const setE = new Set([6, 7, 8])
const setF = new Set([1, 2, 3, 4])
const setG = new Set()
console.log(setA.isSupersetOf(setB))
// -> false
console.log(setA.isSupersetOf(setC))
// -> true
console.log(setA.isSupersetOf(setD))
// -> false
console.log(setA.isSupersetOf(setE))
// -> false
console.log(setA.isSupersetOf(setF))
// -> true
console.log(setA.isSupersetOf(setG))
// -> true
```

### Additional

유사한 데이터를 다루는 Immutable.js 라이브러리에는 Set 의 `union(...iterables)`, `intersection(...interables)` 의 경우
클래스 메소드이기도 합니다. 하지만 스펙 제안에 이 부분은 포함되어 있지 않지만 고려중이라고 합니다.

## Why

보통 컬렉션 데이터를 다루다보면 빈번하게 요구되어지는 유틸리티 메소드가 자바스크립트에서는 풍부하지 않습니다.
그래서 ECMAScript 표준화가 왕성해지기 전에는 lodash 와 같은 별도의 라이브러리에 의존해왔습니다.

현재에 와서 자바스크립트 언어의 변화는 모던 프로그래밍 언어의 특, 장점을 잘 수용해나가고 있으며 이 스펙 제안에서는
Set 객체를 사용할 때 빈번히 구현해서 사용하던 집합의 연산 및 관계 판단을 위한 몇가지 필수 메소드를 추가를 목표로 합니다.

## Practical

Set 객체는 자료형에 관계없이 원시값과 객체참조 모두 유일한 값을 저장할 수 있습니다.
다만 Set 객체는 순서를 보장하지 않고 데이터의 존재 유무만 중요합니다.

그래서 다음과 같은 상황에서 유용하게 사용할 수 있습니다.

> 어떤 웹 사이트에서 하루에 접속하는 사람들 수를 구하고자 합니다.
> 접속하는 IP를 세면 되겠죠.
> 근데 한 사람이 여러번 접속하면 한 IP가 여러번 찍힐 것입니다.
> 이건 한번으로 카운트 해줘야 제대로 된 접속자 수를 구할 수 있습니다.
> 이럴 때 쓰는게 Set 입니다.
> 그냥 수학에서 집합 이라고 보시면 됩니다.

* 인용: https://onsil-thegreenhouse.github.io/programming/java/2018/02/21/java_tutorial_1-23/

## Refereneces

* https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Set
* https://qiita.com/_likr/items/7f74632c64c3f0cd470e