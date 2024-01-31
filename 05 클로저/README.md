## 1. 클로저의 의미 및 원리 이해

클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성이다.

MDN에서는 클로저에 대해 "A closure is the combination of a function and the lexical environment within which that function was decleared."라고 소개 한다.

직역하면, **_"클로저는 함수와 그 함수가 선언될 당시의 lexical environment의 상호관계에 따른 현상"_**이라고 할 수 있다.

선언될 당시의 lexical environment은 실행 컨텍스트의 구성 요소 중 하나인 outerEnvironmentReference에 해당한다. LexicalEnvironment의 environmentRecord와 outerEnvironmentReference에 의해 변수의 유효범위인 스코프가 결정되고 스코프 체인이 가능해진다.

어떤 컨텍스트 A에서 선언한 내부함수 B의 실행 컨텍스트가 활성화된 시점에는 B의 outerEnvironmentReference가 참조하는 대상인 A의 LexicalEnvironment에도 접근이 가능하다. **A에서는 B에서 선언한 변수에 접근할 수 없지만 B에서는 A에서 선언한 변수에 접근 가능하다.**

여기서 'combination'의 의미를 파악할 수 있다.

내부함수 B가 A의 LexicalEnvironment를 언제나 사용하는 것은 아니다. 내부함수에서 외부 변수를 참조하지 않는 경우라면 combination이라고 할 수 없다. **내부함수에서 외부 변수를 참조하는 경우에 한해서만 combination, 즉 '선언될 당시의 LexicalEnvironment와의 상호관계'가 의미가 있을 것이다.**

> 이에 따르면 클로저는 **_"어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상"_**이라고 할 수 있다.

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(1)
var outer = function () {
  var a = 1;
  var inner = function () {
    console.log(++a);
  };
  inner();
};
outer();
```

- inner 함수 내부에서는 a를 선언하지 않았기 때문에 environmentRecord에서 값을 찾지 못해 outerEnvironmentReference에 지정된 상위 컨텍스트인 outer의 LexicalEnvironment에 접근해서 a를 찾는다.
- outer 함수의 실행 컨텍스트가 종료되면 LexicalEnvironment에 지정된 식별자들(a, inner)에 대한 참조를 지운다.
- 각 주소에 저장돼 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집 대상이 된다.

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(2)
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner();
};
var outer2 = outer();
console.log(outer2); // 2
```

- inner 함수를 실행한 결과를 리턴하고 있으므로 outer 함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 없어진다.
- a, inner 변수의 값들은 언젠가 가비지 컬렉터에 의해 소멸된다.

이 두 가지 예시는 outer 함수의 실행 컨텍스트가 종료되기 이전에 inner 함수의 실행 컨텍스트가 종료돼 있으며, 이후 별도로 inner 함수를 호출할 수 없다.

그렇다면 outer의 실행 컨텍스트가 종료된 후에도 inner 함수를 호출할 수 있게 만들면 어떨까?

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(3)
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

- inner 함수 자체를 반환한다.
- outer 함수의 실행 컨텍스트가 종료될 때 outer2 변수는 outer의 실행 결과인 inner 함수를 참조한다.
- outer2를 호출하면 앞서 반환된 함수인 inner가 실행된다.
- inner 함수의 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없다.
- outer-EnvironmentRecord에는 inner 함수가 선언된 위치의 LexicalEnvironment가 참조 복사된다.
- inner 함수는 outer 함수의 LexicalEnvironment가 담긴다.
- 스코프 체이닝에 따라 outer에서 선언한 변수 a에 접근해서 1만큼 증가시킨 후 그 값인 2를 반환하고, inner 함수의 실행 컨텍스트가 종료된다.

inner 함수의 실행 시점에는 outer 함수는 이미 실행이 종료된 상태인데 outer 함수의 LexicalEnvironment에 어떻게 접근할 수 있는 걸까?

이는 가비지 컬렉터의 동작 방식 때문인데, **가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않는다.**

outer 함수는 실행 종료 시점에 inner 함수를 반환한다. **외부함수인 outer의 실행이 종료되더라도 내부함수인 inner 함수는 언젠간 outer2를 실행함으로써 호출될 가능성이 열린 것이다.** 언젠가 inner 함수의 실행 컨텍스트가 활성화되면 outerEnvironmentReference가 outer 함수의 LexicalEnvironment에 필요할 것이므로 수집 대상에서 제외된다. 따라서 inner 함수가 이 변수에 접근할 수 있는 것이다.

