# Chapter 9 타입 제한자

## 9.1 top 타입

top 타입은 시스템에서 가능한 모든 값을 나타내는 타입 → 모든 타입은 top 타입에 할당 가능

### 9.1.1 any 다시 보기

any 타입은 모든 타입의 위치에 제공될 수 있다는 점에서 top 타입처럼 작동

```tsx
let anyValue: any;
anyValue = "abc";
anyValue = 123;

console.log(anyValue);
```

any는 타입스크립트가 타입 검사를 수행하지 않도록 명시적으로 지시하는 문제점

```tsx
function greetComdian(name: any) {
  console.log(`Announcing ${name.toUpperCase()}`);
}

greetComdian({ name: "kwon" }); // Runtime Error
```

toUpperCase() 호출은 문제가 있지만 name이 any로 선언되어 타입스크립트는 타입 오류를 보고하지 않음

`차라리 어떤 값이든 될 수 있음을 나타내려면 unknown 타입이 더 안전함`

### 9.1.2 unknown

타입스크립트에서 unknown 타입은 진정한 top 타입

unknown 타입과 any 타입의 차이점으로는 타입스크립트는 unknown 타입의 값을 더 제한적으로 취급

- 타입스크립트는 unknown 타입 값의 속성에 직접 접근 불가
- unknown 타입은 top 타입이 아닌 타입에는 할당 불가

```tsx
function greetComdian(name: unknown) {
  console.log(`Announcing ${name.toUpperCase()}`); // 'name' is of type 'unknown'.
}
```

타입스크립트가 unknown 타입에 접근할 수 있는 유일한 방법은 instanceof나 typeof 또는 타입 어서션을 사용하는 것처럼 값의 타입이 제한된 경우

```tsx
 function greetComdian(name: unknown) {
  if (typeof name === "string") {
    console.log(`${name.toUpperCase()}`);
  } else {
    console.log("well, im off");
  }
}

greetComdian("kwon");
greetComdian({});
```

## 9.2 타입 서술어

타입스크립트에는 인수가 특정 타입인지 여부를 나타내기 위해 boolean 값을 반환하는 함수를 위한 특별한 구문이 존재

타입 서술어 또는 사용자 정의 타입 가드라고 부름

타입 서술어는 일반적으로 매개변수로 전달된 인수가 매개변수의 타입보다 더 구체적인 타입인지 여부를 나타내는데 사용

```tsx
function typePredicate(input: WideType): input is NarrowType;
```

```tsx
function isNumberOrString(value: unknown): value is number | string {
  return ["number", "string"].includes(typeof value);
}

function logValueIfExists(value: number | string | null | undefined) {
  if (isNumberOrString(value)) {
    // number | string type
    value.toString();
  } else {
    // null | undefined type
    console.log(value);
  }
}
```

타입 서술어는 이미 한 인터페이스의 인스턴스로 알려진 객체가 더 구체적인 인터페이스의 인스턴스인지를 검사하는 데 자주 사용

```tsx
interface Comedian {
  funny: boolean;
}

interface StandupComedian extends Comedian {
  routine: string;
}

function isStandupComedian(value: Comedian): value is StandupComedian {
  return "routine" in value;
}

function workWithComedian(value: Comedian) {
  if (isStandupComedian(value)) {
    console.log(value.routine);
  }
  console.log(value.routine); // Property 'routine' does not exist on type 'Comedian'.
}
```

`타입 서술어는 속성이나 값의 타입을 확인 하는 것 이상을 수행해 잘못 사용하기 쉬움 -> 가능하면 안쓰는 것이 좋다`

## 9.3 타입 연산자

때로는 기존 타입의 속성 일부를 변환해서 두 타입을 결합하는 새로운 타입을 생성해야 할 때도 있음

### 9.3.1 keyof

자바스크립트 객체는 일반적으로 string 타입인 동적값을 사용해 검색된 멤버를 가짐

string 같은 포괄적인 원시 타입을 사용하면 컨테이너 값에 대해 유효하지 않은 키도 허용됨

