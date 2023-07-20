# Chapter 10 제네릭

지금까지 살펴본 모든 구문은 해당 구문이 작성될 때 완전히 알려진 타입과 함께 사용했어야 함

→ 때로는 코드에서 호출하는 방식에따라 다양한 타입으로 작동하도록 의도할 수 있음

```
function identity(input: any) {
  return input;
}

// value: any type
identity("abc");
identity(123);
identity({ title: "abc" });
```

input이 모든 입력을 허용한다면 input 타입과 함수 반환 타입 간의 관계를 말할 수 있는 방법이 필요

`타입스크립트는 제네릭을 사용해 타입 간의 관계를 알아냄`

- 타입스크립트에서 함수와 같은 구조체는 **제네릭 타입 매개변수**를 원하는 수만큼 선언 가능
- 제네릭 타입 매개변수는 제내릭 구조체의 각 사용법에 따라 타입이 결정
- 타입 매개변수는 구조체의 각 인스턴스에서 서로 다른 일부 타입을 나타내기 위해 구조체의 타입으로 사용
- 타입 매개변수는 구조체의 각 인스턴스에 대해 **타입 인수**라고 하는 서로 다른 타입을 함께 제공
- 타입 인수가 제공된 인스턴스 내에서는 일관성을 유지

타입 매개 변수는 전형적으로 T나 U 같은 단일 문자 이름 또는 Key와 Value 같은 파스칼 케이스 이름을 가짐

## 10.1 제네릭 함수

매개변수 괄호 바로 앞 <>로 묶인 타입 매개변수에 별칭을 배치해 함수를 제네릭으로 만듬

해당 타입 매개변수를 함수의 본문 내부의 매개변수 타입 애너테이션, 반환 타입 애너테이션, 타입 애너테이션에서 사용 가능

```tsx
function identity<T>(input: T) {
  return input;
}

const numeric = identity("me"); // type : "me"
const stringy = identity(123); // type : 123
```

화살표 함수도 제네릭을 만들 수 있음

```tsx
const identity = <T>(input: T) => input;

identity(123);
```

### 10.1.1 명시적 제네릭 호출 타입

제네릭 함수를 호출할 때 대부분의 타입스크립트는 함수가 호출되는 방식에 따라 타입 인수를 유추

but, 클래스 멤버와 변수 타입과 마찬가지로 타입 인수를 해석하기 위해 타입스크립트에 알려줘야 하는 함수 호출 정보가 충분하지 않을 수도 있음

```tsx
function logWrapper<Input>(callback: (input: Input) => void) {
  return (input: Input) => {
    console.log(input);
    callback(input);
  };
}

// type: (input: string) => void
logWrapper((input: string) => {
  console.log(input.length);
});

// type: (input: unknown) => void
logWrapper((input) => {
  console.log(input.length); // 'input' is of type 'unknown'.
});
```

매개변수 타입을 모르는 경우에는 타입스크립트는 Input이 무엇이 되야하는지 알아낼 방법이 없음

`기본 값이 unknown으로 설정되는 것을 피하기 위해 타입스크립트에 해당 타입 인수가 무엇인지 명시적으로 알려주는 명시적 제네릭 타입 인수를 사용해 함수를 호출 할 수 있음`

```tsx
// type : (input: string) => void
logWrapper<string>((input) => {
  console.log(input.length);
});

logWrapper<string>((input: boolean) => {});
// Argument of type '(input: boolean) => void' is not assignable to parameter of type '(input: string) => void'.
// Types of parameters 'input' and 'input' are incompatible.
// Type 'string' is not assignable to type 'boolean'.
```

명시적 타입 인수는 제네릭 함수에 지정 가능하지만 때로는 필요하지 않음, 필요 할때만 명시적 타입 인수를 지정하자

### 10.1.2 다중 함수 타입 매개변수

임의의 수의 타입 매개변수를 쉼표로 구분해 함수를 정의 → 제네릭 함수의 각 호출은 각 타입 매개변수에 대한 자체 값 집합을 확인 가능

```tsx
function makeTuple<First, Second>(first: First, second: Second) {
  return [first, second] as const;
}

let tuple = makeTuple(true, "abc"); // type : readonly [boolean, string]
```

