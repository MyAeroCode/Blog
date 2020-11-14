# String

## New Methods

### trimStart()

연속된 좌측 공백문자가 삭제된 무낮열을 반환합니다. 이 함수는 `trimLeft()`를 통해서도 사용할 수 있습니다.

```ts
console.log("  Hello, World!  ".trimStart());
console.log("  Hello, World!  ".trimLeft());
//
// will prints
// "Hello, World!  "
```

---

---

### trimEnd()

연속된 우측 공백문자가 삭제된 무낮열을 반환합니다. 이 함수는 `trimRight()`를 통해서도 사용할 수 있습니다.

```ts
console.log("  Hello, World!  ".trimEnd());
console.log("  Hello, World!  ".trimRight());
//
// will prints
// "  Hello, World!"
```

---

---

# Object

## New Methods

### fromEntries()

`[key, value]의 배열` 또는 `Map`을 사용하여 객체를 생성할 수 있습니다.

**via [key, value][] :**

```ts
const a = { x: 1, y: 2 };
const entries = Object.entries(a);
// entries: [["x", 1], ["y", 2]]
const aPrime = Object.fromEntries(a);
```

**via Map :**

```ts
const aMap = new Map();
aMap.set("x", 1);
aMap.set("y", 2);
const aObj = Object.fromEntries(aMap);
```

---

---

# Array

## Enhanced

### `stable sort()`

이전에는 동레벨 요소들의 상대적 위치가 변화했지만, 이제부터는 동레벨들의 상대적 위치가 변화하지 않습니다.

---

---

## New Methods

### flat()

모든 배열인 요소들의 깊이를 하나 줄입니다.

```ts
const arr = [
    [1, 2, 3],
    [
        [4, 5],
        [6, 7, [8]],
    ],
];

console.log(arr.flat());
//
// will prints
// [1, 2, 3, [4, 5], [6, 7, [8]]];
```

숫자를 넘겨서 재귀적으로 깊이를 줄일 수 있습니다.

```ts
const arr = [
    [1, 2, 3],
    [
        [4, 5],
        [6, 7, [8]],
    ],
];

console.log(arr.flat(2));
//
// will prints
// [1, 2, 3, 4, 5, 6, 7, [8]];
```

깊이를 알 수 없다면 `Infinity`를 넘기면 됩니다.

```ts
arr.flat(Infinity);
```

---

---

### flatMap()

`map()`이 진행된 후에 `flat(1)`이 진행됩니다.

```ts
const arr = [
    [1, 2],
    [[3, 4], [5]],
];

arr.flatMap((subarr) => {
    const doubled = [].concat(subarr).concat(subarr);
    return doubled;
});
// [1, 2, 1, 2, [3, 4], [5], [3, 4], [5]];
```

---

---

# Function

## Enhanced

### toString()

함수에서 `toString()`은 함수의 소스코드를 반환합니다. 이전에는 개행, 주석, 공백이 삭제되었지만, 이제부터는 개행, 주석, 공백도 포함하여 반환합니다.

```ts
function foo() {
    //
    // will prints "Hello, World!"
    console.log("Hello, World!");
}

console.log(foo.toString());
//
// will prints
`function foo() {
    //
    // will prints "Hello, World!"
    console.log("Hello, World!");
}`;
```

---

---

# Symbol

## New Features

### description

읽기전용 속성으로, 심볼에 넘겨진 선택적 설명자를 가져올 수 있습니다.

```ts
const sym = Symbol("Hello, World!");
console.log(sum.description); // "Hello, World!";
```

---

---

# JSON

## Enhanced

### JSON ⊂ ECMAScript

`JSON SuperSet`으로도 불립니다. ES9 이하에서는 문자열에 `U+2028(line separator)` 문자와 `U+2029(paragraph separator)` 문자를 포함할 수 없었지만, JSON 표준에서는 위의 두 문자를 허용했기 때문에 에러가 발생할 수 있었습니다. ES10 부터는 ECMA에서도 위의 두 문자를 허용합니다. 해당 개선으로 인해 ECMA는 JSON를 완전히 호환합니다.

**ES 10:**

```ts
const word1 = `\u{2028}`; // OK
const word2 = `\u{2029}`; // OK
```

**ES 9:**

```ts
const word1 = `\u{2028}`; // Error
const word2 = `\u{2029}`; // Error
```

---

---

### Well-formed stringify()

이전에는 일부 유니코드가 포함된 객체를 문자열로 바꿀 때 폰트가 깨졌습니다.

**ES 9:**

```ts
const obj = {
    key: "\uD83D",
};
const stringified = JSON.stringify(obj);
console.log(stringified);
//
// will prints
// "{key:"�"}"
```

이제는 유니코드 코드를 그대로 표현합니다.

**ES 10:**

```ts
const obj = {
    key: "\uD83D",
};
const stringified = JSON.stringify(obj);
console.log(stringified);
//
// will prints
// "{key:"\uD83D"}"
```

---

---

# Optinal Catch Binding

catch에서 파라미터를 받지 않을경우 소괄호를 생략할 수 있습니다.

**ES 10:**

```ts
try {
    throw Error();
} catch {
    //
}
```

**ES 9:**

```ts
try {
    throw Error();
} catch (unusedReason) {
    //
}
```
