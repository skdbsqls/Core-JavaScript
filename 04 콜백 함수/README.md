## 1. 콜백 함수란?

> **콜백 함수는 다른 코드의 인자로 넘겨주는 함수**로 콜백 함수를 넘겨받은 코드는 이 콜백 함수를 필요에 따라 적절한 시점에 실행한다.

callback은 '부르다', '호출(실행)하다'는 의미인 call과 '뒤돌아오다', '되돌다'는 의미인 back의 합성어로, '되돌아 호출해달라'는 명령이다.

어떤 함수 X를 호출하면서 '특정 조건일 때 함수 Y를 실행해서 나에게 알려달라'는 요청을 함께 보내는 것이다. 이 요청을 받은 함수 X의 입장에서는 해당 조건이 갖춰졌는지 여부를 스스로 판단하고 Y를 직접 호출한다.

## 2. 제어권

> **콜백 함수는 다른 코드(함수 또는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수**이다. 콜백 함수를 위임받은 코드는 자체적인 내부 로직에 의해 이 콜백 함수를 적절한 시점에 실행할 것이다.

### 호출시점

> **setInterval**

```javascript
var intervalID - scope.setInterval(func, delay[, param1, param2, ...]);
```

scope에는 Window 객체 또는 Worker의 인스턴스가 들어올 수 있다. 매개변수로는 첫 번째로 함수 func을, 두 번째로 ms 단위의 숫자인 delay 값을 반드시 전달해야하며, 나머지(param1, param2, ...)는 func 함수를 실행할 때 매개변수로 전달할 인자인데 선택적이다.
**_func에 념겨준 함수는 매 delay(ms)마다 실행되며, 그 결과 어떠한 값도 리턴하지 않는다. setInterval를 실행하면 반복적으로 실행되는 내용 자체를 특정할 수 있는 고유값 ID 값이 반환된다._** 이를 변수에 담는 이유는 반복 실행되는 중간에 종료(clearInterval)할 수 있게 하기 위해서이다.

```javascript
// 콜백 함수 예제 (1-1) setInterval
var count = 0;
var timer = setInterval(function () {
  console.log(count);
  if (++count > 4) clearInterval(timer);
}, 300);
```

```javascript
// 콜백 함수 예제 (1-2) setInterval
var count = 0;
var cbFunc = function () {
  console.log(count);
  if (++count > 4) clearInterval(timer);
};
var timer = setInterval(cbFunc, 300);

// -- 실행 결과 --
// 0  (0.3초)
// 1  (0.6초)
// 2  (0.9초)
// 3  (1.2초)
// 4  (1.5초)
```

- setInterval을 호출할 때 두 개의 매개변수를 전달했는데, 그중 첫 번째는 익명 함수이고 두 번째는 300이라는 숫자이다.
- timer 변수에는 setInterval의 ID 값이 담긴다.
- setInterval에 전달한 첫 번째 인자인 cbFunc 함수는 0.3초마다 자동으로 실행된다.
- 콜백 함수 내부에서는 count 값을 출력하고, count를 1만큼 증가시킨 다음, 그 값이 4보다 크면 반복 실행을 종료하라고 한다.

이 코드를 실행하면 콘솔창에는 0.3초에 한 번씩 숫자가 0부터 1씩 증가하며 출력되다가 4가 출력된 이후 종료된다.

setInterval이라고 하는 '다른 코드'에 첫 번째 인자로서 cbFunc 함수를 넘겨주자 제어권을 넘겨받은 setInterval이 스스로 판단에 따라 적절한 시점에 이 익명 함수를 실행했다.

> 이처럼 **콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가진다.**

### 인자

> **map**

```javascript
Array.propotype.mao(callback[, thisArg])
callback: function(currentValue, index, array)
```

map 메서드는 첫 번째 인자로 callback 함수를 받고, 생략 가능한 두 번째 인자로 콜백 함수 내부에서 this로 인식할 대상을 특정할 수 있다.
_**map 메서드는 메서드의 대상이 되는 배열의 모든 요소들을 처음부터 끝까지 하나씩 꺼내어 콜백 함수를 반복 호출하고, 콜백 함수의 실행 결과들을 모아 새로운 배열을 만든다.**_
콜백 함수의 첫 번째 인자에는 배열의 요소 중 현재값이, 두 번째 인자에는 현재값의 인덱스가, 세 번째 인자에는 map 메서드의 대상이 되는 배열 자체가 담긴다.

```javascript
// 콜백 함수 예제 (2-1) Array.prototype.map
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
  return currentValue + 5;
});
console.log(newArr);

// -- 실행 결과 --
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

- 배열 [10, 20, 30]에 map 메서드를 호출한다. 이때 첫 번째 매개변수로 익명 함수를 전달한다.
- 배열 [10, 20, 30]의 각 요소를 처음부터 하나씩 꺼내어 콜백 함수를 실행한다.
- 첫 번째에에 대한 콜백 함수는 currentValue에 10이, index에는 인덱스 0이 담긴 채 실행된다.
- 각 값을 출력한 다음, 15를 반환한다.
- 두 번째, 세 번째에 대해서도 같은 방식으로 콜백 함수까지 실행을 하고 마치면, [15, 25, 35]라는 새로운 배열이 만들어져 변수 newArr에 담기고, 이 값이 출력된다.

```javascript
// 콜백 함수 예제 (2-2) Array.prototype.map - 인자의 순서를 임의로 바꾸어 사용한 경우
var newArr2 = [10, 20, 30].map(function (index, currentValue) {
  console.log(index, currentValue);
  return currentValue + 5;
});
console.log(newArr2);

// -- 실행 결과 --
// 10 0
// 20 1
// 30 2
// [5, 6, 7]
```

- [5, 6, 7]이 출력된다.

사용자가 첫 번째 인자의 이름을 'index'로 하건 'currentValue'로 칭하건 관계 없이 컴퓨터는 그저 **순서에 의해서만 각각을 구분하고 인식**한다. 따라서 그냥 순회 중인 배열 중 현재 요소의 값을 'index'에 배정하고, 인덱스 값을 'currentValue'에 부여한다.

map 메서드를 호출해서 원하는 배열을 얻으려면 map 매서드에 정의된 규칙에 따라 함수를 작성해야 한다. **map 메서드에 정의된 규칙에는 콜백 함수의 인자로 넘어올 값들 및 그 순서도 포함되어 있다.**

콜백 함수를 호출하는 주체가 사용자가 아닌 map 메서드이므로 map 메서드가 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지는 전적으로 map 메서드에 달리 것이다.

> **이처럼 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권을 가진다.**

### this

> 콜백 함수도 함수이기 때문에 기본적으로는 this가 전역객체를 참조하지만, **제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조한다.**

```javascript
// 콜백 함수 예제 (2-3) Array.prototype.map - 구현
Array.prototype.map = function (callback, thisArg) {
  var mappedArr = [];
  for (var i = 0; i < this.length; i++) {
    var mappedValue = callback.call(thisArg || window, this[i], i, this);
    mappedArr[i] = mappedValue;
  }
  return mappedArr;
};
```

- this에는 thisArg 값이 있을 경우에는 그 값을, 없을 경우에는 전역객체를 지정한다.
- 첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로 배열의 i번째 요소 값을, 두 번째 인자에는 i 값을, 세 번째 인자에는 배열 자체를 지정해 호출한다.
- 그 결과 변수 mappedValue에 담겨 mappedArr의 i번째 인자에 할당된다.

this에 다른 값이 담기는 이유는 바로 제어권을 넘겨받을 코드에서 call/apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩하기 때문이다.

```javascript
// 콜백 함수 내부에서의 this
setTimeout(function () {
  console.log(this);
}, 300); // (1) Window { ... }

[1, 2, 3, 4, 5].forEach(function (x) {
  console.log(this); // (2) Window { ... }
});

document.body.innerHTML += '<button id="a">클릭</button>';
document.body.querySelector("#a").addEventListener(
  "click",
  function (e) {
    console.log(this, e); // (3) <button id="a">클릭</button>
  } // MouseEvent { isTrusted: true, ... }
);
```

- (1) setTimeout : 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 전역객체를 넘기기 때문에 콜백 함수 내부에서의 this가 전역객체를 가리킨다.
- (2) forEach : 별로의 인자로 this를 넘겨주지 않았기 때문에 전역객체를 가리킨다.
- **(3) addEventListener : 내부에서 콜백 함수를 호출할 때 call 메서드의 첫 번째 인자에 addEventListener 메서드의 this를 그대로 넘기도록 정의되어 있기 때문에 콜백 함수 내부에서의 this가 addEventListener를 호출한 주체인 HTML 엘리멘트를 가리킨다.**

## 3. 콜백 함수는 함수다

> 콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 **함수로서 호출**된다.

```javascript
// 메서드를 콜백 함수로 전달한 경우
var obj = {
  vals: [1, 2, 3],
  logValues: function (v, i) {
    console.log(this, v, i);
  },
};
obj.logValues(1, 2); // (1) { vals: [1, 2, 3], logValues: f } 1 2
[4, 5, 6].forEach(obj.logValues); // (2) Window { ... } 4 0
// Window { ... } 5 1
// Window { ... } 6 2
```

(1) : odj 객체의 logValues는 메서드로 정의되었다. obj.logValues(1, 2)에서 이 메서드로서 호출했다. 따라서 this는 obj를 가리키고, 인자로 넘어온 1, 2가 출력된다.

(2) : logValues 메서드를 forEach 함수의 콜백 함수로서 전달했다. **obj를 this로 하는 메서드를 그대로 전달한 것이 아니라, obj.logValues가 가리키는 함수만 전달한 것이다.** 이 함수는 메서드로서 호출할 때가 아닌 한 obj와의 직접적인 연관이 없어진다.

이 함수는 forEach에 의해 콜백이 함수로서 호출되고, 별도로 this를 지정하는 인자를 지정하지 않았으므로 함수 내부에서의 this는 전역객체를 바라본다.

즉, **어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수일 뿐이다.**

## 4. 콜백 함수 내부의 this에 다른 값 바인딩하기

객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없다. 그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고 싶다면 어떻게 해야 할까?

별도의 인지로 this를 받는 함수의 경우에는 여기에 원하는 값을 넘겨주면 되지만 그렇지 않은 경우에는 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀 수 없다.

> 그래서 **전통적으로 this를 다른 변수에 담아 콜백 함수로 활용할 함수에서는 this 대신 그 변수를 사용하게 하고, 이를 클로저로 만다는 방식이 많이 사용되었다.**

```javascript
// 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법(1) - 전통적인 방식
var obj1 = {
  name: "obj1",
  func: function () {
    var self = this;
    return function () {
      console.log(self.name);
    };
  },
};
var callback = obj1.func();
setTimeout(callback, 1000);
```

- obj1.func 메서드 내부에서 self 변수에 this를 담고, 익명 함수를 선언과 동시에 반환한다.
- obj1.func를 호출하면 엎서 선언한 내부함수가 반환되어 callback 변수에 담긴다.
- callback을 setTimeout 함수에 인자로 전달하면 1초 뒤 callback이 실행되면서 obj1을 출력한다.

```javascript
// 콜백 함수 내부에서 this를 사용하지 않은 경우
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(obj1.name);
  },
};
setTimeout(obj1.func, 1000);
```

이는 앞선 예제에서 this를 사용하지 않았을 때의 결과이다. 훨씬 간결하고 직관적이지만 작성한 함수를 this를 이용해 다양한 상황에서 재활용할 수 없다.

처음부터 바라볼 객체를 명시적으로 obj1로 지정했기 때문에 어떤 방법으로든 다른 객체를 바라보게끔 할 수 없다.

```javascript
// func 함수 재활용
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(obj1.name);
  },
};
var obj2 = {
  name: "obj2",
  func: obj1.func,
};
var callback2 = obj2.func();
setTimeout(callback2, 1500);

