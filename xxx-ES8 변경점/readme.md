# Async / Await

## Promise의 한계

프로마이즈는 `Call-Back Hell`을 해결하기엔 충분했지만, 프로마이즈 객체를 생성하는 구문과, 끝없는 `Promise Chain` 때문에 코드의 복잡성은 크게 증가했습니다. `ES8`에 도입된 `await / async`는 프로마이즈의 체이닝을 줄인 채, 비동기 로직을 실행할 수 있도록 도와줍니다.

**ES 8:**

```ts
function getAsyncRandomValue() {
    return new Promise((resolve, reject) => {
        resolve(Math.random());
    });
}

function main() {
    getAsyncRandomValue()
        .then(onSuccess_1)
        .then(onSuccess_2)
        .then(onSuccess_3)
        // ...
        .then(onSuccess_999)
        .catch(onFailure);
}
```

**ES 7:**

```ts
async function getAsyncRandomValue() {
    return Math.random();
}

async function main() {
    try {
        const randomValue = await getAsyncRandomValue();
        onSuccess_1(randomValue);
        onSuccess_2(randomValue);
        // ...
        onSuccess_999(randomValue);
    } catch (reason) {
        onFailure(reason);
    }
}
```

---

---

## async

함수 또는 화살표 함수 앞에 `async` 키워드를 붙이면, 내부적으로 해당 함수를 감싸는 프로마이즈가 생성됩니다.

**ES 8:**

```ts
//
// 클래식 함수
async function getHello() {
    // 3초를 기다린다...
    return "Hello, World!";
}

//
// 화살표 함수
const getHello = async () => {
    // 3초를 기다린다...
    return "Hello, world!";
};
```

**ES 7:**

```ts
function getHello() {
    return new Promise((resolve, reject) => {
        // 3초를 기다린다...
        setTimeout(() => resolve("Hello, World!"), 3000);
    });
}
```

---

---

## await

프로마이즈의 앞에 `await` 키워드를 붙이면, 해당 프로마이즈가 처리될 때 까지 기다리고, `fulfilled`된 값을 꺼내올 수 있습니다. 이 때, 프로마이즈 내부의 값을 밖으로 꺼내기 위해, 내부적으로 `Generator`와 `Generator.prototype.next()`를 사용합니다. 자세한 사항은 `yield 표현식 결과값 지정하기`를 참조해주세요.

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

## 특징 정리

-   `async/await`는 프로마이즈를 대체하기 위해 대두됨.
    -   프로마이즈 체인이 끊기면 안되고...
    -   프로마이즈 객체를 매번 생성하는 것도 보기 나쁘고...
    -   비동기 작업의 워크플로우를 코드상에서 쉽게 확인하고 싶어!
-   `async`는 프로마이즈를 생성하는 문법 설탕.
-   `await`는 제너레이터를 사용하여 비동기 작업을 동기식으로 표현하는 문법.
    -   `yield`의 평가값을 설정하는 기능을 사용하여 구현.

---

---

# Function

## New Features

### Param Trailing Commas

함수를 호출하거나 정의할 때, 인자의 끝으로 컴마가 허용됩니다.

```text
function foo(param1, param2, ) {}
foo(123, 'abc', );
```

---

---

# Object

## New Methods

### entries()

`Object.keys()`와 유사하게, 자신만의 열거가능한 프로퍼티들의 키와 값의 페어를 배열 형태(`Array<[key, val]>`)로 리턴합니다. 즉, 아래의 프로퍼티는 무시됩니다.

-   상위 프로토체인에 있는 부모들의 프로퍼티
-   열거불가 속성으로 정의된 프로퍼티

```ts
//
// 동물 클래스
function Animal(type) {
    this.type = type;
}
Animal.prototype.isAnimal = true;

//
// 고양이 클래스
function Cat(name) {
    Animal.call(this, "cat");
    this.name = name;
    Object.defineProperty(this, "address", {
        value: "secrect",
        // writable: false,
        // enumerable: false,
        // configurable: false,
    });
}
Cat.prototype = Object.create(Animal.prototype);
Cat.prototype.constructor = Cat;
Cat.prototype.age = 0;

//
// 테스팅
const cat = new Cat("navi");
console.log(Object.entries(cat));

//
// will prints
// [ [ 'type', 'cat' ], [ 'name', 'navi' ] ]
```

**출력이 안된 프로퍼티 :**

-   `Animal.prototype.isAnimal` : 상위 프로토체인의 프로퍼티는 무시됩니다.
-   `Cat.address` : 열거불가 프로퍼티는 무시됩니다.

---

---

### values()

기본적으로는 `keys()`, `entries()`와 같지만 값만 반환합니다.

```ts
const cat = new Cat("navi");
console.log(Object.entries(cat));

//
// will prints
// [ 'cat', 'navi' ]
```

---

---

### getOwnPropertyDescriptors()

상위 프로토체인의 프로퍼티를 제외한 모든 프로퍼티의 디스크립터를 가져옵니다.

```ts
const cat = new Cat("navi");
console.log(Object.getOwnPropertyDescriptors(cat));
```

**will prints :**

```ts
{
    type: {
        value: "cat",
        writable: true,
        enumerable: true,
        configurable: true,
    },
    name: {
        value: "navi",
        writable: true,
        enumerable: true,
        configurable: true,
    },
    address: {
        value: "secrect",
        writable: false,
        enumerable: false,
        configurable: false,
    },
};
```

---

---

# String

## New Methods

### padStart()

글자수가 맞춰지도록 문자열의 앞에 적절한 패딩을 삽입합니다. 아래의 예제에서 `padStart`는 문자열 `"123"`이 5글자가 되도록 앞쪽에 2글자 공백을 삽입합니다.

```ts
function wrapString(str) {
    return `"${str}"`;
}
const str = "123".padStart(5);
console.log(wrapString(str));
//
// will prints
// "  123"
```

두 번째 인자로 삽입할 문자열을 지정할 수 있습니다.

```ts
function wrapString(str) {
    return `"${str}"`;
}
const str = "123".padStart(10, "abc");
console.log(wrapString(str));
//
// will prints
// "abcabca123"
```

---

---

### padEnd()

기본적으로는 `padStart()`와 같지만 패딩을 뒤쪽에 삽입합니다.
