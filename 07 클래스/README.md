자바스크립트는 프로토타입 기반 언어라서 '상속' 개념이 존재하지 않았다. 이는 클래스 기반의 다른 언어에 익숙한 많은 개발자들을 혼란스럽게 했고, ES6에는 클래스 문법이 추가되었다.

다만, ES6의 클래스에서도 일정 부분은 프로토타입을 활용하고 있기 때문에, ES5 체제 하에서 클래스르르 흉내내기 위한 구현 방식을 학습하는 것은 의미가 있다.

## 1. 클래스와 인스턴스의 개념 이해

프로그래밍 언어에서의 클래스는 사용자가 직접 정의해야 하며, 클래스를 바탕으로 인스턴스를 만들 때 비로소 어떤 개체가 클래스의 속성을 지니게 된다. 또한 한 인스턴스는 하나의 클래스만을 바탕으로 만들어진다.

즉, 인스턴스를 생성할 때 호출할 수 있는 클래스는 오직 하나뿐이다.

> 클래스는 '공통 요소를 지니는 집단을 분류하기 위한 개념'이면서, 클래스가 먼저 정의되어야만 그로부터 공통적인 요소를 지니는 개체들을 생성할 수 있다.

## 2. 자바스크립트의 클래스

생성자 함수 Array를 new 연산자와 함께 호출하면 인스턴스가 생성된다. 이때 Array를 일종의 클래스라고 하면, Array의 prototype 객체 내부 요소들이 인스턴스에 '상속'된다고 볼 수 있다. 엄밀히는 상속이 아닌 프로토타입 체이닝에 의한 참조지만 결과적으로는 동일하게 동작한다. 한편, Array 내부 프로퍼티들 중 prototype 프로퍼티를 제외한 나머지는 인스턴스에 상속되지 않는다.

인스턴스에 상속되는지(인스턴스가 참조하는지) 여부에 따라 스태틱 멤버(static member)와 인스턴스(instance mamber)로 나뉜다.

이 분류는 다른 언어의 클래스 구성요소에 대한 정의를 차용한 것으로 클래스 입장에서 사용 대상에 따라 구분한 것이다. 그러나 여느 클래스 기반 언어와 달리 자바스크립트에서는 인스턴스에서도 직접 메서드를 정의할 수 있기 때문에 '인스턴스 메서드'라는 명청인 프로토타입에 정의한 메서드를 지칭하는 것인지, 인스턴스에 정의한 메서드를 지칭하는 것인지에 대해 혼란을 야기한다.

따라서 인스턴스 메서드 대신 '프로토타입 메서드'라고 부르는 편이 좋다.

```javascript
// 스태틱 메서드, 프로토타입 메서드
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};
Rectangle.isRectangle = function (instance) {
  return (
    instance instanceof Rectangle && instance.width > 0 && instance.height > 0
  );
};

var rect1 = new Rectangle(3, 4);
console.log(rect1.getArea()); // 12 (O)
console.log(rect1.isRectangle(rect1)); // Error (X)
console.log(Rectangle.isRectangle(rect1)); // true
```

- Rectangle 함수를 new 연산자와 함께 호출해서 생성된 인스턴스를 rect1에 할당
- 이 인스턴스에는 width, height 프로퍼티에 3, 4의 값이 할당되어 있음
- 호출한 getArea는 실제로는 `rect1.__proto__.getArea` 에 접근하는데, 이때 `__proto__` 를 생략했으므로 this가 rect1인 채로 실행됨
- 따라서 결과로는 rect1.width + rect1.height의 계산값이 반환 됨

이처럼 **인스턴스에서 직접 호출할 수 있는 메서드가 바로 프로토타입 메서드**이다.

- rect1 인스턴스에서 isRectangle이라는 메서드에 접근
- 우선, rect1에 해당 메서드가 있는지 검색했는데 없고, `reat1.__proto__` 에도 없으며, `rect1.__proto__.__proto__(= Object.prototype)` 에도 없음
- 결국 undefined를 실행하라는 명령으로, 함수가 아니기 때문에 에러가 발생한다.

