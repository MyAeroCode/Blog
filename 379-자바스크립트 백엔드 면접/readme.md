> # JavaScript

## 프로토타입 언어

#### `프로토타입 언어가 무엇인지 설명해주세요`

클래스 기반 언어에서 상속과 달리 기존 객체를 복제함으로써 상속을 구현하는 객체지향 언어의 한 종류 입니다. 클래스의 개념이 없으므로 `class-less Language`로도 불립니다.

---

---

#### `그럼 자바스크립트에 있는 클래스 문법은 뭔가요?`

ECMA6부터 클래스 문법을 지원하긴 하지만 `문법설탕`에 불과합니다. 클래스 문법을 사용해도 여전히 프로토타입 형태로 동작합니다.

---

---

#### `프로토타입 언어의 특징을 설명해주세요`

클래스 기반 언어와 달리 실행시간에 동적으로 타입이 변경될 수 있습니다. C++의 객체들은 새로운 필드나 메서드가 추가되거나 삭제될 수 없는 것과 대비됩니다.

---

---

#### `프로토타입 언어의 장단점을 설명해주세요`

유연한 타입이 가능하지만, 너무 과한경우 객체의 프로퍼티를 예측할 수 없게됩니다.

---

---

#### `new 키워드로 객체를 생성하는 과정을 설명해주세요`

new 키워드를 사용하여 함수를 호출하면, 먼저 Object를 상속받은 빈 객체가 생성됩니다. 이후에는 빈 객체의 `Prototype-Link`에 호출된 함수의 `Prototype-Object`를 연결하고, new의 결과로 해당 객체가 반환됩니다.

---

---

#### `new 키워드는 객체랑 사용할 수 없나요?`

new의 대상이 되려면 `Prototype-Object`가 존재해야 합니다. 객체는 해당 오브젝트가 존재하지 않기 때문입니다. 반면에 함수는 생성될 때 `Prototype-Object`가 같이 생성되므로 new의 대상이 될 수 있습니다.

---

---

#### `상속받은 필드에 접근할 때, 내부과정을 설명해주세요`

맨 마지막 `Prototype-Object`부터 시작해서 해당 필드가 정의되어 있는지 검사합니다. 해당 객체에서 찾고있는 필드가 발견되었다면 그것을 반환하고, 발견되지 않았다면 상위 `Prototype-Object`로 이동하여 탐색을 반복합니다.

---

**관련 코드 1 :**

```ts
const animal = {
    eats: true,
};
const rabbit = {
    jump: true,
};
rabbit.__proto__ = animal;

console.log(rabbit); // { jump: true } rabbit에 eats는 없다!
console.log(rabbit.jump); // true
console.log(rabbit.eats); // true
```

---

**관련 코드 2 :**

```ts
function Animal() {}
Animal.prototype.eats = true;

function Rabbit() {
    this.jump = true;
}
Rabbit.prototype = Object.create(Animal.prototype);
Rabbit.prototype.constructor = Rabbit;

const rabbit = new Rabbit();
console.log(rabbit); // { jump: true }
console.log(rabbit.jump); // true
console.log(rabbit.eats); // true.

console.log(rabbit.sleep); // undefined
Animal.prototype.sleep = true;
console.log(rabbit.sleep); // true
```

---

---

#### `for in 구문으로 상속된 프로퍼티의 키도 볼 수 있나요?`

네. 볼 수 있습니다.

```ts
function Animal() {
    this.eats = true;
}
function Rabbit() {
    Animal.call(this);
    this.jump = true;
}
Rabbit.prototype = Object.create(Animal.prototype);
Rabbit.prototype.constructor = Rabbit;

const rabbit = new Rabbit();
for (const key in rabbit) {
    console.log(key); // "eats", "jump", "constructor"
}
```

---

---

#### `상속된 프로퍼티를 제외한 프로퍼티 목록을 알려면 어떻게 해야 하나요?`

`Object.getOwnPropertyNames`를 사용합니다.

---

---

## 과거 호환성

#### `클래스 문법 없이 Static Property를 구현하려면 어떻게 해야 하나요?`

함수에 직접 프로퍼티를 대입하면 됩니다.

```ts
function A() {}
A.x = 1;

const a = new A();
console.log(a.x); // undefined
console.log(A.x); // 1;
```

---

---

#### `클래스 문법 없이 Private Property를 구현하려면 어떻게 해야 하나요?`

클로저를 사용하면 됩니다.

```ts
function A() {
    let x = 999;
    function A() {
        this.getX = function () {
            return x;
        };
    }
    return new A();
}

const a = new A();
console.log(a.x); // undefined
console.log(a.getX()); // 999
```

---

---

#### `enum 문법없이 enum을 구현하려면 어떻게 해야 하나요?`

순방향 매핑과 역방향 매핑이 가능한 객체를 생성합니다.

```ts
const Color = {
    // 순방향 매핑
    R: 0,
    G: 1,
    B: 2,

    // 역방향 매핑
    0: "R",
    1: "G",
    2: "B",
};
const color = Color.R;
```

---

---

