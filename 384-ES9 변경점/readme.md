`ES9`는 정규식과 비동기 작업에 대한 새로운 기능이 주요 변경점입니다. 기존의 `Iterator`와 이번에 나온 `Async Iterator`와의 차이점을 숙지해야 합니다.

---

**톺아보기 :**

-   `Async`
    -   `Promise.prototype.finally()`
    -   `Async Iterator`
        -   `for-await-of`
        -   `Symbol.asyncIterator`
-   `String`
    -   `Lifting Template Literal Restriction`
-   `Object`
    -   `New Features`
        -   `Rest Properties`
        -   `Spread Properties`
-   `RegExp`
    -   `Named Capture Groups`
    -   `Unicode Property Escapes`
    -   `Lookbehind Assertions`
    -   `s flag (dotAll)`

---

---

# Async

## Promise.finally()

이행되든 거절되든, 반드시 실행해야 할 로직을 정의할 수 있습니다. `finally` 로직에서 익셉션이 발생되면 `rejected` 상태로 변화함에 유의해주세요.

```ts
Promise.resolve("data")
    .finally(function () {
        console.log("throw error");
        throw new Error();
    })
    .then(function () {
        console.log("then");
    })
    .catch(function () {
        console.log("catch");
    })
    .finally(function () {
        console.log("finally");
    });

//
// will prints
//      throw error
//      catch
//      finally
```

---

---

## Async Iterator

### for-await-of

`for-of`에 주어진 iterator이 프로마이즈를 포함하고 있다면, `for-await-of`으로 이행된 값을 가져올 수 있습니다.

---

**ES 9:**

```ts
for await (const value of promises) {
    console.log(value);
}
```

**ES 8:**

```ts
for (const promise of promises) {
    const value = await promise;
    console.log(value);
}
```

---

---

### Symbol.asyncIterator

`Symbol.iterator`와 다르게 `async`키워드가 허용됩니다.

---

**ES 9:**

```ts
const myIterator = {
    async *[Symbol.asyncIterator]() {
        for (let i = 0; i < 5; i++) {
            await new Promise((resolve) => setTimeout(resolve, 1000));
            yield i;
        }
    },
};
```

**ES 8:**

```ts
const myIterator = {
    *[Symbol.iterator]() {
        for (let i = 0; i < 5; i++) {
            yield new Promise((resolve) => {
                setTimeout(() => resolve(i), 1000);
            });
        }
    },
};
```

---

`[Symbol.iterator]`와 `[Symbol.asyncIterator]`가 동시에 구현된 경우 `for-of`는 `[Symbol.iterator]`를 선택하고, `for-await-of`는 `[Symbol.asyncIterator]`를 선택합니다.

```ts
const myIterator = {
    *[Symbol.iterator]() {
        for (let i = 0; i < 5; i++) {
            yield new Promise((resolve) => {
                setTimeout(() => resolve(i), 1000);
            });
        }
    },

    async *[Symbol.asyncIterator]() {
        for (let i = 5; i < 10; i++) {
            await new Promise((resolve) => setTimeout(resolve, 1000));
            yield i;
        }
    },
};

async function main() {
    //
    // choose [Symbol.iterator]
    for (const promise of myIterator) {
        const item = await promise;
        console.log(item);
    }

    //
    // choose [Symbol.asyncIterator]
    for await (const item of myIterator) {
        console.log(item);
    }
}
main();
```

---

---

# String

## New Features

### Lifting Template Literal Restriction

`템플릿 리터럴 조건 완화` 또는 `느슨한 제약`으로 불리는 기능입니다. 템플릿 리터럴을 사용 할 때, 잘못된 이스케이프 값을 포함하고 있다면 에러가 발생했지만 ES9 부터는 해당 지점의 값이 `undefined`로 대체됩니다. 단, 원래의 모양새를 알 수 있도록 `.raw` 프로퍼티가 추가되었습니다.

```ts
function tag(strs) {
    console.log(strs);
}

tag`올바른 유니코드 : \u{AC00}`;
{
    0 : "올바른 유니코드 : 가",
    raws : ["올바른 유니코드 : \\u{AC00}"]
}

tag`잘못된 유니코드 : \u{0}`;
{
    0 : undefined,
    raws : ["잘못된 유니코드 : \\u{0}"]
}
```

---

---

# Object

## New Features

### Rest Properties

