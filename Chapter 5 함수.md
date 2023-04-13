# Chapter 5 함수

## 5.1 함수 매개변수

타입스크립트를 사용하면 타입 애너테이션으로 함수 매개변수의 타입을 선언 가능

```tsx
function sing(song: string) {
  console.log(`Singing: ${song}!`);
}
```

song 매개변수가 string 타입임을 타입스크립트에게 알림

### 5.1.1 필수 매개변수

타입스크립트는 함수에 선언된 모든 매개변수가 필수라고 가정

```tsx
function singTwo(first: string, second: string) {
  console.log(`${first} / ${second}`);
}

singTwo("one", "two");

singTwo("one"); // Error (Expected 2 arguments, but got 1.)
```

### 5.1.2 선택적 매개변수

타입스크립트에서는 선택적 객체 타입 속성과 유사하게 타입 애너테이션의 : 앞에 ?를 추가해 매개변수가 선택적이라고 표시

선택적 매개변수에는 항상 | undefined가 유니언 타입으로 추가

```tsx
function announceSongBy(song: string, singer?: string) {
  console.log(song);

  if (singer) {
    console.log(singer);
  }
}

announceSongBy("Lee");
announceSongBy("Lee", undefined);
announceSongBy("Lee", "Kim");
```

선택적 매개변수는 | undefined를 포함하는 유니언 타입 매개변수와는 다름, 선택적 매개변수가 아닌 매개변수는 값이 명시적으로 undefined일지라도 항상 제공 되어야함

```tsx
function announceSongBy(song: string, singer: string | undefined) {}

announceSongBy("Lee"); // Error
announceSongBy("Lee", undefined);
announceSongBy("Lee", "Kim");
```

함수에서 사용되는 모든 선택적 매개변수는 마지막 매개변수여야 함

```tsx
function announceSongBy(song?: string, singer: string) {}
// Error (A required parameter cannot follow an optional parameter.)
```

### 5.1.3 기본 매개변수

매개변수에 기본값이 있고 타입 애너테이션이 없는 경우 타입스크립트는 해당 기본값을 기반으로 매개변수 타입을 유추

```tsx
function rateSong(song: string, rating = 0) {
  console.log(`${song} gets ${rating}/5 stars!`);
}

rateSong("a");
rateSong("b", 5);
rateSong("c", undefined);

rateSong("d", "100"); 
// Error (Argument of type 'string' is not assignable to parameter of type 'number'.)
```

rating은 number 타입으로 유추되지만 함수를 호출하는 코드에서는 선택적 number | undefined가 됨

### 5.1.4 나머지 매개변수

자바스크립트 일부 함수는 임의의 수의 인수로 호출할 수 있도록 만들어짐

… 스프레드 연산자는 함수 선언의 마지막 매개변수에 위치, 해당 매개변수에서 시작해 함수에 전달된 나머지 인수가 모두 단일 배열에 저장됨을 나타냄

타입스크립트는 나머지 매개변수의 타입을 일반 매개변수와 유사하게 선언 가능

```tsx
function singAllTheSongs(singer: string, ...songs: string[]) {
  for (const song of songs) {
    console.log(`${song}, by ${singer}`);
  }
}

singAllTheSongs("Lee");
singAllTheSongs("Lee", "Kim", "Park");
singAllTheSongs("Lee", 2000);
//
// Error (Argument of type 'number' is not assignable to parameter of type 'string'.)
```

## 5.2 반환 타입

타입스크립트는 함수가 반환할 수 있는 가능한 모든 값을 이해하면 함수가 반환하는 타입을 알 수 있음

```tsx
function singSongs(songs: string[]) {
  for (const song of songs) {
    console.log(song);
  }
  return songs.length;
}
// singSongs(songs: string[]): number
```

number를 반환하는 것으로 파악

함수에 다른 값을 가진 여러 개의 반환문을 포함한다면 타입스크립트는 반환 타입을 가능한 모든 반환 타입 조합으로 유추

```tsx
function getSongAt(songs: string[], index: number) {
  return index < songs.length ? songs[index] : undefined;
}
// getSongAt(songs: string[], index: number): string | undefined
```

### 5.2.1 명시적 반환 타입

함수에서 반환 타입을 명시적으로 선언하는 방식이 유용할때가 있음

- 가능한 반환값이 많은 함수가 항상 동일한 타입의 값을 반환하도록 강제
- 타입스크립트는 재귀 함수의 반환 타입을 통해 타입을 유추하는 것을 거부
- 매우 큰 프로젝트에서 타입스크립트 타입 검사 속도를 높일 수 있음

