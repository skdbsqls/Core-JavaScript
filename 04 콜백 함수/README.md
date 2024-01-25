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
