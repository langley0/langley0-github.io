---
title: [Typescript]  unknown 과 never 를 이용해 라이브러리 만들어보기
author: langley
---

Typescript 에서 타입을 다루는 많은 기법들이 있다. Intersction 과 Union 과 같은 일반적인 것부터, Type guards 과 Conditional 과 같이 문법자체가 낯선 것들도 있다. 중요한 것은 이런 많은 장치들이 `왜` 필요한가를 알아야 한다는 것이다. 특히 `unknown` 과 `never` 를 사용하기 위해서는 어떠한 상황에서 필요한지 알고 있으면 많은 도움이 된다.

Typescript 를 Javascript 처럼 쓰고 싶으면 모든 타입을 any 로 선언하면 된다. 바꾸어 말하면 any 를 사용하면 Typescript 를 쓰는 이유가 없어진다. Object 도 비슷하다. Object 타입으로 선언된 객체는 모든 클래스인스턴스에 대입이 가능하다. 정말 특별한 목적이 있다면 어쩔수 없지만 대부분의 경우에는 any 그리고 Object 는 타입선언을 번거로움을 피하기 위한 도피처로 선택하는 경우가 많다.

타입스크립트를 쓰는 이유는 타입체크를 통해서 오류를 실행전에 발견할 수 있고, 코드를 더욱 효율적이고 *예쁘게* 작성할 수 있기 때문이다. 
따라서 any 를 사용함으로 이런 이점을 버리는 것은 최대한 막아야 한다.

이를 위해서 `unknown` 과 `never` 를 활용하는 방법을 실제 코드사례를 통해서 알아보도록 하자.

# array null element optimize
배열중간에 null 데이터가 있는 경우가 있다. 이것을 특정한 함수를 통과하면 null 을 없앨 수 있을까?
``` typescript
const arr = [1, 2, null, 4, 5, null];

const arrWithoutNull = fn(arr);
console.log(arrWithoutNull); // [1, 2, 4, 5]
```

우선 상상할 수 있는 것은 filter 를 사용하는 것이다
``` typescript
function fn(src: any[]): any[] {
    return src.filter(item => item !== null);
}
```

자바스크립트였다면 이렇게만 만들어도 충분했을 것이다. 하지만 우리는 타입스크립트니까 any 를 없애보도록 하겠다.
우선 arr 의 타입은 `(number | null)[]` 이 된다. 이것을 입력으로 fn 함수를 다시 작성해보자
``` typescript
function fn(src: (number|null)[]): (number|null)[] {
    return src.filter(item => item !== null);
}
```
여기서 제네릭을 이용해서 좀 더 범용적인 함수로 업그레이드가 가능하다
``` typescript
function fn<T>(src: T[]): T[] {
    return src.filter(item => item !== null);
}
```
이제 이 함수는 `["a", "b", null, "d"]` 와 같은 입력도  처리할 수 있게 되었다.

이렇게 하면 끝난것 같지만 실제로 사용하려고 하면 코드가 예쁘지 않는 문제가 있다. 다음의 상황을 보자
``` typescript
const arr = [1, 2, null, 4, 5, null];
const numToStr = fn(arr).map(item => item.toString());

console.log(numToStr); // expect ['1', '2', '4', '5'] .. but error
```
위의 코드에서 `item.toString()` 에서 __*"객체가 null 일 수 있습니다"*__ 오류가 발생한다. `fn()` 의 리턴타입이 `(number|null)[]` 이기 때문에 item 역시 `number|null` 이 되며, null 을 포함한 변수에 아무런 보호장치 없이 접근하면 타입스크립트는 오류를 내기 때문이다. 하지만 우리는 결과값에 null 이 없다는 것을 알고 있다. 그렇다면 어떻게 이 함수를 구성할 수 있을까?

여기서 사용되는 것은 never 이다.
( Conditional Type 에 대해서는 여기서 설명하지 않는다. )
``` typescript
type NonNullable<T> = T extends null | undefined ? never : T;

declare function fn<T>(src: T[]): NonNullable<T>[];
```

자 실제 상황을 살펴보면 `type T = number|null` 이라고 하면NonNullable 은 다음과 같이 해석된다.
``` typescript
type NonNullable<number|undefine> = 
 (number extends null |undefine ? never : number => number)  | 
 (null extends null |undefine ? never : number => never) 
 
 = number | never = number;
```

`never` 는 그자체의 의미는 아무런 타입도 아니라는 뜻이다. void 와 혼동될 수 있는데, void 는 "빈값" 이라는 특수한 값을 담을 수 있는 타입이다. never 는 담을 수 있는 타입이 없는데, 이를 명확히 보여주는  예제를 준비하였다.

``` typescript
function ReturnVoid(): void {}

function ReturnNeverError(): never {} // 에러: 'never' 를 반환하는 함수에는 연결 가능한 끝점이 있을 수 없습니다

function ReturnNeverOk(): never {
    throw new Error();
    // 이 함수에는 return 이 없기 때문에 리턴타입은 never 가 된다 
} 
```
그리고 추가로 Union Type 규칙에 의해서 `never | void = void` 가 된다. (`never | T = T` 이기 때문에)


