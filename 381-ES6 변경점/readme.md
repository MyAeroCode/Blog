`ES6` 또는 `ES2015`는 그야말로 대격변이 일어났습니다. 하나만 들어와도 벅찬 거대한 개념들이 무수히 많이 추가됬으며, 새로운 문법들도 다수 추가됬습니다. 이번 변경점에서 특히 눈여겨 봐야 할 기능들은 다음과 같습니다.

-   `Block-Scope`
-   `Arrow Functions`
-   `Symbol`
-   `Class`
-   `Promise`
-   `Iterator Protocol`

---

`Class`와 같은 일부 기능은 `ES5`를 사용하여 구현할 수 있습니다. 이런 경우에는 `ES5`로 어떻게 구현할 수 있는지, 어떤 차이점이 있는지 기억해두면 좋습니다. 누구라도 알만한 유니콘 스타트업의 기술면접에서 실제로 출제되었습니다. `ES6`은 단언컨대 ECMAScript 역사상 가장 큰 업데이트입니다 😂

---

**톺아보기 :**

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

---

---

# Scope

## let

중복으로 선언할 수 없는 변수를 생성합니다.

---

**ES 6:**

```ts
let a = 1;
let a = 2; // error
```

**ES 5:**

```
ES5로 구현할 수 없습니다.
```

---

---

## const

값의 변경이 불가능한 변수를 생성합니다.

---

**ES6 :**

```ts
const a = 123;
a = 999; // error
```

**ES5 :**

```ts
Object.defineProperty(this, "a", {
    value: 123,
    enumerable: true,
    writable: false,
    configurable: false,
});
a = 999; // will ignored
```

---

---

## Block-Scoped Variables

`var`은 함수단위 스코프이므로 블럭이 끝나도, 여전히 함수 내부에서 사용할 수 있었습니다.

```ts
function f() {
    {
        var a = 1;
    }
    console.log(a); // will print 1
}
```

또한 `var`은 중복으로 변수를 선언해도 별다른 안전조치가 없습니다.

```ts
var a = 1;
var a = 2; // OK.
```

반면에 `let`과 `const`는 블럭이 끝나면 선언의 효력이 사라지며, 중복선언시 에러가 발생합니다.

```ts
function f() {
    {
        let a = 1;
        let a = 2; // error: a has already been declared.
    }
    console.log(a); // error: a is not defined.
}
```

스코프 원리를 이해하기 위해 인터널하게 들여다보면 인터프리터는 `var`로 선언된 변수의 선언을 최상위로 끌어올려 처리합니다. 이러한 특징을 `hoisting`이라고 합니다.

---

**우리가 작성한 코드 :**

```ts
for (var c = 0; c < 10; c++) {
    console.log(c);
}
var a = 1;
var b = 2;
```

**인터프리터가 실행하는 수도 코드 :**

```ts
[declare] a;
[declare] b;
[declare] c;

for (c = 0; c < 10; c++) {
    //
    // a, b가 위에서 선언되었으므로,
    // a, b를 여기서 사용할 수 있음.
    console.log(c);
}
a = 1;
b = 2;
//
// c가 위에서 선언되었으므로,
// c를 여기서 사용할 수 있음.
```

그러나 위의 동작방식은 이치에 맞지 않으므로 `let`, `const`는 호이스팅되지 않습니다.

---

**let으로 재작성된 코드 :**

```ts
for (let c = 0; c < 10; c++) {
    console.log(c);
}
let a = 1;
let b = 2;
```

**인터프리터가 실행하는 수도 코드 :**

```ts
{
    [declare] c
    for(c = 0; c<10; c++){
        //
        // a, b가 선언되지 않았으므로,
        // a, b를 사용할 수 없음.
        console.log(c);
    }
    [undeclare] c
}
[declare] a
a = 1
[declare] b
b = 2
//
// c가 효력을 잃었으므로,
// c를 사용할 수 없음.
```

즉, `Block-Level Scope`와 `Function-Level Scope`의 특성은 다음과 같습니다.

-   효력범위
    -   함수레벨 : 함수가 끝나야 선언이 사라진다.
    -   블럭레벨 : 블럭이 끝나야 선언이 사라진다.
-   중복선언 가능여부
    -   함수레벨 : 가능
    -   블럭레벨 : 불가능
-   호이스팅 여부
    -   함수레벨 : 호이스팅 적용됨
    -   블럭레벨 : 호이스팅이 적용되지 않음.

---

---

## Block-Scoped Functions

자바스크립트는 `함수선언문`을 다음과 같이 `호이스팅된 함수표현식`으로 대체되는 것을 표준으로 하고 있습니다.

---

**우리가 작성한 함수선언문 :**

```ts
{
    function hello() {
        console.log("Hello, World!");
    }
}
hello(); // 가능? 불가능?
```

**인터프리터가 해석한 코드 :**

```ts
{
    var hello = function () {
        console.log("Hello, World!");
    };
}
hello(); // 가능!
```

그렇다면 다음 코드는 어떻게 동작할까요?

```ts
{
    function hello() {
        console.log("outer");
    }
    {
        function hello() {
            console.log("inner");
        }
        hello();
    }
    hello();
}
hello();
```

