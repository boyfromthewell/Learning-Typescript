# Chapter 6 배열

타입스크립트는 초기 배열에 어떤 데이터 타입이 있는지 기억하고 배열이 해당 데이터 타입에서만 작동하도록 제한 → 배열의 데이터 타입을 하나로 유지시킴

```tsx
const data = ["a", "b"];

data.push("c");

data.push(true); 
// Error (Argument of type 'boolean' is not assignable to parameter of type 'string'.)
```

## 6.1 배열 타입

배열에 대한 타입 애너테이션은 배열의 요소 타입 다음에 []가 와야 함

```tsx
let arrayNumbers: number[];

arrayNumbers = [1, 2, 3, 4];
```

### 6.1.1 배열과 함수 타입

함수 타입에 무엇이 있는지 구별하는 괄호가 필요할 수도 있음

어느 부분이 함수 반환 부분이고 어느 부분이 배열 타입 묶음인지 나타냄

```tsx
let createStrings: () => string[]; // string 배열을 반환하는 함수

let stringCreators: (() => string)[]; // 각각의 string을 반환하는 함수 배열

stringCreators = [() => "hi", () => "bye"];

stringCreators.map((func) => {
  console.log(func()); // hi bye
});
```

### 6.1.2 유니언 타입 배열

배열의 각 요소가 여러 선택 타입 중 하나일 수 있음을 나타내려면 유니언 타입 사용

유니언 타입 배열에서 괄호 사용은 매우 중요

```tsx
let stringOrArrayOfNumbers: string | number[]; // string or number의 배열

let arrayOfStringOrNumbers: (string | number)[]; // number or string인 요소의 배열
```

### 6.1.3 any 배열의 진화

초기에 빈 배열로 설정된 변수는 타입스크립트는 배열을 any[]로 취급, 모든 콘텐츠를 받음

타입 애너테이션이 없는 빈 배열은 잠재적으로 잘못된 값 추가를 허용, 타입 검사기가 갖는 이점을 부분적으로 무력화

```tsx
let values = []; // type : any[]

values.push(""); // type : string[]

values[0] = 0; // type : (number | string)[]
```

any 타입이 되도록 허용하거나, 타입을 사용하도록 허용하면 타입스크립트의 타입 검사 목적을 무효화

`타입스크립트는 값의 타입을 알 때 가장 잘 작동한다!`

### 6.1.4 다차원 배열

2차원 배열 또는 배열의 배열은 두개의 []를 가짐

```tsx
let arrayOfArraysOfNumbers: number[][];

arrayOfArraysOfNumbers = [
  [1, 2, 3],
  [2, 4, 6],
  [3, 6, 9],
];
```

n차원 배열은 n개의 []를 가짐

## 6.2 배열 멤버

타입스크립트는 배열의 멤버를 찾아서 해당 배열의 타입 요소를 되돌려주는 전형적인 인덱스 기반 접근 방식을 이해하는 언어

```tsx
const defenders = ["a", "b"];

const defender = defenders[0]; // type : string
```

유니언 타입으로 된 배열의 멤버는 그 자체로 동일한 유니언 타입

```tsx
const soldiersOrDate = ["Lee", new Date(1996, 2, 9)];

const soldierOrDate = soldiersOrDate[0]; // type : string | Date
```

### 6.2.1 주의 사항: 불안정한 멤버

타입스크립트 타입 시스템은 기술적으로 불안정하다고 알려져 있음

특히 배열은 타입 시스템에서 불안정한 소스

```tsx
function withElements(elements: string[]) {
  console.log(elements[9001].length); // 타입 오류 없음
}

withElements(["a", "b"]);
```

Cannot read property ‘length’ of undefined가 발생하며 충돌할거라고 유추되지만 타입스크립트는 검색된 배열의 멤버가 존재하는지 의도적으로 확인하지 않음

## 6.3 스프레드와 나머지 매개변수

### 6.3.1 스프레드

… 스프레드 연산자를 사용해 배열을 결합하면 타입스크립트는 입력된 배열 중 하나의 값이 결과 배열에 포함될 것임을 이해

```tsx
const array1 = ["a", "b", "c"];
const array2 = [1, 2, 3];

const joined = [...array1, ...array2]; // type : (string | number)[]
```

### 6.3.2 나머지 매개변수 스프레드

