> # 개요

## 접근법

`Trie`라는 용어 자체는 생소할 수 있지만, 알게 모르게 실생활에서도 사용한 적이 있을만큼 매우 간단한 개념입니다. 예를 들어, 영어사전에서 `apple` 이라는 단어를 찾고 싶다면, 앞에서부터 `a - p - p - l - e` 순서로 `prefix`를 조합하면서 찾아갈 수 있습니다.

```ts
// 영어사전에서 apple를 찾고 싶다면...
앞에서   a를 찾는다. -> a
이어지는 p를 찾는다. -> ap
이어지는 p를 찾는다. -> app
이어지는 l를 찾는다. -> appl
이어지는 e를 찾는다. -> apple
```

위와 비슷하게 `xx시 xx고등학교 xx학년 xx반`에 소속된 학생정보를 찾고 싶다면, 앞에서부터 `prefix`를 조합하면서 차례대로 접근하면 됩니다.

```ts
step 1) xx시
step 2) xx시 xx고등학교
step 3) xx시 xx고등학교 xx학년
step 4) xx시 xx고등학교 xx학년 xx반
```

`Trie`는 이러한 개념을 구현한 자료구조의 일종이며 `N중 트리`를 재귀적으로 사용하여 쉽게 구현할 수 있습니다. 아래는 `[CAT, CUTE, TO, B]`가 저장된 트라이의 예시입니다.