이처럼 함수의 실행 컨텍스트가 종료된 이후에도 LexicalEnvironment가 가비지 컬렉터의 수집 대상에서 제외되는 경우 지역변수를 참조하는 내부함수가 외부로 전달된 경우가 유일하다.

즉, "어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상"이란, "외부 함수의 LexicalEnvironment가 가비지 컬렉팅되지 않는 현상"을 말하는 것이다.

> 이를 바탕으로 정의를 다시 고쳐보면, **_클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상_**을 말한다.

다만, 여기서 '외부로 전달'이 곧 return만을 의미하는 것은 아니다.

```javascript
// (1) setInterval/setTimeout
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId);
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})();
```

```javascript
// (2) eventListener
(function () {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "click";
  button.addEventListener("click", function () {
    console.log(++count, "times clicked");
  });
  document.body.appendChild(button);
})();
```

(1)은 별도의 외부객체인 window의 메서드에 전달할 콜백 함수 내부에서 지역변수를 참조한다.
(2)는 별도의 외부객체인 DOM의 메서에 등록할 handler 함수 내부에서 지역변수를 참조한다.

두 상황 모두 두 지역변수를 참조하는 내부함수를 외부에서 전달했기 때문에 클로저이다.

## 2. 클로저와 메모리 관리

메모리 소모는 클로저의 본질적인 특성일 뿐이다. 따라서 이러한 특성을 정확히 이해하고 잘 활용하도록 노력해야 한다.

'메모리 누수'라는 표현은 개발자의 의도와 달리 어떤 값의 참조 카운트가 0이 되지 않아 GC의 수거 대상이 되지 않는 경우에는 맞는 표현이지만 개발자가 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우는 '누수'라고 할 수 없다.

과거에는 의도치 않게 누수가 발생하는 여러 가지 상황들이 있었지만 그중 대부분은 최근의 자바스크립트 엔진에서는 발생하지 않거나 거의 발견하지 힘들어졌다.

이제는 의도대로 설계한 '메모리 소모'에 대한 관리법만 잘 파악해서 적용하는 것으로 충분하다.

클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수를 메모리를 소모하도록 함으로써 발생한다. 그렇다면 그 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 해주면 된다.

> 참조 카운트를 0으로 반드는 방법은 식별자에 참조형이 아닌 기본형 데이터(보통 null이나 undefined)를 할당하면 된다.

```javascript
// (1) return에 의한 클로저의 메모리 해제
var outer = (function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
})();
console.log(outer());
console.log(outer());
outer = null; // outer 식별자의 inner 함수 참조를 끊음
```

```javascript
// (2) setInterval에 의한 클로저의 메모리 해제
(function () {
  var a = 0;
  var intervalId = null;
  var inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId);
      inner = null; // inner 식별자의 함수 참조를 끊음
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})();
```

```javascript
// (3) eventListener에 의한 클로저의 메모리 해제
(function () {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "click";

  var clickHandler = function () {
    console.log(++count, "times clicked");
    if (count >= 10) {
      button.removeEventListener("click", clickHandler);
      clickHandler = null; // clickHandler 식별자의 함수 참조를 끊음
    }
  };
  button.addEventListener("click", clickHandler);
  document.body.appendChild(button);
})();
```

## 3. 클로저 활용 사례

### 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때

```javascript
// 콜백 함수와 클로저(1)
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul"); // (공통 코드)

fruits.forEach(function (fruit) {
  // (A)
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", function () {
    // (B)
    alert("your choice is " + fruit);
  });
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

- fruits 변수를 순회하며 li를 생성하고, 각 li를 클릭하면 해당 리스너에 기억된 콜백 함수를 실행한다.
- forEach 메서드에 넘겨준 익명의 콜백 함수(A)는 그 내부에서 외부 변수를 사용하지 않고 있으므로 클로저가 없다.
- addEventListener에 넘겨준 콜백 함수(B)에는 fruit이라는 외부 변수를 참조하고 있으므로 클로저가 있다.

(A)는 fruits의 개수만큼 실행되며, 그때마다 새로운 실행 컨텍스트가 활성화될 것이다. A의 실행 여부와 무관하게 클릭 이벤트에 의해 각 컨텍스트의 (B)가 실행될 때는 (B)의 outerEnvironmentReference가 (A)의 LexicalEnvironment를 참조하게 될 것이다. 따라서 **최소한 (B) 함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 GC 대상에서 제외되어 계속 참조 가능할 것이다.**

```javascript
// 콜백 함수와 클로저(2)
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul");

