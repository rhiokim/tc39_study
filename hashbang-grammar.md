# Hashbang Grammar

* status: **stage3**
* [tc39/proposal-hashbang](https://github.com/tc39/proposal-hashbang)

## Hashbang (해쉬뱅) 은 무엇?

* [https://en.wikipedia.org/wiki/Shebang_(Unix)](https://en.wikipedia.org/wiki/Shebang_(Unix))

유닉스와 같은 운영체제에서 일반적인 텍스트 파일을 실행형으로 사용하기 위해서 보통 Hashbang 을 사용하는데
파일의 첫줄에 Hashbang 을 설정하고 파일을 실행하게 되면 운영체제는 프로그램 로더 메커니즘에 의해 해당 파일을
파싱하고 그 이후의 내용들 해당 프로그램의 첫번째 인자로 모두 넘기는 방식으로 동작합니다.

> In Unix-like operating systems, when a text file with a shebang is
> used as if it is an executable, the program loader mechanism parses
> the rest of the file's initial line as an interpreter directive.
>
> The loader executes the specified interpreter program, passing to
> it as an argument the path that was initially used when attempting
> to run the script, so that the program may use the file as input data. [8]
>
> For example, if a script is named with the path path/to/script, and it
> starts with the following line, #!/bin/sh, then the program loader is
> instructed to run the program /bin/sh, passing path/to/script as the
> first argument.
>
> In Linux, this behavior is the result of both kernel and user-space code.[9]

현재에선 마우스로 아이콘을 실행하는 방식으로 필요한 애플리케이션을 실행하지만 과거 GUI 가 없던 시절에는
텍스트 형태의 코드가 단일 애플리케이션의 역활에 가까웠고 그 파일마다 실행해주는 프로그램이 달랐기 때문에
이러한 메커니즘도 필요했었습니다.

#### 해쉬뱅 오해

2010년 즈음에 트위터에서 유저의 타임라인 부분을 Single Page Application 으로 구현하면서 이 해쉬뱅
메커니즘을 이용하였는데요. (다음년도에 없앰)

* [https://blog.outsider.ne.kr/698](https://blog.outsider.ne.kr/698)

`https://twitter.com/#!/rhiokim` 과 같이 브라우저의 `location.hash` 값 변경을 탐지해
필요한 사용자 데이터만 서버에서 요청하고 렌더링된 화면을 그대로 사용하는 방식을 취했었습니다.

이때는 SPA 구축을 위한 다양한 방법이 연구되고 있던 때라 인정하지만 이 문서에서 이야기하는 Hashbang Grammar 와는
사뭇 다릅니다.

### 예시

```bash
#!/bin/sh
#!/usr/bin/env python
#!/usr/bin/env node
```

## 언제 필요하지

Hashbang 은 이미 스펙문서에 정리된 내용처럼 CLI(Command Line Interface) 환경에서 필요하고 실상
브라우저에서는 불 필요한 요소에 가깝습니다. (다음 섹션을 보면 사실 이유가 담겨있음)

```bash
$ echo "console.log(1 + 1)" > sum
$ chmod +x sum
$ node sum
2

# 위와 같이 실행 커멘드 node 를 지정하면 해당 파일을 node 가 인터프리팅하게 됩니다.

$ ./sum
./sum: line 1: syntax error near unexpected token `1'
./sum: line 1: `console.log(1 + 1)'

# 위에서 chmod +x sum 으로 파일모드를 실행형으로 변경했지만 위와 같이 에러가 표시됩니다.
# 이유는 당연히 `console.log(1 + 1) 를 인터프리팅할 설정이 없기 때문에 운영체제는 기본으로 설정된
# 쉘로 console.log(1 + 1) 에 대한 처리를 요청하였을 것입니다.

$ echo '#!/usr/bin/env node' > sum
$ echo "console.log(1 + 1)" >> sum
$ ./sum
2

# 다시 위와 같이 node 를 Hashbang 으로 지정하면 원하는 정상적인 결과를 얻을 수 있습니다.
```

* 참고 https://gist.github.com/Leko/ea379ba259e90ea9d2b33ac0e8bf19ff

## 왜 tc39 에서 표준화

TC39 는 ECMAScript 표준화 단체이니까.

근데 Hashbang 은 운영체제가 인식하는 거 아닌가? 라는 의문이 살짝 스쳐가면서 다시 스펙문서를 보니

> A byte-order mark (BOM) character at the beginning of a source text will
> prevent a subsequent #! from being interpreted as a hashbang.
>
> However, web browsers strip this character as part of fetching external
> scripts or modules (in decode or UTF-8 decode respectively), which happens
> before the text is interpreted as ECMAScript

BOM 에 대한 설명은 생략하고 결국 하나의 `.js` 파일이 동작하는 환경이 시스템 레벨과 브라우저 레벨이다 보니
발생할 수 있는 이슈이고 즉 브라우저 벤더의 상황에서는 브라우저가 인터프리팅되기 전에 Hashbang 을 BOM 으로
처리되도록 하는 동작이 필요합니다.

## Node.js

V8 은 [7.4](https://v8.dev/blog/v8-release-74) 에 대응이 되었고 Node.js 12.12.0 기준
v8 7.7.299.13	버젼을 대응하고 있습니다.

## 개인적인 의견

처음 스펙을 봤을 때 이게 단순하게 Hashbang 이 포함된 CLI 모듈의 일부를 브라우저에서 동시에 사용할
상황들이 그렇게 많을까 생각했는데 아래 링크의 마지막 내용을 보니 실제 시스템 레벨의 핸들링을 Node.js 에
의존하거나 한다고 하면 가능한 시나리오가 존재할 수 있겠다 싶지만 그래도 그렇게 할까?

* 참고 https://gist.github.com/Leko/ea379ba259e90ea9d2b33ac0e8bf19ff