나중에 선언된 `hello`가 먼저 선언된 `hello`를 덮어쓰기 때문에 `"inner"`가 출력되는 것을 예상할 수 있으며, 실제로 옛날 자바스크립트 엔진은 이러한 방식으로 동작합니다.

---

**구버전 엔진 :**

```
inner
inner
inner
```

하지만 `ES6` 부터는 선행함수가 후행함수에 덮어씌어지지 않도록 약간 개선됩니다.

---

**신버전 엔진 :**

```
inner
outer
outer
```

하지만 `ES6`을 사용할 수 없어도 `ES5`만으로 해당 기능을 구현할 수 있습니다. `var`이 함수레벨 스코프라는 특성을 이용하는 것 입니다.

---

**ES 5:**

```ts
(function outerScope() {
    function hello() {
        console.log("outer");
    }
    (function innerScope() {
        function hello() {
            console.log("inner");
        }
        hello();
    })();
    hello();
})();
```

위의 코드는 아래와 같이 동작하기 때문입니다.

```ts
(function outerScope() {
    var hello;
    hello = function () {
        console.log("outer");
    };
    (function innerScope() {
        var hello;
        hello = function () {
            console.log("inner");
        };
        hello();
        //
        // var hello는 여기서 만료됨.
    })();
    hello();
    //
    // var hello는 여기서 만료됨.
})();
```

---

---

# Function

## Arrow Functions

### Delcation

기존보다 더 짧은 함수선언이 가능합니다.

---

**ES 6:**

```ts
const hello = () => "Hello, World!";
const hello = () => {
    return "Hello, World!";
};
```

**ES 5:**

```ts
const hello = function () {
    return "Hello, World!";
};
```

---

---

### LexicalThis

그러나 기존의 함수식과 완전히 같은것은 아닙니다. 기존의 함수표현식의 this는 `호출문에서 봤을 때 dot으로 이어진 1단계 상위객체`로 바인딩되지만, 화살표 함수는 `자신을 생성한 함수의 this`로 바인딩됩니다.

---

**기존 함수표현식 :**

기존의 함수표현식의 this는 `호출문에서 봤을 때 dot으로 이어진 1단계 상위객체` 입니다. 따라서, 기존의 함수표현식의 this는 동적입니다.

```ts
function increase() {
    this.count++;
}

const counter = {
    count: 0,
    increase: increase,
    utils: { increase: increase },
};

increase(); // this: global
counter.increase(); // this: counter
counter.utils.increase(); // this: counter.utils

//
// 출력하면 아래와 같다.
{
    count : 1,
    utils : {
        count : NaN
    }
}
```

**화살표 함수 표현식 :**

화살표 함수의 this는 `자신을 생성한 함수의 this`입니다. 따라서 화살표 함수의 this는 정적입니다.

```ts
let increase;
const counter = {
    count: 0,

    init: function () {
        //
        // 자신을 생성한 함수는 counter.init() 이고,
        // 그 함수의 this가 counter 이므로,
        // 이것의 this도 counter이다.
        const increaseImpl = () => {
            this.count++;
        };

        outerIncreaseImpl = increaseImpl;
    },
};
//
// increaseImpl을 increase로 빼낸다.
counter.init();

//
// 밖으로 빼내어도 increaseImpl의 this는 변하지 않는다.
increase();
```

함수화살표의 이러한 this 바인딩 방식을 `Lexical Binding`이라고 합니다.

---

---

### Constructor

화살표 함수로 만들어진 함수는 `prototype` 프로퍼티를 갖지 않습니다. 즉, 생성자 자격이 없으므로 `new` 키워드와 함께 사용할 수 없습니다.

---

---

## Default Param

주어지지 않은 파라미터의 초기값을 설정할 수 있습니다.

---

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

---

---

## Rest Parameter

이후에 오는 모든 파라미터를 배열로 받을 수 있습니다.

---

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

---

---

## Spread Operator

`ES6`부터 추가되는 개념인 `Iterator Protocol`이 구현된 객체는 `...`연산자를 통해 풀어헤쳐서 순서대로 파라미터에 전달할 수 있습니다. 순회 프로토콜에 대해서는 좀 더 아래에서 설명합니다.

---

`ES6`부터 배열은 `Iterator Protocol`가 구현됩니다.

---

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

---

마찬가지로 문자열도 `Iterator Protocol`이 도입됩니다.

---

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

---

---

# Literals

## String

### BackQuote Literal

백틱문자를 사용하여 문자열을 선언할 수 있습니다.

```ts
const name = "World";
const message = `
        Hi, ${name}!
        Hello, ${name}!
        `;
```

---

---

### Tagged String Literal

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

---

---

### Unicode Literal

이제 유니코드를 완벽하게 지원합니다.

---

**ES6 :**

```ts
"𠮷" === "\u{20BB7}";
```

**ES5 :**

```ts
"𠮷" === "\uD842\uDFB7";
```

---

---

## Number

### Binary, Octal

2진 8진 리터럴을 지원합니다.

---

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

---

---

# Regular Expression

## Keep Matching Position

이제 정규식 결과에서, 각 토큰의 마지막 포지션 값을 알 수 있습니다.