#### `const 문법없이 const를 구현하려면 어떻게 해야 하나요?`

`writable = false, configurable = false, else true` 옵션으로 컨텍스트에 defineProperty를 사용합니다.

```ts
const context = typeof global === "object" ? global : window;
Object.defineProperty(context, "PI", {
    value: 3.141593,
    enumerable: true, // 프로퍼티 노출
    writable: false, // 할당 연산자로 변경불가
    configurable: false, // 프로퍼티 덮어쓰기, 삭제 불가
});
```

---

---

#### `ES7`에서는 `Async/Await` 문법이 없는데 동일한 기능을 갖는 코드를 작성하려면 어떻게 해야 하나요?

제네레이터와 프로마이즈를 사용하여 구현할 수 있습니다.

```ts
function __await(thisArg, _arguments, generator) {
    function adopt(value) {
        return value instanceof Promise
            ? value
            : new Promise(function (resolve) {
                  resolve(value);
              });
    }

    return new Promise(function (resolve, reject) {
        generator = generator.apply(thisArg, _arguments || []);

        function fulfilled(value) {
            try {
                step(generator.next(value));
            } catch (e) {
                reject(e);
            }
        }

        function rejected(value) {
            try {
                step(generator.throw(value));
            } catch (e) {
                reject(e);
            }
        }

        function step(result) {
            result.done
                ? resolve(result.value)
                : adopt(result.value).then(fulfilled, rejected);
        }
        step(generator.next());
    });
}
```

이제 `async / await`를 구현할 수 있습니다.

**ES 8:**

```ts
function asyncSleep(t) {
    return new Promise((resolve) => {
        setTimeout(resolve, t);
    });
}

async function asyncGenerateHello() {
    await sleep(123);
    await sleep(456);
    return "Hello, World!";
}

async function main() {
    const hello = await asyncGenerateHello();
    console.log(hello);
}

main();
```

**ES 7:**

```ts
function asyncSleep(t) {
    return new Promise((resolve) => {
        setTimeout(resolve, t);
    });
}

function asyncGenerateHello() {
    return __await(this, undefined, function* () {
        yield asyncSleep(123);
        yield asyncSleep(456);
        return "Hello, World!";
    });
}

function main() {
    return __await(this, undefined, function* () {
        const hello = yield asyncGenerateHello();
        console.log(hello);
    });
}

main();
```

---

---

## ECMA 버전

#### `ECMA가 무엇인가요?`

언어가 지켜야 할 규격입니다. 이 규격을 구현한 언어들을 `ECMA Script (ES)`라고 부릅니다.

---

---

#### `ECMA 버전 이력을 순서대로 말해주세요`

ES2015부터 매년 6월에 새 버전이 출시됩니다.

-   `Edition 5 (ES5)`
-   `Edition 6 (ES6) - ES2015`
-   `Edition 7 (ES7) - ES2016`
-   `Edition 8 (ES8) - ES2017`
-   `Edition 9 (ES9) - ES2018`
-   `Edition 10 (ES10) - ES2019`
-   `Edition 11 (ES11) - ES2020`
-   `ESNext` (다음 해에 추가될 규격)

---

---

#### `ES5에서 추가된 기능을 아는대로 말해주세요`

-   `use strict`
-   `Function`
    -   `New Methods`
        -   `.bind()`
-   `String`
    -   `New Features`
        -   `Index Bracket`
        -   `Strings Over Multiple Lines`
    -   `New Methods`
        -   `.trim()`
        -   `.charAt()`
-   `Array`
    -   `New Features`
        -   `Trailing Commas`
    -   `New Methods`
        -   `.isArray()`
        -   `.forEach()`
        -   `.map()`
        -   `.filter()`
        -   `.reduce()`
        -   `.reduceRight()`
        -   `.every()`
        -   `.some()`
        -   `.indexOf()`
        -   `.lastIndexOf()`
-   `Object`
    -   `New Features`
        -   `Trailing Commas`
        -   `getter / setter`
    -   `New Methods`
        -   `.defineProperty()`
        -   `.defineProperties()`
        -   `.getOwnPropertyDescriptor()`
        -   `.getOwnPropertyNames()`
        -   `.keys()`
        -   `.preventExtensions()`
        -   `.isExtensible()`
        -   `.seal()`
        -   `.isSealed()`
        -   `.freeze()`
        -   `.isFrozen()`
        -   `.create()`
-   `JSON Support`
    -   `.parse()`
    -   `.stringify()`
-   `Date.now()`

