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
console.log(rabbit.jump); // true.
console.log(rabbit.eats); // true. why?
```

---

**관련 코드 2 :**

```ts
function Animal() {
    this.eats = true;
}
function Rabbit() {
    this.__proto__ = new Animal();
    this.jump = true;
}

const rabbit = new Rabbit(); // { jump: true }
console.log(rabbit.jump); // true.
console.log(rabbit.eats); // true. why?

console.log(rabbit.sleep); // undefined
Animal.prototype.sleep = true;
console.log(rabbit.sleep); // true. why?
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
    this.__proto__ = new Animal();
    this.jump = true;
}
const rabbit = new Rabbit();
for (const key in rabbit) {
    console.log(key); // "eats", "jump"
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

#### `let 문법없이 let을 구현하려면 어떻게 해야 하나요?`

`configurable = false` 옵션으로 컨텍스트에 defineProperty를 사용합니다.

```ts
const context = typeof global === "object" ? global : window;
Object.defineProperty(context, "PI", {
    value: 3.141593,
    enumerable: true,
    writeable: true, // 할당 연산자로 변경가능
    configurable: false, // 프로퍼티 덮어쓰기, 삭제 불가
});
```

---

---

#### `const 문법없이 const를 구현하려면 어떻게 해야 하나요?`

`writable = false, configurable = false` 옵션으로 컨텍스트에 defineProperty를 사용합니다.

```ts
const context = typeof global === "object" ? global : window;
Object.defineProperty(context, "PI", {
    value: 3.141593,
    enumerable: true,
    writable: false, // 할당 연산자로 변경불가
    configurable: false, // 프로퍼티 덮어쓰기, 삭제 불가
});
```

---

---

## ECMA 버전

#### `ECMA가 무엇인가요?`

언어가 지켜야 할 규격입니다. 이 규격을 구현한 언어들을 `ECMA Script (ES)`라고 부릅니다.

---

---

#### `ECMA 버전 이력을 순서대로 말해주세요`

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

출처 : [w3schools.com](https://www.w3schools.com/js/js_es5.asp)

**요약 :**

-   `"use strict"`
-   `String`
    -   `New Features`
        -   인덱스 괄호 지원
        -   여러줄에 걸친 문자열 정의
    -   `New Methods`
        -   `.trim()`
        -   `.charAt()`
-   `Array`
    -   `New Features`
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
            -   콤마로 프로퍼티 정의를 끝낼 수 있음. (Trailing Commas)
            -   getter / setter
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
-   `JSON Support`
    -   `.parse()`
    -   `.stringify()`
-   `Date.now()`

---

---

**부연 설명 :**

-   `"use strict"`
-   `String`
    -   `New Features`
        -   인덱스 괄호 지원
        ```ts
        const word = "Hello, World!";
        const char = word[0]; // "H"
        ```
        -   여러줄에 걸친 문자열 정의
        ```ts
        const word1 =
            "Hello,\
        World!";
        const word2 = "Hello, World!";
        console.log(word1 === word2); // true
        ```
    -   `New Methods`
        -   `.trim()`
        ```ts
        const word = "   Hello, World!   ";
        console.log(word.trim()); // "Hello, World!"
        ```
        -   `.charAt()`
        ```ts
        const word = "Hello, World!";
        const char = word.chatAt(0); // "H"
        ```
-   `Array`

    -   `New Features`
        -   콤마로 요소 정의를 끝낼 수 있음. (Trailing Commas)
        ```
        const array = [1, 2, 3,];
        ```
    -   `Object`

        -   `New Features`

            -   콤마로 프로퍼티 정의를 끝낼 수 있음. (Trailing Commas)

            ```
            const object = {
                x : 1,
                y : 2,
            }
            ```

            -   getter / setter

            ```ts
            const human = {
                // private property
                _age: 15,

                //
                // getter
                get age() {
                    return "__" + this._age + "__";
                },

                //
                // setter
                set age(newAge: number) {
                    if (0 <= newAge) {
                        this._age = newAge;
                    }
                },
            };
            human.age = -12345; // will ignored
            console.log(human.age); // "__15__"
            ```

        -   `New Methods`

            -   `.defineProperty()`

                프로퍼티 이름과 프로퍼티 설명자를 사용하여 새 프로퍼티를 생성합니다.

            ```ts
            const obj = {};
            Object.defineProperty(obj, "x", {
                value: 1,
            });
            Object.defineProperty(obj, "y", {
                value: 2,
                writable: false, // readonly
            });
            console.log(obj); // { x:1, y:2 }
            obj.y = 12345; // will ignored.
            ```

            -   `.defineProperties()`
                여러개의 프로퍼티 설명자를 사용하여 새 프로퍼티들을 생성합니다.

            ```ts
            const obj = {};
            Object.defineProperties(obj, {
                x: {
                    value: 1,
                },
                y: {
                    value: 2,
                    writable: false,
                },
            });
            console.log(obj); // { x:1, y:2 }
            ```

            -   `.getOwnPropertyDescriptor()`

            (프로토체인을 탐색하지 않고) 특정 고유 프로퍼티의 설명자를 가져옵니다.

            ```ts
            const a = {};
            Object.defineProperties(a, {
                x: { value: 1 },
                y: { value: 2 },
            });

            const b = {};
            Object.defineProperties(b, {
                p: { value: 1 },
                q: { value: 2 },
            });

            b.__proto__ = a;
            Object.getOwnPropertyDescriptor(b, "p"); // {...}
            Object.getOwnPropertyDescriptor(b, "x"); // undefined
            ```

            -   `.getOwnPropertyNames()`

            (프로토체인을 탐색하지 않고) 모든 고유 프로퍼티의 이름들을 가져옵니다.

            ```ts
            const a = {};
            Object.defineProperties(a, {
                x: { value: 1 },
                y: { value: 2 },
            });

            const b = {};
            Object.defineProperties(b, {
                p: { value: 1 },
                q: { value: 2 },
            });

            b.__proto__ = a;
            Object.getOwnPropertyNames(a); // ["x", "y"]
            Object.getOwnPropertyNames(b); // ["p", "q"]
            ```

            -   `.keys()`
                (프로토체인을 탐색하지 않고) 열거 가능한 모든 고유 프로퍼티의 이름들을 가져옵니다.

            ```ts
            const a = {};
            Object.defineProperties(a, {
                x: { value: 1 },
                y: {
                    value: 2,
                    enumerable: true, // default : false.
                },
            });

            const b = {};
            Object.defineProperties(b, {
                p: { value: 1 },
                q: {
                    value: 2,
                    enumerable: true,
                },
            });

            b.__proto__ = a;
            console.log(Object.keys(a)); // ["y"]
            console.log(Object.keys(b)); // ["q"]
            ```

            ```ts
            const obj = {
                a: 1, // enumerable: true
                b: 2,
                c: 3,
            };
            Object.keys(obj); // ["a", "b", "c"];
            ```

            -   `.preventExtensions()`
                새로운 프로퍼티의 등록을 막습니다.

            ```ts
            const obj = { x: 1 };
            Object.preventExtensions(obj);

            obj.x = 2; // OK
            obj.y = 2; // will ignored (if non-strict)
            Object.defineProperty(obj, "y", {
                value: x,
            }); // will throw exception
            ```

            -   `.isExtensible()`

                preventExtensions가 적용되었는지 검사합니다.

            -   `.seal()`
                신규 프로퍼티 등록을 막고, 기존의 프로퍼티 삭제도 막습니다. 즉, 프로퍼티의 형태를 밀봉시켜 유지합니다.

            ```ts
            const obj = { x: 1 };
            Object.seal(obj);

            //
            // OK
            obj.x = 2;

            //
            // will ignored (if non-strict)
            delete obj.x = 2;
            obj.y = 2;
            ```

            -   `.isSealed()`

                seal이 적용되었는지 검사합니다.

            -   `.freeze()`

                프로퍼티 추가, 변경, 삭제를 방지합니다. `__proto__`도 프로퍼티이므로, 당연히 프로토타입도 변경될 수 없습니다.

            -   `.isFrozen()`

                freeze가 적용되었는지 검사합니다.

---

---

#### `ES2015(ES6)에서 추가된 기능을 아는대로 말해주세요`

출처 : [ES6-Features.org](http://es6-features.org/)

**요약 :**

-   `Definition / Scope`
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
-   `for ... of`
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

---

---

**부연 설명 :**

-   `Definition / Scope`

    -   `let`

    중복으로 선언할 수 없는 변수를 생성합니다.

    **ES6 :**

    ```ts
    let a = 3;
    ```

    **ES5 :**

    ```ts
    Object.defineProperty(thisScope, "a", {
        value: 4,
        enumerable: true,
        writable: true,
        configurable: false,
    });
    ```

    -   `const`

    중복으로 선언할 수 없으며, 값의 변경도 불가능한 변수를 생성합니다.

    **ES6 :**

    ```ts
    const a = 3;
    ```

    **ES5 :**

    ```ts
    Object.defineProperty(thisScope, "a", {
        value: 4,
        enumerable: true,
        writable: false,
        configurable: false,
    });
    ```

    -   `Block-Scoped Variables`

    로컬 스코프에서도 변수를 선언할 수 있습니다.

    **ES6 :**

    ```ts
    for (let i = 0; i < arr.length; i++) {
        // ...
    }
    ```

    **ES5 :**

    ```ts
    var i = 0;
    for (i = 0; i < arr.length; i++) {
        // ...
    }
    ```

    -   `Block-Scoped Functions`

    스코프 컨텍스트 함수를 선언하지 않아도 로컬 함수를 선언할 수 있습니다.

    **ES6 :**

    ```ts
    {
        function foo() {
            return 1;
        }
        foo(); // === 1
        {
            function foo() {
                return 2;
            }
            foo(); // === 2
        }
        foo(); // === 1
    }
    ```

    **ES5 :**

    ```ts
    (function globalScope() {
        //
        var foo = function () {
            return 1;
        };
        foo(); // === 1
        (function localScope() {
            var foo = function () {
                return 2;
            };
            foo(); // === 2
        })();
        foo(); // === 1;
    })();
    ```

-   `Function`

    -   `Arrow Functions`

    이름없는 함수의 가독성을 높일 수 있습니다. 해당 함수 내부의 this는 외부의 this와 같습니다.

    **ES6 :**

    ```ts
    array.map((v) => v + 1);
    array.map((v, i) => ({ v: v, i: I }));
    ```

    **ES5 :**

    ```ts
    array.map(function (v) {
        return v + 1;
    });
    array.map(function (v, i) {
        return {
            v: v,
            i: i,
        };
    });
    ```

    -   `Default Parameter Value`

    선언되지 않은 변수의 초기값을 설정할 수 있습니다.

    **ES6 :**

    ```ts
    function f(x, y = 7, z = 42) {
        return x + y + x;
    }
    ```

    **ES5 :**

    ```ts
    function f(x, y, x) {
        if (y === undefined) y = 7;
        if (z === undefined) z = 42;
        return x + y + x;
    }
    ```

    -   `Rest Parameter`

    뒤따라오는 모든 파라미터를 배열로 받습니다.

    **ES6 :**

    ```ts
    function f(x, y, ...remains) {
        //
    }
    ```

    **ES5 :**

    ```ts
    function f(x, y) {
        var remains = Array.prototype.slice.call(arguments, 2);
    }
    ```

    -   `Spread Operator`

    열거가능한 객체를 풀어헤쳐 순서대로 파라미터로 전달합니다.

    **ES6 :**

    ```ts
    const params = [3, 4];
    function f(a, b, c, d) {
        return a + b + c + d;
    }
    f(1, 2, ...params); // f(1, 2, 3, 4)
    ```

    **ES5 :**

    ```ts
    var params = [3, 4];
    function f(a, b, c, d) {
        return a + b + c + d;
    }
    f.apply(undefined, [1, 2].concat(params));
    ```

    문자열도 열거가능한 객체이므로 풀어헤칠 수 있습니다.

    **ES6 :**

    ```ts
    const word = "foo";
    const chars = [...word]; // ["f", "o", "o"]
    ```

    **ES5 :**

    ```ts
    var word = "foo";
    var chars = word.split(""); // ["f", "o", "o"]
    ```

-   `Literals`

    -   `String`

        -   `BackQoute Literal`

        백틱문자를 사용하여 문자열을 선언할 수 있습니다.

        ```ts
        const name = "World";
        const message = `
        Hi, ${name}!
        Hello, ${name}!
        `;
        ```

        -   `Tagged String Literal`

        백틱 리터럴에서 문자열, 변수 목록을 가져올 수 있습니다.

        ```ts
        function doubler(strings, ...values) {
            // strs : ["a", "b", "c"];
            // vals : [1, 2, 3];
            return strings[1].repeat(3);
        }
        const tagged = doubler`a${1}b${2}c${3}`;
        console.log(tagged); // "bbb"
        ```

        -   `Unicode Literal`

        이제 유니코드를 완벽하게 지원합니다.

        **ES6 :**

        ```ts
        "𠮷" === "\u{20BB7}";
        ```

        **ES5 :**

        ```ts
        "𠮷" === "\uD842\uDFB7";
        ```

    -   `Number`

        -   `Binary`
        -   `Octal`

        **ES6 :**

        ```ts
        0b111110111 === 503;
        0o767 === 503;
        ```

        **ES5 :**

        ```ts
        parseInt("111110111", 2) === 503;
        parseInt("767", 8) === 503;
        ```

-   `Regular Expression`

    -   `Keep Macthing Position`

    이제 정규식 수행시 각 토큰의 마지막 포지션 값을 알 수 있습니다.

    **ES6 :**

    ```ts
    let parser = (input, match) => {
        for (let pos = 0, lastPos = input.length; pos < lastPos; ) {
            for (let i = 0; i < match.length; i++) {
                match[i].pattern.lastIndex = pos;
                let found;
                if ((found = match[i].pattern.exec(input)) !== null) {
                    match[i].action(found);
                    pos = match[i].pattern.lastIndex;
                    break;
                }
            }
        }
    };
    ```

    **ES5 :**

    ```ts
    var parser = function (input, match) {
        for (var i, found, inputTmp = input; inputTmp !== ""; ) {
            for (i = 0; i < match.length; i++) {
                if ((found = match[i].pattern.exec(inputTmp)) !== null) {
                    match[i].action(found);
                    inputTmp = inputTmp.substr(found[0].length);
                    break;
                }
            }
        }
    };
    ```

-   `Object`

    -   `New Features`

        -   `Property Shorthand`

        변수 이름을 프로퍼티 이름으로 사용할 경우 선언문을 축약할 수 있습니다.

        **ES6 :**

        ```ts
        const x = 1;
        const y = 2;
        const obj = { x, y };
        ```

        **ES5 :**

        ```ts
        var x = 1;
        var y = 2;
        var obj = { x: x, y: y };
        ```

        -   `Computed Property Names`

        표현식 결과를 프로퍼티 이름으로 사용할 수 있습니다.

        **ES6 :**

        ```ts
        const obj = {
            foo: "bar",
            ["a".repeat(3)]: "bar",
        };
        ```

        **ES5 :**

        ```ts
        const obj = {
            foo: "bar",
        };
        obj["a".repeat(3)] = "bar";
        ```

        -   `Method Property`

        프로퍼티 메서드 선언을 축약할 수 있습니다.

        **ES6 :**

        ```ts
        const obj = {
            a(x,y){
                //
            }
            b(x,y){
                //
            }
        }
        ```

        **ES5 :**

        ```ts
        var obj = {
            a: function (x, y) {
                //
            },
            b: function (x, y) {
                //
            },
        };
        ```

        -   `Destructuring Assignment`

        열거가능한 객체에서 특정 프로퍼티로 변수를 선언합니다.

        **ES6 :**

        ```ts
        const list = [1, 2, 3];
        const [a, , b] = list; // a=1, b=3
        [b, a] = [a, b]; // b=3, a=1
        ```

        **ES5 :**

        ```ts
        var list = [1, 2, 3];
        var a = list[0],
            b = list[2];
        var tmp = a;
        a = b;
        b = tmp;
        ```

        객체도 열거가능한 객체이므로,

        **ES6 :**

        ```ts
        const obj = {
            x: 1,
            y: 2,
            z: 3,
        };
        const { x, y, z } = obj;
        ```

        **ES5 :**

        ```ts
        var x = obj.x;
        var y = obj.y;
        var z = obj.z;
        ```

        재귀적 분해 및 별칭을 부여할 수 있습니다.

        **ES6 :**

        ```ts
        const outer = {
            value: 1,
            inner: {
                value: 2,
            },
        };
        const {
            value: outer_v,
            inner: { value: inner_v },
        } = outer;
        ```

        **ES5 :**

        ```ts
        var outer_v = outer.value;
        var inner_v = outer.inner.value;
        ```

        존재하지 않는 프로퍼티에 기본값을 부여할 수 있습니다.

        **ES6 :**

        ```ts
        const obj = { a: 1 };
        const { a, b = 2 } = obj;

        const list = [1];
        const [x, y = 2] = list;
        ```

        **ES5 :**

        ```ts
        var a = obj.a;
        var b = obj.b === undefined ? 2 : obj.b;

        var x = list[0];
        var y = list[1] === undefined ? 2 : list[1];
        ```

        함수 시그너쳐에도 적용할 수 있습니다.

        **ES6 :**

        ```ts
        function a([name, val]) {
            console.log(name, val);
        }
        a(["x", 1]);

        function b({ name: n, val: v }) {
            console.log(n, v);
        }
        b({ name: "x", val: 1 });

        function c({ name, val }) {
            console.log(name, val);
        }
        c({ name: "x", val: 1 });
        ```

        **ES5 :**

        ```ts
        function a(arg) {
            var name = arg[0];
            var val = arg[1];
        }

        function b(arg) {
            var n = arg.name;
            var v = arg.val;
        }

        function c(arg) {
            var name = arg.name;
            var val = arg.val;
        }
        ```

        일치하는 프로퍼티가 없다면 undefined가 적용됩니다.

        **ES6 :**

        ```ts
        const list = [7, 42];
        const [a = 1, b = 2, c = 3, d] = list;
        a; // 7
        b; // 42
        c; // 3
        d; // undefined
        ```

        **ES5 :**

        ```ts
        var a = typeof list[0] !== "undefined" ? list[0] : 1;
        var b = typeof list[1] !== "undefined" ? list[1] : 2;
        var c = typeof list[2] !== "undefined" ? list[2] : 3;
        var d = typeof list[3] !== "undefined" ? list[3] : undefined;
        ```

    -   `New Methods`

        -   `.assign()`

        주어진 src의 모든 프로퍼티를 dest에 할당합니다. 연산의 결과로 dest를 반환합니다.

        **ES6 :**

        ```ts
        const dest = {};
        const src1 = { a: 1, b: 2 };
        const src2 = { a: 3, c: 4 };
        Object.assign(dest, ...[src1, src2]);
        console.log(dest.a); // 3
        console.log(dest.b); // 2
        console.log(dest.c); // 4
        ```

        **ES5 :**

        ```ts
        var dest = {};
        var src1 = { a: 1, b: 2 };
        var src2 = { a: 3, c: 4 };
        Object.keys(src1).forEach(function (k) {
            dest[k] = src1[k];
        });
        Object.keys(src2).forEach(function (k) {
            dest[k] = src2[k];
        });
        ```

-   `String`

    -   `New Methods`

        -   `.repeat()`

        해당 문자열은 n번 반복합니다.

        ```ts
        console.log("a".repeat(3)); // "aaa"
        ```

        -   `.startsWith()`

        해당 문자열이 특정 워드로 시작하면 true를 반환합니다.

        ```ts
        "Hello, World!".startsWith("H"); // true
        "Hello, World!".startsWith("ld!"); // false
        ```

        -   `.endsWith()`

        해당 문자열이 특정 워드로 끝나면 true를 반환합니다.

        ```ts
        "Hello, World!".startsWith("H"); // false
        "Hello, World!".startsWith("ld!"); // true
        ```

        -   `.includes()`

        해당 문자열이 특정 워드를 포함하면 true를 반환합니다.

        ```ts
        "Hello, World!".startsWith("H"); // true
        "Hello, World!".startsWith("ld!"); // true
        ```

-   `Number`

    -   `New Methods`

        -   `.isNaN()`

        NaN인지 검사합니다. 등호로는 NaN을 검사할 수 없습니다.

        ```ts
        const target = NaN;
        target === NaN; // false
        Number.isNaN(target); // true
        ```

        -   `.isFinite()`

        양의 무한대 또는 음의 무한대인지 검사합니다. 등호로도 검사할 수 있습니다.

        ```ts
        const target = -Infinity;
        target === -Infinity; // true
        Number.isFinite(target); // true
        ```

        -   `.isSafeInteger()`

        자바스크립트는 부동소수점 또는 매우 큰 숫자는 정확도를 희생합니다. 해당 임계점을 넘었는지 검사합니다.

        ```ts
        Number.isSafeInteger(Number.MAX_SAFE_INTEGER * 1); // true
        Number.isSafeInteger(Number.MAX_SAFE_INTEGER * 2); // false
        Number.isSafeInteger(0.1); // false
        ```

    -   `New Static Constant`

        -   `.EPSILON`

        IEEE754 방식으로 표현할 수 있는 가장 작은 숫자를 가르킵니다. 이것은 매우 특수한 분야의 연구자를 위해 존재하는 값입니다. 일반적인 프로그래머들은 사용할 일이 없습니다.

-   `Math`

    -   `New Methods`

        -   `.trunc()`

        소수점 버림연산입니다.

        **ES6 :**

        ```ts
        Math.trunc(+42.7); // +42
        Math.trunc(+0.1); // +0;
        Math.trunc(-0.1); // -0
        Math.trunc(-42.7); // -42
        ```

        **ES5 :**

        ```ts
        function trunc(n) {
            return n < 0 ? Math.ceil(n) : Math.floor(n);
        }
        ```

        -   `.sign()`

        주어진 수의 부호를 판별합니다.

        **ES6 :**

        ```ts
        Math.sign(+7); // +1
        Math.sign(+0); // +0
        Math.sign(-0); // -0
        Math.sign(-7); // -1
        Math.sign(NaN); // NaN
        ```

        **ES5 :**

        ```ts
        function sign(n) {
            if (Number.isNaN(n)) return NaN;
            if (n === 0) return 0;
            return x < 0 ? -1 : +1;
        }
        ```

-   `Modules`

    모듈 기능이 추가되었습니다.

-   `Classes`

    -   `Definition`

    이제부터 클래스 문법이 지원되지만, 내부적으로는 프로토타입으로 작동합니다.

    **ES6 :**

    ```ts
    class Shape {
        constructor(id, x, y) {
            this.id = id;
            this.move(x, y);
        }
        move(x, y) {
            this.x = x;
            this.y = y;
        }
    }
    ```

    **ES5 :**

    ```ts
    var Shape = function (id, x, y) {
        this.id = id;
        this.move(x, y);
    };
    Shape.prototype.move = function (x, y) {
        this.x = x;
        this.y = y;
    };
    ```

    -   `Inheritance`

    상속도 마찬가지입니다.

    **ES6 :**

    ```ts
    class Rectangle extends Shape {
        constructor(id, x, y, width, height) {
            super(id, x, y);
            this.width = width;
            this.height = height;
        }
    }
    ```

    **ES5 :**

    ```ts
    var Rectangle = function (id, x, y, width, height) {
        Shape.call(this, id, x, y);
        this.width = width;
        this.height = height;
    };
    Rectangle.prototype = Object.create(Shape.prototype);
    Rectangle.prototype.constructor = Rectangle;
    ```

    -   `Expression Based Inheritance`

    표현식의 결과로 무엇을 상속할지 결정할 수 있습니다.

    **ES6 :**

    ```ts
    class A {
        name = "A";
    }
    class B {
        name = "B";
    }
    function getBaseClass() {
        return Math.random() < 0.5 ? A : B;
    }

    class C extends getBaseClass() {}
    console.log(new C()); // "A" or "B"
    ```

    -   `Base Class Access`

    Super 키워드를 사용하여 부모 클래스에 접근할 수 있습니다.

    **ES6 :**

    ```ts
    class A {
        val = 5;
        _getVal() {
            return this.val;
        }
    }
    class B extends A {
        getVal() {
            return super._getVal();
        }
    }
    console.log(new B().getVal()); // 5
    ```

    **ES5 :**

    ```ts
    function A() {
        this.val = 5;
    }
    A.prototype._getVal = function () {
        return this.val;
    };

    function B() {
        A.call(this); // super()
    }
    B.prototype.getVal = function () {
        return A.prototype._getVal.call(this);
    };

    console.log(new B().getVal()); // 5
    ```

    -   `Static Member`

    static 키워드로 클래스 수준에서 프로퍼티를 설정할 수 있습니다.

    **ES6 :**

    ```ts
    class Box {
        val = 0;
        static getRandomBox() {
            const newBox = new Box();
            newBox.val = Math.random();
            return newBox;
        }
    }
    Box.getRandomBox();
    ```

    **ES5 :**

    ```ts
    var Box = function () {
        this.val = 0;
    };
    Box.getRandomBox = function () {
        const newBox = new Box();
        newBox.val = Math.random();
        return newBox;
    };
    Box.getRandomBox();
    ```

    -   `getter/setter`

    get, set을 바로 클래스 내부에서 사용할 수 있습니다.

    **ES6 :**

    ```ts
    class Box {
        _val = 0;
        get val() {
            return this._val;
        }
        set val(newVal) {
            this._val = newVal;
        }
    }
    ```

    **ES5 :**

    ```ts
    var Box = function (){
        this._val = 0;
    }
    Box.prototype = {
        get val(){
            return this._val;
        }
        set val(newVal){
            this._val = newVal;
        }
    }
    ```

-   `Symbol Type`

    프로퍼티를 고유하게 식별하는 원시 자료형입니다. 인자로 프로퍼티 이름을 넣을 수 있지만, 프로퍼티 이름이 같다고 해서 같은 심볼을 가르키지 않습니다. 즉, 디버깅 용도로만 사용됩니다.

    ```ts
    const a = Symbol("x");
    const b = Symbol("x");
    console.log(a === b); // false
    ```

    심볼은 프로퍼티를 고유하게 식별할 수 있습니다.

    ```ts
    const a = Symbol("x");
    const b = Symbol("x");
    const obj = {};
    obj[a] = 1;
    obj[b] = 2;
    console.log(obj[a], obj[b]); // 1, 2
    console.log(obj); // { symbol(x):1, symbol(x):2 }
    ```

    심볼로 정의된 프로퍼티는 Object.getOwnPropertySymbols로 찾을 수 있습니다.

    ```ts
    Object.getOwnPropertyNames(obj); // []
    Object.getOwnPropertySymbols(obj); // [x, x]
    ```

    Symbol.for을 사용하면 컨테이너에 저장된 이름이 같은 심볼을 공유합니다. 이것을 Global Symbol 이라고 합니다.

    ```ts
    const a = Symbol.for("x");
    const b = Symbol.for("x");
    console.log(a === b); // true
    ```

    특정 용도로 사용되는 Symbol은 스태틱 프로퍼티에 동봉되어 있습니다. 이것을 Well-Known Symbol 이라고 합니다.

    **이터레이터 심볼 :**

    -   Symbol.iterator
    -   Symbol.asyncIterator

    **정규표현식 심볼 :**

    -   Symbol.match
    -   Symbol.replace
    -   Symbol.search
    -   Symbol.split

-   `for ... of`

    Iterator Protocol이 구현된 객체는 for of 구문으로 요소들을 순차적으로 가져올 수 있습니다. 해당 프로토콜을 구현하려면 [Symbol.iterator]에 올바른 함수를 구현하면 됩니다. Array는 기본적으로 해당 프로토콜이 구현되어 있습니다.

    ```ts
    const list = [1, 2, 3, 4, 5];
    for (const val of list) {
        console.log(val); // will print 1, 2, 3, 4, 5
    }
    ```

    이터레이터 프로토콜이 구현되어 있다면 `Spread Operator`를 사용할 수 있습니다. 단, 내부적으로 모든 원소들을 불러온 뒤에 할당하므로 Infinity Iterator에 대해 사용하면 안됩니다. 마법처럼 개수만큼만 불러오지 않습니다.

    ```ts
    const arrs = [1, 2, 3, 4, 5];
    const nums = [...arrs];
    ```

    -   `Iterators`

    순회를 목적으로 합니다. done이 false가 될 때 까지 for-of가 반복됩니다.

    ```ts
    const list = {
        entries: {
            0: "a",
            1: "b",
        },
        [Symbol.iterator]: function () {
            let counter = 0;
            const entries = this.entries;
            return {
                next: function () {
                    return {
                        value: entries[counter],
                        done: entries[counter++] === undefined,
                    };
                },
            };
        },
    };

    for (const v of list) {
        console.log(v); // will print a, b
    }

    const values = [...list]; // ["a", "b"];
    ```

    -   `Generators`

    Iterator 훨씬 유연하고 정교한 작업이 가능합니다. iterator는 순회를 목적으로하지만, generator는 생성을 목적으로 합니다. 먼저 generator는 함수 앞에 \*를 붙여놓음으로써 정의할 수 있으며, 더 이상 yield가 사용되지 않을 때 까지 반복합니다.

    **선언 :**

    ```ts
    function* fibonacci() {
        //
    }
    const fibonacci = function* () {
        //
    };
    const fibonacci = {
        *[Symbol.iterator]() {
            //
        },
    };
    ```

    **구현 :**

    ```ts
    function* fibonacci() {
        let pre = 0,
            cur = 1;
        while (true) {
            [pre, cur] = [cur, pre + cur];

            //
            // cur를 넘겨주고 자신은 일시정지.
            yield cur;
        }
    }
    for (const f of fibonacci) {
        //
        // 멈추지 않으면 계속 생성됨.
        if (1000 <= f) {
            break;
        }
        console.log(f);
    }
    ```

    **Iterator와의 차이점 :**

    제너레이터는 함수형태로 작성시 직접적으로 인자를 받을 수 있습니다.

    ```ts
    function* range(srt, end) {
        while (srt < end) {
            yield srt;
            srt++;
        }
    }

    for (const n of range(3, 6)) {
        console.log(n); // will print 3, 4, 5
    }

    const r = [...range(3, 6)]; // [3, 4, 5];
    ```

    제네레이터는 다르게 여러변 yield를 사용할 수 있으므로, 이터레이터보다 더 유연합니다.

    ```ts
    const iteratorExample = {
        [Symbol.iterator]() {
            let count = 0;
            return {
                next() {
                    return {
                        value: count++,
                        done: false,
                    };
                },
            };
        },
    };

    const generatorExample = {
        *[Symbol.iterator]() {
            let count = 0;
            while (true) {
                //
                // 여러변 yield 가능.
                yield count++;
                yield count++;
                yield count++;
            }
        },
    };
    ```

-   `DataStructure`
    -   `Map`
    -   `Set`
    -   `WeakMap`
    -   `WeakSet`
-   `TypedArrays`
-   `Promise`
-   `Meta-Programming`
    -   `Proxy`
    -   `Reflection`
-   `Localization`
    -   `Collation`
    -   `Number Formatting`
    -   `Currency Formatting`
    -   `Date/Time Formatting`

---

---

#### `ES2016(ES7)에서 추가된 기능을 아는대로 말해주세요`

**요약 :**

---

---

#### `ES2017(ES8)에서 추가된 기능을 아는대로 말해주세요`

## **요약 :**

---

#### `ES2018(ES9)에서 추가된 기능을 아는대로 말해주세요`

## **요약 :**

---

#### `ES2019(ES10)에서 추가된 기능을 아는대로 말해주세요`

**요약 :**

---

---

#### `ES2020(ES11)에서 추가된 기능을 아는대로 말해주세요`

**요약 :**

---

---

#### `ESNEXT에서 추가될 기능을 아는대로 말해주세요`

[ESNext Proposal Tables](http://kangax.github.io/compat-table/esnext/)

**Stages :**

-   `stage-4` : Finished (종료) - 정식 문법에 포함되었음
-   `stage-3` : Candidate (후보) - 다음 버전에 포함될 유력 후보
-   `stage-2` : Draft (초안)
-   `stage-1` : Proposal (제안)
-   `stage-0` : Strawman (허수아비) - 폐기될 확률이 높거나, 이미 폐기됨.
