## 1. 상황에 따라 달라지는 this

다른 대부분의 객체지향 언어에서 this는 클랙스로 생성한 인스턴스 객체를 의미한다. 그러나 자바스크립트에서의 this는 어디서든 사용할 수 있다. 자바스크립트에서 this는 함수를 호출할 때 결정된다. 즉, **어떤 방식으로 함수를 호출하느냐에 따라 this의 값이 달라지는 것이다.**

### 전역 공간에서의 this

> **전역 공간에서 this는 전역 객체를 가리킨다.** 전역 객체는 자바스크립트 런타임 환경에 따라 다른 이름과 정보를 가지고 있는데, **브라우저 환경에서 전역객체는 Window이고 Node.js 환경에서 전역객체는 global이다.**

```javascript
// 전역변수와 전역객체(1)
var a = 1;
console.log(a); // (1) 1
console.log(window.a); // (2) 1
console.log(this.a); // (3) 1
```

전역공간에서의 this는 전역객체를 의미하므로 (2)와 (3)이 같은 값을 출력하는 것은 당연하다. 그렇다면 그 값이 1인 이유는 무엇일까?

**자바스크립트의 모든 변수는 실은 특정 객체의 프로퍼티로서 동작하기 때문이다.** 사용자가 var 연산자를 이용해 변수를 선언하더라도 실제 자바스크립트 엔진은 어떤 특정객체의 프로퍼티로 인식한다는 것이다.

이때, 특정 객체란 바로 실행 컨텍스트의 LexicalEnvrionment로, 실행 컨텍스트는 변수를 수집해서 L.E의 프로퍼티에 저장한다. 이후 어떤 변수를 호출하면 L.E를 조회해서 일치하는 프로퍼티가 있는 경우 그 값을 반환하는데, 전역 컨텍스트의 경우 L.E는 전역객체를 그대로 참조한다.

즉, **전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로 할당한다는 것이다.**

따라서 (1)의 출력값이 1인 이유는 변수 a에 접근하고자 하면 스코프 체인에서 a를 검색하다가 가장 마지막에 도달하는 전역 스코프의 L.E, 즉 전역객체에서 해당 프로퍼티 a를 발견해서 그 값을 반환하기 때문이다.

> 전역 공간에서는 var로 변수를 선언하는 대신 window의 프로퍼티에 직접 할당하더라도 결과적으로 var로 선언한 것과 똑같이 동작할 것이라는 예상을 할 수 있다. 이는 대부분의 경우에는 맞지만, **'삭제' 명령의 경우 다르다.**

```javascript
// 전역변수와 전역객체(2)
var a = 1;
window.b = 2;
console.log(a, window.a, this.a); // 1 1 1
console.log(b, window.b, this.b); // 2 2 2

window.a = 3;
b = 4;
console.log(a, window.a, this.a); // 3 3 3
console.log(b, window.b, this.b); // 4 4 4
```

```javascript
// 전역변수와 전역객체(3)
var a = 1;
delete window.a; // false
console.log(a, window.a, this.a); // 1 1 1

var b = 2;
delete b; // false
console.log(b, window.b, this.b); // 2 2 2

window.c = 3;
delete window.c; // true
console.log(c, window.c, this.c); // Uncaught ReferenceError: c is not defined

window.d = 4;
delete d; // true
console.log(d, window.d, this.d); // Uncaught ReferenceError: d is not defined
```

처음부터 전역객체의 프로퍼티로 할당한 경우에는 삭제가 되는 반면, 전역변수로 선언한 경우에는 삭제가 되지 않는다. 왜냐하면 전역변수를 선언하면 자바스크립트 엔진이 이를 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의하기 때문이다.

이처럼 **var로 선언한 전역변수와 전역객체의 프로퍼티는 호이스팅 여부 및 configurable 여부에서 차이를 보인다. **

### 함수 vs 메서드

> 함수를 실행하는 방법에는 여러 가지가 있는데, 가장 일반적인 방법 두 가지는 **함수로서 호출하는 경우와 메서드로서 호출하는 경우**이다. 이 둘을 구분하는 차이는 독립성에 있는데, **함수는 그 자체로 독립적인 기능을 수행하는 반면, 메서드는 자신을 호출한 대상 객체에 관한 동작을 수행한다.**