---

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

---

---

# Object

## New Features

### Property Shorthand

변수 이름을 프로퍼티 이름으로 사용할 경우, 객체 선언문을 축약할 수 있습니다.

---

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

---

---

### Computed Property Names

표현식의 결과를 프로퍼티 이름으로 사용할 수 있습니다.

---

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

---

---

### Method Property

객체 선언문에서 함수 선언문을 사용할 수 있습니다.

---

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

---

---

### Destructuring Assignment

객체의 열거가능한 프로퍼티중 일부분을 추출하여 변수로 선언합니다.

---

배열의 각 원소는 열거가능하므로,

---

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

---

객체도 각 프로퍼티도 열거가능하므로,

---

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

---

객체에서 사용 시, 재귀적으로 분해하거나 별칭을 부여할 수 있습니다.

---

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

---

존재하지 않는 프로퍼티에 대해서는 기본값을 부여할 수 있습니다.

---

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

---

함수 시그너쳐에도 적용할 수 있습니다.

---

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

````

---


일치하는 프로퍼티가 없다면 undefined가 적용됩니다.

---


**ES6 :**

```ts
        const list = [7, 42];
        const [a = 1, b = 2, c = 3, d] = list;
        a; // 7
        b; // 42
        c; // 3
        d; // undefined
````

**ES5 :**

```ts
var a = typeof list[0] !== "undefined" ? list[0] : 1;
var b = typeof list[1] !== "undefined" ? list[1] : 2;
var c = typeof list[2] !== "undefined" ? list[2] : 3;
var d = typeof list[3] !== "undefined" ? list[3] : undefined;
```

---

---

## New Methods

### assign()

주어진 src의 모든 프로퍼티를 dest에 할당합니다. 연산의 결과로 dest를 반환합니다. 중복된 프로퍼티는 후행의 것으로 덮어씌어집니다.

---

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

---

---

# String

## New Methods

### repeat()

해당 문자열을 n번 반복한 것을 반환합니다.

```ts
console.log("a".repeat(3)); // "aaa"
```

---

---

### startsWith()

해당 문자열이 특정 워드를 접두사로 갖는다면 true를 반환합니다.

```ts
"Hello, World!".startsWith("H"); // true
"Hello, World!".startsWith("ld!"); // false
```

---

---

### endsWith()

해당 문자열이 특정 워드를 접미사로 갖는다면 true를 반환합니다.

```ts
"Hello, World!".startsWith("H"); // false
"Hello, World!".startsWith("ld!"); // true
```

---

---

### includes()

해당 문자열이 특정 워드를 포함하면 true를 반환합니다.

```ts
"Hello, World!".startsWith("H"); // true
"Hello, World!".startsWith("ld!"); // true
```

---

---

# Number

## New Methods

### isNaN()

NaN이라면 true를 반환합니다. 등호로는 NaN을 검사할 수 없습니다.

```ts
const target = NaN;
target === NaN; // false
Number.isNaN(target); // true
```

---

---

### isFinite()

양의 무한대 또는 음의 무한대라면 true를 반환합니다. 등호로도 검사할 수 있습니다.

```ts
const target = -Infinity;
target === -Infinity; // true
Number.isFinite(target); // true
```

---

---

### isSafeInteger()

매우 큰 숫자, 매우 작은 숫자, 소수점이 포함된 숫자를 저장할 때 정확도를 희생합니다. 정확도가 포기되지 않았다면 true를 반환합니다.

```ts
Number.isSafeInteger(Number.MAX_SAFE_INTEGER * 1); // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER * 2); // false
Number.isSafeInteger(0.1); // false
```

---

---

## New Constant

### EPSILON

IEEE754 방식으로 표현할 수 있는 가장 작은 숫자를 가르킵니다. 이것은 매우 특수한 분야의 연구자를 위해 존재하는 값입니다. 일반적인 프로그래머들은 사용할 일이 없습니다.

---

---

# Math

## New Methods

### trunc()

소수점 버림연산입니다. 기존에는 `ceil`과 `floor`를 사용하여 직접 구현해야 했습니다.

---

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

---

---

### sign()

주어진 수의 부호를 판별합니다.

---

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

---

---

# Modules

모듈 기능이 추가되었습니다. 모듈에 대해서는 또 다른 포스팅에서 다루겠습니다.

---

---

# Classes

## Definition

클래스 문법이 지원됩니다! 그러나 내부적으로는 여전히 프로토타입으로 작동합니다.

---

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

---

---

## Inheritance

상속도 마찬가지입니다.

---

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

---

---

## Expression Based Inheritance

무엇을 상속할지 동적으로 결정할 수 있습니다.

---

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
console.log(new C().name); // "A" or "B"
```

**ES5 :**

```ts
function A() {}
A.prototype.name = "A";

function B() {}
B.prototype.name = "B";

function getBaseClass() {
    return Math.random() < 0.5 ? A : B;
}

function C() {}
C.prototype = Object.create(getBaseClass().prototype);
C.prototype.constructor = C;

console.log(new C().name); // "A" or "B"
```

---

---

## Base Class Access

Super 키워드를 사용하여 부모 클래스에 접근할 수 있습니다.