`Destructuring assignments`를 사용했을 때, 분해 연산자를 이용하여 사용되지 않은 프로퍼티들을 묶을 수 있습니다.

```ts
const { a, b, ...others } = {
    a: 1,
    b: 2,
    x: 3,
    y: 4,
};
console.log(a); // 1
console.log(b); // 2
console.log(others); // { x: 3, y: 4 }
```

---

---

### Spread Properties

객체 표현식에서 분해 연산자를 사용하여 기존의 객체를 얕은복사 시킬 수 있습니다.

---

**ES 9:**

```ts
const oldBox = { x: 1, y: 2 };
const newBox = {
    a: 1,
    b: 2,
    ...oldBox,
};
{
    a : 1,
    b : 2,
    x : 1,
    y : 2
}
```

**ES 8:**

```ts
const oldBox = { x: 1, y: 2 };
const newBox = Object.assign({ a: 1, b: 2 }, oldBox);
```

프로퍼티가 충돌된다면 후행의 것을 사용합니다.

```ts
const oldBox = { v: 0 };
const newBox = { v: 1, ...oldBox }; // { v: 0 }
const newBox = { ...oldBox, v: 1 }; // { v: 1 }
```

---

---

# RegExp

## Named Capture Groups

기존에는 캡쳐 그룹의 이름이 숫자로만 표현되었지만, 이제는 문자열 이름도 가질 수 있습니다. 문법은 `(?<name>...)` 입니다.

```ts
const regExp = /(?<yyyy>\d{4})-(?<mm>\d{2})-(?<dd>\d{2})/;
const result = regExp.exec("2020-12-25");
{
    0: "2020-12-25",
    1: "2020",
    2: "12",
    3: "25",
    input: "2020-12-25",
    groups: {
        yyyy: "2020",
        mm: "12",
        dd: "25"
    },
};

```

---

---

## Unicode Property Escapes

자주 사용되는 유니코드 패턴을 제공합니다. `\p{유니코드셋}` 형태로 사용합니다. p의 소문자면 `Positive` 대문자면 `Negative` 입니다. 유니코드셋은 다음 중 하나의 형태를 가질 수 있습니다.

-   카테고리명=카테고리값
-   카테고리값

```ts
/\p{General_Category=Letter}/gu.test("a"); // 구체적 명시
/\p{Letter}/gu.test("a"); // 카테고리값만 명시
/\p{L}/gu.test("a"); // Letter의 축약형
/\p{Emoji}/gu.exec("👌"); //
```

유니코드 이스케이프를 통해 사용된 패턴은 나중에 문자가 추가되더라도 `Node.JS` 측에서 자동으로 유지보수를 해주며, 예제를 보더라도 알 수 있듯이 카테고리의 이름은 우리가 읽기 편하게 구성되어 있습니다.

---

카테고리의 종류는 [모질라 도큐먼트](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Unicode_Property_Escapes#Syntax)를 참고해주세요.

---

---

## Lookbehind Assertions

특정 패턴이 앞서 나오거나(`Positive Look-Behind`) 앞서 나오지 않아야 (`Negative Look-Behind`) 매칭됩니다. 영문 이름이 behind인데 앞서나와야 한다고 말한 이유는 문자열의 진행방향이 → 이기 때문입니다. 사람이 → 방향으로 서있다고 생각해보세요. 자신의 뒤에 있는 것이, 문자열에서는 보면 앞서 있는 것입니다.

---

문법

-   `Positive LookBehind` : (?<=regex1)regex2
-   `Negative LookBehind` : (?<!regex1)regex2

**test :**

```ts
/(?<=[xyz])123/.test("x123"); // true
/(?<=[xyz])123/.test("m123"); // false
/(?<![xyz])123/.test("x123"); // false
```

---

LookBehind의 중요한 특징은 앞서 매칭된 정보를 반환하지 않는다는 것입니다.

---

**exec :**

```ts
/(?<=[xyz])123/.exec("x123");
{
    0 : "123",
    input : "x123",
    groups : undefined
}
```

---

---

## s flag (dotAll)

기존에는 닷 기호(`.`)가 개행문자에 매칭되지 않았지만 ES9부터는 `s (dotAll)` 플래그를 사용하여 개행문자에도 매칭시킬 수 있습니다.

---

**ES 9:**

```ts
/^.$/s.test("\n");
```

**ES 8:**

```ts
/^[^]$/.test("\n");
```