```javascript
// 함수로서 호출, 메서드로서 호출
var func = function (x) {
  console.log(this, x);
};
func(1); // (1) Window { ... } 1

var obj = {
  method: func,
};
obj.method(2); // (2) { method: f } 2
```

(1)과 (2) 모두 첫 번째 줄에서 선언한 함수를 참조하지만, (1)에서 func 호출했더니 this로 전역객체 Window가 출력되는 반면, (2)에서 obj.method를 호출했더니 this로 obj가 출력되었다.

즉, **원래의 익명함수는 그대로인데 이를 변수에 담아 호출한 경우와 obj 객체의 프로퍼티에 할당해서 호출한 경우에 this가 달라지는 것이다.**

그렇다면 '함수로서 호출'과 '메서드로서 호출'은 어떻게 구분할까?

```javascript
// 메서드로서 호출 - 점 표기법, 대괄호 표기법
var obj = {
  method: function (x) {
    console.log(this, x);
  },
};
obj.method(1); // { method: f } 1
obj["method"](2); // { method: f } 2
```

바로 **함수 앞에 점(.)이 있는지 여부**만으로 간단하게 구분할 수 있다. (혹은 대괄호 표기법에 따른 경우에도 메서드로서 호출한 것이다.)

즉, 어떤 함수를 호출할 때 그 함수 이름(프로퍼티) 앞에 객체가 명시돼 있는 경우에는 메서드로 호출한 것이고, 그렇지 않은 모든 경우에는 함수로 호출한 것이다.

### 메서드 내부에서의 this

> this에는 호출한 주체에 대한 정보가 담긴다. 즉, 어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체이다. 따라서 **함수명 앞의 객체가 곧 this**가 된다.

```javascript
// 메서드 내부에서의 this
var obj = {
  methodA: function () {
    console.log(this);
  },
  inner: {
    methodB: function () {
      console.log(this);
    },
  },
};
obj.methodA(); // { methodA: f, inner: {...} }    ( === obj)
obj["methodA"](); // { methodA: f, inner: {...} } ( === obj)

obj.inner.methodB(); // { methodB: f }            ( === obj.inner)
obj.inner["methodB"](); // { methodB: f }         ( === obj.inner)
obj["inner"].methodB(); // { methodB: f }         ( === obj.inner)
obj["inner"]["methodB"](); // { methodB: f }      ( === obj.inner)
```

### 함수 내부에서의 this

> this에는 호출한 주체에 대한 정보가 담기는데, 함수로서 호출한 것은 호출 주체를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없다. 실행 컨텍스트를 활성화할 당시에 this가 지정되지 않은 경우 this는 전역 객체를 바라본다. 따라서 **함수에서의 this는 전역 객체를 가리킨다.**

### 메서드의 내부함수에서의 this

```javascript
// 내부함수에서의 this
var obj1 = {
  outer: function () {
    console.log(this); // (1)
    var innerFunc = function () {
      console.log(this); // (2) (3)
    };
    innerFunc();

    var obj2 = {
      innerMethod: innerFunc,
    };
    obj2.innerMethod();
  },
};
obj1.outer();
```

- (1) : 15번째 줄에서 obj1.outer를 호출하면 (1)에서 this는 obj1이 바인딩된다.
- (2) : 7번째 줄에서 innerFunc를 호출하면 함수로서 호출된 (2)에서는 this는 전역 객체(Window)가 바인딩된다.
- (3) : 12번째 줄에서 obj2.innerMethod를 호출하면 (3)에서는 this는 obj2가 바인딩된다.

7번째 줄과 12번째 줄에서는 같은 함수(innerFunc)를 호출했지만, **7번째 줄에서는 함수로서 12번째 줄에서는 메서드로서 호출했기 때문에 바인딩되는 this의 대상이 서로 달라진 것이다.**

결국 this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경(메서드 내부인지, 함수 내부인지 등)은 중요하지 않고, 오직 해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기의 유무가 관건인 것이다.

