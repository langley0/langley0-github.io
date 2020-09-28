---
title: Spread Operator 사용시 주의점
author: langley
---

[Spread Operator](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax)를 사용할때 발생한 문제를 기록하여 팀원들과 공유함을 목적으로 한다.

Spread Operator를 우리말로 하면 전개 연산자가 되는데, 단어 자체가 주는 복잡함이 있어서 이 글에서는 스프레드 혹은 스프레드연산자 라고 부르도록 한다.


# 문법
스프레드 연산자는 ES5 이후의 문법을 사용하는 개발자라면 한번 쯤은 써본 경험이 있을 것이다 (이름은 몰랐다고 하더라도 말이다).
``` javascript
function sum(x, y, z) {
  return x + y + z;
}

const numbers = [1, 2, 3];

console.log(sum(...numbers));
// expected output: 6
```

위의 `[1,2,3]` 은 배열을 표현하는 리터럴 literal 이다. 스프레드는 이런 리터럴를 복사해주는 역활을 한다고 말 할 수 있다.

> literal 에 대해서 혹시 모를 수 있어서 넘어가면, 
> 자바스크립트에서 (사실 많은 언어도들이 같은 의미로 사용한다) 객체를 선언할 때 상수값으로 된 선언문을 리터럴literal 이라고 한다
> *적어놓은 그대로(=literal)* 변수를 생성한다는 의미로 자바스크립트는 문자열, 숫자, 불리언에 배열과 오브젝트도 리터럴로 선언이 가능하다
>
> `"literal"` = 문자열 리터럴,  `[1,2,3]` = 배열 리터럴, `{ a: 1, b: 2: c: 3 }` = 오브젝트 리터럴

