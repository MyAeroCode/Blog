`ECMA International`에서 공식적으로 `ES6`와 같은 대규모 업데이트는 이제 없다고 발표했습니다. 일년마다 이런 대규모 업데이트가 나왔다간 큰일이겠죠. 다만 `ES6`와 같은 벼락치기를 방지하고자 이제부터는 매년 6월에 새로운 버전이 릴리즈됩니다. 다행히도 `ES7`의 변경점은 딱 2개입니다. 중요한 개념도 없으므로 훑어보기만 하면 됩니다.

---

**톺아보기 :**

-   `Array`
    -   `New Methods`
        -   `Array.prototype.includes`
-   `Exponentiation Operator`

---

---

# Array

## New Methods

### includes()

주어진 지점부터 탐색을 시작하여 특정 값의 존재여부를 반환합니다. 일치여부는 `===` 연산자가 사용됩니다.

---

**ES 7:**

```ts
const list = ["1", 2, 3, 4, 5];
list.includes(1); // false
list.includes(2); // true
list.includes(2, 2); // false
```

**ES 6:**

```ts
const list = ["1", 2, 3, 4, 5];
list.indexOf(1) !== 1; // false
list.indexOf(2) !== -1; // true
list.indexOf(2, 2) !== -1; // false
```

---

---

# Exponentiation Operator

`Math.pow`의 연산자 버전입니다.

---

**ES 7:**

```ts
const a = 2 ** 3;
```

**ES 6:**

```ts
const a = Math.pow(2, 3);
```