이렇게 하면 this에 대한 구분은 명확히 할 수 있지만, 그 결과 this라는 단어가 주는 인상과 사뭇 달라져 버렸다. 호출 주체가 없을 때는 자동으로 전역객체를 바인딩하지 않고 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있으면 좋겠다.

**ES5까지 내부 함수에서 this를 우회하는 방법**

```javascript
// 내부함수에서의 this를 우회하는 방법
var obj = {
  outer: function () {
    console.log(this); // (1) { outer: f }
    var innerFunc1 = function () {
      console.log(this); // (2) Window { ... }
    };
    innerFunc1();

    var self = this;
    var innerFunc2 = function () {
      console.log(self); // (3) { outer: f }
    };
    innerFunc2();
  },
};
obj.outer();
```

- (2) : innerFunc1 내부에서 this는 전역객체를 가리킨다.
- (3) : outer 스코프에서 self라는 변수에 this를 저장한 상태에서 호출한 innerFunc2의 경우 객체 obj가 출력된다.

이처럼 **상위 스코프의 this를 변수에 저장하여 내부 함수에서 활용하는 방식으로 this를 우회할 수 있다.**

**ES6에서 this를 바인딩하지 않는 화살표 함수**

```javascript
// this를 바인딩하지 않는 화살표 함수
var obj = {
  outer: function () {
    console.log(this); // (1) { outer: f }
    var innerFunc = () => {
      console.log(this); // (2) { outer: f }
    };
    innerFunc();
  },
};
obj.outer();
```

- (2) : innerFunc 내부에서 this는 obj를 가리킨다.

ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자, this를 바인딩하지 않는 화살표 함수가 새로 도입되었다. **화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지면서, 상위 스코프의 this를 그대로 활용할 수 있다. **

### 콜백 함수 호출 시 그 함수 내부에서의 this

> 함수 A의 제어권을 다른 함수(또는 메서드) B에게 넘겨주는 경우 함수 A를 콜백 함수라 한다. 이때 함수 A는 함수 B의 내부 로직에 따라 실행되며, **this 역시 함수 B 내부 로직에서 정한 규칙에 따라 값이 결정된다. **

콜백 함수도 함수이기 때문에 기본적으로 this가 전역객체를 참조하지만, 제어권을 받은 함수에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조한다.

```javascript
// 콜백 함수 내부에서의 this
setTimeout(function () {
  console.log(this);
}, 300); // (1)

[1, 2, 3, 4, 5].forEach(function (x) {
  // (2)
  console.log(this, x);
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener("click", function (e) {
  // (3)
  console.log(this, e);
});
```

- (1) : setTimeout 함수 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않았기 때문에 전역객체가 출력된다.
- (2) : forEach 메서드 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않았기 때문에 전역객체와 배열의 각 요소가 총 5회 출력된다.
- (3) : addEventListener 메서드는 콜백 함수를 호출할 때 자신의 this를 상속하도록 정의돼어 있기 때문에 버튼을 클릭하면 앞서 지정한 엘리먼트와 클릭 이벤트에 관한 정보가 담긴 객체가 출력된다.

이처럼 콜백 함수에서의 this는 정의할 수 없다. 콜백 함수의 제어권을 가지는 함수(메서드)가 콜백 함수에서의 this를 무엇으로 할지를 결정하며, 특별히 정의하지 않는 경우에는 함수와 마찬가지로 기본적으로 전역객체를 바라본다.

### 생성자 함수 내부에서의 this

생성자 함수는 어떤 공통된 성질을 지니는 객체들을 생성하는 데 사용하는 함수이다. 객체지향 언어에서는 생성자를 클래스, 클래스를 통해 만든 객체를 인스턴스라고 한다.

> 자바스크립트는 함수에 생성자로서의 역할을 함께 부여했다. new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작하게 된다. 그리고 어떤 **함수가 생성자 함수로서 호출된 경우 내부에서의 this는 곧 새로 만들 구체적인 인스턴스 자신이 된다.**

생성자 함수를 호출하면 우선 생성자의 prototype 프로퍼티를 참조하는 `__proto__` 라는 프로퍼티가 있는 객체를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여한다.

