# Chapter 3 유니언과 리터럴

타입스크립트가 값을 바탕으로 추론을 수행하는 두 가지 핵심개념이 존재

- 유니언 : 값에 허용된 타입을 두 개 이상의 가능한 타입으로 확장하는 것
- 내로잉 : 값에 허용된 타입이 하나 이상의 가능한 타입이 되지 않도록 좁히는 것

## 3.1 유니언 타입

```tsx
let mathematician = Math.random() > 0.5 ? undefined : "Kim";
```

mathematician 변수는 undefined이거나 string일 수 있다.

‘이거 혹은 저거’와 같은 타입을 **유니언**이라고 함

타입스크립트는 가능한 값 또는 구성 요소 사이에 | (수직선) 연산자를 사용해 유니언 타입임을 나타냄

### 3.1.1 유니언 타입 선언

변수에 초깃값이 있더라도 변수에 대한 명시적 타입 애너테이션을 제공하는 것이 유용할 때 유니언 타입을 사용

```tsx
let thinker: string | null = null;

if (Math.random() > 0.5) {
  thinker = "kwon";
}
```

### 3.1.2 유니언 속성

값이 유니언 타입일 때 타입스크립트는 유니언으로 선언한 모든 가능한 타입에 존재하는 멤버 속성에만 접근 가능

```tsx
let physicist = Math.random() > 0.5 ? "Kwon" : 84;

physicist.toString(); // OK

physicist.toUpperCase(); // Error

physicist.toFixed(); // Error
```

number | string 타입으로 두 개의 타입에 모두 존재하는 toString()은 사용할 수 있지만 toUpperCase()는 number 타입에 x, toFixed()는 string 타입에 없어서 사용 불가

**객체가 어떤 속성을 포함한 타입으로 확실하게 알려지지 않은 경우 타입스크립트는 해당 속성을 사용하려고 시도하는것이 안전하지 않다고 여김 → 존재하지 않는 타입일 수도 있기 때문**

유니언 타입으로 정의된 여러 타입 중 하나의 타입으로 된 값의 속성을 사용하려면 코드에서 값이 보다 구체적인 타입 중 하나라는 것을 타입스크립트에게 알려야함 → 이 과정을 **내로잉**이라고 함

## 3.2 내로잉

내로잉 : 값이 정의, 선언 혹은 이전에 유추된 것보다 더 구체적인 타입임을 코드에서 유추하는 것

타입스크립트가 값의 타입이 이전에 알려진 것보다 더 좁혀졌다는 것을 알게 되면 값을 더 구체적인 타입으로 취급 → **타입을 좁히는데 사용할 수 있는 논리적 검사를 타입 가드라고 함**

### 3.2.1 값 할당을 통한 내로잉

변수에 값을 직접 할당하면 타입스크립트는 변수의 타입을 할당된 값의 타입으로 좁힘

```tsx
let admiral: number | string;

admiral = "kwon";

admiral.toUpperCase();

admiral.toFixed(); // Error
```

변수에 유니언 타입 애너테이션이 명시되고 초기값이 주어지면 값 할당 내로잉이 작동

타입스크립트는 변수가 나중에 유니언 타입으로 선언된 타입 중 하나의 값을 받을 수 있지만, 처음에는 초기에 할당된 값의 타입으로 시작한다는 것을 이해

```tsx
let admiral: number | string = "kwon";

admiral.toUpperCase();

admiral.toFixed(); // Error
```

### 3.2.2 조건 검사를 통한 내로잉

일반적으로 타입스크립트에서는 변수가 알려진 값과 같은지 확인하는 if 문을 통해 변수의 값을 좁히는 방법을 사용

```tsx
let scientist = Math.random() > 0.5 ? "kwon" : 51;

if (scientist === "kwon") {
  scientist.toUpperCase(); // OK
}

scientist.toUpperCase(); // Error
```

타입스크립트는 if문 내에서 변수가 알려진 값과 동일한 타입인지 확인

### 3.2.3 typeof 검사를 통한 내로잉

타입스크립트는 직접 값을 확인해 값을 좁힐수 있지만, typeof 연산자를 사용할 수도 있음

```tsx
let researcher = Math.random() > 0.5 ? "kwon" : 51;

if (typeof researcher === "string") {
  researcher.toUpperCase(); // Ok : string
}
```

!를 사용한 논리적 부정과 else문도 잘 작동

```tsx
let researcher = Math.random() > 0.5 ? "kwon" : 51;

if (!(typeof researcher === "string")) {
  researcher.toFixed(); // ok : number
} else {
  researcher.toUpperCase(); // ok : string
}
```

## 3.3 리터럴 타입

```tsx
const philosopher = "Hypatia";
```

해당 변수는 단지 string 타입이 아닌 'Hypatia'라는 특별한 값 → 변수의 타입은 기술적으로 더 구체적인 'Hypatia' → 리터럴 타입의 개념

리터럴 타입 : `원시 타입 값 중 어떤 것이 아닌 특정 원싯값으로 알려진 타입`

변수를 const로 선언하고 직접 리터럴 값을 할당하면 타입스크립트는 해당 변수를 할당된 리터럴 값으로 유추