var alertFruit = function (fruit) {
  alert("your choice is " + fruit);
};
fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit);
  $ul.appendChild($li);
});
document.body.appendChild($ul);
alertFruit(fruits[1]);
```

- 공통 함수로 쓰고자 콜백 함수를 외부로 꺼내어 alertFruit라는 변수에 담았다.
- alertFruit를 직접 실행할 수 있고, 정상적으로 'banana'에 대한 얼럿이 실행된다.
- 그러나 각 li를 클릭하면 클릭한 대상의 과일 명이 아닌 [object MouseEvent]가 출력된다.

**콜백 함수의 인자에 대한 제어권을 addEventListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문이다.** 이러한 문제는 bind 메서드를 활용하면 해결할 수 있다.

```javascript
// 콜백 함수와 클로저(3)
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul");

var alertFruit = function (fruit) {
  alert("your choice is " + fruit);
};
fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit.bind(null, fruit));
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

다만 이러한 방법은 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점 및 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야 한다.

이런 변경사항이 발생하지 않게끔 하면서 이슈를 해결하기 위해서는 고차함수를 활용할 수 있다. 이는 함수형 프로그래밍에서 자주 쓰이는 방식이다.

```javascript
// 콜백 함수와 클로저(4)
var fruits = ["apple", "banana", "peach"];
var $ul = document.createElement("ul");

var alertFruitBuilder = function (fruit) {
  return function () {
    alert("your choice is " + fruit);
  };
};
fruits.forEach(function (fruit) {
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruitBuilder(fruit));
  $ul.appendChild($li);
});
document.body.appendChild($ul);
```

- alertFruitBuilder는 함수 내부에서 다시 익명함수를 반환하는데, 이 익명함수는 기존의 alertFruit 함수이다.
- alertFruitBuilder 함수를 실행하면서 fruit 값을 인자로 전달한다.
- 이 함수의 실행 결과는 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로써 전달한다.

언젠가 **클릭 이벤트가 발생하면 비로소 이 함수의 실행 컨텍스가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있다. 즉, alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재한다.**

> 이처럼 **콜백 함수 내부에서 외부변수를 참조하기 위한 방법에는 세 가지**가 있다.

- 첫째는 **콜백 함수를 내부함수로 선언해서 외부변수를 직접 참조하는 방법**으로 클로저를 사용한 방법이다.
- 둘째는 **bind 메서드를 활용한 방법**으로, bind 메서드로 값을 직접 넘겨준 덕분에 클로저는 발생하지 않게 된 반면 여러 가지 제약사항이 따른다.
- 마지막으로 **콜백 함수를 고차함수로 바꿔서 클로저를 적극 활용한 방안**이 있다.

### 접근 권한 제어(정보 은닉)

> 정보 은닉은 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈 간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나이다.

흔히 접근 권한에는 public, private, protected의 세 종류가 있는데, public은 외부에서 접근 가능한 것이고, private은 내부에서만 사용하며 외부에 노출되지 않는 것을 의미한다.

자바스크립트는 기본적으로 변수 자체에 이러한 접근 권한을 직접 부여하도록 설계되어 있진 않지만, 함수 차원에서 public한 값과 private한 값을 구분하는 것이 가능하다.

