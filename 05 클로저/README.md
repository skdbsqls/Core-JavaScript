## 1. 클로저의 의미 및 원리 이해

클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성이다.

MDN에서는 클로저에 대해 "A closure is the combination of a function and the lexical environment within which that function was decleared."라고 소개 한다.

직역하면, ***"클로저는 함수와 그 함수가 선언될 당시의 lexical environment의 상호관계에 따른 현상"***이라고 할 수 있다.

선언될 당시의 lexical environment은 실행 컨텍스트의 구성 요소 중 하나인 outerEnvironmentReference에 해당한다. LexicalEnvironment의 environmentRecord와 outerEnvironmentReference에 의해 변수의 유효범위인 스코프가 결정되고 스코프 체인이 가능해진다.

어떤 컨텍스트 A에서 선언한 내부함수 B의 실행 컨텍스트가 활성화된 시점에는 B의 outerEnvironmentReference가 참조하는 대상인 A의 LexicalEnvironment에도 접근이 가능하다. **A에서는 B에서 선언한 변수에 접근할 수 없지만 B에서는 A에서 선언한 변수에 접근 가능하다.**

여기서 'combination'의 의미를 파악할 수 있다.

내부함수 B가 A의 LexicalEnvironment를 언제나 사용하는 것은 아니다. 내부함수에서 외부 변수를 참조하지 않는 경우라면 combination이라고 할 수 없다. **내부함수에서 외부 변수를 참조하는 경우에 한해서만 combination, 즉 '선언될 당시의 LexicalEnvironment와의 상호관계'가 의미가 있을 것이다.**

> 이에 따르면 클로저는 ***"어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상"***이라고 할 수 있다.

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

> 이를 바탕으로 정의를 다시 고쳐보면, ***클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상***을 말한다.

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
