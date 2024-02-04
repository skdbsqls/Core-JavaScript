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