```javascript
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

outer 함수를 종료할 때 inner 함수를 반환함으로써 outer 함수의 지역변수인 a의 값을 외부에서도 읽을 수 있게 되었다.

이처럼 **클로저를 활용하면 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한을 부여할 수 있다. 바로 return을 활용해서 말이다.**

outer 함수는 외부(전역 스코프)로부터 격리된 닫힌 공간이다. 외부에서는 외부 공간에 노출되어 있는 outer라는 변수를 통해 outer 함수를 실행할 수는 있지만, outer 함수 내부에는 어떠한 개입도 할 수 없다. 외부에서는 오직 outer 함수가 return한 정보에만 접근할 수 있다. **return 값이 외부에 정보를 제공하는 유일한 수단**인 것이다.

> 즉, **외부에 제공하고자 하는 정보들을 모아서 return하고, 내부에서만 사용할 정보들은 return하지 않는 것으로 접근 권한 제어가 가능한 것이다.** return한 변수들은 public이, 그렇지 않은 변수들은 private가 되는 것이다.

```javascript
// 간단한 자동차 객체
var car = {
  fuel: Math.ceil(Math.random() * 10 + 10), // 연료(L)
  power: Math.ceil(Math.random() * 3 + 2), // 연비(km/L)
  moved: 0, // 총 이동거리
  run: function () {
    var km = Math.ceil(Math.random() * 6);
    var wasteFuel = km / this.power;
    if (this.fuel < wasteFuel) {
      console.log("이동불가");
      return;
    }
    this.fuel -= wasteFuel;
    this.moved += km;
    console.log(km + "km 이동 (총 " + this.moved + "km)");
  },
};
```

- car 변수에 객체를 직접 할당한다.
- fuel과 power는 무작위로 생성하고, moved라는 프로퍼티에 총 이동거리를 부여했으며, run 메서드를 실행할 때마다 car 객체의 fuel, moved 값이 변한다.

```javascript
car.fuel = 10000;
car.power = 100;
car.moved = 1000;
```

하지만 이 코드는 위와 같은 방법으로 마음껏 연료, 연비, 이동거리를 바꿀 수 있다. 이 값을 바꾸지 못하도록 하기 위해서는 클로저를 활용해야 한다. **즉, 객체가 아닌 함수로 만들고 필요한 것만 retrun 하는 것이다.**

```javascript
// 클로저로 변수를 보호한 자동차 객체(1)
var createCar = function () {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료(L)
  var power = Math.ceil(Math.random() * 3 + 2); // 연비(km / L)
  var moved = 0; // 총 이동거리
  return {
    get moved() {
      return moved;
    },
    run: function () {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel;
      moved += km;
      console.log(km + "km 이동 (총 " + moved + "km). 남은 연료: " + fuel);
    },
  };
};
var car = createCar();
```

- createCar라는 함수를 실행함으로써 객체를 생성한다.
- fuel, power 변수는 private으로 지정해 외부에서는 접근을 제한한다.
- moved 변수는 getter만을 부여해 읽기 전용 속성을 부여한다.

**이제 외부에서는 오직 run 메서드를 실행하는 것과 현재의 moved 값을 확인하는 두 가지 동작만 할 수 있다.** 비록 run 메서드를 다른 내용으로 덮어씌우는 어뷰징은 여전히 가능하지만 앞선 코드보단 훨씬 안전한 코드가 되었다. 이런 어뷰징을 막기 위해서는 객체를 return 하기 전에 미리 변경할 수 없게끔 조치를 취해야 한다.

```javascript
// 클로저로 변수를 보호한 자동차 객체(2)
var createCar = function () {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료(L)
  var power = Math.ceil(Math.random() * 3 + 2); // 연비(km / L)
  var moved = 0; // 총 이동거리
  var publicMembers = {
    get moved() {
      return moved;
    },
    run: function () {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel;
      moved += km;
      console.log(km + "km 이동 (총 " + moved + "km). 남은 연료: " + fuel);
    },
  };
  Object.freeze(publicMembers);
  return publicMembers;
};
var car = createCar();
```

> **클로저를 활용해 접근제한을 제어하는 방법**에는 두 가지가 있다.

- 첫째는 **함수에서 지역변수 및 내부함수 등을 생성**하는 것이다.
- 둘째는 **외부에 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터**(대상이 여럿일 때는 객체 또는 배열, 하나일 때는 함수)**를 return 하는 것**이다.

### 부분 적용 함수

> 부분 적용 함수란 n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수이다.

this를 바인딩해야 하는 점을 제외하면 앞서 살펴본 bind 메서드의 실행 결과가 바로 부분 적용 함수이다.

```javascript
// bind 메서드를 활용한 부분 적용 함수
var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55
```

- addPartial 함수는 인자 5개를 미리 적용하고, 추후 추가적으로 인자들을 전달하면 모든 인자를 모아 원래의 함수가 실행되는 부분 적용 함수이다.

add 함수는 this를 사용하지 않으므로 bind 메서드만으로도 문제 없이 구현되었다. 그러나 **this의 값을 변경할 수밖에 없기 때문에 메서드에서는 사용할 수 없다.** this에 관여하지 않는 별도의 부분 적용 함수가 있다면 범용성 측면에서 더 좋겠다.

```javascript
// 부분 적용 함수 구현(1)
var partial = function () {
  var originalPartialArgs = arguments;
  var func = originalPartialArgs[0];
  if (typeof func !== "function") {
    throw new Error("첫 번째 인자가 함수가 아닙니다.");
  }
  return function () {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
    var restArgs = Array.prototype.slice.call(arguments);
    return func.apply(this, partialArgs.concat(restArgs));
  };
};

