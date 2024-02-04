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