이처럼 **인스턴스에서 직접 접근할 수 없는 메서드를 스태틱 메서드**라고 한다. 스태틱 메서드는 생성자 함수를 this로 해야만 호출할 수 있다.

> 프로그래밍 언어에서 클래스는 일반적인 사용 방식, 즉 구체적인 인스턴스가 사용할 메서드를 정의한 '틀'의 역할을 담당하는 목적을 가질 때의 클래스는 추상적인 개념이지만, 클래스 자체를 this로 해서 직접 접근해야만 하는 스태틱 메서드를 호출할 때의 클래스는 그 자체가 하나의 개체로서 취급된다.

## 3. 클래스 상속

### 기본 구현

```javascript
// Grade 생성자 함수 및 인스턴스
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);
```

해당 코드는 몇 가지 문제점을 가진다.

1. length 프로퍼티가 configurable(삭제 가능)하다는 점
2. Grade.prtotype에 빈 배열을 참조시켰다는 점

**1. length 프로퍼티를 삭제한 경우**

```javascript
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);

g.push(90);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }

delete g.length;
g.push(70);
console.log(g); // Grade { 0: 70, 1: 80, 2: 90, length: 1 }
```

- length 프로퍼티를 삭제하고 다시 push
- push한 값이 0번째 인덱스에 들어감
- length가 1이 됨

이는 Grade.prototype이 빈 배열을 가리키고 있기 때문이다. push 명령에 의해 자바스크립트 엔진이 g.length를 읽고자 하는데, 없으므로 프로토타입 체이닝을 차고 `g.__proto__.length` 를 읽은 것이다.

즉, 내장객체인 배열 인스턴스의 length 프로퍼티는 configurable 속성이 false라서 삭제 불가능하지만, Grade 클래스의 인스턴스는 배열 메서드를 상속하지만 기본적으로 일반 객체의 성질을 그대로 지니므로 삭제가 가능하다.

**2. 요소가 있는 배열을 prototype에 매칭한 경우**

```javascript
var Grade = function () {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = ["a", "b", "c", "d"];
var g = new Grade(100, 80);

g.push(90);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }

delete g.length;
g.push(70);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, ___ 4: 70, length: 5 }
```

- prototype에 length가 4인 배열 할당

이번에는 length를 삭제하고 나니 g.length가 없어 `g.__proto__.length` 를 찾고, 값이 4이므로 인덱스 4에 70을 넣고, 다시 g.length에 5를 부여하는 순서로 동작한다.

즉, 클래스에 있는 값이 인스턴스의 동작에 영향을 끼친다.

이처럼 인스턴스와의 관계에서는 구체적인 데이터를 지니지 않고 오직 인스턴스가 사용할 메서드만을 지니는 추상적인 틀로서만 작용하게끔 작성하지 않으면 예기치 않은 오류가 발생할 수 있다.

### 클래스가 구체적인 데이터를 지니지 않게 하는 방법

**1. 인스턴스 생성 후 프로퍼티 제거하기**

클래스가 구체적인 데이터를 지니지 않게 하는 방법에는 여러 가지가 있는데, 그 중 가장 쉬운 방법은 일단 만들고 나서 프로퍼티들을 일일이 지우고 더는 새로운 프로퍼티를 추가할 수 없게 하는 것이다.

```javascript
var extendClass1 = function (SuperClass, SubClass, subMethods) {
  SubClass.prototype = new SuperClass();
  for (var prop in SubClass.prototype) {
    if (SubClass.prototype.hasOwnProperty(prop)) {
      delete SubClass.prototype[prop];
    }
  }
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};

var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};
var Square = extendClass1(Rectangle, function (width) {
  Rectangle.call(this, width, width);
});
var sq = new Square(5);
console.log(sq.getArea()); // 25
```