var obj3 = { name: "obj3" };
var callback3 = obj1.func.call(obj3);
setTimeout(callback3, 2000);
```

- callback2에는 obj1의 func을 복사한 obj2의 func를 실행한 결과를 담아 이를 콜백으로 사용한다.
- callback3의 경우 obj1의 func를 실행하면서 this를 obj3가 되도록 지정해 이를 콜백으로 사용한다.

이를 실행하면 실행 시점으로부터 1.5초 후에는 'obj2'가, 실행 시점으로부처 2초 후에는 'obj3'가 출력된다.

이는 번거롭긴 하지만 **this를 우회적으로나마 활용함으로써 다양한 상황에서 원하는 객체를 바라보는 콜백 함수를 만들 수 있는 방법이다. 하지만 불편하고 메모리를 낭비하는 측면도 있다.**

```javascript
// 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법(2) - bind 메서드 활용
var obj1 = {
  name: "obj1",
  func: function () {
    console.log(this.name);
  },
};
setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name: "obj2" };
setTimeout(obj1.func.bind(obj2), 1500);
```

이를 보완한 방법이 바로 **ES5에서 등장한 bind 메서드를 이용하는 방법**이다.

## 5. 콜백 지옥과 비동기 제어

> **콜백 지옥은 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드를 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상이다.** 주로 이벤트 처리나 서버 통신과 같이 비동기적인 작업을 수행하기 위해 이런 형태가 자주 등장하곤 한다.

### 동기와 비동기

> **동기적인 코드는 현재 실행 중인 코드가 완료된 후에야 다음 코드를 실행하는 방식**이다.
> 반대로 **비동기적인 코드는 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어간다.**

CPU의 계산에 의해 즉시 처리가 가능한 대부분의 코드는 동기적인 코드이다.
계산식이 복잡해서 CPU가 계산하는데 시간이 많이 필요한 경우라 하더라도 이는 동기적인 코드이다.

반면, **별도의 요청, 실행 대기, 보류 등과 관련된 코드는 비동기적 코드이다.**

- 사용자의 요청에 의해 특정 시간이 경과되기 전까지 어떤 함수의 실행을 보류하는 경우
- 사용자의 직접적인 개입이 있을 때 비로소 어떤 함수를 실행하도록 대기하는 경우
- 웹브라우저 자체가 아닌 별도의 대상에 무언가를 요청하고 그에 대한 응답이 왔을 때 비로소 어떤 함수를 실행하도록 대기하는 경우

```javascript
// 콜백 지옥 예시(1-1)
setTimeout(
  function (name) {
    var coffeeList = name;
    console.log(coffeeList);

    setTimeout(
      function (name) {
        coffeeList += ", " + name;
        console.log(coffeeList);

        setTimeout(
          function (name) {
            coffeeList += ", " + name;
            console.log(coffeeList);

            setTimeout(
              function (name) {
                coffeeList += ", " + name;
                console.log(coffeeList);
              },
              500,
              "카페라떼"
            );
          },
          500,
          "카페모카"
        );
      },
      500,
      "아메리카노"
    );
  },
  500,
  "에스프레소"
);
```

- 0.5초 주기마다 커피 목록을 수집하고 출력한다.
- 각 콜백은 커피 이름을 전달하고 목록에 이름을 추가한다.

목적 달성에는 지장이 없지만 들여쓰기 수준이 과도하게 깊어졌을뿐만 아니라 값이 전달되는 순서가 '아래에서 위로'향하고 있어 어색하게 느껴진다.

이러한 가독성 문제와 어색함을 동시에 해결하는 가장 단순한 방법은 익명의 콜백 함수를 모두 기명함수로 전환하는 것이다.

```javascript
// 콜백 지옥 해결 - 기명함수로 변환
var coffeeList = "";

