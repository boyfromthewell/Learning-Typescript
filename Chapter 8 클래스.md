# Chapter 8 클래스

## 8.1 클래스 메서드

타입스크립트는 독립 함수를 이해하는 것과 동일한 방식으로 메서드를 이해

메서드를 호출하려면 허용 가능한 수의 인수가 필요, 재귀 함수가 아니라면 대부분 반환 타입을 유추 함

```jsx
class Greeter {
  greet(name: string) {
    console.log(`${name}, do your stuff`);
  }
}

new Greeter().greet("Miss Frizzle");

new Greeter().greet(); 
// Expected 1 arguments, but got 0.
```

클래스 생성자는 매개변수와 관련하여 전형적인 클래스 메서드 처럼 취급

메서드 호출 시 올바른 타입의 인수가 올바른 수로 제공되는지 확인하기 위해 타입 검사를 수행

```jsx
class Greeted {
  constructor(message: string) {
    console.log(`As I always say: ${message}!`);
  }
}

new Greeted("abcde");

new Greeted();
// Expected 1 arguments, but got 0.
```

## 8.2 클래스 속성

타입스크립트에서 클래스의 속성을 읽거나 쓰려면 클래스에 명시적으로 선언해야 함

클래스 속성은 인터페이스와 동일한 구문을 사용해 선언, 클래스 속성 이름 뒤에는 선택적으로 타입 애너테이션이 붙음

```tsx
class FieldTrip {
  destination: string;

  constructor(destination: string) {
    this.destination = destination;
    console.log(`We are going to ${this.destination}`);

    this.nonexistent = destination;
    // Property 'nonexistent' does not exist on type 'FieldTrip'.
  }
}
```

클래스 속성을 명시적으로 선언하면 타입스크립트는 클래스 인스턴스에서 무엇이 허용되고 허용되지 않는지 빠르게 이해 가능

```jsx
const trip = new FieldTrip("abc");

trip.destination;

trip.nonexistent;
// Property 'nonexistent' does not exist on type 'FieldTrip'.
```

### 8.2.1 함수 속성

자바스크립트에는 클래스의 멤버를 호출 가능한 함수로 선언하는 두가지 구문이 있음

메서드 접근 방식은 함수를 클래스 프로토타입에 할당 → 모든 클래스 인스턴스는 동일한 함수 정의를 사용

```jsx
class WithMethod {
  myMethod() {}
}

new WithMethod().myMethod === new WithMethod().myMethod; // true
```

값이 함수인 속성을 선언하는 방식도 존재 → 클래스의 인스턴스당 새로운 함수가 생성

항상 클래스 인스턴스를 가리켜야하는 화살표 함수에서 this 스코프를 사용하면 클래스 인스턴스당 새로운 함수를 생성하는 시간과 메모리 비용 측면에서 유용

```jsx
class WithProperty {
  myProperty: () => {};
}

console.log(new WithProperty().myProperty === new WithProperty().myProperty); // false
```

함수 속성에는 클래스 메서드와 독립 함수의 동일한 구문을 사용해 매개변수와 반환 타입을 지정 가능 → 결국 함수 속성은 클래스 멤버로 할당된 값 → 그 값은 함수

```jsx
class WithPropertyParameters {
  takesParameters = (input: boolean) => (input ? "yes" : "no");
}

const instance = new WithPropertyParameters();

instance.takesParameters(true);

instance.takesParameters(123);
// Argument of type 'number' is not assignable to parameter of type 'boolean'.
```

### 8.2.2 초기화 검사

엄격한 컴파일러 설정이 활성화된 상태에서 타입스크립트는 undefined 타입으로 선언된 각 속성이 생성자에서 할당되었는지 확인 → 클래스 속성에 값을 할당하지 않는 실수를 예방할 수 있어 유용

```jsx
class WithValue {
  immediate = 0;
  later: number;
  myBeUndefined: number | undefined;
  unused: number; // Error
  // Property 'unused' has no initializer and is not definitely assigned in the constructor.
  constructor() {
    this.later = 1;
  }
}
```