이는 extendClass1 함수는 SuperClass와 SubClass, SubClass에 추가할 메서드들이 정의된 객체를 받아서 SubClass의 prototype 내용을 정리하고 freeze하는 내용으로 구성되어 있다.

**2. 빈 함수(Bridge)를 활용하기**

두 번째는 더글라스 크락포드가 제시해서 대중적으로 널리 알려진 방법으로, SubClass의 prototype에 직접 SuperClass의 인스턴스를 할당하는 대신 아무런 프로퍼티를 생성하지 않는 빈 생성자 함수(Bridge)를 하나 더 만들어서 그 prototype이 SuperClass의 prototype을 바라보게끔 한 다음, SubClass의 prototype에는 Bridge의 인스턴스를 할당하게 하는 것이다.

```javascript
var extendClass2 = (function () {
  var Bridge = function () {};
  return function (SuperClass, SubClass, subMethods) {
    Bridge.prototype = SuperClass.prototype;
    SubClass.prototype = new Bridge();
    if (subMethods) {
      for (var method in subMethods) {
        SubClass.prototype[method] = subMethods[method];
      }
    }
    Object.freeze(SubClass.prototype);
    return SubClass;
  };
})();

var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};
var Square = extendClass2(Rectangle, function (width) {
  Rectangle.call(this, width, width);
});
var sq = new Square(5);
console.log(sq.getArea()); // 25
```

Bridge라는 빈 함수를 만들고, Bridge.prototype이 Rectangle.prototype을 참조하게 한 다음, Square.prototype에 new Bridge()로 할당하면, Rectangle 자리에 Bridge가 대체하게 될 것이다. 이로써 인스턴스를 제외한 프로토타입 체인 경로상에는 더는 구체적인 데이터가 남아있지 않게 된다.

**3. Object.create 이용하기**

이는 SubClass의 prototype의 `__proto__`가 SuperClass의 prototype을 바라보되, SuperClass의 인스턴스가 되지 않으므로 앞의 두 방법보다 간단하면서 안전하다.

```javascript
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};
var Square = function (width) {
  Rectangle.call(this, width, width);
};
Square.prototype = Object.create(Rectangle.prototype);
Object.freeze(Square.prototype);

var sq = new Square(5);
console.log(sq.getArea()); // 25
```

### constructor 복구하기

앞선 방법으로 모두 기본적인 상속에는 성공했지만 SubClass 인스턴스의 constructor는 여전히 SuperClass를 가리키는 상태이다. 엄밀히는 SubClass 인스턴스에는 constructor가 없고, SubClass.prototype의 constructor에서 가리키는 대상, 즉 SuperClass가 출력될 뿐이다.

**1. 인스턴스 생성 후 프로퍼티 제거하기**

```javascript
var extendClass1 = function (SuperClass, SubClass, subMethods) {
  SubClass.prototype = new SuperClass();
  for (var prop in SubClass.prototype) {
    if (SubClass.prototype.hasOwnProperty(prop)) {
      delete SubClass.prototype[prop];
    }
  }
  SubClass.prototype.consturctor = SubClass;
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};
```

**2. 빈 함수(Bridge)를 활용하기**

```javascript
var extendClass2 = (function () {
  var Bridge = function () {};
  return function (SuperClass, SubClass, subMethods) {
    Bridge.prototype = SuperClass.prototype;
    SubClass.prototype = new Bridge();
    SubClass.prototype.consturctor = SubClass;
    Bridge.prototype.constructor = SuperClass;
    if (subMethods) {
      for (var method in subMethods) {
        SubClass.prototype[method] = subMethods[method];
      }
    }
    Object.freeze(SubClass.prototype);
    return SubClass;
  };
})();
```

**3. Object.create 이용하기**

```javascript
var extendClass3 = function (SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};
```

이제 가장 기본적인 기능 상속 및 추상화만큼은 성공적으로 달성할 수 있다.

### 상위 클래스에 접근 수단 제공

