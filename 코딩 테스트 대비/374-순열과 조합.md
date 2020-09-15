> # 개요

`순열`<sup>Permutation</sup>과 `조합`<sup>Combination</sup>은 코딩테스트에서 매우 빈번하게 사용되는 도구 중 하나입니다. `어떤 배열의 순열 또는 조합을 구하라!`라는 직접적인 문제는 출제되지 않지만, 이것을 사용해야 문제가 풀리는 경우가 많으므로 `순열과 조합 알고리즘 구현`에 대해 정리하고자 합니다.

---

해당 포스팅에서는 `TypeScript`를 사용하여 구현했지만, 다른 언어에서도 충분히 구현할 수 있을만큼 상세하게 설명하겠습니다.

---

> # 순열 (Permutation)

## 정의

길이가 $n$인 배열에서 $r$개의 요소를 차례대로 뽑아 새로운 배열을 만들었을 때, 가능한 모든 배열의 합입니다. 예를 들어 `[1, 2, 3, 4]`라는 배열에서 3개의 요소를 뽑아 새로운 배열을 만든다고 한다면, 아래와 같이 24개의 배열이 도출됩니다.

```ts
[1, 2, 3][(1, 2, 4)][(1, 3, 2)][(1, 3, 4)][(1, 4, 2)][(1, 4, 3)][(2, 1, 3)][
  (2, 1, 4)
][(2, 3, 1)][(2, 3, 4)][(2, 4, 1)][(2, 4, 3)][(3, 1, 2)][(3, 1, 4)][(3, 2, 1)][
  (3, 2, 4)
][(3, 4, 1)][(3, 4, 2)][(4, 1, 2)][(4, 1, 3)][(4, 2, 1)][(4, 2, 3)][(4, 3, 1)][
  (4, 3, 2)
];
```

---

## 구현

### 간단한 방법

가장 간단한 방법은 `for`반복문을 $r$번 중첩하는 것 입니다. 이 때, 반복문에 의해 선택된 `i, j, k`가 모두 다른값을 가르키도록 강제해야 합니다.

```ts
const arr: number[] = [1, 2, 3, 4];
const ans: number[][] = []; // 순열이 저장될 배열
for (let i = 0; i < arr.length; i++) {
  for (let j = 0; j < arr.length; j++) {
    for (let k = 0; k < arr.length; k++) {
      // 중복 배제
      if (i === j || j === k || k === i) continue;
      const current = [arr[i], arr[j], arr[k]];
      ans.push(current);
    }
  }
}
```

`i, j, k`의 중복을 배제하는 또 다른 아이디어는, 다음과 같이 `boolean[]`을 사용하는 것 입니다.

```ts
const arr: number[] = [1, 2, 3, 4];
const ans: number[][] = []; // 순열이 저장될 배열
//
// isUsed[i] = i번째 요소를 사용중이라면 true.
const isUsed: boolean[] = [false, false, false, false];
for (let i = 0; i < arr.length; i++) {
  if (isUsed[i] === true) continue;
  isUsed[i] = true;

  for (let j = 0; j < arr.length; j++) {
    if (isUsed[j] === true) continue;
    isUsed[j] = true;

    for (let k = 0; k < arr.length; k++) {
      if (isUsed[k] === true) continue;
      isUsed[k] = true;

      const current = [arr[i], arr[j], arr[k]];
      ans.push(current);
      isUsed[k] = false;
    }
    isUsed[j] = false;
  }
  isUsed[i] = false;
}
```

물론 `Set<number>`을 사용해도 됩니다.

```ts
//
// isUsed.has(i) = i번째 요소를 사용중이라면 true.
const isUsed = new Set<number>();
```

---

### 재귀로 구현

`for`문을 사용한 구현은 간단하지만 $r$이 작은 경우에만 유효합니다. $r$이 커지면 커질수록 일일이 적어야 하는 코드의 량이 늘어나기 때문이죠. 이런 경우, 재귀함수를 사용하면 유연하게 대처할 수 있습니다.