함수가 여러 개의 타입 매개변수를 선언하면 해당 함수에 대한 호출은 명시적으로 제네릭 타입을 모두 선언하지 않거나 모두 선언해야 함 

(타입스크립트는 아직 제네릭 호출 중 일부 타입만을 유추하지는 못함)

```tsx
function makePair<Key, Value>(key: Key, value: Value) {
  return { key, value };
}

makePair("abc", 123); // type: { key: string; value: number }

makePair<string, number>("abc", 123); // type: { key: string; value: number }
makePair<"abc", 123>("abc", 123); // type: { key: "abc"; value: 123 }

makePair<string>("abc", 123);
// Expected 2 type arguments, but got 1.
```

## 10.2 제네릭 인터페이스

인터페이스도 제네릭으로 선언 가능, 함수와 유사한 제네릭 규칙을 따름

```tsx
interface Box<T> {
  inside: T;
}

let stringyBox: Box<string> = {
  inside: "abc",
};

let numberBox: Box<number> = {
  inside: 123,
};

let incorrectBox: Box<number> = {
  inside: true, // Type 'boolean' is not assignable to type 'number'.
};
```

### 10.2.1 유추된 제네릭 인터페이스 타입

제네릭 인터페이스의 타입 인수는 사용법에서 유추 가능, 타입스크립트는 제네릭 타입을 취하는 것으로 선언된 위치에 제공된 값의 타입에서 타입 인수를 유추

```tsx
interface LinkedNode<Value> {
  next?: LinkedNode<Value>;
  value: Value;
}

function getLast<Value>(node: LinkedNode<Value>): Value {
  return node.next ? getLast(node.next) : node.value;
}

let lastDate = getLast({ value: new Date("09-13-1993") });

let lastFruit = getLast({
  next: {
    value: "banana",
  },
  value: "apple",
});

let lastMismatch = getLast({
  next: {
    value: 123,
  },
  value: "abc", // Type 'string' is not assignable to type 'number'.
});
```

인터페이스가 타입 매개변수를 선언하는 경우, 해당 인터페이스를 참조하는 모든 타입 애너테이션은 이에 상응하는 타입 인수를 제공해야 함

```tsx
interface CrateLike<T> {
  contents: T;
}

let missingGeneric: CrateLike = {
  // Generic type 'CrateLike<T>' requires 1 type argument(s).
};
```

## 10.3 제네릭 클래스

클래스도 나중에 멤버에서 사용할 임의의 수의 타입 매개변수를 선언할 수 있음

```tsx
class Secret<Key, Value> {
  key: Key;
  value: Value;

  constructor(key: Key, value: Value) {
    this.key = key;
    this.value = value;
  }

  getValue(key: Key): Value | undefined {
    return this.key === key ? this.value : undefined;
  }
}

const storage = new Secret(12345, "abc"); // type : Secret<number, string>

storage.getValue(1987); // type : string | undefined
```

### 10.3.1 명시적 제네릭 클래스 타입

제네릭 클래스 인스턴스화는 제네릭 함수를 호출하는 것과 동일한 타입 인수 유추 규칙을 따름

생성자에 전달된 인수에서 클래스 타입 인수를 유추할 수 없는 경우 타입 인수의 기본값은 unknown이 됨

```tsx
class CurriedCallback<Input> {
  #callback: (input: Input) => void;

  constructor(callback: (input: Input) => void) {
    this.#callback = (input: Input) => {
      console.log("Input", input);
      callback(input);
    };
  }

  call(input: Input) {
    this.#callback(input);
  }
}

new CurriedCallback((input: string) => {
  console.log(input.length);
});

new CurriedCallback((input) => {
  console.log(input.length); // 'input' is of type 'unknown'.
});
```

클래스 인스턴스는 다른 제네릭 함수 호출과 동일한 방식으로 명시적 타입 인수를 제공 → 기본값  unknown이 되는 것을 피할 수 있음