---

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

---

---

## Static Member

static 키워드로 클래스 수준에서 프로퍼티를 설정할 수 있습니다.

---

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

---

---

## getter/setter

get, set을 바로 클래스 내부에서 사용할 수 있습니다.

---

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
var Box = function () {
    this._val = 0;
};
Box.prototype = {
    get val() {
        return this._val;
    },
    set val(newVal) {
        this._val = newVal;
    },
};
```

---

---

# Symbol Type

프로퍼티를 고유하게 식별할 수 있는 새로운 원시 자료형입니다. 인자로 프로퍼티 이름을 넣을 수 있지만, 프로퍼티 이름이 같다고 해서 같은 심볼을 가르키지 않습니다. 디버깅을 할 때 보기편하게 이름을 설정하는 것 뿐입니다.

```ts
const a = Symbol("x");
const b = Symbol("x");
console.log(a === b); // false
```

---

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

---

객체에 넣어진 심볼기반의 프로퍼티는 `Object.getOwnPropertySymbols`로 찾을 수 있습니다.

```ts
Object.getOwnPropertyNames(obj); // []
Object.getOwnPropertySymbols(obj); // [x, x]
```

---

`Symbol.for`을 사용하면 컨테이너에 저장된 이름이 같은 심볼을 공유합니다. 이것을 `Global Symbol` 이라고 합니다.

```ts
const a = Symbol.for("x");
const b = Symbol.for("x");
console.log(a === b); // true
```

---

특정 용도로 사용되는 Symbol은 스태틱 프로퍼티에 동봉되어 있습니다. 이것을 `Well-Known Symbol` 이라고 합니다.

---

**이터레이터 심볼 :**

-   Symbol.iterator
-   Symbol.asyncIterator

**정규표현식 심볼 :**

-   Symbol.match
-   Symbol.replace
-   Symbol.search
-   Symbol.split

---

---

# Iterator Protocol

## for of

`Iterator Protocol`이 구현된 객체는 `for of` 구문이나 `Spread Operator`를 사용할 수 있으며, 해당 구문을 사용시 프로토콜에 요청하여 요소들을 하나씩 가져옵니다. 커스텀 객체에 프로토콜을 정의하려면 [Symbol.iterator] 프로퍼티에 올바른 함수를 구현하면 됩니다. `Array`와 `String`은 기본적으로 해당 프로토콜이 구현되어 있습니다.

```ts
const list = [1, 2, 3, 4, 5];
const word = "Hello, World!";

//
// for of
for (const val of list) {
    console.log(val);
}
for (const char of word) {
    console.log(char);
}

//
// Spread Operator
const cloned = [...list];
foo(...list);
const chars = [...word];
```

단, `Array`와 `Object`는 기본적으로 유한하므로 별 문제가 없지만, 커스텀 프로토콜은 무한하게 요소를 생성해낼 수 있으므로 주의해주세요. `Infinite Series`에 `for of` 또는 `Spread Operator`를 사용하면 무한루프에 빠질 수 있습니다.

```ts
//
// 1부터 Infinite까지 저장되어 있다고 가정합니다.
const list = [1 ... Infinite];

//
// 반복문이 끝나지 않습니다.
for(const val of list){
    console.log(val);
}

//
// 요것도 끝나지 않습니다!
const cloned = [...list];
```

---

---

## Iterators

순회를 목적으로 하는 반복자입니다. 이 방식으로 프로토콜을 구현하고 싶다면, 다음 요소를 반환하는 함수를 반환하면 됩니다.

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

---

---

## Generators

### 기본 사용법

`yield` 키워드로 요소를 프로토콜에 제공할 수 있으며, `yield`는 독특한 특성이 있기 때문에 `iterator`보다 훨씬 유연하고 정교한 작업이 가능합니다. 함수의 이름 앞에 \*을 붙이면 `Generators`로 정의됩니다.

---

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

---

---

### 프로토콜 읽기/쓰기

먼저 `yield v`표현식을 통해, 값을 프로토콜에 제공할 수 있습니다.

```ts
function* generator() {
    yield "Hello, World!";
    yield 12345;
}
```

위의 형식으로 프로토콜에 제공된 요소들은 `Spread Operator`, `for-of`로 읽을 수 있습니다.

```ts
//
// Spread Operator
const recived = [...generator()];