### 확실하게 할당된 속성

엄격한 초기화 검사가 유용한 경우가 대부분이지만 클래스 생성자 다음에 클래스 속성을 의도적으로 할당하지 않는 경우가 있을 수 있음

엄경한 초기화 검사를 적용하면 안 되는 속성인 경우에는 이름 뒤에 !를 추가해 검사를 비활성화하도록 설정 → 타입스크립트에 속성이 처음 사용되기 전에 undefined 값이 할당

```tsx
class ActivitiesQueue {
  pending!: string[];

  initialize(pending: string[]) {
    this.pending = pending;
  }

  next() {
    return this.pending.pop();
  }
}

const activities = new ActivitiesQueue();

activities.initialize(["a", "b", "c"]);
activities.next();
```

`! 어서션을 추가해 속성에 대한 타입 안정성을 줄이는 대신 클래스를 리팩토링해 어서션이 필요하지 않도록 하는것이 좋음`

### 8.2.3 선택적 속성

인터페이스와 마찬가지로 클래스는 선언된 속성 이름 뒤에 ?를 추가해 속성을 옵션으로 선언

```jsx
class MissingInitializer {
  property?: string;
}

new MissingInitializer().property?.length; // ok
new MissingInitializer().property.length; // Error
// Object is possibly 'undefined'.
```

MissingInitializer 클래스는 property를 옵션으로 정의했으므로 엄격한 속성 초기화 검사와 관계없이 클래스 생성자에서 할당하지 않아도 됨

### 8.2.4 읽기 전용 속성

인터페이스와 마찬가지로 클래스도 선언된 속성 이름 앞에 readonly 키워드를 추가해 속성을 읽기 전용으로 선언

```tsx
class Quote {
  readonly text: string;

  constructor(text: string) {
    this.text = text;
  }

  emphasize() {
    this.text += "!";
    // Cannot assign to 'text' because it is a read-only property.
  }
}

const quote = new Quote("ABC");

quote.text = "ha";
// Cannot assign to 'text' because it is a read-only property.
```

원시 타입의 초깃값을 갖는  readonly 선언 속성은 다른 속성과 조금 다름

값의 타입이 가능한 한 좁혀진 리터럴 타입으로 유추 됨

```tsx
class RandomQuote {
  readonly explicit: string = "abc";
  readonly implicit = "abc";

  constructor() {
    if (Math.random() > 0.5) {
      this.explicit = "def";
      this.implicit = "def";
      // Type '"def"' is not assignable to type '"abc"'.
    }
  }
}

const quote = new RandomQuote();

quote.explicit; // type: string
quote.implicit; // type : "abc"
```

클래스 속성이 처음에는 모두 문자열 리터럴로 선언되므로 둘 중 하나를 string으로 확장하기 위해선 타입 애너테이션이 필요

## 8.3 타입으로서의 클래스

타입 시스템에서의 클래스는 클래스 선언이 런타임 값과 타입 애너테이션에서 사용할 수 있는 타입을 모두 생성함

```tsx
class Teacher {
  sayHello() {
    console.log("hello");
  }
}

let teacher: Teacher;

teacher = new Teacher();
teacher = "woo";
// Type 'string' is not assignable to type 'Teacher'.
```

teacher 변수에는 Teacher 클래스의 자체 인스턴스처럼 Teacher 클래스에 할당할 수 있는 값만 할당해야 함을 타입스크립트에 알려줌

타입스크립트는 클래스의 동일한 멤버를 모두 포함하는 모든 객체 타입을 클래스에 할당할 수 있는 것으로 간주 → 타입스크립트의 구조적 타이핑이 객체의 형태만 고려하기 때문

```tsx
class SchoolBus {
  getAbilitites() {
    return ["magic", "shapeshifiting"];
  }
}

function withSchoolBus(bus: SchoolBus) {
  console.log(bus.getAbilitites());
}

withSchoolBus(new SchoolBus()); // ok

withSchoolBus({ getAbilitites: () => ["transmogrification"] }); // ok

withSchoolBus({ getAbilitites: () => 123 });
// Type 'number' is not assignable to type 'string[]'.
```

