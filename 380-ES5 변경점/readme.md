`ES5`는 기본적으로 편의성 개선에 초점이 맞추어져 있습니다. 기존에 프로그래머가 구현해야 했던 기능이나, 있으면 편리하겠다 싶을만한 기능들이 해당 버전에서 제공됩니다. 눈여겨 보아야 할 변경점은 `use strict` `Array Methods` `JSON Support` 입니다.

---

**톺아보기 :**

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

---

---

# use strict

코드의 상단에 `"use strict"`를 선언하면 평소라면 무시할만한 `위험행동`을 더 이상 무시하지 않고 익셉션을 발생시킵니다.

---

**정의되지 않은 변수 사용 :**

```ts
console.log(hello); // error.
```

**delete 구문으로 변수 삭제 :**

```ts
var x = 0;
delete x; // error.
```

**eval로 만든 변수 사용 :**

```ts
eval("var x = 2");
console.log(x); // error.
```

**미래에 사용할 예약어를 변수의 이름으로 사용 :**

```ts
const yield = 3;
```

---

---

# Function

## New Methods

### bind()

함수의 `this`는 호출문에서 봤을 때 닷(`.`)으로 이어진 1단계 상위객체입니다. 즉, 함수의 `this`는 동적으로 변할 수 있습니다.

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

`bind()`는 `this`를 정적으로 고정한 새로운 함수를 반환합니다.

```ts
function increase() {
    this.count++;
}

const counter = {
    count: 0,
    increase: increase,
    utils: { increase: increase },
};

increase.bind(counter)();
counter.increase.bind(counter)();
counter.utils.increase.bind(counter)();

//
// 출력하면 아래와 같다.
{
    count : 3,
}
```

---

---

# String

## New Features

### Index Bracket

인덱스 괄호를 사용하여 글자단위 접근이 가능합니다.

```ts
const word = "Hello, World!";
const char = word[0]; // "H"
```

---

---

### Strings Over Multiple Lines

여러줄에 걸쳐 문자열을 정의할 수 있습니다.

```ts
"Hello, \
World!" === "Hello, World!"; // true
```

---

---

## New Methods

### trim()

양 옆의 연속된 공백과 개행을 제거합니다.

```ts
const word = " \n\n  Hello, World!   \n\n";
console.log(word.trim()); // "Hello, World!"
```

---

---

### charAt()

문자열에서 n번째 문자를 반환합니다.

```ts
const word = "Hello, World!";
const char = word.chatAt(0); // "H"
```

---

---

# Array

## New Features

### Trailing Commas

콤마로 요소 정의를 끝낼 수 있습니다.

```
const array = [ 1, 2, 3, ];
```

---

---

## New Methods

### isArray()

해당 객체가 Array 생성자로 만들어졌는지 검사합니다.

```ts
Array.isArray([]); // true
Array.isArray({}); // true
```

---

---

### forEach()

각 요소를 한번씩 방문합니다.

```ts
const list = [2, 4, 6];
list.forEach(function (val, idx, array) {
    console.log(val, idx, array);
});
//
// will print
// 2, 0, [2, 4, 6]
// 4, 1, [2, 4, 6]
// 6, 2, [2, 4, 6]
```

---

---

### forEach()

각 요소를 다른 값으로 변환한 새로운 배열을 반환합니다.

```ts
[2, 4, 6]
    .map(function (val, idx, array) {
        return {
            v: val,
            i: idx,
        };
    })
    .forEach(function (val, idx, array) {
        console.log(val);
    });
//
// will print
// [
//    { v:2, i:0 },
//    { v:4, i:1 },
//    { v:6, i:2 },
// ]
```

---

---

### filter()

조건결과가 true인 요소만 남긴 새로운 배열을 반환합니다.

```ts
[0, 1, 2, 3, 4, 5, 6]
    .filter(function (val, idx, array) {
        return val <= 3;
    })
    .forEach(function (val, idx, array) {
        console.log(val);
    });
//
// will print
// 0, 1, 2, 3
```

---

---

### reduce()

요소를 왼쪽부터 순회하면서 acc에 누적을 진행합니다. 순회가 끝났으면 acc를 반환합니다.

```ts
const array = [1, 1, 1, 9];
const count = array.reduce(function (acc, val, idx, array) {
    if (acc[val] === undefined) {
        acc[val] = 0;
    }
    acc[val]++;
}, new Object());

console.log(count);
//
// will print
// {
//     1 : 3,
//     9 : 1,
// }
```

---

---

### reduceRight()