```tsx
interface Ratings {
  audience: number;
  critics: number;
}

function getRating(ratings: Ratings, key: string): number {
  return ratings[key];
}

const ratings: Ratings = { audience: 66, critics: 84 };
getRating(ratings, "audience");
getRating(ratings, "not valid"); // 허용 되지만 사용되면 안됨
```

다른 옵션으론 허용되는 키를 위한 리터럴 유니언 타입을 사용하는 것

```tsx
interface Ratings {
  audience: number;
  critics: number;
}

function getRating(ratings: Ratings, key: "audience" | "critics"): number {
  return ratings[key];
}

const ratings: Ratings = { audience: 66, critics: 84 };
getRating(ratings, "audience");
getRating(ratings, "not valid"); // Argument of type '"not valid"' is not assignable to parameter of type '"audience" | "critics"'.
```

인터페이스에 수십개 이상의 멤버가 있다면 번거로운 작업이 요구 될 것임

→ 타입스크립트에서는 기존에 존재하는 타입을 사용하고, 해당 타입에 허용되는 모든 키의 조합을 반환하는 keyof 연산자를 제공

```tsx
interface Ratings {
  audience: number;
  critics: number;
}

function getRating(ratings: Ratings, key: keyof Ratings): number {
  return ratings[key];
}

const ratings: Ratings = { audience: 66, critics: 84 };
getRating(ratings, "audience");
getRating(ratings, "not valid"); // Argument of type '"not valid"' is not assignable to parameter of type 'keyof Ratings'.
```

### 9.3.2 typeof

typeof는 제공되는 값의 타입을 반환 → 값의 타입을 수동으로 작성하는 것이 복잡한 경우에 유용

```tsx
const original = {
  medium: "movie",
  title: "Mean Girls",
};

let adptaion: typeof original;

if (Math.random() > 0.5) {
  adptaion = { ...original, medium: "play" };
} else {
  adptaion = { ...original, medium: 2 }; // Type 'number' is not assignable to type 'string'.
}
```

### keyof typeof

타입스크립트는 두 키워드를 함께 연결해 값의 타입에 허용된 키를 간결하게 검색 가능

인터페이스를 생성하는 것 대신, keyof typeof를 사용해 키가 ratings 값 타입의 키 중 하나여야 함을 나타냄

```tsx
const ratings = {
  imdb: 8.4,
  metacritic: 82,
};

function logRating(key: keyof typeof ratings) {
  console.log(ratings[key]);
}

logRating("imdb");
logRating("invalid"); // Argument of type '"invalid"' is not assignable to parameter of type '"imdb" | "metacritic"'.
```

## 9.4 타입 어서션

타입스크립트는 코드가 강력하게 타입화 될때 가장 잘 작동 → 코드의 모든 값이 정확히 알려진 타입을 가지는 경우

경우에 따라서는 코드가 어떻게 작동하는지 타입 시스템에 100% 정확히 알리는 것이 불가능 할 때도 있음

예를 들어 JSON.parse는 의도적으로 top 타입인 any를 반환

JSON.parse에 주어진 특정 문자열값이 특정한 값 타입을 반환해야 한다는 것을 타입 시스템에 안전하게 알릴 방법은 없음

타입스크립트는 값의 타입에 대한 타입 시스템의 이해를 재정의하기 위한 구문으로 타입 어서션을 제공 → 다른 타입을 의미하는 값의 타입 다음에 as 키워드를 배치

```tsx
const rawData = '["grace", "frankie"]';

JSON.parse(rawData); // type : any

JSON.parse(rawData) as string[]; // type : string[]

JSON.parse(rawData) as [string, string]; // type : [string, string]

JSON.parse(rawData) as ["grace", "frankie"]; // type : ["grace", "frankie"]
```

타입스크립트 모범 사례는 가능한 타입 어서션을 사용하지 않는 것, but 타입 어서션이 유용하고 심지어 필요한 경우가 종종 있음 

### 9.4.1 포착된 오류 타입 어서션

