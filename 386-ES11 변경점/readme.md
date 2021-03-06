`ES11`은 보다 깊은 `편의성 개선`을 제공합니다. 기존의 문법에서 걸림돌이 되거나 불편했던 제약사항들을 완화하거나, 그것을 해결한 기능을 새로운 문법으로 풀어냈습니다.

---

**톺아보기 :**

-   `New Features`
    -   `BigInt`
    -   `globalThis`
    -   `Well defined for-in order`
    -   `Optional Chaining`
    -   `Nullish Coalescing`
-   `Class`
    -   `Private Fields`
    -   `Static Fields`
-   `Async`
    -   `Top Level Await`
    -   `Promise.allSettled()`
-   `String`
    -   `matchAll()`
-   `Module`
    -   `Dynamic Import`
    -   `Module Namespace Exports`
    -   `import.meta`

---

---

# New Featrues

## BigInt

자바스크립트는 `Number.MAX_SAFE_INTEGER` 또는 `2^53-1`을 넘어가는 매우 큰 정수는 일부 정확도를 포기하고 저장하게 됩니다. 두 번째 줄의 연산결과가 틀린것에 주목해주세요.

```ts
const M = Number.MAX_SAFE_INTEGER;
console.log(M + 9 + 0); // 9007199254741000
console.log(M + 9 + 1); // 9007199254741000
```

이러한 상황을 예방하기 위해 `BigInt` 자료형이 등장했으며, `2^53-1`를 넘어가는 매우 큰 수를 정확하게 저장할 수 있습니다. 그러나 `Number`와 달리 상수시간 계산이 아니므로 암호화같은 집중연산에 적합하지 않음에 유의해주세요.

```ts
const M = BigInt(Number.MAX_SAFE_INTEGER);
console.log(M + 9n + 0n); // 9007199254741000n
console.log(M + 9n + 1n); // 9007199254741001n
```

---

---

## globalThis

`JavaScript`는 실행환경에 따라 최상위 문맥의 이름이 다르므로, 각각의 경우를 일일이 체크해야 했습니다. 그러나 `globalThis`가 도입됨으로써 이러한 불편이 해소되었습니다.

-   Browser : `window`
-   Service Worker : `self`
-   Node : `global`

**ES 11:**

```ts
function getGlobal() {
    return globalThis;
}
```

**ES 10:**

```ts
function getGlobal() {
    if (typeof self !== "undefined") return self;
    if (typeof window !== "undefined") return window;
    if (typeof global !== "undefined") return global;

    throw new Error("unable to locate global object");
}
```

---

---

## Well defined for-in order

이전에는 `for-in`의 순서가 `Object.keys()`의 순서와 일치하지 않았습니다. 그러나 ES11 부터는 두 순서가 일치됩니다.

```ts
//
// for-in 으로 키목록 획득
function getKeys1(obj) {
    const keys = [];
    for (const key in obj) {
        keys.push(key);
    }
    return keys;
}

//
// Object.keys() 로 키목록 획득
function getKeys2(obj) {
    return Object.keys();
}
```

---

---

## Optional Chaining

퀘스쳔 닷(`?.`)을 사용하여 속성에 접근할 경우, 해당 속성이 존재하지 않는다면 `undefined`를 반환하고 탐색을 멈춥니다. 즉, `Cannot read property '...' of undefined` 에러가 발생하지 않습니다.

---

**ES 11:**

```ts
const outer = {};

console.log(otuer?.inner?.value); // undefined
```

**ES 10:**

```ts
const outer = {};

//
// cannot read property 'value' of undefined.
console.log(outer.inner.value);
```

퀘스쳔 닷의 앞이 `undefined`라면 탐색을 멈춘다는 특징은 굉장히 유용합니다. 아래와 같은 상황에 적용할 수 있습니다.

---

**배열 요소에 접근 :**

```ts
let arr = [0, 1, 2, 3];

//
// 중간에 배열이 손상됨.
arr = null;

console.log(arr?.[1]); // undefined
console.log(arr[1]); // error
```

**함수 호출 :**

```ts
let helloWorld = function () {
    console.log("Hello, World!");
};

//
// 중간에 함수가 손상됨.
helloWorld = null;

helloWorld?.(); // 함수가 호출되지 않고 undefined가 반환됨.
helloWorld(); // error
```

---

---

## Nullish Coalescing

버티컬바 2개를 사용한 논리 연산자(`||`)는 좌측값이 `Falsy Value`라면 우측값을 반환하는 성질에 의해 `FallBack Value`를 계산할 때 매우 유용하게 사용되지만, `0`도 거짓으로 판단되기 때문에 아래와 같은 불상사가 발생합니다.

```ts
function getValue(v: undefined | null | number) {
    //
    // v값이 지정되지 않았다면 기본값인 50을 반환한다.
    return v || 50;
}

getValue(undefined); // 50
getValue(null); // 50
getValue(0); // 50
```

위의 불상사를 막기 위해 `||`와 비슷하지만 `undefined`와 `null`에 대해서만 동작하는 더블 퀘스쳔(`??`) 연산자가 등장했습니다. 이것이 `Nullish Coalescing` 입니다.

```ts
function getValue(v: undefined | null | number) {
    //
    // v값이 지정되지 않았다면 기본값인 50을 반환한다.
    return v ?? 50;
}

getValue(undefined); // 50
getValue(null); // 50
getValue(0); // 0
```