//
// for-of
const recived = [];
for (const e of generator()) {
    recived.push(e);
}
```

또는 `Generator.prototype.next()`를 사용하여 읽을 수 있습니다. 해당 메서드는 다음과 같은 형태의 값을 반환합니다.

```ts
{
    value : "Hello, World!", // 이번 차례에 제공된 값.
    done : false // 더 이상 제공할 요소가 없다면 true.
}
```

즉, 더 이상 제공할 요소가 없어질 때 까지 `next()`를 반복하면 됩니다.

```ts
const recived = [];
const g = generator();
let next = g.next();
while (next.done === false) {
    recived.push(next.value);
    next = g.next();
}
```

---

---

### 라이프사이클

제네레이터 함수는 기본적으로 프로토콜 읽기 요청을 받은 경우에만 동작합니다. 예를 들어, 아래의 코드는 제네레이터를 생성하긴 했지만, 프로토콜을 읽는 요청이 없으므로 `console.log()`에 도달하지 않습니다.

```ts
function* generator() {
    console.log("Hello, World!"); // 여기에 도달하지 못함.
    yield 1;
}
const g = generator();
```

---

프로토콜 읽기 요청을 감지하면 제네레이터 함수의 내부가 실행되며, 다음 `yield`에 제공된 값을 프로토콜에 넘기고, 즉시 일시정지합니다.

```ts
function* generator() {
    yield 1; // 프로토콜에 값을 넘기고 즉시 일시정지.
    console.log("Hello, World!"); // 여기에 도달하지 못함.
}
const g = generator();
console.log(g.next().value); // 읽기 요청
```

---

다음 읽기 요청이 들어오면, 일시정지했던 지점에서 다시 시작합니다.

```ts
function* generator() {
    yield 1; // 실행 후, 첫 번째 일시정지
    yield 2; // 실행 후, 두 번째 일시정지
    console.log("Hello, World!"); // 여기에 도달하지 못함.
}
const g = generator();
console.log(g.next().value); // 첫 번째 읽기 요청
console.log(g.next().value); // 두 번째 읽기 요청
```

---

마지막으로 제네레이터 함수의 끝(또는 리턴문)에 도달하면 프로토콜에 `return value`를 전달하고 프로토콜을 닫습니다. 리턴문이 없다면 `undefined`가 반환됩니다.

```ts
function* generator() {
    yield 1; // 실행 후, 첫 번째 일시정지
    yield 2; // 실행 후, 두 번째 일시정지
    return 3; // 실행 후, 프로토콜 닫음
}
const g = generator();
console.log(g.next().value); // 1, done: false
console.log(g.next().value); // 2, done: false
console.log(g.next().value); // 3, done: true
```

---

---

### yield 표현식의 값 설정하기

`yield n`도 표현식이므로 값으로 평가될 수 있으며, 기본값은 `undefined` 입니다. (아래의 코드에서 yield)

```ts
function* generator() {
    //
    // yield가 호출되면 즉시 일시정지되므로,
    // console.log는 다음번 읽기요청에서 처리됩니다.
    console.log(yield 1);

    //
    // yield2는 호출되지만,
    // console.log에는 도달하지 않습니다.
    console.log(yield 2);
}
const g = generator();
console.log(g.next().value);
console.log(g.next().value);

//
// will prints
// 1
// undefined
// 2
```

---

`yield n`의 표현식의 결과값을 설정하고 싶다면 `Generator.prototype.next()`에 그것을 넘기면 됩니다.

```ts
function* generator() {
    console.log(yield 1); // yield 1이 "a"로 평가됨.
    console.log(yield 2);
}
const g = generator();
console.log(g.next("a").value);
console.log(g.next("b").value);

//
// will prints
// 1
// "a"
// 2
```

---

---

### Iterator와 비교

-   제네레이터의 함수 한 번당 여러개의 요소를 전달할 수 있지만, 이터레이터는 함수 한 번당 1개의 요소만을 전달할 수 있습니다.
-   제네레이터는 함수가 중간에 일시정지될 수 있지만, 이터레이터는 그렇지 않습니다.
-   제네레이터는 함수 형태로 작성하면, 직접적으로 인자를 받을 수 있습니다.

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

-   모든 이터레이터는 제네레이터로 바꿔쓸 수 있지만, 어떤 제네레이터는 이터레이터로 바꿔쓸 수 없습니다.

---

---

# DataStructure

## Map

`Key-Value` 형태의 딕셔너리 자료구조입니다. `Object`와 유사하지만 다음과 같은 차이점이 있습니다.

---

### 의도하지 않은 키 방지

`Object`를 딕셔너리를 사용하면 프로토타입에 정의된 요소들을 엔트리로 인식할 수 있습니다. 반면에 `Map`은 프로토타입이 없는 `Entry Container`에 엔트리를 담고 있으므로 명시적으로 제공한 키 외에는 어떤 키도 가지지 않습니다.

---

**object :**

```ts
const map = {};
map[1] = 1;
console.log(map[0] !== undefined); // false
console.log(map[1] !== undefined); // true
console.log(map["toString"] !== undefined); // true
```

**map :**

```ts
const map = new Map();
map.set(1, 1);
console.log(map.has(0)); // false
console.log(map.has(1)); // true
console.log(map.has("toString")); // false
```

---

---

### 모든 형태의 키 허용

`Object`의 키는 `String` 또는 `Symbol` 타입이어야 합니다. 만약 이외의 키를 저장한 경우 `String`으로 캐스팅됩니다. 반면에 `Map`은 모든 형태의 키를 허용하며, 키를 손상시키지 않습니다.

---

**object :**

```ts
const map = {};
map[1] = 123;
map[true] = 456;
map[{ name: "AeroCode" }] = 789;

console.log(Object.keys(map));
//
// will print
// ["1", "true", "[object Object]"]
```

**map :**

```ts
const map = new Map();
map.set(1, 123);
map.set(true, 456);
map.set({ name: "AeroCode" }, 789);

