## 1. 프로토타입의 개념 이해

자바스크립트는 프로토타입 기반 언어이다. 클래스 기반 언에에서는 '상속'을 사용하지만 프로토타입 기반 언어에서는 어떤 객체를 원형으로 삼고 이를 복제(참조)함으로써 상속과 비슷한 효과를 얻는다.

```javascript
var instance = new Constructor();
```

- 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면
- Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스(instance)가 생성된다.
- 이때 instance에는 `__proto__` 라는 프로퍼티가 자동으로 부여되는데,
- 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조한다.

prototype이라는 프로퍼티와 `__proto__` 라는 프로퍼티의 관계가 프로토타입 개념의 핵심이라고 할 수 있다. prototype은 객체이고 이를 참조하는 `__proto__` 역시 당연히 객체이다. prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장한다. 그러면 인스턴스에서도 숨겨진 프로퍼티인 `__proto__` 를 통해 이 메서드들에 접근할 수 있다.

```javascript
var Person = function (name) {
  this._name = name;
};
Person.prototype.getName = function () {
  return this._name;
};
```

- Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정한다.
- Person의 인스턴스는 `__proto__` 프로퍼티를 통해 getName을 호출할 수 있다.
- instance의 `__proto__` 가 Constructor의 prototype 프로퍼티를 참조하기 때문에 둘은 같은 객체를 바라본다.

```javascript
var suzi = new Person("Suzi");
suzi.__proto__.getName(); // undefined
```

```javascript
Person.prototype === suzi.__proto__; // true
```

어떤 변수를 실행해 undefined가 나왔다는 것은 곧 이 변수가 '호출할 수 있는 함수'에 해당한다는 것을 의미한다. 만약 실행할 수 없는, 즉 함수가 아닌 다른 데이터 타입이었다면 TypeError가 발생했을 것이다.

따라서 메서드 호출 결과로 undefined가 나왔다는 것은 getName이 실행됐음을 의미하고 이로부터 getName은 함수라는 것이 입증되는 셈이다.

getName의 함수 내부를 살펴보면 this.name 값을 리턴하도록 되어있다. 그렇다면 this에 원래 의도와 다른 값이 할당된 것은 아닐까 의심해 볼 수 있다.

어떤 함수를 '메서드로서' 호출할 때는 메서드명 바로 앞의 객체가 곧 this가 된다. 따라서 `thomas.__proto__.getName()` 에서 getName 함수 내부에서의 this는 thomas가 아니라 `thomas.__proto__` 라는 객체가 되는 것이다.

즉, **이 객체 내부에는 name 프로퍼티가 없으므로 '찾고자 하는 식별자가 정의돼 있지 않을 때는 Error 대신 undefined를 반환한다'라는 자바스크립트 규약에 의해 undefined가 반환된 것**이다.

```javascript
var suzi = new Person("Suzi");
suzi.__proto__._name = "SUZI__proto__";
suzi.__proto__.getName(); // SUZI__proto__
```

`__proto__` 객체에 name 프로퍼티가 있다면 예상대로 출력된다. 따라서 관건은 this이다.

**this를 인스턴스로 하려면 `__proto__` 없이 인스턴스에서 곧바로 메서드를 쓰면 된다.**

```javascript
var suzi = new Person("Suzi", 28);
suzi.getName(); // Suzi
var iu = new Person("Jieun", 28);
iu.getName(); // Jieun
```

이처럼 `__proto__` 를 빼면 this는 instance가 된다. 이유는 바로 `__proto__` 가 생략 가능한 프로퍼티이기 때문이다. 원래부터 생략 가능하도록 정의돼 있다.

**`__proto__` 를 생락하지 않으면 this는 `suzi.__proto__` 를 가리키지만, 이를 생략하면 `suzi` 를 가리킨다. `suzi.__proto__` 에 있는 메서드인 getName을 실행하지만 this는 suzi를 바라보게 할 수 있게 된 것이다. **

> 즉, **new 연산자로 Constructor를 호출하면 instance가 만들어지는데, 이 instance의 생략 가능한 프로퍼티인 `__proto__`는 Constructor의 prototype을 참조한다.**