var addEspresso = function (name) {
  coffeeList = name;
  console.log(coffeeList);
  setTimeout(addAmericano, 500, "아메리카노");
};
var addAmericano = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
  setTimeout(addMocha, 500, "카페모카");
};
var addMocha = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
  setTimeout(addLatte, 500, "카페라떼");
};
var addLatte = function (name) {
  coffeeList += ", " + name;
  console.log(coffeeList);
};

setTimeout(addEspresso, 500, "에스프레소");
```

이 방식은 코드의 가독성을 높일뿐 아니라 함수 선언과 함수 호출만 구분할 수 있다면 위에서부터 아래로 순서대로 읽어내려가는 데 어려움이 없다. 또한 변수를 최상단으로 끌어올림으로써 외부에 노출되게 됐지만 전체를 즉시 실행 함수 등으로 감싸면 간단히 해결할 수 있는 문제이다.

다만, 코드명을 일일이 따라다녀야 하므로 오히려 헷길릴 소지가 있다.

지난 십수 년간 자바스크립트 진영은 비동기적인 일련의 작업을 동기적으로 혹은 동기적인 것처럼 보이게끔 처리해주는 장치를 마련하고자 끊임없이 노력해 왔고, 그 결과 ES6에서는 Promise, Generator 등이 도입됐고, ES2017에서는 async/await가 도입됐다.

### Promise

> Promise 객체는 비동기 작업이 맞이할 미래의 완료 또는 실패와 그 결과 값을 나타낸다.

Promise는 프로미스가 생성된 시점에는 알려지지 않았을 수도 있는 값을 위한 대리자로, 비동기 연산이 종료된 이후에 결과 값과 실패 사유를 처리하기 위한 처리기를 연결할 수 있다. **프로미스를 사용하면 비동기 메서드에서 마치 동기 메서드처럼 값을 반환할 수 있다. 다만 최종 결과를 반환하는 것이 아니고, 미래의 어떤 시점에 결과를 제공하겠다는 '약속'(프로미스)을 반환한다.**

Promise는 다음 중 하나의 상태를 가진다.

- 대기(pending): 이행하지도, 거부하지도 않은 초기 상태
- 이행(fulfilled): 연산이 성공적으로 완료됨
- 거부(rejected): 연산이 실패함

대기 중인 프로미스는 값과 함께 이행할 수도, 어떤 이유(오류)로 인해 거부될 수도 있다. 이행이나 거부될 때, 프로미스의 then 메서드에 의해 대기열(큐)에 추가된 처리기들이 호출된다. 이미 이행했거나 거부된 프로미스에 처리기를 연결해도 호출되므로, 비동기 연산과 처리기 연결 사이에 경합 조건은 없다.

`Promise.prototype.then()` 및 `Promise.prototype.catch()` 메서드의 반환 값은 새로운 프로미스이므로 서로 연결할 수 있다.

```javascript
// 비동기 작업의 동기적 표현 - Promise
new Promise(function (resolve) {
  setTimeout(function () {
    var name = "에스프레소";
    console.log(name);
    resolve(name);
  }, 500);
})
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 아메리카노";
        console.log(name);
        resolve(name);
      }, 500);
    });
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 카페모카";
        console.log(name);
        resolve(name);
      }, 500);
    });
  })
  .then(function (prevName) {
    return new Promise(function (resolve) {
      setTimeout(function () {
        var name = prevName + ", 카페라떼";
        console.log(name);
        resolve(name);
      }, 500);
    });
  });