요소를 오른쪽 순회하면서 acc에 누적을 진행합니다. 순회가 끝났으면 acc를 반환합니다.

---

---

### every()

모든 요소가 조건을 만족했을 때 true를 반환합니다.

```ts
const array = [1, 2, 3, 4];

array.every(function (val, idx, array) {
    return val !== 3;
}); // false

array.every(function (val, idx, array) {
    return val !== 5;
}); // true
```

---

---

### some()

하나라도 조건을 만족하는 요소가 있다면 true를 반환합니다.

```ts
const array = [1, 2, 3, 4];

array.some(function (val, idx, array) {
    return val === 5;
}); // false

array.some(function (val, idx, array) {
    return val === 4;
}); // true
```

---

---

### indexOf()

찾고싶은 값이 배열에서 몇번째에 있는지 반환합니다. 엄격한 같음(`===`)을 통해 비교하며, 없다면 `-1`을 반환합니다. 시작점을 포함하여 우측으로 진행합니다.

```ts
const array = [1, 2, 1, 2, 3];

array.indexOf(String(2)); // -1
array.indexOf(Number(2)); // +1
array.indexOf(Number(2), 1); // +1
array.indexOf(Number(2), 2); // +3
```

---

---

### lastIndexOf()

기본적으로는 `indexOf()`와 같지만, 시작점을 포함하여 좌측으로 진행합니다.

```ts
const array = [1, 2, 1, 2, 3];

array.lastIndexOf(String(2)); // -1
array.lastIndexOf(Number(2)); // +3
array.lastIndexOf(Number(2), 3); // +3
array.lastIndexOf(Number(2), 2); // +1
```

---

---

# Object

## New Features

### Trailing Commas

콤마로 프로퍼티 정의를 끝낼 수 있습니다.

```
const object = {
    x : 1,
    y : 2,
}
```

---

---

### getter, setter

객체에 특정 속성을 읽거나 쓰려고 할 때, 해당 함수가 호출됩니다. 올바르지 않은 값의 대입을 막거나, 현재 값을 더욱 처리해야 할 때 유용하게 사용됩니다.

```ts
const human = {
    //
    // private property
    _age: 15,

    //
    // age에 대해 읽으려고 하면 아래의 함수가 실행됨.
    get age() {
        return "__" + this._age + "__";
    },

    //
    // age에 대해 적으려고 하면 아래의 함수가 실행됨.
    set age(newAge: number) {
        if (0 <= newAge) {
            this._age = newAge;
        }
    },
};
human.age = -12345; // will ignored
console.log(human.age); // "__15__"
```

프로퍼티 이름과 getter, setter의 이름이 같다면 무한루프가 발생합니다.

```ts
const human = {
    age: 15,

    get age() {
        //
        // 바로 아랫줄은 다시 get age()를 호출합니다.
        const thisAge = this.age;
        return thisAge;
    },
};
```

---

---

## New Methods

### defineProperty()

프로퍼티 이름과 프로퍼티 디스크립터를 사용하여, 특정 객체에 새 프로퍼티를 생성합니다.

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

---

---

### defineProperties()

여러개의 프로퍼티 디스크립터가 포함된 객체를 사용하여, 특정 객체에 새 프로퍼티들을 생성합니다.

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

---

---

### getOwnPropertyDescriptor()

(프로토체인을 탐색하지 않고) 특정 객체가 갖고있는 특정 프로퍼티의 디스크립터를 가져옵니다.

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

---

---

### getOwnPropertyNames()

(프로토체인을 탐색하지 않고) 특정 객체가 갖고있는 모든 고유 프로퍼티의 이름들을 가져옵니다.

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

---

---

### keys()

(프로토체인을 탐색하지 않고) 특정 객체의 열거 가능한 모든 고유 프로퍼티의 이름들을 가져옵니다.

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

---

---

### preventExtensions()

새로운 프로퍼티의 등록을 막습니다. 새 프로퍼티 등록을 시도하면 엄격모드에서는 익셉션이 발생합니다. 엄격모드가 아니라면 대입문을 통한 시도는 익셉션이 발생하지 않습니다. (단, 시도 자체는 무시됩니다.)

```ts
const obj = { x: 1 };
Object.preventExtensions(obj);

obj.y = 2;
Object.defineProperty(obj, "y", { value: x });
```

---

---

### isExtensible()

특정 객체에 새로운 프로퍼티를 등록할 수 있다면 true를 반환합니다.

```ts
const obj = {};
Object.isExtensible(obj); // true
```