```javascript
// 생성자 함수
var Cat = function (name, age) {
  this.bark = "야옹";
  this.name = name;
  this.age = age;
};
var choco = new Cat("초코", 7); // (1)
var nabi = new Cat("나비", 5); // (2)
console.log(choco, nabi);

/* 결과
Cat { bark: '야옹', name: '초코', age: 7 }
Cat { bark: '야옹', name: '나비', age: 5 }
*/
```

(1)에서 실행한 생성자 함수 내부에서의 this는 choco 인스턴스를, (2)에서 실행한 생성자 함수 내부에서의 this는 nabi 인스턴스를 가리킴을 알 수 있다.

## 2. 명시적으로 this를 바인딩하는 방법

### call 메서드

```javascript
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

> **call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령이다.** 이때 메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 한다.

```javascript
// call 메서드(1)
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // Window{ ... } 1 2 3
func.call({ x: 1 }, 4, 5, 6); // { x: 1 } 4 5 6
```

함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

```javascript
// call 메서드(2)
var obj = {
  a: 1,
  method: function (x, y) {
    console.log(this.a, x, y);
  },
};

obj.method(2, 3); // 1 2 3
obj.method.call({ a: 4 }, 5, 6); // 4 5 6
```

메서드에 대해서도 마찬가지로 객체의 메서드를 그냥 호출하면 this는 객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있다.

### apply 메서드

```javascript
Function.prototype.apply(thisArg[, argsArray])
```

> **apply 메서드는 call 메서드와 기능적으로 완전히 동일하다.** call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면, **apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다는 점**에서만 차이가 있다.

```javascript
// apply 메서드
var func = function (a, b, c) {
  console.log(this, a, b, c);
};
func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6

var obj = {
  a: 1,
  method: function (x, y) {
    console.log(this.a, x, y);
  },
};
obj.method.apply({ a: 4 }, [5, 6]); // 4 5 6
```

### call / apply 메서드의 활용

**(1) 유사배열객체에 배열 메서드를 적용**

> 객체에는 배열 메서드를 직접 적용할 수 없다. 그러나 키가 0 또는 양의 정수인 프로퍼티가 존재하고 length 프로퍼티의 값이 0 또는 양의 정수인 객체, 즉 배열의 구조와 유사한 객체인 유사배열객체인 경우 call 또는 apply 메서드를 이용해 배열 메서드를 차용할 수 있다.

```javascript
// call/apply 메서드의 활용(1-1) 유사배열객체에 배열 메서드를 적용
var obj = {
  0: "a",
  1: "b",
  2: "c",
  length: 3,
};
Array.prototype.push.call(obj, "d"); // (1)
console.log(obj); // { 0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4 }

var arr = Array.prototype.slice.call(obj); // (2)
console.log(arr); // [ 'a', 'b', 'c', 'd' ]
```

- (1) : 배열 메서드인 `push` 를 객체 obj에 적용해 프로퍼티 3에 'd'를 추가했다.
- (2) : `slice` 메서드를 적용해 객체를 배열로 전환했다.
- `slice` 메서드는 매개변수를 아무것도 넘기지 않을 경우 그냥 원본 배열의 얕은 복사본을 반환한다.

```javascript
// call/apply 메서드의 활용(1-2) arguments, NodeList에 배열 메서드를 적용
function a() {
  var argv = Array.prototype.slice.call(arguments);
  argv.forEach(function (arg) {
    console.log(arg);
  });
}
a(1, 2, 3);

document.body.innerHTML = "<div>a</div><div>b</div><div>c</div>";
var nodeList = document.querySelectorAll("div");
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function (node) {
  console.log(node);
});
```

함수 내부에서 접근할 수 있는 arguments 객체, querySelectorAll, getElementsByClassName 등의 Node 선택자로 선택한 결과인 NodeList도 위의 방법으로 배열로 전환해서 활용할 수 있다.

```javascript
// call/apply 메서드의 활용(1-3) 문자열에 배열 메서드를 적용
var str = "abc def";

Array.prototype.push.call(str, ", pushed string");
// Error: Cannot assign to read only property 'length' of object [object String]

Array.prototype.concat.call(str, "string"); // [String {"abc def"}, "string"]

Array.prototype.every.call(str, function (char) {
  return char !== " ";
}); // false

Array.prototype.some.call(str, function (char) {
  return char === " ";
}); // true