## 8.4 클래스와 인터페이스

타입스크립트는 클래스 이름 뒤에 implements 키워드와 인터페이스 이름을 추가함으로써 클래스의 해당 인스턴스가 인터페이스를 준수한다고 선언 가능

```tsx
interface Learner {
  name: string;
  study(hours: number): void;
}

class Student implements Learner {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  study(hours: number) {
    for (let i = 0; i < hours; i++) {
      console.log("studying...");
    }
  }
}

class Slacker implements Learner {
  // Class 'Slacker' incorrectly implements interface 'Learner'.
  // Property 'study' is missing in type 'Slacker' but required in type 'Learner'.
  name = "kwon";
}
```

### 8.4.1 다중 인터페이스 구현

타입스크립트의 클래스는 다중 인터페이스를 구현해 선언 가능

클래스에 구현된 인터페이스 목록은 인터페이스 이름 사이에 쉼표를 넣고, 개수 제한 없이 인터페이스를 사용 가능

```tsx
interface Graded {
  grades: number[];
}

interface Reporter {
  report: () => string;
}

class ReportCard implements Graded, Reporter {
  grades: number[];

  constructor(grades: number[]) {
    this.grades = grades;
  }

  report() {
    return this.grades.join(",");
  }
}

class Empty implements Graded, Reporter {} // Error
```

## 8.5 클래스 확장

타입스크립트는 다른 클래스를 확장하거나 하위 클래스를 만드는 자바스크립트 개념에 타입 검사를 추가

기본 클래스에 선언된 모든 메서드나 속성은 파생 클래스라고 하는 하위 클래스에서 사용 가능

```tsx
class Teacher {
  teach() {
    console.log("teach");
  }
}

class StudentTeacher extends Teacher {
  learn() {
    console.log("learn");
  }
}

const teacher = new StudentTeacher();

teacher.learn();
teacher.teach();
```

### 8.5.1 할당 가능성 확장

파생 인터페이스가 기본 인터페이스를 확장하는 것처럼 하위 클래스도 기본 클래스의 멤버를 상속

하위 클래스의 인스턴스는 기본 클래스의 모든 멤버를 가지므로 기본 클래스의 인스턴스가 필요한 모든 곳에서 사용할 수 있음

```tsx
class Lesson {
  subject: string;

  constructor(subject: string) {
    this.subject = subject;
  }
}

class OnlineLesson extends Lesson {
  url: string;

  constructor(subject: string, url: string) {
    super(subject);
    this.url = url;
  }
}

let lesson: Lesson;
lesson = new Lesson("coding");
lesson = new OnlineLesson("coding", "naver.com");

let online: OnlineLesson;
online = new OnlineLesson("coding", "naver.com");
online = new Lesson("coding"); // Error
// Property 'url' is missing in type 'Lesson' but required in type 'OnlineLesson'.
```

타입스크립트의 구조적 타입에 따라 하위 클래스의 모든 멤버가 동일한 타입의 기본 클래스에 이미 존재하는 경우 기본 클래스의 인스턴스를 하위 클래스 대신 사용 가능

```tsx
class PastGrades {
  grades: number[] = [];
}

class LabeledPastGrades extends PastGrades {
  label?: string;
}

let subClass: LabeledPastGrades;
subClass = new PastGrades(); // ok
subClass = new LabeledPastGrades(); // ok
```

LabeledPastGrades는 선택적 속성인 PastGrades만 추가 → 하위 클래스 대신 기본 클래스의 인스턴스 사용 가능

### 8.5.2 재정의된 생성자

타입스크립트에서 하위 클래스는 자체 생성자를 정의할 필요가 없음 → 자체 생성자가 없는 하위 클래스는 암묵적으로 기본 클래스의 생성자 사용

자바스크립트에서 하위 클래스가 자체 생성자를 선언하면 super 키워드를 통해 기본 클래스 생성자를 호출해야 함

타입스크립트의 타입 검사기는 기본 클래스 생성자를 호출할 때 올바른 매개변수를 사용하는지 확인