---

---

# Class

## Private Fields

필드 이름의 앞에 `#`를 붙이면 비공개 필드로 선언됩니다. 기존에는 클래스로 이것을 구현할 수 없었으므로 `functional class`와 `closure`를 통해 구현해야 했습니다.

---

**ES 11:**

```ts
class Box {
    #value = 0;

    getValue() {
        return this.#value;
    }

    setValue(value) {
        this.#value = value;
    }
}

const box = new Box();
console.log(box.getValue()); // 0
console.log(box.#value); // error
```

**ES 10:**

```ts
function Box() {
    let _value = 0;

    return new (function Box() {
        this.getValue = function () {
            return _value;
        };

        this.setValue = function (value) {
            _value = value;
        };
    })();
}

const box = new Box();
console.log(box.getValue()); // 0
console.log(box._value); // error
```

메서드도 내부적으로는 필드에 불과하므로, 메서드의 앞에도 #을 붙일 수 있습니다.

```ts
//
// 이렇게 써도
class Box {
    #foo() {
        console.log("Hello, World!");
    }
}

//
// 이것과 같다.
class Box {
    #foo = function () {
        console.log("Hello, World!");
    };
}
```

---

---

## Static Fields

필드의 앞에 `Static`을 붙이면, 해당 필드는 클래스에 소유됩니다.

---

**ES 11:**

```ts
class Box {
    static hello() {
        console.log("Hello, World!");
    }
}
Box.hello();
```

**ES 10:**

```ts
class Box {}
Box.hello = function () {
    console.log("Hello, World!");
};
Box.hello();
```

---

---

# Async

## Top Level Await

기존의 `await`는 항상 `async function` 내부에서만 허용되었지만, 이제 최상위 문맥에서도 `await` 키워드를 사용할 수 있습니다.

---

**ES 11:**

```ts
function asyncSleep(ms) {
    return new Promise((resolve) => {
        setTimeout(resolve, ms);
    });
}

await asyncSleep(1000);
console.log("Hello, World!");
```

**ES 10:**

```ts
function asyncSleep(ms) {
    return new Promise((resolve) => {
        setTimeout(resolve, ms);
    });
};

async main(){
    await asyncSleep(1000);
    console.log("Hello, World!");
}
main();
```

`Node 14.8.0` 미만에서는 `--harmony-top-level-await` 플래그를 함께 주어야 해당 기능이 활성화되지만, 이상부터는 항상 활성화됩니다. 단, 해당 기능은 `ES Modules`에서만 동작하므로 package.json에 `"type": "module"`을 추가하거나 `.mjs`로 확장자를 변경해야 합니다.

---

---

## Promise.allSettled()

`Promise.all()`은 실패한 프로마이즈가 하나라도 있다면 전체가 실패(`.catch()`로 이동)합니다. 그러나 `Promise.allSettled()`는 프로마이즈의 성공여부와는 관계없이 `결과`만 추출하여 `then`으로 넘깁니다. 즉, 항상 Resolved 상태로 변경됩니다.

```ts
Promise.allSettled([
    new Promise((resolve, reject) => setTimeout(resolve, 1000)),
    new Promise((resolve, reject) => setTimeout(reject, 1000)),
]).then(function (results) {
    console.log(results.length); // 2
    console.log(results[0]); // { status: 'fulfilled', value: undefined }
    console.log(results[1]); // { status: 'rejected', reason: undefined }
});
```

---

---

# String

## matchAll()

```ts
const regex = /x[a-z]+/g;
const text = "x y z xx xy xz";
const tokens = text.matchAll(regex);

console.log(tokens);
//
// will prints
// [RegExp String Iterator] {}

for (const token of tokens) {
    console.log(token);
}
//
// will prints
[ 'xx', index: 6, input: 'x y z xx xy xz', groups: undefined ]
[ 'xy', index: 9, input: 'x y z xx xy xz', groups: undefined ]
[ 'xz', index: 12, input: 'x y z xx xy xz', groups: undefined ]
```

---

---

# Module

## Dynamic Import

기존에는 최상단에서만 모듈을 불러올 수 있었지만, 이제부터는 코드의 중간에서도 동적으로 모듈을 불러올 수 있습니다.

```ts
async function main() {
    const cp = await import("child_process");
}
main();
```

---

---

## Module Namespace Exports

`export * from "..."`를 여러번 사용할 경우, 이름이 충돌되는 경우가 있었습니다. 이제부터 네임스페이스를 함께 주어 이름충돌을 방지할 수 있습니다.

```ts
// moduleA.ts
export const a = 1;
export const b = 2;

// moduleB.ts
export const b = 3;
export const c = 4;

//
// modules.ts
export * from "./moduleA";
export * from "./moduleB"; // error! Module "./moduleA" has already exported a member named 'b'.
```

```ts
//
// modules.ts
export * as A from "./moduleA";
export * as B from "./moduleA";

//
// main.ts
import { A, B } from "./modules";
console.log(A.a); // 1
console.log(A.b); // 2
console.log(B.b); // 3
console.log(B.c); // 4
```

---

---

## import.meta

현재 모듈파일의 메타정보를 가져올 수 있습니다.

```ts
console.log(import.meta);

//
// will prints
{
    url: "file:///home/user/my-module.js";
}
```