var newArr = Array.prototype.map.call(str, function (char) {
  return char + "!";
});
console.log(newArr); // ['a!', 'b!', 'c!', ' !', 'd!', 'e!', 'f!']

var newStr = Array.prototype.reduce.apply(str, [
  function (string, char, i) {
    return string + char + i;
  },
  "",
]);
console.log(newStr); // "a0b1c2 3d4e5f6"
```

배열처럼 인덱스와 length 프로퍼티를 지니는 문자열에 대해서도 마찬가지이다.

단, 문자열의 경우 length 프로퍼티가 읽기 전용이기 때문에 원본 문자열에 변경을 가하는 메서드( `push` , `pop` , `shift` , `unshift` , `splice` 등)는 에러를 던지며, `concat` 처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없다.

사실 call/apply를 이용해 형변환하는 것은 'this를 원하는 값으로 지정해서 호출한다'라는 본래의 메서드의 의도와는 다소 동떨어진 활용법이라 할 수 있다.

```javascript
// call/apply 메서드의 활용(1-4) ES6의 Array.from 메서드
var obj = {
  0: "a",
  1: "b",
  2: "c",
  length: 3,
};
var arr = Array.from(obj);
console.log(arr); // ['a', 'b', 'c']
```

이에 ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터 타입을 배열로 전환하는 `Array.from` 메서드를 새로 도입했다.

**(2) 생성자 내부에서 다른 생성자를 호출**

> 생성자 내부에서 다른 생성자와 공통된 내용이 있을 경우 call 또는 apply를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있다.

```javascript
// call.apply 메서드의 활용(2) 생성자 내부에서 다른 생성자를 호출
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}
function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}
function Employee(name, gender, company) {
  Person.apply(this, [name, gender]);
  this.company = company;
}
var by = new Student("보영", "female", "단국대");
var jn = new Employee("재난", "male", "구골");
```

- Student, Employee 생성자 함수 내부에서 Person 생성자 함수를 호출해서 인스턴스의 속성을 정의했다.

**(3) 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용**

> 여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 apply 메서드를 활용할 수 있다.

```javascript
// call.apply 메서드의 활용(3-1) 최대/최솟값을 구하는 코드를 직접 구현
var numbers = [10, 20, 3, 16, 45];
var max = (min = numbers[0]);
numbers.forEach(function (number) {
  if (number > max) {
    max = number;
  }
  if (number < min) {
    min = number;
  }
});
console.log(max, min); // 45 3
```

```javascript
// call.apply 메서드의 활용(3-2) 여러 인수를 받는 메서드(Math.max/Math.min)에 apply 적용
var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);
console.log(max, min); // 45 3
```

- 배열에서 최대/최솟값을 구해야 할 경우 직접 구현하는 것보다, Math.max/Math.min 메서드에 apply를 적용하는 것이 훨씬 간단하다.

> **단, ES6의 전개 연산자를 이용하면 apply를 적용하는 것보다 더욱 간단하다.**

```javascript
// call.apply 메서드의 활용(3-2) ES6의 전개 연산자 활용
const numbers = [10, 20, 3, 16, 45];
const max = Math.max(...numbers);
const min = Math.min(...numbers);
console.log(max, min); // 45 3
```

### bind 메서드

```javascript
Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]])
```

> bind 메서드는 ES5에서 추가된 기능으로, **call과 비슷하지만 즉시 호출하지 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드이다.** 다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록된다.

**(1) this 지정과 부분 적용 함수 구현**

bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 지닌다.

```javascript
// bind 메서드 - this 지정과 부분 적용 함수 구현
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};
func(1, 2, 3, 4); // Window{ ... } 1 2 3 4

var bindFunc1 = func.bind({ x: 1 }); // (1)
bindFunc1(5, 6, 7, 8); // { x: 1 } 5 6 7 8