```tsx
class GradeAnnouncer {
  message: string;

  constructor(grade: number) {
    this.message = grade >= 65 ? "good" : "pass";
  }
}

class PassingAnnouncer extends GradeAnnouncer {
  constructor() {
    super(100);
  }
}

class FailingAnnouncer extends GradeAnnouncer {
  constructor() {}
  // Constructors for derived classes must contain a 'super' call.
}
```

자바스크립트 규칙에 따르면 하위 클래스의 생성자는 this or super에 접근하기 전에 반드시 기본 클래스의 생성자를 호출해야 함

타입스크립트는 super()를 호출하기 전에 this or super에 접근하려고 하는 경우 타입 오류를 보고

```tsx
class GradesTally {
  grades: number[] = [];

  addGrades(...grades: number[]) {
    this.grades.push(...grades);
    return this.grades.length;
  }
}

class ContinuedGradesTally extends GradesTally {
  constructor(previousGrades: number[]) {
    this.grades = [...previousGrades]; // Error
    // 'super' must be called before accessing 'this' in the constructor of a derived class.
    super();

    console.log(this.grades.length); // ok
  }
}
```

### 8.5.3 재정의된 메서드

하위 클래스의 메서드가 기본 클래스의 메서드에 할당될 수 있는 한 하위 클래스는 기본 클래스와 동일한 이름으로 새 매서드를 다시 선언 가능

```tsx
class GradeCounter {
  countGrades(grades: string[], letter: string) {
    return grades.filter((grade) => grade === letter).length;
  }
}

class FailureCounter extends GradeCounter {
  countGrades(grades: string[]) {
    return super.countGrades(grades, "F");
  }
}

class AnyFailureChecker extends GradeCounter {
  countGrades(grades: string[]) {
    // Property 'countGrades' in type 'AnyFailureChecker' is not assignable to the same property in base type 'GradeCounter'.
    // Type '(grades: string[]) => boolean' is not assignable to type '(grades: string[], letter: string) => number'.
    // Type 'boolean' is not assignable to type 'number'.
    return super.countGrades(grades, "F") !== 0;
  }
}

// 예상 타입 : number, 실제 타입 : boolean
const counter: GradeCounter = new AnyFailureChecker();
const count = counter.countGrades(["A", "B", "F"]);
```

### 8.5.4 재정의된 속성

하위 클래스는 새 타입을 기본 클래스의 타입에 할당할 수 있는 한 동일한 이름으로 기본 클래스의 속성을 명시적으로 다시 선언 가능

재정의된 메서드와 마찬가지로 하위 클래스는 기본 클래스와 구조적으로 일치해야 함

속성을 다시 선언하는 대부분의 하위 클래스는 해당 속성을 유니언 타입의 더 구체적인 하위 집합으로 만들거나 기본 클래스 속성 타입에서 확장되는 타입으로 만듬

```tsx
class Assignment {
  grade?: number;
}

class GradeAssignment extends Assignment {
  grade: number;

  constructor(grade: number) {
    super();
    this.grade = grade;
  }
}
```

속성의 유니언 타입의 허용된 값 집합을 확장할 수는 없음 → 확장한다면 하위 클래스 속성은 더 이상 기본 클래스 속성 타입에 할당 불가

```tsx
class NumericGrade {
  value = 0;
}

class VagueGrade extends NumericGrade {
  value = Math.random() > 0.5 ? 1 : "...";
  // Property 'value' in type 'VagueGrade' is not assignable to the same property in base type 'NumericGrade'.
  // Type 'string | number' is not assignable to type 'number'.
  // Type 'string' is not assignable to type 'number'.
}

const instance: NumericGrade = new VagueGrade();

// 실제 타입 : number | string
instance.value;
```

## 8.6 추상 클래스

일부 메서드의 구현을 선언하지 않고 하위 클래스가 해당 메서드를 제공할 것을 예상하고 기본 클래스를 만드는것이 유용할 수 있음

추상화하려는 클래스 이름과 메서드 앞에 타입스크립트의 abstract 키워드를 추가