함수 선언 반환 타입 애너테이션은 매개변수 목록이 끝나는 ) 다음에 배치

함수 선언의 경우는 { 앞에 배치

```tsx
function singSongsRecursive(songs: string[], count = 0): number {
  return songs.length ? singSongsRecursive(songs.slice(1), count + 1) : count;
}
```

함수 반환문이 함수의 반환 타입으로 할당할 수 없는 값을 반환하는 경우 타입스크립트는 할당 가능성 오류를 표시

```tsx
function getSongRecordingDate(song: string): Date | undefined {
  switch (song) {
    case "a":
      return new Date("April 20, 2023");

    case "b":
      return "unknown"; // Error (Type 'string' is not assignable to type 'Date'.)

    default:
      return undefined;
  }
}
```

## 5.3 함수 타입

함수 타입은 화살표 함수와 유사, 함수 본문 대신 타입이 존재

- 매개변수가 없고 특정 타입을 반환 할때
    
    ```tsx
    let nothingInGivesString: () => string;
    ```
    
- 매개변수 존재, 특정 타입 반환 할때
    
    ```tsx
    let inputAndOutput: (songs: string[], count?: number) => number;
    ```
    

### 5.3.1 함수 타입 괄호

함수 타입은 다른 타입이 사용되는 모든곳에 배치 가능 (유니언 타입도 포함)

유니언 타입의 애너테이션에서 함수 반환 위치를 나타내거나 유니언 타입을 감싸는 부분을 표시할때 괄호 사용

```tsx
let returnStringOrUndefined: () => string | undefined;

let maybeReturnsString: (() => string) | undefined;
```

### 5.3.2 매개변수 타입 추론

매개변수로 사용되는 인라인 함수를 포함, 작성한 모든 함수에 대해 매개변수를 선언하는것은 번거로움

타입스크립트는 선언된 타입의 위치에 제공된 함수의 매개변수 타입을 유추 가능

```tsx
let singer: (song: string) => string;

singer = function (song) {
  // song: string type
  return `Singing: ${song.toUpperCase()}!`;
};
```

singer 변수는 string 타입의 매개변수를 갖는 함수로 알려져 있고 나중에 변수가 할당 되는 함수 내의 매개변수는 string으로 예측됨

### 5.3.3 함수 타입 별칭

함수 타입에서도 동일하게 타입 별칭을 사용 가능

```tsx
type StringToNumber = (input: string) => number;

let stringToNumber: StringToNumber;

stringToNumber = (input) => input.length;

stringToNumber = (input) => input.toUpperCase(); // Error
```

함수 매개변수도 함수 타입을 참조하는 별칭을 입력 가능

```tsx
type NumberToString = (input: number) => string;

function usesNumberToString(numberToString: NumberToString) {
  console.log(`The string is: ${numberToString(1234)}`);
}

usesNumberToString((input) => `${input}!`);

usesNumberToString((input) => input * 2); 
// Error (Type 'number' is not assignable to type 'string'.)
```

## 5.4 그 외 반환 타입

### 5.4.1 void 반환 타입

반환 값이 없는 함수의 반환타입은 타입스크립트는 void 키워드를 사용

```tsx
function logSong(song: string | undefined): void {
  if (!song) {
    return;
  }
  console.log(song);

  return true; // Error
}
```

자바스크립트 함수는 실젯값이 반환되지 않으면 기본으로 모두 undefined를 반환, void는 undefined와 동일하지 않음

void는 함수의 반환 타입이 무시된다는 것을 의미

undefined는 반환되는 리터럴 값

```tsx
function returnVoid() {
  return;
}

let lazyValue: string | undefined;

lazyValue = returnVoid();
// Error (Type 'void' is not assignable to type 'string | undefined'.)
```

### 5.4.2 never 반환 타입

never 반환 함수는 항상 오류를 발생시키거나 무한 루프를 실행하는 함수

함수가 절대 반환하지 않도록 의도하려멱 명시적 : never 타입 에너테이션을 추가, 해당 함수를 호출한 후 모든 코드가 실행되지 않음을 나타냄

```tsx
function fail(message: string): never {
  throw new Error(`Invariant failure: ${message}.`);
}

function workWithUnsafeParam(param: unknown) {
  if (typeof param !== "string") {
    fail(`param should be a string, not ${typeof param}`);
  }

  param.toUpperCase(); // param의 타입은 string으로 좁혀짐
}
```