# Chapter 4 객체

## 4.1 객체 타입

객체 값의 속성에 접근하려면 **value.멤버** 또는 **value[’멤버’]** 구문을 사용

```tsx
const poet = {
  born: 1996,
  name: "kwon",
};

poet["born"]; // number
poet.name; // string

poet.end;
// Error (Property 'end' does not exist on type '{ born: number; name: string; }'.)
```

다른 속성 이름으로 접근하려고 하면 해당 이름이 존재하지 않는다는 타입 오류가 발생

객체 타입은 타입스크립트가 자바스크립트 코드를 이해하는 방법에 대한 핵심 개념

null, undefined를 제외한 모든 값은 그 값에 대한 실제 타입의 멤버 집합을 가짐

→ 타입스크립트는 모든 값의 타입을 확인하기 위해 객체 타입을 이해해야 함

### 4.1.1 객체 타입 선언

객체 타입은 객체 리터럴과 유사하게 보이지만 필드 값 대신 타입을 사용해 설명

```tsx
let poet: {
  born: number;
  name: string;
};

poet = {
  born: 1996,
  name: "kwon",
};

poet = "kim";
// Error (Type 'string' is not assignable to type '{ born: number; name: string; }'.)
```

### 4.1.2 별칭 객체 타입

각 객체 타입에 타입 별칭을 할당해 사용하는 방법이 일반적

```tsx
type Poet = {
  born: number;
  name: string;
};

let poetLater: Poet;

poetLater = {
  born: 1996,
  name: "kwon",
};

poetLater = "kim"; // Error (Type 'string' is not assignable to type 'Poet'.)
```

## 4.2 구조적 타이핑

타입스크립트의 타입 시스템은 **구조적으로 타입화** 되어 있음 → 타입을 충족하는 모든 값을 해당 타입의 값으로 사용 가능

```tsx
type WithfirstName = {
  firstName: string;
};

type WithLastName = {
  lastName: string;
};

const hasBoth = {
  firstName: "kwon",
  lastName: "soonyong",
};

let withFirstName: WithfirstName = hasBoth;
let withLastName: WithLastName = hasBoth;
```

WithfirstName과 WithLastName은 오직 string 타입의 단일 멤버만 선언

hasBoth 변수는 명시적으로 선언되지 않았어도 두 개의 별칭 객체 타입을 모두 가지므로 두 개의 별칭 객체 타입 내에 선언된 변수를 모두 제공 가능

구조적 타이핑은 **덕 타이핑**과는 다름

- 타입스크립트의 타입 검사기에서 구조적 타이핑은 정적 시스템이 타입을 검사하는 경우
- 덕 타이핑 : 런타임에서 사용될 때까지 객체 타입을 검사하지 않는것

`자바스크립트는 덕 타입인 반면 타입스크립트는 구조적으로 타입화`

### 4.2.1 사용 검사

객체 타입에 필요한 멤버가 객체에 없다면 타입스크립트는 타입 오류를 발생

```tsx
type FirstAndLastNames = {
  first: string;
  last: string;
};

const hasBoth: FirstAndLastNames = {
  first: "kwon",
  last: "soonyong",
};

const hasOnlyOne: FirstAndLastNames = {
  first: "Lee",
};
// Error (Property 'last' is missing in type '{ first: string; }'
// but required in type 'FirstAndLastNames'.)
```

둘 사이에 일치하지 않는 타입도 허용되지 않음

```tsx
type TimeRange = {
  start: Date;
};

const hasStartString: TimeRange = {
  start: "1996-02-09",
}; // Error (Type 'string' is not assignable to type 'Date'.)
```

### 4.2.2 초과 속성 검사

별칭 객체 타입에 정의된 속성과 다르게 초과 속성이 존재하면 타입 오류를 발생

```tsx
type Poet = {
  born: number;
  name: string;
};

const extraProperty: Poet = {
  activity: "walking",
  born: 1996,
  name: "kwon",
};
// Error (Type '{ activity: string; born: number; name: string; }' is not assignable
// to type 'Poet'.Object literal may only specify known properties,
// and 'activity' does not exist in type 'Poet'.)
```

### 4.2.3 중첩된 객체 타입