- 자바스크립트는 함수에 자동 객체인 prototype 프로퍼티를 생성해 놓는데,
- 해당 함수를 생성자 함수로서 사용할 경우, 즉 new 연산자와 함께 함수를 호출할 경우,
- 그로부터 생성된 인스턴스에는 숨겨진 프로퍼티인 `__proto__` 가 자동으로 생성되며,
- 이 프로퍼티는 생성자 함수의 prototype 프로퍼티를 참조한다.
- `__proto__` 프로퍼티는 생략 가능하도록 구현돼 있기 때문에
- 생성자 함수의 prototype에 어떤 메서드나 프로퍼티가 있다면 인스턴스에서도 마치 자신의 것처럼 해당 메서드나 프로퍼티에 접근할 수 있다.

### prototype과 `__proto__`

```javascript
var Constructor = function (name) {
  this.name = name;
};
Constructor.prototype.method1 = function () {};
Constructor.prototype.property1 = "Constructor Prototype Property";

var instance = new Constructor("Instance");
console.dir(Constructor);
console.dir(instance);
```

![](https://velog.velcdn.com/images/skdbsqls/post/51c1ef06-5b93-4c2d-8da0-19294fe10fa1/image.png)

1. **`console.dir(Constructor);` 결과**

- 함수라는 의미의 f와 함수 이름인 Constructor, 인자 name 확인
- 내부에는 옅은 색의 arguments, caller, length, name, prototype 등의 프로퍼티 확인
- prototype을 열어보면 method1, property1 등의 값이 짙은 색으로 표기
- constructor 등이 옅은 색으로 보인다.

2. **`console.dir(instance);` 결과**

- Constructor이 출력 : 어떤 생성자 함수의 인스턴스는 해당 생성자 함수의 이름을 표기함으로써 해당 함수의 인스턴스임을 표기
- Constructor을 열어보면 name 프로퍼티가 짙은 색으로 표기
- `[[ Prototype ]]` (`__proto__`)을 열어보면 method1, property1, constructor 등이 보인다. 이는 Constructor의 prototype과 동일한 내용으로 구성되어 있다.

\*\* 색상 차이는 { enumerable: false } 속성이 부여된 프로퍼티인지 여부에 따른다. 짙은색은 enumerable, 즉 열거 가능한 프로퍼티임을 의미하고, 옅은 색은 innumerable, 즉 열거할 수 없는 프로퍼티임을 의미한다.

```javascript
var arr = [1, 2];
console.dir(arr);
console.dir(Array);
```

![](https://velog.velcdn.com/images/skdbsqls/post/eef62fa3-0b11-431b-9a6c-0b4557935167/image.png)

![](https://velog.velcdn.com/images/skdbsqls/post/eee3bb7f-3956-406b-aaf5-c531343c7bbc/image.png)

1. **`console.dir(arr);` = 변수 arr의 출력 결과**

- Array(2) : Array라는 생성자 함수를 원형으로 삼아 생성되었고, length가 2이다.
- 인덱스 0, 1이 짙은 색으로, length와 `__proto__` 가 옅은 색으로 표기
- `__proto__` 를 열어보면 옅은 색상의 배열에 사용하는 메서드들이 거의 모두 나열되어 있다.

2. **`console.dir(Array);` = 생성자 함수인 Array의 출력 결과**

- 함수라는 의미의 f가 표기
- 함수의 기본적인 프로퍼티들인 length, name 등이 옅은 색으로 표기
- Array 함수의 정적 메서드인 from, isArray, of 확인
- prototype을 열어보면 변수 arr의 출력 결과 중 `__proto__` 와 완전히 동일한 내용으로 구성되어 있다.

**Array를 new 연산자와 함께 호출해서 인스턴스를 생성하든, 그냥 배열 리터럴를 생성하든, 어쨋든 instance [1, 2,]가 만들어진다. 이 인스턴스의 `__proto__` 는 Array.prototype을 참조하는데, `__proto__` 가 생략 가능하도록 설계돼 있기 때문에 인스턴스가 push, pop, forEach 등의 메서드를 마치 자신의 것처럼 호출 할 수 있다.**

```javascript
var arr = [1, 2];
arr.forEach(function () {}); // (O)
Array.isArray(arr); // (O)
arr.isArray(); // (X) TypeError: arr.isArray is not a function
```

한편 Array의 prototype 프로퍼티 내부에 있지 않은 from, isArray 등의 메서드들은 인스턴스가 직접 호출할 수 없다. 이들은 Array 생성자 함수에서 직접 접근해야 한다.

### constructor 프로퍼티

생성자 함수의 프로퍼티인 prototype 객체 내부에는 constructor 라는 프로퍼티가 있다. 인스턴스의 `__proto__` 객체 내부에도 마찬가지이다. 이 프로퍼티는 단어 그대로 원래의 생성자 함수(자기 자신)을 참조한다.

```javascript
var arr = [1, 2];
Array.prototype.constructor === Array; // true
arr.__proto__.constructor === Array; // true
arr.constructor === Array; // true

var arr2 = new arr.constructor(3, 4);
console.log(arr2); // [3, 4]
```

인스턴스의 `__proto__` 가 생성자 함수의 prototype 프로퍼티를 참조하며 `__proto__` 가 생략 가능하기 때문에 인스턴스에서 직접 constructor에 접근할 수 있는 수단이 생긴 것이다.

한편, constructor는 읽기 전용 속성이 부여된 예외적인 경우(기본형 리터럴 변수 - number, string, boolean)를 제외하고는 값을 바꿀 수 있다.

```javascript
var NewConstructor = function () {
  console.log("this is new constuctor!");
};
var dataTypes = [
  1, // Number & false
  "test", // String & false
  true, // Boolean & false
  {}, // NewConstructor & false
  [], // NewConstructor & false
  function () {}, // NewConstructor & false
  /test/, // NewConstructor & false
  new Number(), // NewConstructor & false
  new String(), // NewConstructor & false
  new Boolean(), // NewConstructor & false
  new Object(), // NewConstructor & false
  new Array(), // NewConstructor & false
  new Function(), // NewConstructor & false
  new RegExp(), // NewConstructor & false
  new Date(), // NewConstructor & false
  new Error(), // NewConstructor & false
];

dataTypes.forEach(function (d) {
  d.constructor = NewConstructor;
  console.log(d.constructor.name, "&", d instanceof NewConstructor);
});
```

모든 데이터가 `d instanceof NewConstructor` 명령에 대해 false를 반환한다. 즉, **constructor를 변경하더라도 참조하는 대상이 변경될 뿐 이미 만들어진 인스턴스의 원형이 바뀐다거나 데이터 타입이 변하는 것은 아니다.**

따라서 인스턴스의 생성자 정보를 알아내기 위해 constructor 프로퍼티에 의존하는 게 항상 안전하지는 않다. 이런 면에서 클래스 상속을 흉내내는 것이 가능하다.

## 2. 프로토타입 체인

### 메서드 오버라이드

만약 인스턴스가 동일한 이름의 프로퍼티 또는 메서드를 가진다면?

```javascript
var Person = function (name) {
  this.name = name;
};
Person.prototype.getName = function () {
  return this.name;
};

var iu = new Person("지금");
iu.getName = function () {
  return "바로 " + this.name;
};
console.log(iu.getName()); // 바로 지금
```

- `iu.__proto__.geName` 이 아닌 iu 객체에 있는 getName 메서드가 호출

> 여기서 일어난 현상을 **메스드 오버라이드**라고 한다. 메서드 위에 메서드를 덮어씌웠다는 표현이다. **원본을 제거하고 다른 대상으로 교체하는 것이 아니라 원본이 그대로 있는 상태에서 다른 대상을 그 위에 얹는** 이미지가 정확하다.

자바스크립트 엔진이 getName이라는 메서드를 찾는 방식은 가장 가까운 대상인 자신의 프로퍼티를 검색하고, 없으면 그다음으로 가까운 대상인 `__proto__` 를 검색하는 순서로 진행된다.

따라서 `__proto__` 에 있는 메서드는 자신에게 있는 메서드보다 검색 순서에서 밀려 호출되지 않은 것이다.

교체하는 형태라면 원본에 접근할 수 없는 형태가 되겠지만 얹는 형태라면 원본이 아래에 유지되고 있으니 원본에 접근할 수 있는 방법도 있다.

메서드 오버라이딩이 이뤄져 있는 상황에서 prototype에 있는 메서드에 접근하려면?

```javascript
console.log(iu.__proto__.geName()); // undefined

Person.prototype.name = "이지금";
console.log(iu.__proto__.geName()); // 이지금
```

- prototype 상에 name 프로퍼티가 없을 때는 undefined 출력
- prototype 상에 name 프로퍼티가 있을 때는 이지금 출력

```javascript
console.log(iu.__proto__.geName.call(iu)); // 지금
```

call이나 apply를 사용하여 this가 prototype이 아니라 인스턴스를 바라보도록 한다.

> 즉, 일반적으로 메서드가 오버라이드된 경우에는 자신으로부터 가장 가까운 메서드에만 접근할 수 있지만, 그다음으로 가까운 `__proto__` 의 메서드도 우회적인 방법을 통해서 접근할 수 있다.

### 프로토타입 체인

**객체의 내부 구조**

![](https://velog.velcdn.com/images/skdbsqls/post/135f34e7-5b93-44e3-959d-643e57d19732/image.png)

- Object의 인스턴스임을 알 수 있음
- 프로퍼티 a의 값 1이 확인
- `__proto__` 내부에는 hasOwnProperty, isPrototypeOf, toLocaleString, toString, valueOf 등의 메서드 확인
- constructor는 생성자 함수인 Object를 가리킴

**배열의 내부 구조**

![](https://velog.velcdn.com/images/skdbsqls/post/977590d0-db02-4a84-8dfc-485f19fa194c/image.png)

![](https://velog.velcdn.com/images/skdbsqls/post/e47b21d3-567b-4d28-9859-c92392bf2feb/image.png)

- `__proto__` 에는 배열 메서드 및 constructor가 있음
- 추가로, 이 `__proto__` 안에는 또다시 `__proto__` 가 등장하는데,
- 이를 열어보면 객체의 `__proto__` 와 동일한 내용으로 이루어져 있음

이는 prototype 객체가 객체이기 때문이다. **기본적으로 모든 객체의 `__proto__` 에는 Object.prototype이 연결된다.** prototype 객체도 예외가 아니다.

`__proto__` 는 생략이 가능하다. 따라서 Object.prototype 내부의 메서드도 자신의 것처럼 실행할 수 있다. 생략 가능한 `__proto__` 를 한 번 더 따라가면 Object.prototype을 참조할 수 있기 때문이다.

```javascript
var arr = [1, 2];
arr.push(3);
arr.hasOwnProperty(2); // true
```

> **어떤 데이터의 `__proto__` 프로퍼티 내부에 다시 `__proto__` 프로퍼티가 연쇄적으로 이어진 것을 프로토타입 체인이라 하고, 이 체인을 따라가며 검색하는 것을 프로토타입 체이닝이라고 한다.**
> 프로토타입 체이닝은 메서드 오버라이드와 동일한 맥락이다. 어떤 메서드를 호출하면 자바스크립트 엔진은 데이터 자신의 프로퍼티들을 검색해서 원하는 메서드가 있으면 그 메서드를 실행하고, 없으면 `__proto__` 를 검색해서 있으면 그 메서드를 실행하고, 없으면 다시 `__proto__` 를 검색해서 실행하는 식이다.

```javascript
var arr = [1, 2];
Array.prototype.toString.call(arr); // 1,2
Object.prototype.toString.call(arr); // [object Array]
arr.toString(); // 1,2

arr.toString = function () {
  return this.join("_");
};
arr.toString(); // 1_2
```

arr 변수는 배열이므로 `arr.__proto__` 는 Array.prototype을 참조한다.
Array.prototype은 객체이므로 `Array.prototype.__proto__` 는 Object.prototype을 참조한다.

toString 메서드는 Array.prototype와 Object.prototype에 둘 다 존재한다.

Array와 Object의 각 프로토타입에 있는 toString 메서드를 arr에 적용하면 arr.toString 실행 결과는 Array.prototype.toString을 적용한 것과 동일하다.

이후 arr에 직접 toString 메서드를 부여하면, Array.prototype.toString이 아닌 arr.toString이 바로 실행된다.

### 객체 전용 메서드의 예외사항

어떤 생성자 함수이든 prototype은 반드시 객체이기 때문에 Object.prototype이 언제나 프로토타입 체인의 최상단에 존재한다.

따라서 객체에서만 사용할 메서드는 다른 여느 데이터 타입처럼 프로토 타입 객체 안에 정의할 수 없다.

객체에서만 사용할 매서드를 Object.prototype 내부에 정의하면 다른 데이터 타입도 해당 메서드를 사용할 수 있게 되기 때문이다.

```javascript
Object.prototype.getEntries = function () {
  var res = [];
  for (var prop in this) {
    if (this.hasOwnProperty(prop)) {
      res.push([prop, this[prop]]);
    }
  }
  return res;
};
var data = [
  ["object", { a: 1, b: 2, c: 3 }], // [["a",1], ["b", 2], ["c",3]]
  ["number", 345], // []
  ["string", "abc"], // [["0","a"], ["1","b"], ["2","c"]]
  ["boolean", false], // []
  ["func", function () {}], // []
  ["array", [1, 2, 3]], // [["0", 1], ["1", 2], ["2", 3]]
];
data.forEach(function (datum) {
  console.log(datum[1].getEntries());
});
```

- 객체에서만 사용할 용도로 getEntries라는 메서드를 만들었으나,
- forEach에 따라 각 데이터마다 getEntries를 실행하면
- 모든 데이터가 오류 없이 결과를 반환한다.

원래 의도대로라면 객체가 아닌 다른 데이터 타입에 대해서는 오류를 던지게끔 돼야 할 텐데, 어느 데이터 타입이건 거의 무조건 프로토타입 체이닝을 통해 getEntries 메서드에 접근할 수 있으니 그렇게 동작하지 않는 것이다.

**Object 출력 결과**

![](https://velog.velcdn.com/images/skdbsqls/post/26b97901-8459-4572-a76c-6d6aa59b658c/image.png)

> 이러한 이유로 **객체만을 대상으로 동작하는 객체 전용 메서드들은 부득이 Object.prototype이 아닌 Object에 스태틱 메서드로 부여**할 수밖에 없다.

> 또한 생성자 함수인 Object와 인스턴스인 객체 리터럴 사이에는 this를 통한 연결이 불가능하기 때문에 여느 전용 메서드처럼 '메서드명 앞의 대상이 곧 this'가 되는 방식 대신 **this의 사용을 포기하고 대상 인스턴스를 인자로 직접 주입해야 하는 방식**으로 구현되어 있다.

객체 한정 메서드들은 Object.prototype이 아닌 Object에 직접 부여할 수밖에 없었던 이유는 Object.prototype이 여타의 참조형 데이터뿐 아니라 기본형 데이터조차 `__proto__` 에 반복 접근함으로써 도달할 수 있는 최상위 존재이기 때문이다.

같은 이유에서 Object.prototype에는 어떤 데이터에서도 활용할 수 있는 범용적인 메서드들만 있다.

\*\* ~~예외적으로 Object.create를 이용하면 Object.prototype의 메서드에 접근할 수 없는 경우가 있다. Object.create(null)은 `__proto__` 가 없는 객체를 생성한다.~~

### 다중 프로토타입 체인

자바스크립트의 기본 내장 데이터 타입들은 모두 프로토타입 체인이 1단계(객체)이거나 2단계(나머지)로 끝나는 경우만 있었지만 사용자가 새롭게 만드는 경우에는 그 이상도 가능하다.

> `__proto__` 가 가리키는 대상, 즉 생성자 함수의 prototype이 연결하고자 하는 상위 생성자 함수의 인스턴스를 바라보게끔 하면 된다.

```javascript
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
var g = new Grade(100, 80);
```

- 변수 g는 Grade의 인스턴스를 바라봄
- Grade의 인스턴스는 유사배열객체

```javascript
Grade.prototype = [];
```

유사배열객체인 Grade에 배열 메서드를 적용하는 방법으로 `g.__proto__`, 즉 Grade.prototype이 배열의 인스턴스를 바라보게 한다.

```javascript
console.log(g); // Grade(2) [100, 80]
g.pop();
console.log(g); // Grade(1) [100]
g.push(90);
console.log(g); // Grade(2) [100, 90]
```

이제 Grade의 인스턴스인 g에서 직접 배열의 메서드를 사용할 수 있다.

g 인스턴스의 입장에서는 프로토타입 체인에 따라 g 객체 자신이 지니는 멤버, Grade의 prototype에 있는 멤버, Array.prototype에 있는 멤버, 끝으로 Object.prototype에 있는 멤버까지 접근할 수 있게 되었다.