```

new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만 그 내부에 resolve 또는 reject 함수를 호출하는 구문이 있을 경우 둘 중 하나가 실행되기 전까지는 다음(then) 또는 오류 구문(catch)으로 넘어가지 않는다.

따라서 **비동기 작업이 완료될 때 비로소 resolve 또는 reject를 호출하는 방법으로 비동기 작업의 동기적 표현이 가능하다.**

### Generator

> Generator 객체는 generator function 으로부터 반환되며, 반복 가능한 프로토콜과 반복자 프로토콜을 모두 준수한다.

Generator는 다음과 같은 메서드를 가진다.

- `Generator.prototype.next()` : yield 표현식을 통해 yield된 값을 반환한다.
- `Generator.prototype.return()` : 주어진 값을 반환하고 제너레이터를 종료한다.
- `Generator.prototype.throw()` : 제너레이터에 오류를 발생시킨다. (해당 제너레이터 내에서 오류가 발생한 경우가 아닌 한 제너레이터도 완료)

```javascript
// 비동기 작업의 동기적 표현(3) - Generator
var addCoffee = function (prevName, name) {
  setTimeout(function () {
    coffeeMaker.next(prevName ? prevName + ", " + name : name);
  }, 500);
};
var coffeeGenerator = function* () {
  var espresso = yield addCoffee("", "에스프레소");
  console.log(espresso);
  var americano = yield addCoffee(espresso, "아메리카노");
  console.log(americano);
  var mocha = yield addCoffee(americano, "카페모카");
  console.log(mocha);
  var latte = yield addCoffee(mocha, "카페라떼");
  console.log(latte);
};
var coffeeMaker = coffeeGenerator();
coffeeMaker.next();
```

Generator 함수를 실행하면 Itertator가 반환되는데, Iterator는 next라는 메서드를 가지고 있다. 이 next 메서드를 호출하면 Generator 함수 내부에서 가장 먼저 등장하는 yield에서 함수의 실행을 멈춘다. 이후 다시 next 메서드를 호출하면 앞서 멈췄던 부분부터 시작해서 그다음에 등장하는 yield에서 함수의 실행을 멈춘다.

따라서 **비동기 작업이 완료되는 시점마다 next 메서드를 호출한다면, Generator 함수 내부의 소스가 위에서부터 아래로 순차적으로 진행된다.**

### Async/await

> async function 선언은 AsyncFunction객체를 반환하는 하나의 비동기 함수를 정의한다.

비동기 함수는 이벤트 루프를 통해 비동기적으로 작동하는 함수로, 암시적으로 Promise를 사용하여 결과를 반환한다. 그러나 비동기 함수를 사용하는 코드의 구문과 구조는, 표준 동기 함수를 사용하는 것과 많이 비슷하다.

```javascript
// 비동기 작업의 동기적 표현(4) - Promise + Async/await
var addCoffee = function (name) {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(name);
    }, 500);
  });
};
var coffeeMaker = async function () {
  var coffeeList = "";
  var _addCoffee = async function (name) {
    coffeeList += (coffeeList ? "," : "") + (await addCoffee(name));
  };
  await _addCoffee("에스프레소");
  console.log(coffeeList);
  await _addCoffee("아메리카노");
  console.log(coffeeList);
  await _addCoffee("카페모카");
  console.log(coffeeList);
  await _addCoffee("카페라떼");
  console.log(coffeeList);
};
coffeeMaker();
```

비동기 작업을 수행하고자 하는 함수 앞에 async를 표기하고, **함수 내부에서 실질적인 비동기 작업이 필요한 위치마다 await를 표기하는 것만으로 뒤의 내용을 Promise로 자동 전환하고, 해당 내용이 resolve된 이후에야 다음으로 진행한다.** 즉, Promise의 then과 흡사한 효과를 얻을 수 있다.

> ## 📌 정리

- 콜백 함수는 다른 코드에 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수이다.
- 제어권을 넘겨받은 코드는 다음과 같은 제어권을 가진다.
  - 콜백 함수를 호출하는 시점을 스스로 판단해서 실행한다.
  - 콜백 함수를 호출할 때 인자로 넘겨줄 값들 및 그 순서가 정해져있다.
  - 콜백 함수의 this가 무엇을 바라도록 할지가 정해져 있는 경우도 있다. (정하지 않은 경우에는 전역객체를 바라보며, 사용자 임의로 this를 바꾸고 싶을 경우 bind 메서드를 활용한다.)
- 어떤 함수에 인자로 메서드를 전달하더라도 이는 결국 함수로서 실행된다.
- 비동기 제어를 위해 콜백 함수를 사용하다 보면 콜백 지옥에 빠지기 쉽다.
- 최근의 ECMAScript에는 Promise, Generator, async/await 등 콜백 지옥에서 벗어날 수 있는 훌륭한 방법들이 속속 등장하고 있다.