var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = partial(add, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55

var dog = {
  name: "강아지",
  greet: partial(function (prefix, suffix) {
    return prefix + this.name + suffix;
  }, "왈왈, "),
};
dog.greet("입니다!"); // 왈왈, 강아지입니다.
```

- 첫 번째 인자에는 원본 함수를, 두 번째 인자 이후부터는 미리 적용할 인자들을 전달한다.
- 반환할 함수에는 다시 나머지 인자들을 받아 이를 한데 모아(concat) 원본 함수를 호출(apply)한다.
- 실행 시점의 this를 그대로 반영함으로써 this에는 아무런 영향을 주지 않는다.

**다만 부분 적용 함수에 넘길 인자를 반드시 앞에서부터 차례로 전달할 수밖에 없다는 점**이 아쉽다. 인자들을 원하는 위치에 미리 넣고 나중에 빈 자리에 인자를 채워넣어 실행하면 더 좋겠다.

```javascript
// 부분 적용 함수 구현(2)
Object.defineProperty(window, "_", {
  value: "EMPTY_SPACE",
  writable: false,
  configurable: false,
  enumerable: false,
});

var partial2 = function () {
  var originalPartialArgs = arguments;
  var func = originalPartialArgs[0];
  if (typeof func !== "function") {
    throw new Error("첫 번째 인자가 함수가 아닙니다.");
  }
  return function () {
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
    var restArgs = Array.prototype.slice.call(arguments);
    for (var i = 0; i < partialArgs.length; i++) {
      if (partialArgs[i] === _) {
        partialArgs[i] = restArgs.shift();
      }
    }
    return func.apply(this, partialArgs.concat(restArgs));
  };
};

var add = function () {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};
var addPartial = partial2(add, 1, 2, _, 4, 5, _, _, 8, 9);
console.log(addPartial(3, 6, 7, 10)); // 55

var dog = {
  name: "강아지",
  greet: partial2(function (prefix, suffix) {
    return prefix + this.name + suffix;
  }, "왈왈, "),
};
dog.greet(" 배고파요!"); // 왈왈, 강아지 배고파요!
```

- '비워놓음'을 표시하기 위해 미리 전역객체에 \_라는 프로퍼티를 준비하면서 삭제 변경 등의 접근에 대한 방어 차원에서 여러 가지 프로퍼티 속성을 설정한다.
- 처음에 넘겨준 인자들 중 \_로 비워놓은 공간마다 나중에 넘어온 인자들이 차례대로 끼워넣어진다.

부분 적용 함수를 만들 때 미리부터 실행할 함수의 인자 개수를 맞춰 빈 공간을 확보하지 않아도 된다. 실행할 함수 내부 로직에만 문제가 없다면 최종 실행 시 인자 개수가 많든 적든 잘 실행된다.

**미리 일부 인자를 넘겨두어 기억하게끔하고 추후 필요한 시점에 기억했던 인자들까지 함께 실행한다는 개념 자체가 클로저의 정의에 정확히 부합한다.**

### 부분 적용 함수 예시 - 디바운스

> **디바운스(devounce)는 짧은 시간 동안 동일한 이벤트가 많이 발생할 경우 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리하는 것**으로, 프런트엔드 성능 최적화에 큰 도움을 주는 기능 중 하나이다.

```javascript
var debounce = function (eventName, func, wait) {
  var timeoutId = null;
  return function (event) {
    var self = this;
    console.log(eventName, "event 발생");
    clearTimeout(timeoutId);
    timeoutId = setTimeout(func.bind(self, event), wait);
  };
};

var moveHandler = function (e) {
  console.log("move event 처리");
};
var wheelHandler = function (e) {
  console.log("wheel event 처리");
};
document.body.addEventListener("mousemove", debounce("move", moveHandler, 500));
document.body.addEventListener(
  "mousewheel",
  debounce("wheel", wheelHandler, 700)
);
```

\*\* 클로저로 처리되는 변수에는 eventName, func, wait, timeoutId가 있다.

- 출력 용도로 지정한 eventName과 실행할 함수(func), 마지막으로 발생한 이벤트인지 여부를 판단하기 위한 대기시간(wait(ms))을 받는다.
- 내부에서는 timeoutId 변수를 생성하고, 클로저로 EventListener에 의해 호출될 함수를 반환한다.
- 반환될 함수 내부에서는 setTimeout을 사용하기 위해 this를 별도의 변수에 담고 무조건 대기큐를 초기화하게 한다.
- 마지막으로 setTimeout으로 wait 시간만큼 지연시킨 다음, 원래의 func를 호출한다.

최초 event가 발생하면 timeout의 대기열에 'wait 시간 뒤에 func를 실행할 것'이라는 내용이 담긴다. 그러나 wait 시간이 경과하기 이전에 다시 동일한 event가 발생하면 앞서 저장했던 대기열을 초기화하고 다시 새로운 대기열을 등록한다.

결국 각 이벤트가 바로 이전 이벤트로부터 wait한 시간 이내에 발생하는 한 마지막에 발생한 이벤트만이 초기화되지 않고 무사히 실행될 것이다.

> ### ES6 Symbol.for
>
> ES5 환경에서의 '비워놓음'은 ES6의 Symbol.for 활용하면 더 좋다. **Symbol.for 메서드는 전역 심볼공간에 인자로 넘어온 문자열이 이미 있으면 해당 값을 참조하고, 선언돼 있지 않으면 새로 만드는 방식**으로, 어디서든 접근 가능하면서 유일무이한 상수를 만들고자 할 때 적합하다.

### 커링 함수

> 커링 함수란 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있는 체인 형태로 구성한 것을 말한다.

부분 적용 함수와 기본적인 맥락은 일치하지만,

- 커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다는 점
- 중간 과정상의 함수를 실행한 결과는 그다음을 인자를 받기 위해 대기만 할 뿐 마지막 안지가 전달되기 전까지는 원본 함수가 실행되지 않는다는 점에서 다르다.

참고로 부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할 때 원본 함수가 무조건 실행된다.

```javascript
// 커링 함수(1)
var curry3 = function (func) {
  return function (a) {
    return function (b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8)); // 10
console.log(getMaxWith10(25)); // 25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8)); // 8
console.log(getMinWith10(25)); // 10
```

부분 적용 함수와 달리 커링 함수는 필요한 상황에 직접 만들어 쓰기 용이하다. 필요한 인자 개수만큼 함수를 만들어 계속 리턴해주다가 마지막에만 조합해서 리턴하면 되기 때문이다.

```javascript
// 커링 함수(2)
var curry5 = function (func) {
  return function (a) {
    return function (b) {
      return function (c) {
        return function (d) {
          return function (e) {
            return func(a, b, c, d, e);
          };
        };
      };
    };
  };
};
var getMax = curry5(Math.max);
console.log(getMax(1)(2)(3)(4)(5));
```

다만 인자가 많아질수록 가독성이 떨어진다는 단점이 있다.

> ### ES6 화살표 함수

```javascript
var curry5 = (func) => (a) => (b) => (c) => (d) => (e) => func(a, b, c, d, e);
```

ES6의 화살표 함수를 활용하면 단 한 줄에 표기할 수 있다. 화살표 함수로 커링 구현하면 화살표 순서에 따라 함수에 값을 차례로 넘겨주면 마지막에 func가 호출될 거라는 흐름을 한 눈에 파악할 수 있다.

각 단계에서 받은 인자들은 모두 마지막 단계에서 참조할 것이므로 GC되지 않고 메모리에 차곡차곡 쌓였다가, 마지막 호출로 실행 컨텍스트가 종료된 후에야 비로소 한꺼번에 GC의 수거 대상이 된다.

### 커링 함수 예시 - 지연실행

> 커링 함수는 지연실행에 유용하다. 지연실행이란 당장 필요한 정보만 받아서 전달하고 또 필요한 정보가 들어오면 전달하는 식으로 하며 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 것이다.

원하는 시점까지 지연시켰다가 실행하는 것이 유용한 상황이거나 프로젝트 내에서 자주 쓰이는 함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우라면 커링을 쓰기에 적합하다.

> ## 📌 정리

- 클로저란 어떤 함수에서 선언한 변수를 참조하는 내부함수를 외부로 전달할 경우, 함수의 실행 컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상이다.
- 내부 함수를 외부로 전달하는 방법에는 함수를 return하는 경우와 콜백으로 전달하는 경우가 있다.
- 클로저는 그 본질이 메모리를 계속 차지하는 개념이므로 더는 사용하지 않게 된 클로저에 대해서는 메모리를 차지하지 않도록 관리해줘야 한다.