오류를 처리할 때 타입 어서션이 매우 유용할 수 있음 

catch 블록에서 포착된 오류가 어떤 타입인지 아는것은 일반적으로 불가능

코드 영역이 Error 클래스의 인스턴스를 발생시킬 거라 틀림없이 확신한다면 타입 어서션을 사용해 포착된 어서션을 오류로 처리 가능

```tsx
try {
} catch (error) {
  console.warn((error as Error).message);
}
```

발생된 오류가 예상된 오류 타입인지를 확인하기 위해 instanceof 검사와 같은 타입 내로잉을 사용하는 것이 더 안전

```tsx
try {
} catch (error) {
  console.warn(error instanceof Error ? error.message : error);
}
```

### 9.4.2 non-null 어서션

실제로가 아닌 이론적으로만 null or undefined를 포함할 수 있는 변수에서 null or undefined을 제거할 때 타입 어서션을 주로 사용

타입스크립트에서는 이와 관련된 약어를 제공 → null or undefined를 제외한 값의 전체 타입을 작성하는 대신 !를 사용하면 됨

```tsx
let maybeDate = Math.random() > 0.5 ? undefined : new Date();

maybeDate as Date; // type이 Date라고 간주
maybeDate!; // type이 Date라고 간주
```

non-null 어서션은 값을 반환하거나 존재하지 않는 경우 undefined를 반환하는 Map.get과 같은 API에서 특히 유용

```tsx
const seasonCounts = new Map([
  ["I Love Lucy", 6],
  ["The Golden Girls", 7],
]);

const maybeValue = seasonCounts.get("I Love Lucy"); // type: string | undefined
console.log(maybeValue.toString()); // 'maybeValue' is possibly 'undefined'.

const knownValue = seasonCounts.get("I Love Lucy")!;
console.log(knownValue.toString()); // ok
```

### 9.4.3 타입 어서션 주의 사항

any 타입과 마찬가지로 타입 어서션은 타입스크립트의 타입 시스템에 필요한 하나의 도피 수단 → 꼭 필요한 경우가 아니라면 가능한 한 사용하지 말아야 함

예를 들어 Map이 시간이 지남에 따라 다른 값을 갖도록 변경된다고 가정, non-null 어서션은 여전히 코드가 타입스크립트 검사를 통과하도록 만들지만 런타임 오류가 발생할 수 있음

```tsx
const seasonCounts = new Map([
  ["Broad City", 5],
  ["Community", 6],
]);

const knownValue = seasonCounts.get("I Love Lucy")!;
console.log(knownValue.toString()); // 타입 오류가 아니지만, 런타임 오류가 발생
// Cannot read properties of undefined (reading 'toString')
```

`타입 어서션은 자주 사용하면 안 되고, 사용하는 것이 안전하다고 확실히 확신할 때만 사용해야 함`

### 어서션 vs 선언

변수 타입을 선언하기 위해 타입 애너테이션을 사용하는 것과 초깃값으로 변수 타입을 변경하기 위해 타입 어서션을 사용하는 것 사이에는 차이가 있음

변수의 타입 애너테이션과 초깃값이 모두 있을때, 타입 검사기는 변수의 타입 애너테이션에 대한 변수의 초깃값에 대해 할당 가능성 검사를 수행

but 타입 어서션은 타입 검사중 일부를 건너뛰도록 명시적으로 지시

```tsx
interface Entertainer {
  acts: string[];
  name: string;
}

const declare: Entertainer = {
  // Property 'acts' is missing in type '{ name: string; }' but required in type 'Entertainer'.
  name: "kwon",
};

const asserted = {
  name: "kwon",
} as Entertainer; // 허용되지만 런타임 시 오류 발생

// 런타임 오류 발생
console.log(declare.acts.join(","));
console.log(asserted.acts.join(","));
```

### 어서션 할당 가능성

타입스크립트는 타입 중 하나가 다른 타입에 할당 가능한 경우에만 두 타입 간의 타입 어서션을 허용