```tsx
class CurriedCallback<Input> {
  #callback: (input: Input) => void;

  constructor(callback: (input: Input) => void) {
    this.#callback = (input: Input) => {
      console.log("Input", input);
      callback(input);
    };
  }

  call(input: Input) {
    this.#callback(input);
  }
}

new CurriedCallback<string>((input) => {
  console.log(input.length);
});

new CurriedCallback<string>((input: boolean) => {});
// Argument of type '(input: boolean) => void' is not assignable to parameter of type '(input: string) => void'.
// Types of parameters 'input' and 'input' are incompatible.
// Type 'string' is not assignable to type 'boolean'.
```

### 10.3.2 제네릭 클래스 확장

제네릭 클래스는 extends 키워드 다음에 오는 기본 클래스로 사용 가능

기본값이 없는 모든 타입 인수는 명시적 타입 애너테이션을 사용해 지정해야 함

```tsx
class Quote<T> {
  lines: T;

  constructor(lines: T) {
    this.lines = lines;
  }
}

class SpokenQuote extends Quote<string[]> {
  speak() {
    console.log(this.lines.join("\n"));
  }
}

new Quote("abc").lines; // type: string
new Quote([1, 2, 3, 4]).lines; // type: number[]

new SpokenQuote(["abc", "def"]).lines; // type: string[]
new SpokenQuote([1, 2, 3, 4]);
// Type 'number' is not assignable to type 'string'.
```

제네릭 파생 클래스는 자체 타입 인수를 기본 클래스에 번갈아 전달가능, 타입 이름은 일치하지 않아도 됨

```tsx
class AttributedQuote<Value> extends Quote<Value> {
  speaker: string;
  constructor(value: Value, speaker: string) {
    super(value);
    this.speaker = speaker;
  }
}

new AttributedQuote("ABV", "CCCDCD");
// AttributedQuote { lines: 'ABV', speaker: 'CCCDCD' }
```

### 10.3.3 제네릭 인터페이스 구현

제네릭 인터페이스는 제네릭 기본 클래스를 확장하는 것과 유사하게 작동

기본 인터페이스의 모든 타입 매개변수는 클래스에 선언되어야 함

```tsx
interface ActingCredit<T> {
  role: T;
}

class MoviePart implements ActingCredit<string> {
  role: string;
  speaking: boolean;

  constructor(role: string, speaking: boolean) {
    this.role = role;
    this.speaking = speaking;
  }
}

const part = new MoviePart("abc", true);
part.role; // type: string

class IncorrectExtension implements ActingCredit<string> {
  role: boolean;
  // Property 'role' in type 'IncorrectExtension' is not assignable to the same property in base type 'ActingCredit<string>'.
  // Type 'boolean' is not assignable to type 'string'.
}
```

### 10.3.4 메서드 제네릭

클래스 메서드는 클래스 인스턴스와 별개로 자체 제네릭 타입을 선언 가능

제네릭 클래스 메서드에 대한 각각의 호출은 각 타입 매개변수에 대해 다른 타입 인수를 가짐

```tsx
class CreatePairFactory<T> {
  key: T;

  constructor(key: T) {
    this.key = key;
  }

  createPair<Value>(value: Value) {
    return { key: this.key, value };
  }
}

const factory = new CreatePairFactory("role");

const numberPair = factory.createPair(10);
const stringPair = factory.createPair("abc");
```

### 10.3.5 정적 클래스 제네릭

클래스의 정적 멤버는 인스턴스 멤버와 구분, 클래스의 특정 인스턴스와 연결되어 있지 않음

클래스의 정적 멤버는 클래스 인스턴스에 접근하거나 타입 정보를 지정할 수 없음

`정적 클래스 메서드는 자체 타입 매기변수는 선언할 수 있지만 클래스에 선언된 어떠한 타입 매개변수에도 접근 할 수 없음`

```tsx
class BothLogger<T> {
  instanceLog(value: T) {
    console.log(value);
    return value;
  }

  static staticLog<U>(value: U) {
    let fromInstace: T; // Static members cannot reference class type parameters.

    console.log(value);
    return value;
  }
}

const logger = new BothLogger<number[]>();
logger.instanceLog([1, 2, 3]);

BothLogger.staticLog([false, true]); // boolean[]
BothLogger.staticLog<string>("abc"); // string
```

## 10.4 제네릭 타입 별칭

각 타입 별칭에는 T와 함꼐 임의의 수의 타입 매개변수가 주어짐

```tsx
type Nullish<T> = T | null | undefined;
```

