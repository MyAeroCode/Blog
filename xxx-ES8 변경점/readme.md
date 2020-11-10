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
async function getHello() {
    // 3초를 기다리는 프로마이즈가 fulfilled 될 때 까지 대기한다.
    await new Promise((resolve, reject) => {
        // 3초를 기다리고 resolve 한다.
        setTimeout(resolve, 3000);
    });

    return "Hello, World!";
}

async function main() {
    const hello = await getHello();
    console.log(hello);
}
```

**ES 7:**

```ts
//
// 제네레이터를 이용한 await 문법의 구현
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

function getHello() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve("Hello, World!");
        }, 3000);
    });
}

function main() {
    return __await(this, undefined, function* () {
        const hello = yield getHello();
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