var bindFunc2 = func.bind({ x: 1 }, 4, 5); // (2)
bindFunc2(6, 7); // { x: 1 } 4 5 6 7
bindFunc2(8, 9); // { x: 1 } 4 5 8 9
```

- (1) this 지정 : bindFunc1 변수에는 func에 this를 { x: 1 }로 지정한 새로운 함수가 담긴다.
- (2) 부분 적용 함수 구현 : bindFunc2에는 func에 this를 { x: 1 }로 지정하고, 앞에서부터 두 개의 인수를 각각 4, 5로 지정한 새로운 함수를 담는다.

**(2) name 프로퍼티**

bind 메서드를 적용해서 새로 만든 함수는 name 프로퍼티에 동사 bind의 수동태인 'bound'라는 접두어가 붙는다.

```javascript
// bind 메서드 - name 프로퍼티
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};
var bindFunc = func.bind({ x: 1 }, 4, 5);
console.log(func.name); // func
console.log(bindFunc.name); // bound func
```

**(3) 상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기**

```javascript
// 내부함수에 this 전달 - call
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this);
    };
    innerFunc.call(this);
  },
};
obj.outer();
```

```javascript
// 내부함수에 this 전달 - bind
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this);
    }.bind(this);
    innerFunc();
  },
};
obj.outer();
```

메서드의 내부함수에서 메서드의 this를 그대로 바라보게 하기 위한 방법으로 변수를 활용한 우회법보다는 call, apply 또는 bind 메서드를 이용하면 더 깔끔하다.

```javascript
// bind 매서드 - 내부함수에 this 전달
var obj = {
  logThis: function () {
    console.log(this);
  },
  logThisLater1: function () {
    setTimeout(this.logThis, 500);
  },
  logThisLater2: function () {
    setTimeout(this.logThis.bind(this), 1000);
  },
};
obj.logThisLater1(); // Window { ... }
obj.logThisLater2(); // obj { logThis: f, ... }
```

콜백 함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수 또는 메서드에 대해서도 bind 메서드를 이용하면 this 값을 사용자 입맛에 맞게 바꿀 수 있다.

### 화살표 함수의 예외사항

> ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외되었다. 즉, 이 **함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근한다.** 따라서 화살표 함수를 활용하면 변수로 this를 우회하거나 call/apply/bind를 적용할 필요가 없어 더욱 편리하다.

```javascript
// 화살표 함수 내부에서의 this
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = () => {
      console.log(this);
    };
    innerFunc();
  },
};
obj.outer();
```

### 콜백 함수 내에서의 this

> **콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있다.** 이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있다. **이런 형태는 여러 내부 요소에 대해 같은 동작을 반복 수행해야 하는 배열 메서드에 많이 포진**돼 있으며, 같은 이유로 ES6에서 새로 등장한 Set, Map 등의 메서드에서도 일부 존재한다.

```javascript
// thisArg를 받는 경우 예시 - forEach 메서드
var report = {
  sum: 0,
  count: 0,
  add: function () {
    var args = Array.prototype.slice.call(arguments);
    args.forEach(function (entry) {
      // (1)
      this.sum += entry;
      ++this.count;
    }, this);
  },
  average: function () {
    return this.sum / this.count;
  },
};
report.add(60, 85, 95); // (2)
console.log(report.sum, report.count, report.average()); // 240 3 80
```

- (1) : 배열을 순회하면서 콜백 함수를 실행하는데, 이때 콜백 함수 내부에서의 this는 forEach의 두 번째 인자로 전달해준 this가 바인딩된다.
- (2) 60, 85, 95를 인자로 삼아 add 메서드를 호출하면 이 세 인자를 배열로 만들어 forEach 메서드가 실행되는데, 콜백 함수 내부에서의 this는 add 메서드의 this가 전달된 상태이므로 add 메서드의 this(report)를 그대로 가리킨다.

아래는 이 밖에 thisArg를 인자로 받는 메서드 예시이다.

```javascript
// 콜백 함수와 함께 thisArg를 인자로 받는 메서드
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(arrayLike[, callback[, thisArg]])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

> ## 📌 정리

**상황에 따라 달라지는 this**

- 전역공간에서의 this는 전역객체(브라우저에서는 window, Node.js에서는 global)를 참조한다.
- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조한다.
- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조한다. 메서드의 내부함수에서도 같다.
- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조한다.
- 생성자 함수에서의 this는 생성될 인스턴스를 참조한다.

**명시적 this 바인딩**

- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출한다.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만단다.
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 한다.