이제 함수의 내용을 채워보도록 하자
``` typescript
function fn<T>(src: T[]): NonNullable<T>[] {
    return src.filter(item => item !== null); // error: T[] 형식은 NonNullable<T>[] 에 할당할 수 없습니다
}
```
T 는 NonNullable<T> 보다 큰 집합이기 때문에 `NonNullable<T> => T : ok,  T => NonNullable<T> : error` 가 된다. 여기서 사용할 수 있는 것이 타입을 한정지어주는 타입가드이다
``` typescript
function isNonNullable<T>(src: T): src is NonNullable<T> {
  return (src !== null && src !== undefined);
}

let foo: Foo | null;
if (isNonNullable(foo)) { 
    // foo 는 null 이나 undefined 가 아닌값이다
    // 따라서 멤버 메소드를 호출할때 null 체크를 하지 않아도 타입오류가 발생하지 않는다
    foo.fn();
}
```

위에서 작성한 타입가드를 이용해서 실제 작성된 코드이다
``` typescript
function isNonNullable<T>(src: T): src is NonNullable<T> {
    return (src !== null && src !== undefined);
  }
  
function nonNullable<T>(array: T[]): NonNullable<T>[] {
    const initialValue: NonNullable<T>[] = [];
    return array.reduce((list, item) => {
        if (isNonNullable(item)) {
        list.push(item);
        }
        return list;
    }, initialValue);
}

const arr: (number | null)[] = [1, 2, null, 4, 5, null];
const arrWithoutNull: number[] = nonNullable(arr);
console.log(arrWithoutNull); // [1, 2, 4, 5]
```

어때요. 간단하죠?

# 숫자 배열의 총합
배열의 총합을 구하는 함수를 만들어보자
``` typescript
function sum<T>(array: T[]): number {
  // error : operator '+' cannot be applied to types 'number' and 'T'
  return array.reduce((prev, item) => prev + item, 0); 
}
```
안타깝게도 + 연산자를 T 에 적용할 수 없다면 오류가 난다. + 연산자는 미리 선언된 특별한 타입에만 적용가능하기 때문이다. 
이경우에 우리는 위에서 배운 타입가드로 해결할 수 있다

``` typescript
function sum<T>(array: T[]): number {
    return array.reduce((prev, item) => {
        return prev + (isNumber(item) ? item : 0)
    } , 0); 
}
  

function isNumber<T>(value: T): value is number {  // error: 
    // A type predicate's type must be assignable to its parameter's type.
    // Type 'number' is not assignable to type 'T'.
    // 'T' could be instantiated with an arbitrary type which could be unrelated to 'number'.(2677)
    return (typeof (value) === 'number');
}
```

T 를 number 로 변환하기 위해서는 number 가 T 의 서브셋이어야 하는데,  T 의 조건이 너무 광범위하기 때문에 이 상황을 특정지을수 없다라는 의미이다. 
일반적으로 이런 경우 any 를 사용할 수 있다
``` typescript
function isNumber(value: any): value is number {
    return (typeof (value) === 'number');
}
```
하지만 우리는 any 를 최대한 없애고 싶다. 그럴때 사용하는 것이 `unknown` 이다. 

`unknown` 은 any 대신에 코드를 좀 더 명확하게 만들어주는 역활을 한다. 다음 코드를 비교해보자
``` typescript
const x: any = 1;
const y: unknown = 2;

console.log(x.toString()); // '1'
console.log(y.toString()); // error: Object is of type 'unknown'
```

any 는 위와 같이 프라퍼티나 메소드를 사용하는데 컴파일러가 제한을 두지 않는데, unknown 은 프라퍼티나 메소드를 전혀 사용할 수 없게 지정한다. 이것은 값은 전파하되, 사용은 금지하는 효과를 가져오기 때문에 리팩토링 과정에서 any 와는 다르게 타입을 바꾸었을때 undefined 오류가 나지 않는 이점이 있다.

`unknown` 은 어느 타입으로나 변경이 가능한 장점이 있다
``` typescript
function prettyPrint(x: unknown): string {
  if (Array.isArray(x)) {
    return "[" + x.map(prettyPrint).join(", ") + "]"
  }
  if (typeof x === "string") {
    return `"${x}"`
  }
  if (typeof x === "number") {
    return String(x)
  } 
  return "etc."
}
```

`unknown` 을 이용해서 배열의 숫자들의 합을 구하는 함수를 완성하도록 하자
``` typescript
function isNumber(src: unknown): src is number {
  return (typeof (src) === 'number');
}

function sum<T>(array: T[]): number {
    return array.reduce((prev, item) => {
        return prev + (isNumber(item) ? item : 0)
    } , 0); 
}
console.log(sum([1, 2, null, 3])); // 6
```

again 참 쉽죠?