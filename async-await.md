# Top-level Await

status: stage 3
spec: https://github.com/tc39/proposal-top-level-await

## Top-level means?

`Top-Level Await` 은 `async` 함수밖에서 `await` 키워드를 사용할 수 있게 됩니다.

```js
const dynamic = await import('./file.js')
```

하지만 이전에는 `await` 만 별도로 사용하게 되면 `SyntaxError` 가 발생합니다.

그래서 async/await 는 쌍으로 함께하기 마련입니다.

```js
await Promise.resolve(console.log('🎉'));
// → SyntaxError: await is only valid in async function

(async function() {
  await Promise.resolve(console.log('🎉'));
  // → 🎉
}());
```

딱히 불편함이 없는게 사실입니다. `async () => { }` 으로 감싸버리면 되니까요.

그런데 왜? `Top-Level Await` 이 필요할까?

## Use Cases

* Dynamic dependency pathing

```js
const strings = await import(`/i18n/${navigator.language}`);
```

런타임에 의존성을 결정해야 할 경우 예를 들어 국제화에 필요한 리소스라던지, 개발환경과 프로덕션레벨에 따라
나눠야 할 것들 등에서 유용하게 사용될 수 있습니다.

* Resource initialization

```js
const conn = await dbConnector()
```

DB 클라이언트 모듈과 같이 기본적으로 비동기 형태로 동작이 요구되어지는 리소스 관리자 초기화 시에도 매우
유용합니다.

* Dependency fallback

```js
let jQuery
try {
  jQuery = await import('https://cnd-a.example.com/jQuery')
} catch {
  jQuery = await import('https://cdn-b.example.com/jQuery')
}
```

의존성 폴백처리 jQuery 라이브러리를 CDN-A 에서 로딩이 실패하면 CDN-B 에서 로딩하도록 시도합니다.

사실 위의 것들은 개인적으로 크게 와닿지는 않았지만 고개는 끄덕여지는 정도였습니다.

다음의 예시를 보시죠.

## Modules are Asynchronous

```js
exports.handler = async function(event, context) {
  console.log("EVENT: \n" + JSON.stringify(event, null, 2))
  return context.logStreamName
}

import { handler } from '.'

await handler(event, context)
```

위의 코드는 Node.js 로 작성된 AWS Lambda 함수 핸들러 입니다. 즉 모듈이기도 합니다.

만약 위와 같이 단일 모듈이 비동기적으로 동작하는 코드들이 빈번해지는 Serverless, Event Loop 환경에서는
유용해보입니다.

## Wrap up

특히 이런 비동기 처리가 많은 상황에서는 Async/Await 가 빛을 발하는데 Top-Level Await 가 가능해지므로써
비동기 로직의 재사용성이 높아지고 복잡한 비동기 로직이 많은 환경에서 코드의 가독성이 좋아지리라 봅니다.

stage3 단계로 모든 브라우저에 곧 안착되리라 봅니다.

## References

* v8: https://v8.dev/features/top-level-await
* 2 Minutes Standards: https://bkardell.com/blog/TopLevelAwaitIn2m.html
* footgun: https://gist.github.com/Rich-Harris/0b6f317657f5167663b493c722647221