console.log([...map.keys()]);
//
// will print
// [ 1, true, { name: 'AeroCode' } ]
```

---

---

### 삽입순서 유지

`Map`은 추가적인 메모리를 사용하여 삽입된 순서를 유지합니다, 그러므로 `Map`을 순회하면 먼저 삽입된 요소가 먼저 출력됩니다. 반면에 `Object`의 키는 삽입순서를 유지하지 않습니다.

---

**object :**

```ts
const map = {};
map[+1] = +1;
map[-1] = -1;
map[0] = +0;

console.log(Object.keys(map));
//
// will print
// [ '0', '1', '-1' ]
```

**map :**

```ts
const map = new Map();
map.set(+1, +1);
map.set(-1, -1);
map.set(0, 0);

console.log([...map.keys()]);
//
// will print
// [1, -1, 0]
```

---

---

### 엔트리 개수 파악

`Map`의 엔트리 개수는 `size` 프로퍼티를 통해 쉽고 빠르게 알아낼 수 있지만, `Object`의 항목 수는 직접 계산해야 하고, 매우 느립니다.

---

**object :**

```ts
const map = {};
const keys = Object.keys(map); // 오버헤드 지점
const size = keys.length;
console.log(size);
```

**map :**

```ts
const map = new Map();
console.log(map.size);
```

---

---

### 순회 프로토콜 지원

`Map`은 순회 프로토콜을 지원하므로 `for of` 구문과 `Spread Operator`를 사용하여 간편하게 순회할 수 있지만, `Object`는 그렇지 않으므로, 먼저 모든 키를 알아내는 과정이 필요합니다.

---

**object :**

```ts
const map = {};
const keys = Object.keys(map); // 키부터 알아내야 한다.
for (const key of keys) {
    const val = map[key];
    console.log(key, val);
}
```

**map :**

```ts
const map = new Map();
map.set(0, 1);
map.set(2, 3);

//
// for of
for (const [key, val] of map) {
    console.log(key, val);
}

//
// spread operator
const entries = [...map];
console.log(entries); // [ [ 0, 1 ], [ 2, 3 ] ]
```

---

---

### 퍼포먼스 비교

데이터의 개수와 연산의 종류 상관없이, Key가 중복될수록 `Object`가 유리하고, Key가 중복되지 않을수록 `Map`이 유리합니다.

---

**object가 유리한 케이스 :**

```ts
map[1] = 1;
map[1] = 2;
map[1] = 3;
map[1] = 4;
...
map[1] = 987654321;
```

**map이 유리한 케이스 :**

```ts
map.set(1, 1);
map.set(2, 2);
map.set(3, 3);
...
map.set(987654321, 987654321);
```

---

---

### 언제 사용해야 하나?

**object를 사용해야 하는 경우 :**

-   저장할 데이터의 키가 중복되는 것이 많을 때. (= 대부분이 갱신일 때)
-   객체인 경우가 더 자연스러운 경우. (= 쓸데없이 객체를 분리하여 맵 형태로 저장하지 말란 의미)

```ts
//
// do
const human = {
    name: "AeroCode",
    age: 25,
    addr: "...",
    hello: function () {
        console.log("Hello, My name is " + this.name + "!");
    },
};

