`ES6 (ES2015)` 에서 추가된 기능들에 대해 정리합니다. 기존의 `ES5`를 사용하여 구현할 수 있는 경우가 대부분이므로, 구현방법을 함께 알아두면 좋습니다.

---

---

# Scope

## let

중복으로 선언할 수 없는 변수를 생성합니다.

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

**ES6 :**

```ts
const a = 123;
a = 999; // error
```

**ES5 :**

```ts
Object.defineProperty(globalThis, "a", {
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

**구버전 엔진 :**

```
inner
inner
inner
```

하지만 `ES6` 부터는 선행함수가 후행함수에 덮어씌어지지 않도록 약간 개선됩니다.

**신버전 엔진 :**

```
inner
outer
outer
```

하지만 사정이 있어 `ES6`을 사용할 수 없어도 `ES5`만으로 해당 기능을 구현할 수 있습니다. `var`이 함수레벨 스코프라는 특성을 이용하는 것 입니다.

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

화살표 함수로 만들어진 함수는 `prototype` 프로퍼티를 갖지 않으므로, 생성자 자격이 없으므로 `new` 키워드와 함께 사용할 수 없습니다.

---

---

## Default Param

주어지지 않은 파라미터의 초기값을 설정할 수 있습니다.

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
Rectangle.prototype = Shape.prototype;
Rectangle.prototype.constructor = Rectangle;
```

---

---

## Expression Based Inheritance

무엇을 상속할지 동적으로 결정할 수 있습니다.

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
C.prototype = getBaseClass().prototype;
C.prototype.constructor = C;

console.log(new C().name); // "A" or "B"
```

---

---

## Base Class Access

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

---

---

## Static Member

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

---

---

## getter/setter

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

`yield` 키워드로 요소를 제공합니다. `iterator`보다 훨씬 유연하고 정교한 작업이 가능합니다. 함수의 이름 앞에 \*을 붙이면 `Generators`로 정의됩니다.

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

---

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

---

제네레이터는 다르게 여러곳에서 `yield`를 사용할 수 있으므로, 이터레이터보다 더 유연하게 요소를 제공할 수 있습니다.

```ts
//
// 이터레이터
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

//
// 제너레이터
const generatorExample = {
    *[Symbol.iterator]() {
        let count = 0;

        //
        // 여러 군데에서 yield 가능!
        yield count++;
        while (true) yield count++;
        yield count++;
    },
};
```

---

---

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
