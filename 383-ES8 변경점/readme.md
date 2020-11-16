`ES8`에서 새롭게 추가된 기능들은 적지만 그것을 엎어버릴 정도로 `Async/Await`는 심오합니다. 특히 `ES7`로도 구현할 수 있기에, 깐깐한 기술면접에서는 `Async/Await`를 `ES7`로 구현해보라는 질문도 나옵니다. `Async/Await`를 제외한 다른 변경점은 단순한 편의점 개선이므로 편안하게 읽어내려가면 됩니다.

---

**톺아보기 :**

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

---

---

# Async / Await

## Promise의 한계

프로마이즈는 `Call-Back Hell`을 해결하기엔 충분했지만, 프로마이즈 객체를 생성하는 구문과, 끝없는 `Promise Chain` 때문에 코드의 복잡성은 크게 증가했습니다. `ES8`에 도입된 `await / async`는 프로마이즈의 체이닝을 줄인 채, 비동기 로직을 실행할 수 있도록 도와줍니다.

---

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
        const v = await getAsyncRandomValue();
        const a = onSuccess_1(v);
        const b = onSuccess_2(a);
        const c = onSuccess_3(b);
    } catch (reason) {
        onFailure(reason);
    }
}
```

---

---

## async

`Async/Await`는 제네레이터로 구현됩니다. 함수 또는 화살표 함수 앞에 `async` 키워드를 붙이면 내부 로직을 감싸는 제네레이터가 생성되고, 해당 제너레이터를 `await 구현체`가 실행합니다.

---

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
//
// 클래식 함수
function getHello() {
    function* getHello() {
        // 3초를 기다린다...
        return "Hello, World!";
    }
    return __await(this, undefined, getHello);
}

//
// 화살표 함수
const getHello = () => {
    function* getHello() {
        // 3초를 기다린다...
        return "Hello, World!";
    }
    return __await(this, undefined, getHello);
};
```

---

---

## await

`async`에 의해 내부로직이 제너레이터가 되었습니다. 여기서 상기시켜야 할 것은, 제너레이터에 `yield 표현식의 결과값`을 제너레이터 외부에서 지정할 수 있다는 것 입니다. 자세한 사항은 `ES7 변경점 - yield 표현식 결과값 지정하기`를 참조해주세요.

```ts
function* generator() {
    const hello = yield 1; // yield 1의 결과값을 외부에서 "Hello, World!"로 대체.
    console.log(hello);
}
const g = generator();
g.next();
g.next("Hello, World!");
//
// will prints
// Hello, World!
```

정리하자면 `next()`를 사용하여 제너레이터 내부에 비동기적으로 값을 전달할 수 있으며 `yield`는 자신이 호출된 직후에 일시정지되므로 `다음번 next()`가 호출되지 않는 한, 앞으로 나아가지도 않습니다. 이러한 특징을 사용하여 `await`를 구현할 수 있습니다.

---

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
function __await(thisArg, _arguments, generator) {
    //
    // value를 프로마이즈로 변환하는 함수.
    // 이미 프로마이즈라면 그대로 반환한다.
    function adopt(value) {
        return value instanceof Promise
            ? value
            : new Promise(function (resolve) {
                  resolve(value);
              });
    }

    //
    // 프로마이즈로 감싸지 않으면 동기적으로 수행됨에 주의.
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

        //
        // 내부로직이 끝날 때 까지 반복.
        function step(result) {
            result.done
                ? resolve(result.value)
                : adopt(result.value).then(fulfilled, rejected);
        }
        step(generator.next());
    });
}

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
    -   프로마이즈 체인이 안 끊기게 관리하는 것도 힘들고...
    -   프로마이즈 객체를 매번 생성하는 것도 보기 나쁘고...
    -   비동기 작업의 워크플로우를 코드상에서 쉽게 확인하고 싶어...
-   `async/await`는 제너레이터를 사용하여 구현됨.
    -   `next()`를 사용하여 `yield`의 평가값을 비동기적으로 전달.
    -   `next()`를 호출되기 전까지는 제너레이터 내부가 일시정지됨.

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