하지만 때론 하위 클래스의 메서드에서 상위 클래스의 메서드 실행 결과를 바탕으로 추가적인 작업을 수행하고 싶을 때가 있다. 하위 클래스에서 상위 클래스의 프로토타입 메서드에 접근하기 위한 별도의 수단이 있다면 좋겠다.

```javascript
var extendClass = function (SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  SubClass.prototype.super = function (propName) {
    // 추가된 부분 시작
    var self = this;
    if (!propName)
      return function () {
        SuperClass.apply(self, arguments);
      };
    var prop = SuperClass.prototype[propName];
    if (typeof prop !== "function") return prop;
    return function () {
      return prop.apply(self, arguments);
    };
  }; // 추가된 부분 끝
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};

var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function () {
  return this.width * this.height;
};
var Square = extendClass(
  Rectangle,
  function (width) {
    this.super()(width, width); // super 사용 (1)
  },
  {
    getArea: function () {
      console.log("size is :", this.super("getArea")()); // super 사용 (2)
    },
  }
);
var sq = new Square(10);
sq.getArea(); // size is : 100
console.log(sq.super("getArea")()); // 100
```

- 인자가 비었을 경우에는 SuperClass 생성자 함수에 직접 접근하는 것으로 간주
- this가 달라지는 것을 막기 위해 클로저 활용
- SuperClass의 prototype 내부의 propName에 해당하는 값이 함수가 아닌 경우에는 해당 값을 그대로 반환
- 함수인 경우 클로저를 활용해 메서드 접근

즉, SuperClass의 생성자 함수에 직접 접근하고자 할 때는 this.super(), SuperClass의 프로토타입 메서드에 접근하고잫 할 때는 this.super(propName)와 같이 사용하면 된다.

## 4. ES6의 클래스 및 클래스 상속

**ES5와 ES6의 클래스 문법 비교**

```javascript
var ES5 = function (name) {
  this.name = name;
};
ES5.staticMethod = function () {
  return this.name + " staticMethod";
};
ES5.prototype.method = function () {
  return this.name + " method";
};
var es5Instance = new ES5("es5");
console.log(ES5.staticMethod()); // es5 staticMethod
console.log(es5Instance.method()); // es5 method

var ES6 = class {
  constructor(name) {
    this.name = name;
  }
  static staticMethod() {
    return this.name + " staticMethod";
  }
  method() {
    return this.name + " method";
  }
};
var es6Instance = new ES6("es6");
console.log(ES6.staticMethod()); // es6 staticMethod
console.log(es6Instance.method()); // es6 method
```

**ES6의 클래스 상속**

```javascript
var Rectangle = class {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
};
var Square = class extends Rectangle {
  constructor(width) {
    super(width, width);
  }
  getArea() {
    console.log("size is :", super.getArea());
  }
};
```

> ## 📌 정리

- 클래스는 어떤 사물의 공통 속성을 모아 정의한 추상적인 개념이다.
- 인스턴스는 클래스의 속성을 지니는 구체적인 사례이다.
- 상위 클래스(superclass)의 조건을 충족하면서 더욱 구체적인 조건이 추가된 것을 하위 클래스(subclass)라고 한다.
- 클래스의 prototype 내부에 정의된 메서드를 프로토타입 메서드라고 하며, 이들은 인스턴스가 마치 자신의 것처럼 호출할 수 있다.
- 클래스(생성자 함수)에 직접 정의한 메서드를 스태틱 메서드라고 하며, 이들은 인스턴스가 직접 호출할 수 없고 클래스(생성자 함수)에 의해서만 호출할 수 있다.
- 클래스 상속을 흉내 내기 위한 방법에는 3 가지가 있는데, SubClass.prototype에 SuperClass의 인스턴스를 할당한 다음 프로퍼티를 모두 삭제하는 방법, 빈 함수(Bridge)를 활용하는 방법, Object.create를 이용하는 방법이다.
- 세 가지 방법 모두 constructor 프로퍼티가 원래의 생성자 함수를 바라보도록 조정해야 한다.