![Untitled](https://user-images.githubusercontent.com/86250281/231386273-0aab3a4e-5451-4e49-b622-9d8240abd81f.png)

원시 타입은 해당 타입의 가능한 모든 리터럴 값의 집합

boolean, null, undefined 타입 외에 number, string과 같은 모든 원시 타입에는 무한한 수의 리터럴 타입이 있음

- boolean : true | false
- null, undefined : 둘 다 자기자신 → 오직 하나의 리터럴 값만 가짐
- number
- string

유니언 타입 에너테이션에서 리터럴과 원시 타입을 섞어서 사용 가능

```tsx
let lifespan: number | "ongoing" | "uncertain";

lifespan = 89;
lifespan = "ongoing";

lifespan = true; // Error
```

### 3.3.1 리터럴 할당 가능성

0과 1처럼 동일한 원시 타입일지라도 서로 다른 리터럴 타입은 서로 할당할 수 없음

```tsx
let specificallyKwon: "kwon";

specificallyKwon = "kwon"; // ok

specificallyKwon = "kim"; // Error

let someString = ""; // type: string

specificallyKwon = someString; // Error
```

## 3.4 엄격한 null 검사

타입스크립트는 두려운 ‘10억 달러의 실수’를 바로 잡기 위해 엄격한 null 검사를 사용

### 3.4.1 십억 달러의 실수

십억 달러의 실수 : 다른 타입이 필요한 위치에서 null 값을 사용하도록 허용하는 많은 타입 시스템을 가리키는 업계 용어

엄격한 null 검사가 없는 언어에서는 예제 코드의 변수 할당이 허용

```tsx
const firstName: string = null;
```

타입스크립트 컴파일러는 실행 방식을 변경할 수 있는 다양한 옵션 제공

- strictNullChecks : 엄격한 null 검사를 활성화할지 여부를 결정

(비활성화하면 코드의 모든 타입에 | null | undefined를 추가해야 모든 변수가 null 또는 undefined 할당 가능)

엄격한 null 검사가 활성화되면 타입스크립트는 코드에서 발생하게 될 잠재적인 충돌을 확인

```tsx
let nameMaybe = Math.random() > 0.5 ? "kwon" : undefined;

nameMaybe.toLowerCase();
// 'nameMaybe' is possibly 'undefined'.
```

엄격한 null 검사를 활성화 해야만 코드가 null or undefined 값으로 인한 오류로부터 안전한지 여부를 쉽게 파악 가능

### 3.4.2 참 검사를 통한 내로잉

타입스크립트는 잠재적인 값 중 truthy로 확인된 일부에 한해서만 변수의 타입을 좁힐 수 있음

```tsx
let geneticist = Math.random() > 0.5 ? "kwon" : undefined;

if (geneticist) {
  geneticist.toUpperCase(); // OK
}

geneticist.toUpperCase(); // Error
```

geneticist는 string | undefined 타입이며 undefined는 항상 falsy
→ if 문의 코드 블록에서는 geneticist가 string 타입이 되어야 한다고 추론 가능

```tsx
geneticist && geneticist.toUpperCase();
geneticist?.toUpperCase();
```

논리 연산자인 &&와 ?는 참 여부를 검사하는 일 잘 수행

but string | undefined 값에 대해 알고 있는 것이 falsy라면 그것이 빈 문자열인지 undefined 인지는 알수 없음

### 3.4.3 초깃값이 없는 변수

자바스크립트에서 초깃값이 없는 변수는 기본적으로 undefined

타입스크립트는 값이 할당될 때까지 변수가 undfined임을 이해

```tsx
let mathematician: string;

mathematician?.length; // Error

mathematician = "kwon";
mathematician.length; // OK
```

값이 할당되기 전에 해당 변수를 사용하려고 시도하면 오류 발생

```tsx
let mathematician: string | undefined;

mathematician?.length; // Ok

mathematician = "kwon";
mathematician.length; // OK
```

변수 타입에 undefined가 포함되어 있는 경우 오류가 보고 되지 않음

변수 타입에 | undefined를 추가하면 undefined는 유효한 타입이기 때문에 사용 전에는 정의할 필요가 없음을 타입스크립트에 나타냄

## 3.5 타입 별칭

유니언 타입 대부분은 두 세개의 구성 요소만 가짐, but 반복해서 입력하기 불편한 긴 형태의 유니언 타입도 존재

```tsx
let data1: boolean | number | string | null | undefined;
let data2: boolean | number | string | null | undefined;
let data3: boolean | number | string | null | undefined;
```

타입스크립트에는 재사용하는 타입에 더 쉬운 이름을 할당하는 **타입 별칭(type alias)**이 존재

```tsx
type MyName = ...;
```

타입스크립트가 타입 별칭을 발견하면 해당 별칭이 참조하는 실제 타입을 입력한 것처럼 동작

```tsx
type RawData = boolean | number | string | null | undefined;

let data1: RawData;
let data2: RawData;
let data3: RawData;
```

읽기에도 훨씬 편리함

### 3.5.1 타입 별칭은 자바스크립트가 아닙니다

타입 별칭은 타입 애너테이션처럼 자바스크립트로 컴파일 되지 않음

`순전히 타입스크립트 타입 시스템에만 존재`

타입 별칭은 타입 시스템에만 존재 → 런타임 코드에서는 참조 할 수 없음

```tsx
type SomeType = string | undefined;

console.log(SomeType);
// Error: 'SomeType' only refers to a type, but is being used as a value here.
```

**타입 별칭은 순전히 개발시에만 존재**

### 3.5.2 타입 별칭 결합

타입 별칭은 다른 타입 별칭을 참조 가능

```tsx
type Id = number | string;
type IdMaybe = Id | undefined | null; // == number | string | undefined | null
```

사용 순서대로 타입 별칭을 선언할 필요는 없음

파일 내에서 타입 별칭을 먼저 선언하고 참조할 타입 별칭을 나중에 선언할 수 있음

```tsx
type IdMaybe = Id | undefined | null; // OK
type Id = number | string;
```