//
// don't
const human = new Map();
human.set("name", "AeroCode");
human.set("age", 25);
human.set("addr", "...");
human.set("hello", function () {
    console.log("Hello, My name is " + this.name + "!");
});
```

**map을 사용해야 하는 경우 :**

-   사용자의 입력에서 키를 받는 경우. (입력으로 `toString`와 같은 것이 들어올 수 있는 경우 `object.prototype.toString`과 충돌)
-   저장된 데이터의 개수의 파악이 중요한 경우.
-   문자열 외의 키를 허용해야 하는 경우.
-   메모리를 좀 더 잡아먹더라도, 삽입된 순서가 중요한 경우.

---

---

## Set

`Key`만 저장할 수 있는 `Map`이라고 생각하면 됩니다. 그것을 제외한 대부분의 특성은 `Map`과 같습니다.

-   삽입순 정렬
-   의도치 않은 키 없음
-   모든 형태의 키 허용
-   순회 프로토콜 지원
-   쉬운 엔트리 개수 파악

`Object`와의 비교결과도 `Map`과 같습니다.

-   중복될수록 object가 유리
-   중복되지 않을수록 set이 유리

---

---

## WeakMap

### 메모리 누수

엔트리의 키로 객체가 사용된 경우, 해당 객체가 스코프에서 사라지더라도 `Map`이 키의 목록을 유지하려고 강하게 참조하고 있기 때문에 `GC`의 대상이 되지 않습니다.

```ts
const map = new Map();
function foo() {
    const obj = {
        hello: function () {
            console.log("Hello, World!");
        },
    };
    map.set(obj, "Hello!");

    //
    // 함수가 끝나는 시점에서 obj는 GC의 수집대상이 되어야 하지만,
    // map이 obj를 강하게 붙잡고 있으므로 GC의 수집대상에서 벗어남.
}
foo();
console.log([...map.keys()]); // [obj];
```

그러나 위와 같은 동작은 메모리 누수의 원인이 될 수 있습니다.

```ts
const map = new Map();
function foo() {
    for (let i = 0; i < 123456789; i++) {
        const key = { idx: i };
        const val = { val: i };
        map.set(idx, val);
    }
}
foo();
console.log(map.size);
//
// will print 123456789
// 메모리가 해제되지 않아...!
```

---

---

### 느슨한 참조

따라서 `Map`에서 키 목록 유지 기능을 빼버린 `WeakMap`이 함께 등장했습니다. 더 이상 키 목록을 유지할 필요가 없기 때문에 키와의 연결이 느슨해지므로 `키로 사용된 객체`는 `GC`에 의해 수집될 수 있습니다.

```ts
const weakMap = new WeakMap();
function foo() {
    const obj = {
        hello: function () {
            console.log("Hello, World!");
        },
    };
    weakMap.set(obj, "Hello!");
    //
    // weakMap은 obj를 붙잡고 있지 않으므로,
    // obj는 GC에 의해 수거될 수 있다!
}
foo();
```

---

---

### Map과의 비교

-   `Map`과 달리 키 목록 유지기능이 없음.
    -   즉, 키 목록을 얻을 수 있는 방법이 없음.
    -   즉, `.keys()` 메서드를 지원하지 않음.
    -   즉, `Iterator Protocol`을 지원하지 않음.
    -   즉, 메모리 누수를 예방할 수 있음.
-   `Map`과 달리 `Primitive Type`을 키로 지정할 수 없음.

---

---

## WeakSet

`WeakMap`과 그 특성이 같습니다.

---

---

# TypedArrays

## 개요

기본적으로는 `Array`와 같지만 숫자값만 저장할 수 있다는 것이 특징입니다. 내부적으로는 `ArrayBuffer`를 사용하여 바이트 단위로 읽기/쓰기를 수행하는 `DataView`의 일종입니다.

```ts
Int8Array;
Uint8Array;
Int16Array;
Uint16Array;
Int32Array;
Uint32Array;
Float32Array;
Float64Array;
```

---

---

## Array와의 비교

`Array`는 동적크기 배열이지만 `TypedArray`는 고정크기 배열입니다. 내부적으로 참조하는 `ArrayBuffer`의 크기를 변경할 수 없기 때문입니다.

```ts
//
// Array는 배열의 크기가 늘어날 수 있다.
const array = new Array(5);
console.log(array.length); // 5
array.push(5);
console.log(array.length); // 6
```

---

`TypedArray`는 `Array`와 다르게 범위를 벗어난 데이터를 저장 시, 손실이 발생할 수 있습니다.

```ts
const array = new Int8Array(1);

//
// 12345 = 0b 00110000 00111001
// Int8Array는 1바이트만 저장할 수 있으므로,
// 12345의 마지막 바이트인 00111001만 저장된다.
array[0] = 12345;

//
// 0b00111001 = 57
console.log(array[0]); // 57
```

---

---

# Promise

## Callback Hell

프로마이즈는 아래와 같은 콜백지옥 문제를 해결하기 위해 등장한 `비동기 작업 실행자`입니다. 콜백지옥은 코드의 패딩을 증가시키고, 가독성을 크게 저하시킵니다.

```ts
do_1st(
    initVal,
    function (result) {
        do_2nd(
            result,
            function (result) {
                do_3nd(
                    result,
                    function (result) {
                        do_4nd(result, function (result) {
                            // ...
                        });
                    },
                    failureCallback_1
                );
            },
            failureCallback_2
        );
    },
    failureCallback_3
);
```

---

---

## Chaining

프로마이즈는 또 다른 프로마이즈와 연결될 수 있는데, 이러한 특성을 사용하여 콜백지옥 문제를 해결합니다. 이것을 `Promise Chaining`이라고 부릅니다.

```ts
promiseObject
    .then(nextCallback_1, failureCallback_1)
    .then(nextCallback_2, failureCallback_2)
    .then(nextCallback_3, failureCallback_3);
```

`failureCallback`이 모두 같다면 다음과 같이 축약할 수 있습니다.

```ts
promiseObject
    .then(nextCallback_1)
    .then(nextCallback_2)
    .then(nextCallback_3)
    .catch(failureCallback);
```

---

---

## 프로마이즈의 상태

프로마이즈는 다음 3가지 중 하나를 갖습니다.

-   대기 (`pending`) : `resolve` 또는 `reject`되지 않은 초기 상태
-   이행 (`fulfilled`) : 프로마이즈가 `resolve`된 상태
-   거부 (`rejected`) : 프로마이즈가 `reject`된 상태

---

```ts
//
// 프로마이즈가 생성될 당시에는 아직 pending 상태.
// 곧, 프로마이즈의 내부로직이 실행됨.
const promiseObject = new Promise((resolve, reject) => {
    console.log("in Promise");
    if (Math.random() < 0.5) {
        //
        // 아래 메서드 실행 후, fulfilled 상태로 변함.
        resolve("Success");
    } else {
        //
        // 아래 메서드 실행 후, rejected 상태로 변함.
        reject("Fail");
    }
});

