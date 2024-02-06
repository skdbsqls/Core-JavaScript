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