이런 추상화 메서드 선언은 추상화 기본 클래스에서 메서드의 본문을 제공하는것을 건너뛰고 인터페이스와 동일한 방식으로 선언

```tsx
abstract class School {
  readonly name: string;

  constructor(name: string) {
    this.name = name;
  }

  abstract getStudentTypes(): string[];
}

class Preschool extends School {
  getStudentTypes() {
    return ["preschooler"];
  }
}

class Absence extends School {}
// Non-abstract class 'Absence' does not implement inherited abstract member 'getStudentTypes' from class 'School'.
```

구현이 존재한다고 가정할 수 있는 일부 메서드에 대한 정의가 없기 때문에 추상 클래스를 직접 인스턴스화는 불가능 → 추상 클래스가 아닌 클래스만 인스턴스화 할 수 있음

```tsx
let school: School;
school = new Preschool("dfdfdf");
school = new School("fdfdfdf"); // Error
// Cannot create an instance of an abstract class.
```

## 8.7 멤버 접근성

자바스크립트에서는 클래스 멤버 이름 앞에 #을 추가해 private 클래스 멤버임을 나타냄

타입스크립트의 클래스 지원은 자바스크립트의 # 프라이버시보다 먼저 만들어짐

타입스크립트는 private 클래스 멤버를 지원, 타입 시스템에만 존재하는 클래스 메서드와 속성에 대해 조금 더 미묘한 프라이버시 정의 집합을 허용

타입스크립트의 멤버 접근성은 클래스 멤버의 선언 이름 앞에 다음 키워드 중 하나를 추가해 만듬

- public(기본값): 모든 곳에서 누구나 접근가능
- protected: 클래스 내부 또는 하위 클래스에서만 접근 가능
- private: 클래스 내부에서만 접근 가능

이러한 키워드는 순수하게 타입 시스템 내에만 존재 → 코드가 자바스크립트로 컴파일 되면 타입 시스템 구문과 함께 키워드도 제거

```tsx
class Base {
  isPublicImplicit = 0;
  public isPublicExplicit = 1;
  protected isProtected = 2;
  private isPrivate = 3;
  #truePrivate = 4;
}

class SubClass extends Base {
  exmples() {
    this.isPublicImplicit;
    this.isPublicExplicit;
    this.isProtected;

    this.isPrivate; // Error
    this.#truePrivate; // Error
  }
}

new SubClass().isPublicExplicit;
new SubClass().isPublicImplicit;

new SubClass().isProtected; // Error

new SubClass().isPrivate; // Error
```

타입스크립트의 멤버 접근성은 타입 시스템에서만 존재, 자바스크립트의 private 선언은 런타임에도 존재

자바스크립트 런타임에서는 # private 필드만 진정한 private

접근성 제한자는 readonly와 함께 표시 가능, readonly와 명시적 접근성 키워드로 멤버를 선언하려면 접근성 키워드를 먼저 적어야함

```tsx
class TwoKeywords {
  private readonly name: string;

  constructor() {
    this.name = "abc";
  }

  log() {
    console.log(this.name);
  }
}

const two = new TwoKeywords();

two.name = "def";
// Property 'name' is private and only accessible within class 'TwoKeywords'.
// Cannot assign to 'name' because it is a read-only property.
```

### 8.7.1 정적 필드 제한자

자바스크립트는 static 키워드를 사용해 클래스 자체에서 멤버를 선언

타입스크립트는 static 키워드를 단독으로 사용하거나 readonly와 접근성 키워드를 함께 사용할 수 있도록 지원

```tsx
class Question {
  protected static readonly answer: "javascript";
  protected static readonly prompt =
    "what's a kwon's favorite programming language?";

  guess(getAnswer: (prompt: string) => string) {
    const answer = getAnswer(Question.prompt);

    if (answer === Question.answer) {
      console.log("you got it");
    } else {
      console.log("try again");
    }
  }
}

Question.answer; // Error
// Property 'answer' is protected and only accessible within class 'Question' and its subclasses.
```