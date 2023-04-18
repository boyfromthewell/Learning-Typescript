# Chapter 7 인터페이스

인터페이스는 연관된 이름으로 객체 형태를 설명하는 또 다른 방법

별칭으로 된 객체 타입과 유사하지만 일반적으로 더 읽기 쉬운 오류 메세지, 더 나은 컴파일러 성능, 클래스와의 더 나은 상호 운영성을 위해 선호

## 7.1 타입 별칭 vs 인터페이스

```jsx
interface Poet {
  born: number;
  name: string;
}

let valueLater: Poet;

valueLater = {
  born: 1996,
  name: "kwon",
};

valueLater = "Lee"; // Error

valueLater = {
  born: true, // Error
  name: "Lee",
};
```

인터페이스와 타입 별칭 사이에는 몇 가지 차이점이 존재

- 인터페이스는 속성 증가를 위해 병합할 수 있음
- 인터페이스는 클래스가 선언된 구조의 타입을 확인하는데 사용 가능, 타입 별칭은 사용 불가
- 일반적으로 인터페이스에서 타입스크립트 타입 검사기가 더 빨리 작동
- 인터페이스는 이름 있는 객체로 간주되므로 어려운 특이 케이스에서 나타나는 오류 메시지를 더 쉽게 읽을 수 있음

## 7.2 속성 타입

### 7.2.1 선택적 속성

타입 애너테이션 : 앞에 ?를 사용해 인터페이스의 속성이 선택적 속성임을 나타낼 수 있음

```jsx
interface Book {
  author?: string;
  pages: number;
}

const ok: Book = {
  author: "kim",
  pages: 89,
};

const missing: Book = {
  pages: 80,
};
```

### 7.2.2 읽기 전용 속성

타입스크립트는 속성 이름 앞에 readonly 키워드를 추가해 다른 값으로 설정될 수 없음을 나타냄

(새로운 값으로 재할당 불가)

```jsx
interface Page {
  readonly text: string;
}

function read(page: Page) {
  console.log(page.text);

  page.text += "!"; // Error (Cannot assign to 'text' because it is a read-only property.)
}
```

readonly 제한자는 타입시스템에만 존재하며 인터페이스에만 사용 가능

but, 타입 시스템 구성 요소일 뿐 컴파일된 자바스크립트 출력 코드에는 존재하지않음

`단지 타입스크립트 타입 검사기를 사용해 개발 중에 그 속성이 수정되지 못하도록 보호하는 역할`

### 7.2.3 함수와 메서드

타입스크립트는 인터페이스 멤버를 함수로 선언하는 두 가지 방법을 제공

- 메서드 구문
- 속성 구문

```jsx
interface HasBothFunctionTypes {
  property: () => string; // 속성 구문
  method(): string; // 메서드 구문
}

const hasBoth: HasBothFunctionTypes = {
  property: () => "",
  method() {
    return "";
  },
};

hasBoth.property();
hasBoth.method();
```

메서드와 속성 선언은 서로 교환하여 사용 가능, 주요 차이점은 다음과 같음

- 메서드는 readonly로 선언할 수 없지만 속성은 가능
- 인터페이스 병합은 메서드와 속성을 다르게 처리
- 타입에서 수행되는 일부 작업은 메서드와 속성을 다르게 처리 (15장 참고)

### 7.2.4 호출 시그니처

호출 시그니처 : 값을 함수처럼 호출하는 방식에 대한 타입 시스템의 설명

인터페이스와 객체 타입은 호출 시그니처로 선언 가능

호출 시그니처는 함수 타입과 비슷하지만 : 대신 ⇒로 표시함

```jsx
type FunctionAlias = (input: string) => number;

interface CallSignature {
  (input: string): number;
}

const typedFunctionAlias: FunctionAlias = (input) => input.length;

const typedCallSignature: CallSignature = (input) => input.length;
```

### 7.2.5 인덱스 시그니처

타입스크립트는 인덱스 시그니처 구문을 제공해 인터페이스의 객체가 임의의 키를 받고, 해당 키 아래의 특정 타입을 반환할 수 있음을 나타냄

일반 속성 정의와 유사하지만 키 다음에 타입이 있고 {[i: string] : … }과 같이 배열의 대괄호를 가짐

```jsx
interface WordCounts {
  [i: string]: number;
}

const counts: WordCounts = {};

counts.apple = 0;
counts.banana = 1;

counts.cherry = false; // Error
```

인덱스 시그니처는 객체에 값을 할당할 때 편리하지만 타입 안정성을 완벽하게 보장하지는 않음

객체가 어떤 속성에 접근하든 간에 값을 반환해야 함을 나타냄

```jsx
interface DatesByName {
  [i: string]: Date;
}

const publishDates: DatesByName = {
  Kwon: new Date("1 January 1996"),
};

publishDates.Kwon;
console.log(publishDates.Kwon.toString());

publishDates.Beloved; // 런타임 값은 undefined
console.log(publishDates.Beloved.toString());
// 타입 시스템에서는 오류가 나지 않지만 런타임에서 오류 발생
```

타입스크립트는 Beloved가 정의되지 않았음에도 불구하고 정의되었다고 생각함

### 속성과 인덱스 시그니처 혼합