//
// 이미 프로마이즈가 실행되어 pending 상태에서 벗어났으므로,
// "done" 에 앞서 "in Promise"가 출력됨.
console.log("done");
```

```
in Promise
done
```

---

---

## 콜백과의 차이

`CallBack`은 동기식이지만 `Promise`는 비동기로 작동합니다. 즉, `CallBack`은 후행 코드들을 블럭킹합니다.

---

**CallBack :**

```ts
function doSomething(onSuccess, onFailure) {
    try {
        console.log("in doSomething");
        if (Math.random() < 0.5) {
            throw new Error();
        }
        const result = "Hello, World!";
        onSuccess(result);
    } catch (reason) {
        onFailure(reason);
    }
}

function main() {
    console.log("start");
    doSomething(
        (result) => console.log("success"),
        (reason) => console.log("failure")
    );
    console.log("end");
}
main();
```

```
start
in doSomething
success // or failure
end
```

**Promise :**

```ts
const doSomething = new Promise((resolve, reject) => {
    console.log("in doSomething");
    if (Math.random() < 0.5) {
        const result = "Hello, World!";
        resolve(result); // = return
    } else {
        reject(); // = throw
    }
});

function main() {
    console.log("start");
    doSomething
        .then((result) => console.log("success"))
        .catch((reason) => console.log("failure"));
    console.log("end");
}
main();
```

```
in doSomething
start
end
success // or failure
```

---

---

## 다수의 프로마이즈 관리

### Promise.all()

`iteratorable`에 저장된 프로마이즈가 전부 이행되어야 `fulfilled`, 하나라도 거절되면 `rejected` 상태로 변하는 프로마이즈를 생성합니다.

```ts
function makeRandomPromise() {
    return new Promise((resolve, reject) => {
        if (Math.random() < 0.5) {
            resolve();
        } else {
            reject();
        }
    });
}

const promises = [];
for (let i = 0; i < 3; i++) {
    const promise = makeRandomPromise();
    promises.push(promise);
}

Promise.all(promises)
    .then(() => console.log("모든 프로마이즈가 이행됨."))
    .catch(() => console.log("어떤 프로마이즈가 거절됨."));
```

---

---

### Promise.race()

`iteratorable`에 저장된 프로마이즈 중, 가장 빠르게 상태가 변화한 프로마이즈의 상태를 사용합니다. 즉, 가장 빨리 처리된 프로마이즈의 상태가 이행이라면 `fulfilled`로, 거절이라면 `rejected`로 변화하는 프로마이즈를 생성합니다.

```ts
function makeDelayPromise(delay: number) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (Math.random() < 0.5) {
                resolve(`resolved with ${delay}`);
            } else {
                reject(`rejected with ${delay}`);
            }
        }, delay);
    });
}

const promises = [];
for (let i = 0; i < 3; i++) {
    const delay = Math.floor(Math.random() * 1000);
    const promise = makeDelayPromise(delay);
    promises.push(promise);
}

Promise.race(promises)
    .then((v) => console.log(v))
    .catch((v) => console.log(v));
```

---

---

# Meta Programming

## Proxy

특정 객체에 접근하기 전에 훅을 먼저 실행하는 대리자를 생성합니다.

```ts
const object = {
    name: "Sample Object",
    desc: "Hello, World!",
};

const objectProxy = new Proxy(object, {
    get: function (target, key) {
        if (key in target) {
            return target[key];
        }
        throw new Error(`No such key.`);
    },
});

const name = objectProxy.name; // OK
const addr = objectProxy.addr; // error
```

---

가능한 훅은 다음과 같습니다.

```ts
interface ProxyHandler<T extends object> {
    getPrototypeOf?(target: T): object | null;
    setPrototypeOf?(target: T, v: any): boolean;
    isExtensible?(target: T): boolean;
    preventExtensions?(target: T): boolean;
    getOwnPropertyDescriptor?(
        target: T,
        p: PropertyKey
    ): PropertyDescriptor | undefined;
    has?(target: T, p: PropertyKey): boolean;
    get?(target: T, p: PropertyKey, receiver: any): any;
    set?(target: T, p: PropertyKey, value: any, receiver: any): boolean;
    deleteProperty?(target: T, p: PropertyKey): boolean;
    defineProperty?(
        target: T,
        p: PropertyKey,
        attributes: PropertyDescriptor
    ): boolean;
    enumerate?(target: T): PropertyKey[];
    ownKeys?(target: T): PropertyKey[];
    apply?(target: T, thisArg: any, argArray?: any): any;
    construct?(target: T, argArray: any, newTarget?: any): object;
}
```

---

---

## Reflect

흩어져있는 객체, 함수, 생성자 관련 함수들을 한데 묶어놓은 유틸리티 객체입니다.

```ts
Reflect.apply();
Reflect.construct();
Reflect.defineProperty();
Reflect.deleteProperty();
Reflect.get();
Reflect.getOwnPropertyDescriptor();
Reflect.getPrototypeOf();
Reflect.has();
Reflect.isExtensible();
Reflect.ownKeys();
Reflect.preventExtensions();
Reflect.set();
Reflect.setPrototypeOf();
```

---

---

# Localization

각 나라에 맞는 통화, 날짜 형식으로 포맷팅하는 유틸리티를 제공합니다.