타입스크립트의 객체 타입도 타입 시스템에서 중첩된 객체 타입을 나타낼 수 있어야 함

이름 대신에 {…} 객체 타입을 사용

```tsx
type Poet = {
  author: {
    firstName: string;
    lastName: string;
  };
  name: string;
};

// OK
const poemMatch: Poet = {
  author: {
    firstName: "kwon",
    lastName: "soonyong",
  },
  name: "kwon",
};

const poemMismatch: Poet = {
  author: {
    name: "Lee", // Error
  },
  name: "kim",
};
```

타입을 작성할때 중첩 객체 속성의 형태를 자체 별칭 객체 타입으로 추출하면 타입스크립트의 오류 메세지에 더 많은 정보를 담을 수 있음

```tsx
type Author = {
  firstName: string;
  lastName: string;
};

type Poem = {
  author: Author;
  name: string;
};

const poemMismatch: Poem = {
  author: {
    name: "Lee",
    // Error (Type '{ name: string; }' is not assignable to type 'Author'.
    // Object literal may only specify known properties, and 'name' does not exist in
    // type 'Author'.)
  },
  name: "kim",
};
```

### 4.2.4 선택적 속성

타입 속성 애너테이션에서 : 앞에 ?를 추가하면 선택적 속성임을 나타낼수 있음

```tsx
type Book = {
  author?: string;
  pages: number;
};

const ok: Book = {
  author: "kwon",
  pages: 80,
};

const missing: Book = {
  author: "Kim", // Error
};
```

선택적 속성과 undefined를 포함한 유니언 타입의 속성에는 차이가 존재

필수로 선언된 속성과 | undefined는 그 값이 undefined라도 반드시 존재해야함

```tsx
type Writers = {
  author: string | undefined;
  editor?: string;
};

const hasRequired: Writers = {
  author: undefined,
};

const missingRequired: Writers = {}; // Error
```

## 4.3 객체 타입 유니언

### 4.3.1 유추된 객체 타입 유니언

변수에 여러 객체 타입 중 하나가 될 수 있는 초기값이 주어지면 타입스크립트는 해당 타입을 객체 타입 유니언으로 유추

```tsx
const poem =
  Math.random() > 0.5
    ? { name: "kim", pages: 7 }
    : { name: "Lee", rhymes: true };

poem.name; // string
poem.pages; // number | undefined
poem.rhymes; // boolean | undefined
```

![Untitled (1)](https://user-images.githubusercontent.com/86250281/231396828-892c4c97-7a1f-43d1-9e91-7f2e43943f36.png)

다음 변수는 항상 string 타입인 name 속성을 가지며 pages, rhymes는 있을 수도 있고 없을 수도 있음

### 4.3.2 명시된 객체 타입 유니언

객체 타입의 조합을 명시하면 객체 타입을 더 명확히 정의 가능

`값의 타입이 객체 타입으로 구성된 유니언이면 타입스크립트의 타입 시스템은 모든 유니언 타입에 존재하는 속성에 대한 접근만 허용`

```tsx
type PoemWithPages = {
  name: string;
  pages: number;
};

type PoemWithRhymes = {
  name: string;
  rhymes: boolean;
};

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem =
  Math.random() > 0.5
    ? { name: "kim", pages: 7 }
    : { name: "Lee", rhymes: true };

poem.name; // Ok

poem.pages; // Error

poem.rhymes; // Error
```

잠재적으로 존재하지 않은 객체의 멤버에 대한 접근을 제한하면 코드의 안전을 지킬 수 있음

### 4.3.3 객체 타입 내로잉

타입 검사기가 유니언 타입 값에 특정 속성이 포함된 경우에만 코드 영역을 실행할 수 있음을 알게 되면 값의 타입을 해당 속성을 포함하는 구성 요소로만 좁힘

→ 코드에서 객체의 형태를 확인하고 타입 내로잉이 객체에 적용

```tsx
if ("pages" in poem) {
  poem.pages; // poem은 PoemWithPages로 좁혀짐
} else {
  poem.rhymes; // poem은 PoemWithRhymes로 좁혀짐
}
```

타입스크립트는 if (poem.pages)와 같은 형식으로 참 여부를 확인하는것을 허용하지 않음

```tsx
if (poem.pages) {
} // Error
```

존재하지 않은 객체의 속성에 접근하려고 시도하면 타입 가드처럼 작동하는 방식으로 사용되더라도 타입 오류로 간주

### 4.3.4 판별된 유니언

판별된 유니언 : 객체의 속성이 객체의 형태를 나타내도록 하는 것, 객체의 타입을 가리키는 속성이 판별값

```tsx
type PoemWithPages = {
  name: string;
  pages: number;
  type: "pages";
};

type PoemWithRhymes = {
  name: string;
  rhymes: boolean;
  type: "rhymes";
};

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem =
  Math.random() > 0.5
    ? { name: "kim", pages: 7, type: "pages" }
    : { name: "Lee", rhymes: true, type: "rhymes" };

if (poem.type === "pages") {
  console.log(`It's got pages: ${poem.pages}`);
} else {
  console.log(`It rhymes: ${poem.rhymes}`);
}