인터페이스는 명시적으로 선언된 속성과 포괄적 용도의 string 인덱스 시그니처를 한번에 포함할 수 있음

```jsx
interface Novels {
  Oroonoko: number;
  [i: string]: number;
}

const novels: Novels = {
  Outlander: 1991,
  Oroonoko: 1688,
};

const missing: Novels = {
  // Error (Property 'Oroonoko' is missing in type '{ Outlander: number; }' but required in type 'Novels'.)
  Outlander: 1991,
};
```

### 숫자 인덱스 시그니처

객체의 키로 숫자만 허용하려면 인덱스 시그니처 키로 number 타입 사용

```jsx
interface MoreNarrowNumbers {
  [i: number]: string;
  [i: string]: string | undefined;
}

const mixesNumbersAndStrings: MoreNarrowNumbers = {
  0: "",
  key1: "",
  key2: undefined,
};
```

### 7.2.6 중첩 인터페이스

인터페이스 타입도 자체 인터 페이스 타입 혹은 객체 타입을 속성으로 가질 수 있음

```jsx
interface Novel {
  author: {
    name: String;
  };
  setting: Setting;
}

interface Setting {
  place: string;
  year: number;
}

let myNovel: Novel;

myNovel = {
  author: {
    name: "Kwon",
  },
  setting: {
    place: "korea",
    year: 1996,
  },
};
```

## 7.3 인터페이스 확장

타입스크립트는 인터페이스가 다른 인터페이스의 모든 멤버를 복사해서 선언할 수 있는 확장된 (extend) 인터페이스를 허용

확장할 인터페이스의 이름뒤에 extends 키워드를 추가

`파생 인터페이스를 준수하는 모든 객체가 기본 인터페이스의 모든 멤버를 가져야함`

```jsx
interface Writing {
  title: string;
}

interface Novella extends Writing {
  pages: number;
}

let myNovella: Novella = {
  title: "a",
  pages: 195,
};

let missingPages: Novella = {
  title: "a",
  // Error (Property 'pages' is missing in type '{ title: string; }'
  // but required in type 'Novella'.)
};
```

### 7.3.1 재정의된 속성

파생 인터페이스는 다른 타입으로 속성을 다시 선언해 기본 인터페이스의 속성을 재정의 하거나 대체 가능

속성을 재선언하는 대부분의 파생 인터페이스는 해당 속성을 **유니언 타입의 더 구체적인 하위 집합으로 만들거나** 속성을 기본 인터페이스의 타입에서 확장된 타입으로 만들기 위해 사용

```jsx
interface WithNullableName {
  name: string | null;
}

interface WithNonNullableName extends WithNullableName {
  name: string;
}

interface WithNumericName extends WithNonNullableName {
  name: number | string; // Error (number | string을 string | null에 할당 할 수는 없음)
}
```

### 7.3.2 다중 인터페이스 확장

타입스크립트의 인터페이스는 여러 개의 다른 인터페이스를 확장해서 선언 가능

extends 키워드 뒤에 쉼표로 인터페이스 이름을 구분해 사용

```jsx
interface GivesNumber {
  giveNumber(): number;
}

interface GivesString {
  giveString(): string;
}

interface GivesBothAndEither extends GivesNumber, GivesString {
  giveEither(): number | string;
}

function useGivesBoth(instance: GivesBothAndEither) {
  instance.giveEither(); // numer | string
  instance.giveNumber(); // number
  instance.giveString(); // string
}
```

## 7.4 인터페이스 병합

두 개의 인터페이스가 동일한 이름으로 동일한 스코프에 선언된 경우, 선언된 모든 필드를 포함하는 더 큰 인터페이스가 코드에 추가

```jsx
interface Merged {
  fromFirst: string;
}

interface Merged {
  fromSecond: number;
}

/* 
다음과 같은 형태로 병합
interface Merged {
  fromFirst: string;  
  fromSecond: number;
}
*/
```

but, 인터페이스가 여러 곳에 선언되면 코드를 이해하기 어려워짐, 가능하면 인터페이스 병합을 사용하지 않는것이 좋음

인터페이스 병합은 외부 패키지 또는 Window 같은 내장된 전역 인터페이스를 보강하는데 유용

```jsx
interface Window {
  myEnvironmentVariable: string;
}

window.myEnvironmentVariable; // type : stirng
```

### 7.4.1 이름이 충돌되는 멤버

병합된 인터페이스는 타입이 다른 동일한 이름의 속성을 여러 번 선언할 수 없음

속성이 이미 선언되어 있다면 병합된 인터페이스에서도 동일한 타입을 사용해야함

```jsx
interface MergedProperties {
  same: (input: boolean) => string;
  different: (input: string) => string;
}

interface MergedProperties {
  same: (input: boolean) => string;
  different: (input: number) => string;
  // Error (Subsequent property declarations must have the same type.
  // Property 'different' must be of type '(input: string) => string', 
  // but here has type '(input: number) => string'.)
}
```

병합된 인터페이스는 동일한 이름과 다른 시그니처를 가진 **메서드**는 정의할 수 있음 → 메서드에 대한 함수 오버로드 발생

```jsx
interface MergedMethods {
  different(input: string): string;
}

interface MergedMethods {
  different(input: number): string; // OK
}
```