완전히 서로 관련이 없는 두 타입 사이에 타입 어서션이 있는 경우에는 타입스크립트가 타입 오류를 감지

```tsx
let myValue = "abc" as number;
// Conversion of type 'string' to type 'number' may be a mistake
// because neither type sufficiently overlaps with the other.
// If this was intentional, convert the expression to 'unknown' first.
```

하나의 타입에서 값을 완전히 관련 없는 타입으로 전환해야 하는 경우 이중 타입 어서션을 사용

```tsx
let myValueDouble = "1337" as unknown as number;
```

타입 시스템의 도피 수단으로 이중 타입 어서션을 사용하면 주변 코드를 변경, 이전에 작동하던 코드에 문제가 발생할 경우 타입 시스템이 구해주지 못함 → 쓰면 안댐

## 9.5 const 어서션

const 어서션은 배열, 원시 타입, 값, 별칭 등 모든 값을 상수로 취급해야 함을 나타내는 데 사용

as const는 수신하는 모든 타입에 세 가지 규칙을 적용

- 배열은 가변 배열이 아닌 읽기 전용 튜플로 취급
- 리터럴은 일반적인 원시 타입과 동등하지 않고 리터럴로 취급
- 객체의 속성은 읽기 전용으로 간주

### 9.5.1 리터럴에서 원시 타입으로

타입 시스템이 리터럴 값을 일반적인 원시 타입으로 확장하기보다 특정 리터럴로 이해하는 것이 유용할 수 있음

```tsx
const getName = () => "Maria Bamford"; // type : () => string
const getNameConst = () => "Maria Bamford" as const; // type : () => "Maria Bamford"
```

값의 특정 필드가 더 구체적인 리터럴 값을 갖도록 하는 것도 유용

많은 인기 있는 라이브러리는 값의 판별 필드가 특정 리터럴이 되도록 요구

```tsx
interface Joke {
  quote: string;
  style: "story" | "one-liner";
}

function tellJoke(joke: Joke) {
  if (joke.style === "one-liner") {
    console.log(joke.quote);
  } else {
    console.log(joke.quote.split("\n"));
  }
}

const narrowJoke = {
  quote: "abcd",
  style: "one-liner" as const,
};

tellJoke(narrowJoke);

const wideObject = {
  quote: "abc",
  style: "one-liner",
};

tellJoke(wideObject);
// Argument of type '{ quote: string; style: string; }' is not assignable to parameter of type 'Joke'.
// Types of property 'style' are incompatible.
// Type 'string' is not assignable to type '"one-liner" | "story"'.
```

### 9.5.2 읽기 전용 객체

변수의 초기값으로 사용되는 것과 같은 객체 리터럴은 let 변수의 초깃값이 확장되는 것과 동일한 방식으로 속성 타입을 확장

값의 일부 또는 전체를 나중에 특정 리터럴 타입이 필요한 위치에서 사용해야 할 때 잘 맞지 않을 수 있음

as const를 사용해 값 리터럴을 어서션하면 유추된 타입이 가능한 한 구체적으로 전환 됨

→ 모든 멤버 속성은 readonly가 되고 리터럴은 일반적인 원시 타입 대신 고유한 리터럴 타입으로 간주, 배열은 읽기 전용 튜플이 됨

```tsx
function describlePreference(preference: "maybe" | "no" | "yes") {
  switch (preference) {
    case "maybe":
      return "I suppose...";
    case "no":
      return "No thanks";
    case "yes":
      return "Yes please";
  }
}

const preferenceMutable = {
  movie: "maybe",
  standup: "yes",
};

describlePreference(preferenceMutable.movie);
// Argument of type 'string' is not assignable to parameter of type '"maybe" | "no" | "yes"'.

preferenceMutable.movie = "no"; // ok

const preferenceReadOnly = {
  movie: "maybe",
  standup: "yes",
} as const;

describlePreference(preferenceReadOnly.movie); // ok

preferenceReadOnly.movie = "no";
// Cannot assign to 'movie' because it is a read-only property.
```