일반적으로 제네릭 함수의 타입을 설명하는 함수와 함께 사용

```tsx
type CreateValue<T, U> = (input: T) => U;

let creator: CreateValue<string, number>;

creator = (text) => text.length;
creator = (text) => text.toUpperCase(); // Type 'string' is not assignable to type 'number'.
```

### 10.4.1 제네릭 판별된 유니언

데이터의 성공적인 결과 또는 오류로 인한 실패를 나타내는 제네릭 ‘결과’ 타입을 만들기 위한 타입 인수를 추가하는데 주로 사용

```tsx
type Result<T> = FailureResult | SuccessfulResult<T>;

interface FailureResult {
  error: Error;
  succeeded: false;
}

interface SuccessfulResult<T> {
  data: T;
  succeeded: true;
}

function handleResult(result: Result<string>) {
  if (result.succeeded) {
    console.log(result.data);
  } else {
    console.log(result.error);
  }
}
```

## 10.5 제네릭 제한자

타입스크립트는 제네릭 타입 매개변수의 동작을 수정하는 구문도 제공

### 10.5.1 제네릭 기본값

타입 매개변수 선언 뒤에 =와 기본 타입을 배치해 타입 인수를 명시적으로 제공 가능

기본값은 타입 인수가 명시적으로 선언되지 않고 유추할 수 없는 모든 후속 타입에 사용

```tsx
interface Quote<T = string> {
  value: T;
}

let explicit: Quote<number> = { value: 123 };

let implicit: Quote = { value: "agdfsdaf" };

let mismatch: Quote = { value: 123 };
// Type 'number' is not assignable to type 'string'.
```

타입 매개변수는 동일한 선언 안의 앞선 타입 매개변수를 기본값으로 가질 수 있음

각 타입 매개변수는 선언에 대한 새로운 타입을 도입 → 해당 선언 이후의 타입 매개변수에 대한 기본값으로 이를 사용 가능

```tsx
interface KeyValuePair<Key, Value = Key> {
  key: Key;
  value: Value;
}

let allExplicit: KeyValuePair<string, number> = {
  key: "fdf",
  value: 10,
};

let oneDefaulting: KeyValuePair<string> = {
  key: "12321",
  value: "ten",
};

let firstMissing: KeyValuePair = {
  // Generic type 'KeyValuePair<Key, Value>' requires between 1 and 2 type arguments.
};
```

모든 기본 타입 매개변수는 기본 함수 매개변수처럼 선언 목록의 제일 마지막에 와야 함

기본값이 없는 제네릭 타입은 기본값이 있는 제네릭 타입 뒤에 오면 안됨

```tsx
function inTheEnd<First, Second, Third = number, Fourth = string>() {}

function inTheMiddle<First, Second = boolean, Third = number, Fourth>() {}
// Required type parameters may not follow optional type parameters.
```

## 10.6 제한된 제네릭 타입

기본적으로 제네릭 타입에는 클래스, 인터페이스, 원싯값, 유니언, 별칭 등 모든 타입을 제공

그러나 일부 함수는 제한된 타입에서만 작동해야 함

타입 매개변수를 제한하는 구문은 매개변수 이름 뒤에 extends 키워드를 배치하고 그 뒤에 이를 제한할 타입을 배치

```tsx
interface WithLength {
  length: number;
}

function logWithLength<T extends WithLength>(input: T) {
  console.log(input.length);
  return input;
}

logWithLength("12345");
logWithLength([false, true]);
logWithLength({ length: 123 });

logWithLength(new Date());
// Argument of type 'Date' is not assignable to parameter of type 'WithLength'.
// Property 'length' is missing in type 'Date' but required in type 'WithLength'.
```

### 10.6.1 keyof와 제한된 타입 매개변수

extends와 keyof를 함께 사용하면 타입 매개변수를 이전 타입 매개변수의 키로 제한할 수 있음, 제네릭 타입의 키를 지정하는 유일한 방법

```tsx
function get<T, Key extends keyof T>(container: T, key: Key) {
  return container[key];
}

const roles = {
  favorite: "Fargo",
  others: ["abc", "cde", "fgh"],
};

const favorite = get(roles, "favorite");
const others = get(roles, "others");

const missing = get(roles, "extras");
// Argument of type '"extras"' is not assignable to parameter of type '"favorite" | "others"'.
```