[##_Image|kage@dgc773/btqILLUVzr7/t25diDkAGpDjGi9NakjOy0/img.png|alignCenter|width="100%"|_##]

---

## 목적

위에서는 이해를 위해 `어떤 자료를 찾아가는 방법`처럼 트라이를 설명했지만, 검색의 측면에서는 `HashMap`을 사용하는게 더 낫겠죠. 트라이는 `공통된 접두사를 공유하는 요소`를 찾는데 특화되어 있습니다.

---

예를 들어, `[hey, hey, hi, he, his, her, ex]`에서 `he`로 시작하는 단어는 몇개일까요? `Trie`를 사용하지 않는다면, 각 요소마다 `he`로 시작하는지 체크해야 되므로 $O(n)$의 시간복잡도가 걸리겠지만, 아래처럼 `Trie`를 사용한다면 $O(string.length)$로 줄일 수 있습니다.

[##_Image|kage@cSve37/btqITt6lo0P/H6RWoSHpHUKJxu7qobn5FK/img.png|alignCenter|width="100%"|_##]

위의 특징으로 인해 `Trie`는 `Prefix Tree`라고 불리기도 합니다.

---

> # 구현

개념적인 구현은 다음과 같습니다.

```ts
class TrieNode {
  size: number;
  data: Map<string, TrieNode>;

  of(char: string): TrieNode {
    let childTrie = this.data.get(char);
    if (childTrie === undefined) {
      childTrie = new TrieNode();
      this.data.set(char, childTrie);
    }
    return childTrie;
  }

  insert(str: string): void {
    if (str.length === 0) return;
    const first = str[0];
    const others = str.slice(1);
    const childTrie = this.of(first);
    childTrie.size++;
    childTrie.insert(others);
  }
}
```

기본적인 사용방법은 다음과 같습니다.

```ts
const root = new TrieNode();
root.insert("hey");
root.insert("hell");
root.insert("happy");

//
// he로 시작하는 단어의 개수
root.of("h").of("e").size;
```

---

> # 응용

## 접두사 트리

일반적인 경우의 트라이입니다.

---

## 접미사 트리

문자열을 뒤집어서 삽입하면 `공통된 접미사를 공유하는 요소`를 찾을 수 있습니다. 예를 들어, `[hey, eye, wey, hi]`에서 `ey`로 끝나는 문자열을 찾으려면, 각 요소를 뒤집어서 만든 `[yeh, eye, yew, ih]`로 트라이를 구성하면 됩니다.

---

## 와일드카드가 허용된 트라이

### Question Mark

`?(물음표 문자)`는 임의의 1개 글자에 대응되는 와일드카드 문자입니다. 예를 들어 `h?`는 `h`로 시작하는 2글자 단어를 의미하며 `[hi, ha]`와는 일치하지만 `[hey, her]`와는 일치하지 않습니다. 이러한 요구조건을 구현하기 위해, 2가지의 접근법이 사용될 수 있습니다.

---

- 자신의 모든 자식 트라이를 순회

```ts
const root = new TrieNode();
...

//
// h로 시작하는 2글자 단어의 개수
let answer = 0;
for(const childTrie of root.of("h").data) {
    answer += childTrie.size;
}
```

- `?`에 대응되는 자식 트라이를 생성하고, 중복저장.

```ts
class TrieNode {
    ...

    insert(str: string, recursive = true) : void {
        if(str.length === 0) return;
        const first  = str[0];
        const others = str.slice(1);

        //
        // 자신의 자식 트라이에 한번 저장.
        const childTrie = this.of(first);
        childTrie.size++;
        childTrie.insert(others);

        //
        // 자신의 퀘스쳔 마크 트라이에 두번 저장.
        const questionMarkTrie = this.of("?");
        questionMarkTrie.insert(str, false);
    }
}

root
    .of("h")
    .of("?")
    .size;
```

---

### Asterisk

`*(별 문자)`는 임의의 0개 이상의 글자에 대응되는 와일드카드 입니다. 예를 들어 `h*ix`는 `[hix, haix, halix]`와 일치합니다. 이러한 요구조건을 구현하기 위해, 재귀를 사용할 수 있습니다.

---

예를 들어, `hi*ok*ts`이 주어졌을 때, 다음과 같이 재귀적으로 검색할 수 있습니다.

```ts
루트 노드에서 [hi]를 검색.
-> 매칭된 트라이의 모든 하위요소에 대해 [ok]를 검색.
-> 매칭된 트라이의 모든 하위요소에 대해 [ts]를 검색.
```

예를 들어 `Hello*World`가 `["HelloWorld", "Hello~~World"]`를 찾는 과정은 다음과 같습니다.

1. 루트 노드에서 `Hello` 노드로 이동.
2. `Hello` 노드의 하위요소 `Hello`, `Hello~`, `Hello~~`에 대해 `World`를 검색.
   - `Hello` 노드에서 1건.
   - `Hello~` 노드에서 0건.
   - `Hello~~` 노드에서 1건.
3. 따라서 `Hello*World`의 결과는 2건.

---

## 문자열이 아닌 트라이

접두사의 개념은 문자열에만 있는 것이 아닙니다. 예를 들어, 아래의 두 데이터는 `세 번째 바이트 까지 공통된 접두사`를 가지고 있습니다.

```ts
  b[0]     b[1]     b[2]     b[3]
10110000 01110101 11001100 00000000
10110000 01110101 11001100 11111111
```

코드로 표현하면 다음과 같습니다.

```ts
root
  .of(176) // 10110000
  .of(117) // 01110101
  .of(204).size; // 11001100
```

---

## 동등조건이 아닌 트라이

각 접두사의 조건은 `동등(Equal)`만 있는 것이 아닙니다.

- 학생1 : 수학 90점, 영어 80점, 과학 70점
- 학생2 : 수학 80점, 영어 20점, 과학 91점
- 학생3 : 수학 80점, 영어 40점, 과학 50점

---

위와 같은 상황에서,

1. `수학은 70점 이상, 과학은 70점 이하인 학생의 수?`
2. `수학은 80점, 영어는 짝수, 과학은 홀수인 학생의 수?`

---

> # 연습문제

[2020 카카오 블라인드 공채 - 가사 검색](https://programmers.co.kr/learn/courses/30/lessons/60060)
[백준 - 전화번호 목록](https://www.acmicpc.net/problem/5052)