타입스크립트는 나머지 매개변수로 배열을 스프레드하는 자바스크립트 실행을 인식, 이에 대한 타입 검사 수행

나머지 매개변수를 위한 인수로 사용되는 배열은 나머지 매개변수와 동일한 배열 타입을 가져야함

```tsx
function logWarriors(greeting: string, ...names: string[]) {
  for (const name of names) {
    console.log(`${greeting}, ${name}`);
  }
}

const warriors = ["kim", "lee", "park"];

logWarriors("hi", ...warriors);

const years = [1996, 1997, 2000];

logWarriors("Born in", ...years); 
// Error (Argument of type 'number' is not assignable to parameter of type 'string'.)
```

## 6.4 튜플

튜플이라고 하는 고정된 크기에 배열을 사용하는 것이 유용할 때가 있음

튜플 타입을 선언하는 구문은 배열 리터럴처럼 보이지만 요소의 값 대신 타입을 적음

```tsx
let yearAndWarrior: [number, string];

yearAndWarrior = [530, "kim"];
yearAndWarrior = [false, "kim"]; // Error
```

### 6.4.1 튜플 할당 가능성

타입스크립트에서 튜플 타입은 가변 길이의 배열 타입보다 더 구체적으로 처리 → 가변 길이의 배열 타입은 튜플 타입에 할당 불가

```tsx
const pairLoose = [false, 123]; // type : (number | boolean)[]

const pairTupleLoose: [boolean, number] = pairLoose; 
// Error (Type '(number | boolean)[]' is not assignable to type '[boolean, number]'.
// Target requires 2 element(s) but source may have fewer.)
```

### 나머지 매개변수로서의 튜플

타입스크립트는 … 나머지 매개변수로 전달된 튜플에 정확한 타입 검사를 제공 가능

```tsx
function logPair(name: string, value: number) {
  console.log(`${name} has ${value}`);
}

const pairArray = ["kim", 1];
logPair(...pairArray); 
// Error (A spread argument must either have a tuple type or be passed to a rest parameter.)

const pairTupleCorrect: [string, number] = ["Lee", 1];
logPair(...pairTupleCorrect); // OK
```

(string | number)[] 타입의 값을 인수로 전달하려고 하면 둘 다 동일한 타입이거나 타입의 잘못된 순서로 인해 내용이 일치하지 않을 가능성 존재, 타입의 안전 보장할 수 없음

값이 [string, number] 튜플이라고 알고 있으면 값이 일치한다는 것을 알게 됨

### 6.4.2 튜플 추론

타입스크립트는 배열이 변수의 초깃값 또는 함수에 대한 반환값으로 사용되는 경우, 고정된 크기의 튜플이 아니라 유연한 크기의 배열로 가정

```jsx
function firstCharAndSize(input: string) {
  return [input[0], input.length]; // return type : (string | number)[]
}

const [firstChar, size] = firstCharAndSize("abc");
```

타입스크립트에서는 값이 튜플 타입이어야 함을 두가지 방법으로 나타냄

### 명시적 튜플 타입

함수가 튜플 타입을 반환한다고 선언, 배열 리터럴을 반환한다면 해당 배열 리터럴은 튜플로 간주

```jsx
function firstCharAndSize(input: string): [string, number] {
  return [input[0], input.length];
}

const [firstChar, size] = firstCharAndSize("abc");
```

### const 어서션

타입스크립트는 값 뒤에 넣을 수 있는 const 어서션인 as const 연산자를 제공

`const 어서션은 타입스크립트에 타입을 유추할 때 읽기 전용이 가능한 값 형식을 사용하도록 지시`

```jsx
const unionArray = [1157, "Kim"];

const readOnlyTuple = [1157, "Lee"] as const; // type : readonly [1157, "Lee"]
```

const 어서션은 해당 튜플이 읽기 전용이고 값 수정이 예상되는 곳에서 사용할 수 없음을 나타냄

```jsx
const pairMutable: [number, string] = [1157, "abc"];
pairMutable[0] = 123; // Ok

const pairAlsoMutable: [number, string] = [1157, "abc"] as const;
// Error (The type 'readonly [1157, "abc"]' is 'readonly' and cannot be assigned to the mutable type '[number, string]'.)

const pairConst = [1157, "abc"] as const;
pairConst[0] = 1247;
// Error (Cannot assign to '0' because it is a read-only property.)
```