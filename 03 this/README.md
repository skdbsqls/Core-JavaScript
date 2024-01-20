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