[출처] [w3schools.com](https://www.w3schools.com/js/js_es5.asp)

---

---

#### `ES2015(ES6)에서 추가된 기능을 아는대로 말해주세요`

-   `Definition`
    -   `let`
    -   `const`
    -   `Block-Scoped Variables`
    -   `Block-Scoped Functions`
-   `Function`
    -   `Arrow Functions`
    -   `Default Parameter Value`
    -   `Rest Parameter`
    -   `Spread Operator`
-   `Literals`
    -   `String`
        -   `BackQoute Literal`
        -   `Tagged String Literal`
        -   `Unicode Literal`
    -   `Number`
        -   `Binary`
        -   `Octal`
-   `Regular Expression`
    -   `Keep Macthing Position`
-   `Object`
    -   `New Features`
        -   `Property Shorthand`
        -   `Computed Property Names`
        -   `Method Property`
        -   `Destructuring Assignment`
    -   `New Methods`
        -   `.assign()`
        -
-   `String`
    -   `New Methods`
        -   `.repeat()`
        -   `.startsWith()`
        -   `.endsWith()`
        -   `.includes()`
-   `Number`
    -   `New Methods`
        -   `.isNaN()`
        -   `.isFinite()`
        -   `.isSafeInteger()`
    -   `New Static Constant`
        -   `.EPSILON`
-   `Math`
    -   `New Methods`
        -   `.trunc()`
        -   `.sign()`
-   `Modules`
-   `Classes`
-   `Symbol Type`
-   `Iterator Protocol`
    -   `for of`
    -   `Iterators`
    -   `Generators`
-   `DataStructure`
    -   `Map`
    -   `Set`
    -   `WeakMap`
    -   `WeakSet`
-   `TypedArrays`
-   `Promises`
-   `Meta-Programming`
    -   `Proxy`
    -   `Reflection`
-   `Localization`
    -   `Collation`
    -   `Number Formatting`
    -   `Currency Formatting`
    -   `Date/Time Formatting`

[출처] [es6-features.org](http://es6-features.org/)

---

---

#### `ES2016(ES7)에서 추가된 기능을 아는대로 말해주세요`

-   `Array`
    -   `New Methods`
        -   `Array.prototype.includes`
-   `Exponentiation Operator`

[출처] [JavaScript History es7~es9](https://medium.com/@madasamy/javascript-brief-history-and-ecmascript-es6-es7-es8-features-673973394df4)

---

---

#### `ES2017(ES8)에서 추가된 기능을 아는대로 말해주세요`

-   `Async / Await`
-   `Function`
    -   `New Features`
        -   `Param Trailing Commas`
-   `Object`
    -   `New Methods`
        -   `values()`
        -   `entries()`
-   `String`
    -   `New Methods`
        -   `padStart()`
        -   `padEnd()`
        -   `getOwnPropertyDescriptors()`

[출처] [JavaScript History es7~es9](https://medium.com/@madasamy/javascript-brief-history-and-ecmascript-es6-es7-es8-features-673973394df4)

---

#### `ES2018(ES9)에서 추가된 기능을 아는대로 말해주세요`

-   `Async`
    -   `Async Iterator / for await of`
    -   `Promise.prototype.finally()`
-   `Object`
    -   `New Features`
        -   `Rest Properties`
        -   `Spread Properties`
-   `RegExp`
    -   `Named Capture Groups`
    -   `Unicode Property Escapes`
    -   `Lookbehind Assertions`
    -   `s(dotAll)`

[출처] [JavaScript History es7~es9](https://medium.com/@madasamy/javascript-brief-history-and-ecmascript-es6-es7-es8-features-673973394df4)

---

#### `ES2019(ES10)에서 추가된 기능을 아는대로 말해주세요`

-   `String`
    -   `New Methods`
        -   `trimStart()`
        -   `trimEnd()`
-   `Object`
    -   `New Methods`
        -   `fromEntries()`
-   `Array`
    -   `New Methods`
        -   `flat()`
        -   `flatMap()`
        -   `Stable Array.sort()`
-   `Function`
    -   `Changed`
        -   `toString()`
-   `Symbol`
    -   `New Features`
        -   `description`
-   `JSON`
    -   `Superset`
    -   `Well-formed JSON.stringify()`
-   `Optional Catch Binding`

[출처] [JavaScript History es10, es11](https://medium.com/@amandeepkochhar/javascript-es10-new-features-in-ecmascript-10-es2019-version-2e5ac8c46493)

---

---

#### `ES2020(ES11)에서 추가된 기능을 아는대로 말해주세요`

-   `BigInt`
-   `globalThis`
-   `Module`
    -   `Dynamic Import`
    -   `Module Namespace Exports`
    -   `import.meta`
-   `New Syntax`
    -   `Nullish Coalescing`
    -   `Optional Chaining`
-   `New Methods`
    -   `Promise.allSettled`
    -   `String.matchAll()`
-   `Well defined for-in order`

[출처] [JavaScript History es10, es11](https://medium.com/@amandeepkochhar/javascript-es10-new-features-in-ecmascript-10-es2019-version-2e5ac8c46493)

---

---

#### `ESNEXT에서 추가된 기능을 아는대로 말해주세요`

[ESNext Proposal Tables](http://kangax.github.io/compat-table/esnext/)

**Stages :**

-   `stage-4` : Finished (종료) - 정식 문법에 포함되었음
-   `stage-3` : Candidate (후보) - 다음 버전에 포함될 유력 후보
-   `stage-2` : Draft (초안)
-   `stage-1` : Proposal (제안)
-   `stage-0` : Strawman (허수아비) - 폐기될 확률이 높거나, 이미 폐기됨.