poem.type; // 'pages' | 'rhymes'
poem.pages;
// Error (Property 'pages' does not exist on type 'Poem'.
// Property 'pages' does not exist on type 'PoemWithRhymes'.)
```

Poem 타입은 두개의 타입이 둘 다 될수 있는 객체를 설명, type 속성으로 어느 타입인지를 나타냄

poem.type이 pages이면 타입스크립트는 poem을 poemWithPages로 유추 (타입 내로잉 없이는 값에 존재하는 속성을 보장할 수 없음)

## 4.4 교차 타입

타입스크립트에도 & (교차 타입)을 사용해 여러 타입을 동시에 나타냄

```tsx
type Artwork = {
  genre: string;
  name: string;
};

type Writing = {
  pages: number;
  name: string;
};

type WrittenArt = Artwork & Writing;
```

```tsx
type ShortPoem =
  | ({ author: string } & { kigo: string; type: "haiku" })
  | { meter: number; type: "villanelle" };

const morningGlory: ShortPoem = {
  author: "Kim",
  kigo: "Morning",
  type: "haiku",
};
```

다음 타입은 항상 author 속성을 가지며 하나의 type 속성으로 판별된 유니언 타입

### 4.4.1 교차 타입의 위험성

교차 타입을 사용할 때는 가능한 한 코드를 간결하게 유지해야 함

### 긴 할당 가능성 오류

복잡한 교차 타입을 만들게 되면 할당 가능성 오류 메세지는 읽기 어려워짐

타입스크립트가 해당 이름을 출력하도록 타입을 일련의 별칭으로 된 객체 타입으로 분할하면 읽기가 쉬워짐

```tsx
type ShortPoemBase = { author: string };
type Haiku = ShortPoemBase & { kigo: string; type: "haiku" };
type Villanelle = ShortPoemBase & { meter: number; type: "villanelle" };
type shortPoem = Haiku | Villanelle;

const oneArt: shortPoem = {
  author: "Kim",
  type: "villanelle",
};
// Error (Type '{ author: string; type: "villanelle"; }' is not assignable to type 'shortPoem'.
// Type '{ author: string; type: "villanelle"; }' is not assignable to type 'Villanelle'.
// Property 'meter' is missing in type '{ author: string; type: "villanelle"; }'
// but required in type '{ meter: number; type: "villanelle"; }'.)
```

### never

교차 타입은 잘못 사용하기 쉽고 불가능한 타입을 생성

원시 타입의 값은 동시에 여러 타입이 될수 없음 → 교차 타입의 구성 요소로 결합 불가

두 개의 원시타입을 함께 시도하면 **never** 키워드로 표시되는 **never** 타입이 됨

```tsx
type notPossible = number & string; // type: never
```

never 키워드와 타입은 프로그래밍 언어에서 bottom 타입 or empty 타입을 뜻함

bottom 타입은 값을 가질수도 없고 참조할 수도 없음 → 어떠한 타입도 제공 불가

```tsx
let notNumber: NotPossible = 0; // Error

let notString: NotPossible = ""; // Error
```

`대부분의 타입스크립트 프로젝트는 never 타입을 거의 사용하지 않음` → `대부분 교차 타입을 잘못 사용해 발생한 실수일 가능성이 높음`