```ts
function getPermutation(
  arr: number[],
  r: number,
  tmp: number[] = [], // 순열 중 하나를 임시로 저장할 배열
  ans: number[][] = [], // 순열이 저장될 배열
  isUsed: Set<number> = new Set<number>()
) {
  // 순열 중 하나가 완성된 경우.
  if (tmp.length === r) {
    ans.push(tmp);
    return ans;
  }
  // 사용되지 않은 요소 중 하나를 사용한다.
  for (let i = 0; i < arr.length; i++) {
    if (isUsed.has(i)) continue;
    isUsed.add(i);
    tmp.push(arr[i]);
    getPermutation(arr, r, tmp, ans, isUsed);
    tmp.pop();
    isUsed.delete(i);
  }
  return ans;
}
//
// [1, 2, 3, 4] 에서 3개를 선택.
const permutation = getPermutation([1, 2, 3, 4], 3);
```

위의 재귀함수 호출을 트리 형태로 표현하면 다음과 같습니다. 색칠된 부분은 `isUsed` 조건문에 의해 호출되지 않았음을 의미합니다.

[##_Image|kage@bhRtYX/btqIBOcNsGf/oQ01Urxk6XfcKkhQAwbD8K/img.png|alignCenter|width="100%"|_##]

---

추가적으로 `tmp`에 포함된 원소는 이미 사용되었다는 것을 이해했다면 `isUsed`를 제거할 수 있습니다. (단, `Set`을 이용한 구현법이 조금 더 빠릅니다.)

```ts
function getPermutation(
  arr: number[],
  r: number,
  tmp: number[] = [], // 순열 중 하나를 임시로 저장할 배열
  ans: number[][] = [] // 순열이 저장될 배열
) {
  // 순열 중 하나가 완성된 경우.
  if (tmp.length === r) {
    ans.push(tmp);
    return ans;
  }
  // 사용되지 않은 요소 중 하나를 사용한다.
  for (let i = 0; i < arr.length; i++) {
    if (temp.includes(i)) continue;
    tmp.push(arr[i]);
    getPermutation(arr, r, tmp, ans, isUsed);
    tmp.pop();
  }
  return ans;
}
//
// [1, 2, 3, 4] 에서 3개를 선택.
const permutation = getPermutation([1, 2, 3, 4], 3);
```

---

> # 조합 (Combination)

## 정의

직관적으로 설명하자면 `순서가 중요하지 않은 순열`입니다. 예를 들어, 아래의 배열들은 `순열`로 바라봤을 때는 `Not Equal`이지만, `조합`으로 봤을 때는 `Equal`입니다.

```ts
[1, 2, 3][(1, 3, 2)][(2, 1, 3)][(2, 3, 1)][(3, 1, 2)][(3, 2, 1)];
```

따라서 `[1, 2, 3, 4]`에서 3개를 선택하는 조합은 다음과 같이 4개밖에 없습니다.

```ts
[
  [1, 2, 3],
  [1, 2, 4],
  [1, 3, 4],
  [2, 3, 4],
];
```

위에서 보았듯이, 조합에서는 순서가 그다지 중요하지 않습니다. 위의 결과에서 `[1, 2, 3]`대신에 `[2, 1, 3]`, `[1, 3, 2]`와 같이 다른 것을 사용해도 의미는 달라지지 않습니다. 하지만 계산상의 편의를 위해 `오름차순으로 정렬된 것`을 선택하겠습니다. 이것을 `정규화`라고 합니다.

---

## 구현

### 간단한 방법

순열과 마찬가지로 `for`반복문을 $r$번 반복하여 구현할 수 있습니다. `[1, 2, 3, 4]`에서 3개를 선택하는 조합을 계산해보겠습니다.

```ts
const arr: number[] = [1, 2, 3, 4];
const ans: number[][] = []; // 조합이 저장될 배열
for (let i = 0; i < arr.length; i++) {
  for (let j = i + 1; j < arr.length; j++) {
    for (let k = j + 1; k < arr.length; k++) {
      ans.push([arr[i], arr[j], arr[k]]);
    }
  }
}
```

순열과 다른 점은 `j = i + 1` , `k = j + 1`로 초기화되는 것입니다. 우리는 조합의 결과를 오름차순으로 강제했으므로 `[i=0, j=2, k=1]`과 같은 경우를 방지하기 위해, 직전 반복문에서 사용한 다음 요소부터 사용해야 합니다.

---

### 재귀로 구현

순열과 마찬가지로 `재귀`를 사용하여 $r$이 큰 경우에도 유연하게 대처할 수 있습니다. 순열과 다른점은, 우리가 설정한 강제사항으로 인해 `매 인덱스는 오름차순`으로 정렬되어 있어야 하므로, 마지막에 선택한 인덱스보다 낮은 인덱스를 사용할 수 없다는 것 입니다. 따라서 `isUsed`는 필요하지 않습니다.

```ts
function getCombination(
  arr: number[],
  r: number,
  tmp: number[] = [], // 조합 중 하나를 임시로 저장할 배열
  ans: number[][] = [] // 조합이 저장될 배열
) {
  // 조합 중 하나가 완성된 경우.
  if (tmp.length === r) {
    ans.push(tmp);
    return ans;
  }
  // 마지막으로 사용한 요소의 다음 것 부터 사용한다.
  const lastIndex = tmp.length === 0 ? -1 : tmp[tmp.length - 1];
  for (let i = lastIndex + 1; i < arr.length; i++) {
    tmp.push(arr[i]);
    getCombination(arr, r, tmp, ans);
    tmp.pop();
  }
  return ans;
}
//
// [1, 2, 3, 4] 에서 3개를 선택.
const combination = getCombination([1, 2, 3, 4], 3);
```

---

### 비트셋 응용 구현

$n$개의 아이템을 갖는 배열을 생각해보겠습니다.

```ts
const items = ["사과", "포도", "파인애플"];
```

각 아이템의 존재유무는 `1 또는 0`으로 표현될 수 있으므로, 결론적으로 $n$개의 비트를 사용하면 하나의 상태를 표현할 수 있습니다. `items[i]의 존재유무를 가르키는 비트는 1<<i`이라고 표현하면, 아래의 값은 `사과와 포도가 존재하는 경우`로 해석할 수 있습니다.

```ts
const state = 0b011;
// 사과가 가르키는 비트는 `1<<0 (0b001)`.
// 포도가 가르키는 비트는 `1<<1 (0b010)`.
// 파인이 가르키는 비트는 `1<<2 (0b100)`.
```

유심히 살펴보면, 이것은 조합과 동일하다는 것을 알 수 있습니다. 예를 들어 `[1, 2, 3, 4]`에서 3개를 선택하는 경우, 아래와 같이 4개의 비트셋과 연관됩니다. (여기서는 편의를 위해 맨 왼쪽 아이템이, 맨 왼쪽 비트에 대응된다고 생각하겠습니다.)

```ts
[
  0b1110, // 14,  -> [1, 2, 3]
  0b1101, // 13,  -> [1, 2, 4]
  0b1011, // 11,  -> [1, 3, 4]
  0b0111, // 7,   -> [2, 3, 4]
];
```

이제 눈치채셨겠지만 \[$0$, $2^n$\)를 순회하면서 비트셋을 해석하면, 가능한 모든 $r$에 대한 조합들을 얻어낼 수 있습니다. (단, $r$에 대해 정렬되어 있지는 않습니다.)

```
1 -> 0b0001 -> [ 1 ]
2 -> 0b0010 -> [ 2 ]
3 -> 0b0011 -> [ 1, 2 ]
4 -> 0b0100 -> [ 3 ]
```

이러한 아이디어를 구현하면 다음과 같습니다.

```ts
function getCombination(arr: number[]) {
  const ans: number[][] = [];
  for (let bitset = 0; bitset < 2 ** arr.length; bitset++) {
    const current: number[] = [];
    for (let idx = 0; idx < arr.length; idx++) {
      if (bitset & (1 << idx)) current.push(arr[idx]);
    }
    ans.push(current);
  }
  //
  // 필요하다면 r에 대해 정렬합니다.
  // 여기서는 생략합니다.
  return ans;
}
//
// [1, 2, 3, 4] 에서 가능한 모든 조합의 경우를 계산.
const combination = getCombination([1, 2, 3, 4]);
```

---

> # 중복된 선택이 허용된 순열과 조합

## 정의

몇몇 경우에는 중복이 허용될 수 있습니다. `오지선다`같은 경우가 대표적이죠. `[1, 2, 3, 4, 5]` 중에서 하나를 선택하는 것을 $r$번 반복한다 생각하면 편합니다.

---

## 순열

아이러니하지만 중복된 선택이 가능해짐으로써, 구현 난이도가 크게 내려갑니다. 중복된 선택을 하지 못하게끔 `isUsed`와 같은 것을 도입하고 있었는데, 그럴 필요가 없어졌기 때문입니다.

```ts
const arr: number[] = [1, 2, 3, 4];
const ans: number[][] = []; // 순열이 저장될 배열
for (let i = 0; i < arr.length; i++) {
  for (let j = 0; j < arr.length; j++) {
    for (let k = 0; k < arr.length; k++) {
      // 이제 중복을 배제 할 필요가 없다.
      // if( i === j || j === k || k === i ) continue;
      const current = [arr[i], arr[j], arr[k]];
      ans.push(current);
    }
  }
}
```

---

## 조합

조합도 중복된 선택을 할 수 있지만, `오름차순 제약`이 사라진 것이 아닙니다. 중복선택을 허용하면서 오름차순으로 정렬하는 경우는 다음과 같습니다. ($r = 5$)

```ts
[1, 1, 1, 2, 2]
[1, 1, 1, 2, 3]
[1, 2, 2, 4, 4]
...
```

즉, `마지막으로 선택된 것을 다시 사용할 수 있다` 정도로 해석할 수 있습니다.

```ts
const arr: number[] = [1, 2, 3, 4];
const ans: number[][] = []; // 조합이 저장될 배열
for (let i = 0; i < arr.length; i++) {
  for (let j = i + 0; j < arr.length; j++) {
    for (let k = j + 0; k < arr.length; k++) {
      ans.push([arr[i], arr[j], arr[k]]);
    }
  }
}
```

기존에는 `j = i + 1` , `k = j + 1`로 초기화되었지만, 마지막으로 선택된 요소를 다시 선택할 수 있도록 `j = i + 0` , `k = j + 0`으로 초기화하면 됩니다.

---

> # 중복된 요소가 허용된 순열과 조합

## 정의

반면에, `[1, 1, 2, 3, 3]`처럼 배열의 모든 것이 유니크하게 주어지지 않을 수 있습니다. `값을 기반`으로 계산했다면 코드를 뜯어고쳐야 하겠지만, 지금까지 우리는 `인덱스를 기반`으로 계산했기 때문에, 이것에 대해서는 크게 걱정하지 않아도 괜찮습니다.

---

그러나 `중복 선택과 중복 요소가 동시에 허용된 경우`에는 조금 생각해봐야 합니다. 예를 들어, `[1, 1, 1]`에서 중복선택이 허용된 수열을 구해보겠습니다. 알고리즘의 관점에서는 `[a, b, c]`로 간주되기 때문에, 아래처럼 서로다른 방법으로 배치되겠지만, 결과적으로 보면 `[1, 1, 1]` 하나밖에 없습니다.

```ts
[a, b, c] -> [1, 1, 1]
[a, c, b] -> [1, 1, 1]
[b, a, c] -> [1, 1, 1]
[b, c, a] -> [1, 1, 1]
[c, a, b] -> [1, 1, 1]
[c, b, a] -> [1, 1, 1]
```

---

## 구현

즉, 중복된 결과를 걸러내는 것이 추가로 요구됩니다. 가장 보편적인 방법은, 하나의 임시결과가 만들어질 때 마다, 그것을 문자열로 변환하고 `Set`에 들어있는지 확인하면 됩니다.

```ts
const ans : number[][]  = [];
const set : Set<string> = new Set<string>();
for(...){
    // 이번에 만들어진 순열 또는 조합
    const current : number[] = [...];

    // 해당 순열 또는 조합을 문자열로 변환한다.
    // ex) "1,2,3,", "2,3,4,"
    let key : string = "";
    for(let i=0; i<current.length; i++){
        key += (i + ",");
    }

    //
    // 이전에 본적이 있다면 건너뛴다.
    if(!set.has(key)) {
        set.add(key);
        ans.push(current);
    }
}
```

---

> # 백준 연습문제

순열과 조합이 [`N과 M`](https://www.acmicpc.net/workbook/view/2052)이라는 항목으로 분류되어 있습니다.