## 10.7 Promise

임의의 값 타입에 대해 유사한 작업을 나타내는 Promise의 기능은 타입스크립트의 제네릭과 자연스럽게 융합

Promise는 타입스크립트 타입 시스템에서 최종적으로 resolve된 값을 나타내는 Promise 클래스로 표현

### 10.7.1 Promise 생성

값을 resolve하려는 Promise를 만들려면 Promise 타입 인수를 명시적으로 선언해야 함

```tsx
// type : Promise<unknown>
const resolvesUnknown = new Promise((resolve) => {
  setTimeout(() => resolve("Done"), 1000);
});

// type : Promise<string>
const resolvesString = new Promise<string>((resolve) => {
  setTimeout(() => resolve("Done"), 1000);
});
```

Promise의 제네릭 .then 메서드는 반환되는 Promise의 resolve된 값을 나타내는 새로운 타입 매개변수를 받음

```tsx
const textEventually = new Promise<string>((resolve) => {
  setTimeout(() => resolve("DONE"), 1000);
});

const lengthEventually = textEventually.then((text) => text.length);
```

### 10.7.2 async 함수

자바스크립트에서 async 키워드를 사용해 선언한 모든 함수는 Promise를 반환

```tsx
// type: (text: string) => Promise<number>
async function lengthAftrerSecond(text: string) {
  await new Promise((resolve) => setTimeout(resolve, 1000));
  return text.length;
}

// type: (text: string) => Promise<number>
async function lengthImmediately(text: string) {
  return text.length;
}
```

Promise를 명시적으로 언급하지 않더라도 async 함수에서 수동으로 선언된 반환 타입은 항상 Promise 타입이 됨

```tsx
async function givePromiseForString(): Promise<string> {
  return "Done";
}

async function giveString(): string {
  // The return type of an async function or method must be the global Promise<T> type.
  return "Done!";
}
```

## 10.8 제네릭 올바르게 사용하기

제네릭은 코드에서 타입을 설명하는데 많은 유연성을 제공하지만 코드가 복잡해질수 있음

필요할 때만 제네릭을 사용하고, 제네릭을 사용할 때는 무엇을 위해 사용하는지 명확히 해야 함

### 10.8.1 제네릭 황금률

제네릭은 타입 간의 관계를 설명, 제네릭 타입 매개변수가 한 곳에만 나타나면 여러 타입 간의 관계를 정의할 수 없음

`각 함수 타입 매개변수는 매개변수에 사용되어야 하고, 적어도 하나의 다른 매개변수 또는 함수의 반환 타입에서도 사용되어야 함`

```tsx
function logInput<Input extends string>(input: Input) {
  console.log(input);
}
```

logInput은 타입 매개변수로 더 많은 매개변수를 반환하거나 선언하는 작업을 하지 않음, Input 타입 매개변수를 선언하는 것은 별로 쓸모가 없음

```tsx
function logInput(input: string) {
  console.log(input);
}
```

### 10.8.2 제네릭 명명 규칙

타입 매개변수에 대한 표준 명명 규칙은 기본적으로 첫 번쨰 타입 인수로 T를 사용, 후속 타입 매개변수가 존재하면 U,V 등을 사용하는 것

타입 인수가 어떻게 사용되어야 하는지 맥락과 정보가 알려진 경우 명명 규칙은 해당 용어의 첫 글자를 사용하는 것으로 확장

- 상태관리 라이브러리에서는 제네릭 상태를 S
- 데이터 구조의 키와 값은 K, V로 나타내기도 함

but 하나의 문자를 사용하는 타입 인수명은 하나의 문자로 함수나 변수의 이름을 사용하는 것만큼 혼란스러울수 있음

```tsx
// L, V는 과연 무엇?
function labelBox<L, V>(l: L, v: V) {}
```

제네릭의 의도가 단일 문자 T에서 명확하지 않은 경우에는 타입이 사용되는 용도를 가리키는 설명적인 제네릭 타입 이름을 사용하는 것이 좋음

```tsx
function labelBox<Label, Value>(l: Label, v: Value) {}
```