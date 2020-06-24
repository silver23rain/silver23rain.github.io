---
layout: post
title: "ES10 private method 적용해보기 "
date: 2020-06-24
author: Eunbi
tags: babel esmascript javascript eslint
cover: "https://miro.medium.com/max/1280/1*AYmSGc1mM-kZTElGpuaWIA.jpeg"
---

# ES10 private method 적용해보기 (+Babel)

이전에는 private 를 표현하기 위해 생성자 함수를 생성하여 public 하게 사용할 변수, 메서드들만 return 하는 방식으로 주로 사용했었다.

```javascript
const Test = function () {
    // private field
    name = "math";

    // private method
    testing = () => {
        console.log(name);
    };

    start = () => {
        testing();
    };

    getName = () => {
        return name;
    };

    setName = (newName) => {
        name = newName;
    };
    return {
        start,
        getName,
        setName,
    };
};

const test = new Test();
test.start();

test.setName("english");
console.log(test.getName()); //english

console.log(test.name); // undefined
test.testing(); // test.testing is not a function
```

ES10 (ES2019) 부터 `Class`에 `private field` ( [TC39 stage 3](https://ahnheejong.name/articles/ecmascript-tc39/))를 사용할 수 있다.

위 코드를 Class 형태로 바꾸면 이렇게 쓸 수 있다.

```javascript
class Test {
  #name = "math";

  #testing() {
			console.log(this.#name);
	}

	start() {
		this.#testing();
	}

	get name () {
		return this.#name;
	}

	set name = (newName) => {
		this.#name = newName;
	}
}

const test = new Test();
test.start();

test.setName("english");
console.log(test.#name);
console.log(test.name);
```

해당 코드를 그대로 사용하면 **Babel**이 번들링 하는 과정에서 Support for the experimental syntax 'classPrivateMethods' isn't currently enabled 이런 오류가 발생한다.

## **Support for the experimental syntax 'classPrivateMethods' isn't currently enabled** 오류 해결

1. [플러그인 설치](https://babeljs.io/docs/en/babel-plugin-proposal-private-methods)

```javascript
npm install @babel/plugin-proposal-private-methods --save -dev
```

2. babel.config.js 수정

```javascript
//babel.config.js
module.exports = {
    presets: ["@babel/preset-env"],
    plugins: ["@babel/plugin-proposal-private-methods"],
};
```

이렇게까지 하면 정상적으로 private method를 사용하고 제대로 번들링 된다🙂

---

현재 프로젝트에 ESLint도 함께 사용 중이여서 눈에 밟히는 메세지가 떴다.

> ESLint: Parsing error: This experimental syntax requires enabling the parser plugin: 'classPrivateMethods'

위 이슈가 `babel-eslint@11.0.0-beta.0` 에서 수정됐다는 [깃헙 이슈](https://github.com/babel/babel-eslint/pull/711#issuecomment-456203896)를 발견 해서

해당 버전 설치

```javascript
npm install babel-eslint@11.0.0-beta.0 --save -dev
```

해당 베타 버전을 설치 후... ESLint error는 아니지만 규칙이 어긋난다는 메세지가 출력된다... 😕

> ESLint: 'initialize' is not defined.(no-undef)

[11.0.0-beta.0 - Private class methods no-undef error · Issue #746 · babel/babel-eslint](https://github.com/babel/babel-eslint/issues/746)

beta.2 버전( 글 적는 일자 기준 최신)이 올라왔길래 설치해봤지만... 같은 eslint 메세지가 출력된다.

[beta.2 버전에서도 해당 이슈는 해결되지 않았다](https://github.com/babel/babel-eslint/issues/826)

babel-eslint@11 정식버전에서는 수정 되어 있길 바라며...
찝찝한 마무리

![안녕](https://lh3.googleusercontent.com/proxy/Nqrum3byFyaOnuphwcgiHdA8aTPfiI-vosTIMbgrb6r3SFThcWXtsOpLKyUK_9th57SY4H6pk9_dTZLnKMGW-Hu39plaCaNOEnjWm_ZAvh8ZLtw4d2tcxCQOAWdt620CbIzekkZd43hSOKsdC5vipx5_Xy0qsTqoqzrNFr-zuo_CQY3ijYvY)

---

**참고한 링크**

[JavaScript's New Private Class Fields](https://www.sitepoint.com/javascript-private-class-fields/)

[@babel/plugin-proposal-private-methods · Babel](https://babeljs.io/docs/en/babel-plugin-proposal-private-methods)

[How do I enable ESLint classPrivateMethods parser plugin?](https://stackoverflow.com/questions/53675785/how-do-i-enable-eslint-classprivatemethods-parser-plugin)