좀더 정확한 의미를 기술하자면, 오브젝트 생성시에  `enumerable:true` 인 프로퍼티를 주어진 오브젝트에서 생성되는 오브젝트로 복사한다라는 뜻이다. 
( [스프레드/레스트 ECMA 정의](https://github.com/tc39/proposal-object-rest-spread) )

여기서 리터럴은 자바스크립트에서 인정하는 오브젝트 생성방식이기 때문에 스프레드가 가능하다. 그것이외에도 많은 부분에서 스프레드는 사용가능하다
예를 들어 위에 함수의 인자값을 스프레드로 사용했는데, 기본적으로 자바스크립트에서 함수의 인자값은 고정이 아니라 배열의 형태로 전달된다. 따라서 배열 스프레드를 이용해서 새롭게 배열을 생성해서 전달하는 형태로 인자값에 스프레드를 사용할 수 있게 된다. 단 타입스크립트에서는 Rest Operator (= three dot operator) 를 사용했을때에만 스프레드가 가능하며 일반적인 함수에서는 컴파일오류가 발생한다
``` typescript
// 타입스크립트에서 스프레드를 함수인자에 적용
function foo(var1: number, var1: number)) {}
function bar(...args: number[]) {}

foo(...[1,2]); // error
bar(...[1,2]); // success
```

스프레드를 사용하여 다음과 같은 코드를 작성할 수 있다.
``` javascript
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1 = [...arr1, ...arr2]; // arr1 은 이제 [0, 1, 2, 3, 4, 5]
arr2 = [...arr2, ...arr1]; // arr2 은 이제 [3, 4, 5, 0, 1, 2] 가 됨
```

``` javascript
var obj1 = { foo: 'bar', x: 42 };
var obj2 = { foo: 'baz', y: 13 };

var clonedObj = { ...obj1 };
// Object { foo: "bar", x: 42 }

var mergedObj = { ...obj1, ...obj2 };
// Object { foo: "baz", x: 42, y: 13 }
```

# Enumerable
위의 스프레드의 정의를 할 때 enumerable 프로퍼티를 복사한다고 했다. 스프레드를 사용할 때 문제점을 이해하려면 우선 이 속성을 알아야 한다.

오브젝트 프라퍼티는 여러 속성을 가지며 enumerable 은 그중 하나가 된다. 프라퍼티를 정의할때 사용되는 속성은 다음과 같다.

**configurable**
이 속성의 값을 변경할 수 있고, 대상 객체에서 삭제할 수도 있다면 true.
기본값은 false.

**enumerable**
이 속성이 대상 객체의 속성 열거 시 노출된다면 true.
기본값은 false.
데이터 서술자는 다음 키를 선택사항으로 가집니다.

**value**
속성에 연관된 값. 아무 유효한 JavaScript 값(숫자, 객체, 함수 등)이나 가능합니다.
기본값은 undefined

**writable**
할당 연산자로 속성의 값을 바꿀 수 있다면 true.
기본값은 false.
접근자 서술자는 다음 키를 선택사항으로 가집니다.

**get**
속성 접근자로 사용할 함수, 접근자가 없다면 undefined. 속성에 접근하면 이 함수를 매개변수 없이 호출하고, 그 반환값이 속성의 값이 됩니다. 이 때 this 값은 이 속성을 가진 객체(상속으로 인해 원래 정의한 객체가 아닐 수 있음)입니다.
기본값은 undefined.

**set**
속성 설정자로 사용할 함수, 설정자가 없다면 undefined. 속성에 값을 할당하면 이 함수를 하나의 매개변수(할당하려는 값)로 호출합니다. 이 때 this 값은 이 속성을 가진 객체입니다.
기본값은 undefined.


프라퍼티를 정의하는 방법은 여러가지가 있다.
우선 리터럴을 사용하는 방법이다
``` javascript
var obj = { key: "value" }
```
이것을 defineProperty 를 사용해서 동일하게 만들 수 있다
``` javascript
Object.defineProperty(obj, 'key', {
  enumerable: true,
  configurable: true,
  writable: true,
  value: 'value'
});
```

readonly 같은 특수한 속성 역시 프라퍼티 정의로 표현이 가능하다
``` javascript
class Foo {
    readonly key: "value";
}
```
이것은 다음과 같이 정의할 수 있다
``` javascript
Object.defineProperty(obj, 'key', {
  enumerable: true,
  configurable: false,
  writable: false,
  value: 'value'
});
```

그럼 enumerable 이 false 라면 어떤 일이 발생할까?
우선 `Object.keys()` 나 `Object.values()` 를 사용해서 프라퍼티에 엑세스할 수 없게 된다. 이렇게 되면 오브젝트 복사를 할때 누락되게 된다.
이런 속성이 유용하게 쓰이는 일이 자주있는데, 특히 옵저버패턴을 사용해서 오브젝트를 감시할 때, 오브젝트 복사에서 옵저버는 복사하고 싶지 않기 때문에 관련 항목들을 전부 `enumerable: false` 로 구성하게 된다.

# Sequelize 사용시 문제점
Sequelize 를 잘 모른다면 넘어가도 좋다

우선 Sequelize 는 Model 이라는 핵심 클래스가 있다. Model 은 데이터베이스와 동기화 되면서, 스키마와 값을 동시에 가지게 된다.
Model 을 동기화 하고 값을 읽어오는 코드를 살펴보자
``` javascript
const { Sequelize, Model, DataTypes } = require("sequelize");
const sequelize = new Sequelize("sqlite::memory:");

const User = sequelize.define("user", {
  name: DataTypes.TEXT,
  age: DataTypes.INTEGER,
  cash: DataTypes.INTEGER
});

const jane = User.build({ name: "Jane" });
console.log(jane.name); // "Jane"
```

위의 코드에서처럼 name, age, cach 는 모델에서 바로 해당 프로퍼티로 값을 가져올 수 있다.
그럼 다음의 코드를 이어서 보자
``` javascript
const jane = User.build({ name: "Jane" });
console.log(jane.name); // "Jane"

const cloneJane = { ... jane };
console.log(jane.name); // undefined
```

놀랍게도 name 은 복사되지 않는다!
이유는 name 이 `getter` 이면서 `enumerable: false` 로 되어 있기 때문이다. 
이유를 살펴보면 실제 데이터는 내부에 다른 오브젝트로 설정되어 있고, name 이라는 프라퍼티는 이 오브젝트에서 값을 읽어오도록 되어 있다. 하지만 스프레드를 사용하면 이 getter 함수는 복사되지 않으므로 name 에 접근하려고 하면 `undefined` 가 나오게 되는 것이다

# 종합적인 스프레드 사용시 주의점
스프레드가 복잡한 인스턴스에 적용하려고 하면 발생하는 문제를 살펴보자
``` javascript
class Foo {
  constructor() { this.var1 = "foo";  }
  foo() { }
}

const obj1 = new Foo();
const obj2 = { ...(new Foo()) };

console.log(obj1.var1, obj1.foo); // foo, foo() {}
console.log(obj1.var1, obj2.foo); // foo, undefined
```

기본적으로 함수는 프로토타입 링크를 통해서 할당된다. 하지만 이 프로토타입 링크는 복사대상이 되지 않기 때문에 발생하는 문제이다.
아래예시를 보면  오브젝트를 온전히 복사하기 위해서 스프레드만으로는 안된다는것을 알 수 있다.
``` javascript
console.log(obj1.__proto__ === obj2.__proto__); // false
obj2.__proto__ = obj1.__proto__
console.log(obj2.foo) // foo() {}
```
함수는 그래도 쉽게 파악이 되는데 문제는 getter 의 경우이다
``` javascript
class Foo {
  constructor() { this.var1 = "foo";  }
  get foo() { 
    return this.var1;
  }
}

const obj1 = new Foo();
const obj2 = { ...(new Foo()) };

console.log(obj1.foo); // "foo"
console.log(obj2.foo); // undefined
```

게터 getter 역시 함수이기 때문에 복사가 되지 않는다. 하지만 우리는 이것이 일반적인 프라퍼티인지 게터인지 사용시점에서는 구분이 되지 않는다.

# 결론
스프레드는 리터럴로 표현이 가능한 오브젝트에 대해서는 복사가 가능하지만 복작한 오브젝트에서는 그 결과를 예측하기 어렵다. 따라서 리터럴 선언이 명확한 경우에만 스프레드를 사용하도록 하자