---

---

### seal()

신규 프로퍼티 등록을 막고, 기존의 프로퍼티 삭제도 막습니다. 즉, 프로퍼티의 형태를 밀봉(유지)합니다. 엄격모드에서는 항상 익셉션이 발생하지만, 엄격모드가 아니라면 대입문 또는 delete 호출은 익셉션이 발생하지 않습니다. (단, 해당 시도 자체는 무시됩니다.)

```ts
const obj = { x: 1 };
Object.seal(obj);

delete obj.x;
obj.y = 2;
```

---

---

### isSealed()

`sealed()`가 적용되었다면 true를 반환합니다.

---

---

### freeze()

특정 객체에서 프로퍼티 추가, 삭제, 값의 변경을 방지합니다. `__proto__`도 프로퍼티이므로, 당연히 프로토타입도 변경될 수 없습니다. 엄격모드에서는 항상 익셉션이 발생합니다. 엄격모드가 아닌 경우 대입연산자, `delete`에서는 익셉션이 발생하지 않습니다. (단, 시도 자체는 무시됩니다.)

```ts
const obj = {
    x: 0,
};
Object.freeze(obj);

obj.x = 1; // will ignored
obj.y = 2; // will ignored
delete obj.x; // will ignored
```

---

---

### isFrozen()

`freeze()`가 적용되었다면 true를 반환합니다.

---

---

### create()

`prototype object`를 상속받은 빈 객체를 생성합니다. `null`을 전달하면 프로토타입이 존재하지 않는 객체를 생성할 수 있습니다.

```ts
const o1 = {};
const o2 = Object.create(Object.prototype);
const o3 = Object.create(null);

console.log(o1 instanceof Object); // true
console.log(o2 instanceof Object); // true
console.log(o3 instanceof Object); // false
console.log(o3.__prototype__); // undefined
```

두 번째 인자로 `PropertyDescriptorsObject`를 사용하여 프로퍼티를 부여할 수 있습니다.

```ts
const obj = Object.create(null, {
    val: {
        value: 12345,
    },
});

console.log(obj);
//
// will print
// {
//     val : 12345
// }
```

위의 특성을 사용하여 고전적인 상속을 구현할 수 있습니다.

```ts
//
// 부모 클래스
function Shape() {
    this.center = {
        x: 0,
        y: 0,
    };
}
Shape.prototype.moveTo = function (x, y) {
    this.center.x = x;
    this.center.y = y;
};

//
// 자식 클래스
function Rect() {
    Shape.call(this);
}
Rect.prototype = Object.create(Shape.prototype);
Rect.prototype.constructor = Rect;

//
// 테스팅
const rect = new Rect();
console.log(rect instanceof Rect); // true
console.log(rect instanceof Shape); // true
console.log(rect.moveTo !== undefined); // true
```

프로토체인은 기본적으로 1차원 형태이므로, 다중상속을 수행하려면 매번 프로토체인의 끝에 프로토타입 오브젝트를 등록해야 합니다. 아래 코드는 그러한 역할을 하는 함수입니다.

```ts
function mixin() {
    var superClasses = arguments;
    var holder = Object.create(Object.prototype);
    for (var i = superClasses.length - 1; 0 <= i; i--) {
        //
        // 프로토체인의 끝을 찾고,
        var tail = holder;
        while (tail.__proto__.constructor !== Object) {
            tail = tail.__proto__;
        }

        //
        // 프로토체인의 끝을 부모 클래스로 대체한다.
        tail.__proto__ = Object.create(superClasses[i].prototype);
    }
    return holder;
}
```

위의 함수를 사용하여 다중상속을 구현할 수 있습니다. 이 때, 선행 프로퍼티는 후행 프로퍼티에 의해 가려집니다.

```ts
//
// A 클래스 선언
function A() {
    this.isA = true;
}
A.prototype.hello = function () {
    console.log("Hello, A!");
};

//
// B 클래스 선언
function B() {
    this.isB = true;
}
B.prototype.hello = function () {
    console.log("Hello, B!");
};

//
// A, B를 동시에 상속받는 클래스 선언
function AB() {
    A.call(this);
    B.call(this);
}
AB.prototype = mixin(A, B);
AB.prototype.constructor = AB;

//
// 테스팅
const ab = new AB();
console.log(ab); // AB { isA: true, isB: true }
console.log(ab instanceof AB); // true
console.log(ab instanceof A); // true
console.log(ab instanceof B); // true
ab.hello(); // "Hello, B!"
```
