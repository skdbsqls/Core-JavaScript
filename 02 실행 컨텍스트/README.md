## 1. 실행 컨텍스트란?

> **실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체**로, 자바스크립트의 동적 언어로서의 성격을 가장 잘 파악할 수 있는 개념이다.

### 스택과 큐

![](https://velog.velcdn.com/images/skdbsqls/post/fee33fc1-113d-4bba-9550-e6c2860bdb4a/image.png)

- **스택** : 출입구가 하나뿐인 깊은 우물 같은 데이터 구조로, **비어있는 스택에 순서대로 a, b, c, d를 저장했다면, 꺼낼 때는 반대로 d, c, b, a의 순서대로 꺼낼** 수 밖에 없다.
- **큐** : 양쪽이 모두 열려있는 파이프와 같은 데이터 구조로, 비어있는 큐에 순서대로 a, b, c, d를 저장했다면, 꺼낼 때도 역시 a, b, c, d의 순서로 꺼낼 수 밖에 없다.

실행 컨텍스트는 동일한 환경에 있는 코드들을 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하고, 이를 콜 스택에 쌓아올렸다가, 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드들을 실행하는 식으로 전테 코드의 환경과 순서를 보장한다.

이때 '동일한 환경', 즉 하나의 실행 컨텍스트를 구성할 수 있는 방법에는 전역공간, eval() 함수, 함수 등이 있으나 우리가 흔히 **실행 컨텍스트를 구성하는 방법은 함수를 실행하는 것**뿐이다.

```javascript
// 실행 컨텍스트와 콜 스택
// -------------------------- (1)
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
  }
  inner(); // ------------ (2)
  console.log(a); // 1
}
outer(); // ---------------- (3)
console.log(a); // 1
```

![](https://velog.velcdn.com/images/skdbsqls/post/2fb55ad3-4d55-4c7e-b252-6df0f8a64d52/image.png)

1. 처음 자바스크립트를 실행하는 순간 (1)전역 컨텍스트가 콜 스택에 담긴다.
   - 전역 컨텍스트란? 최상단의 공간은 코드 내부에서 별도의 실행 명령이 없어도 브라우저에서 자동으로 실행하므로 자바스크립트 파일이 열리는 순간 전역 컨텍스트가 활성화된다고 이해하면 된다.
2. (3)에서 outer 함수를 호출하면 자바스크립트 엔진은 outer에 대한 환경 정보를 수집해서 outer 실행 컨텍스트를 생성한 후 콜 스텍에 담는다.
3. 전역 컨텍스트와 관련된 코드의 실행을 일시중단하고 대신 outer 실행 컨텍스트와 관련된 코드, 즉 outer 함수 내부의 코드들을 순차로 실행한다.
4. (2)에서 inner 함수의 실행 컨텍스트를 생성한 후 콜 스택의 가장 위에 담기면, outer 컨텍스트와 관련된 코드의 실행을 중단하고 inner 함수 내부의 코드를 순서대로 진행한다.
5. inner 함수 내부에서 a 변수에 값 3을 할당하고 나면 inner 함수의 실행이 종료되면서 inner 실행 컨텍스트가 콜 스택에서 제거된다.
6. a 변수의 값을 출력하고 나면 outer 함수의 실행이 종료되어 outer 실행 컨텍스트가 콜 스택에서 제거된다.
7. 전역 컨텍스트만 남은 콜 스택은 실행을 중단했던 (3)의 다음 줄부터 이어서 실행하고 a 변수의 값을 출력하고 나면 전역 컨텍스트도 제거되어 콜 스택은 아무것도 남지 않은 상태로 종료된다.

**활성화될 때 자바스크립트 엔진은 해당 컨텍스트에 관련된 코드들을 실행하는 데 필요한 환경 정보들을 수집해서 실행 컨텍스트 객체에 저장**하는데, 이 객체는 자바스크립트 엔진이 활용할 목적으로 생성할 뿐 **개발자가 코드를 통해 확인할 수 없다.**

![](https://velog.velcdn.com/images/skdbsqls/post/7c6ca035-b9fa-49b2-bcff-236b18fa9487/image.png)

활성화된 실행 컨텍스트의 수집 정보는 다음과 같다.

- VariableEnvironment: 현재 컨텍스트 내의 식별자들에 대한 정보 + 외부 환경 정보. 선언 시점의 LexicalEnvironment의 스냅샷으로 변경 사항은 반영되지 않음
- LexicalEnvironment: 처음 VariableEnvironment와 같지만 변경 사항이 실시간으로 반영됨.
  -ThisBinding: this 식별자가 봐라봐야 할 대상 객체

VariableEnvironment와 LexicalEnvironment의 내부는 **envrionmentRecord와 outer-EnvironmentReference로 구성**되어 있다. 다만, **VariableEnvironment은 최초 실행 시의 스냅샷을 유지**한다는 점에서 LexicalEnvironment와 다르다.

## 2. envrionmentRecord와 호이스팅

> **envrionmentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장**된다. 컨텍스트를 구성하는 함수에 지정된 매개변수 식별자, 선언한 함수가 있을 경우 그 함수 자체, var로 선언된 변수의 식별자 등이 식별자에 해당한다. **컨텍스트 내부 전체를 처음부터 끝까지 쭉 훑어나가며 순서대로 수집**한다.

변수 정보를 수집하는 과정을 모두 마쳤더라도 아직 실행 컨텍스트가 관여할 코드들은 실행되지 전의 상태이다. 코드가 실행되기 전임에도 불구하고 자바스크립트 엔진은 해당 환경에 속한 코드의 변수명들을 모두 알고 있게 되는 셈이다.

> 여기서 호이스팅이라는 개념이 등장하는데, **호이스팅이란 변수 정보를 수집하는 과정을 더욱 이해하기 쉬운 방법으로 대체한 가상 개념**이다. 즉, 자바스크립트 엔진이 실제로 끌어올리지는 않지만 편의상 끌어올린 것으로 간주하자는 것이다.

### 호이스팅 규칙

**1) 매개변수와 변수에 대한 호이스팅**

```javascript
// 매개변수와 변수에 대한 호이스팅(1) - 원본 코드
function a(x) {
  // 수집 대상 1(매개변수)
  console.log(x); // (1)
  var x; // 수집 대상 2(변수 선언)
  console.log(x); // (2)
  var x = 2; // 수집 대상 3(변수 선언)
  console.log(x); // (3)
}
a(1);
```

```javascript
// 매개변수와 변수에 대한 호이스팅(2) - 매개변수를 변수 선언/할당과 같다고 간주해거 변환한 상태
function a() {
  var x = 1; // 수집 대상 1(매개변수 선언)
  console.log(x); // (1)
  var x; // 수집 대상 2(변수 선언)
  console.log(x); // (2)
  var x = 2; // 수집 대상 3(변수 선언)
  console.log(x); // (3)
}
a();
```

호이스팅이 되지 않았을 때 (1), (2), (3)에서 어떤 값들이 출력될지 예상해보면, (1)에서는 함수 호출 시 전달한 1이, (2)에서는 선언된 변수 x에 할당한 값이 없으므로 undefined가, (3)에서는 2가 출력될 것으로 예상할 수 있다.

이 상태에서 변수 정보를 수집하는 과정, 즉 호이스팅을 처리해보자.

```javascript
// 매개변수와 변수에 대한 호이스팅(3) - 호이스팅을 마친 상태
function a() {
  var x; // 수집 대상 1의 변수 선언 부분
  var x; // 수집 대상 2의 변수 선언 부분
  var x; // 수집 대상 3의 변수 선언 부분

  x = 1; // 수집 대상 1의 할당 부분
  console.log(x); // (1)
  console.log(x); // (2)
  x = 2; // 수집 대상 3의 할당 부분
  console.log(x); // (3)
}
a(1);
```

**envrionmentRecord는 현재 실행될 컨텍스트의 대상 코드 내에 어떤 식별자들이 있는지에만 관심 있고, 각 식별자에 어떤 값이 할당될 것인지는 관심이 없다.** 따라서 변수를 호이스팅할 때 변수명만 끌어올리고 할당 과정은 원래 자리에 그대로 남겨준다. 매개변수의 경우도 마찬가지다.

호이스팅을 마친 후 코드를 실행하면 예상[_(1)1, (2)undefined, (3)2_]과 다르게, **실제로는 (1)과 (2)에서는 1이, (3)에서는 2가 출력된다.**

**2) 함수 선언의 호이스팅**

```javascript
// 함수 선언의 호이스팅(1) - 원본 코드
function a() {
  console.log(b); // (1)
  var b = "bbb"; // 수집 대상 1(변수 선언)
  console.log(b); // (2)
  function b() {} // 수집 대상 2(함수 선언)
  console.log(b); // (3)
}
a();
```

마찬가지로 출력 결과를 미리 예상해보면, (1)에는 b의 값이 없으니 에러가 나거나 undefined가, (2)는 'bbb'가, (3)은 b 함수가 출력될 것으로 예상할 수 있다.

이제 호이스팅 처리를 해보자.

```javascript
// 함수 선언의 호이스팅(2) - 호이스팅을 마친 상태
function a() {
  var b; // 수집 대상 1. 변수는 선언부만 끌어올립니다.
  function b() {} // 수집 대상 2. 함수 선언은 전체를 끌어올립니다.

  console.log(b); // (1)
  b = "bbb"; // 변수의 할당부는 원래 자리에 남겨둡니다.
  console.log(b); // (2)
  console.log(b); // (3)
}
a();
```

```javascript
// 함수 선언의 호이스팅(3) - 함수 선언문을 함수 표현식으로 바꾼 코드
function a() {
  var b;
  var b = function b() {}; // ← 바뀐 부분

  console.log(b); // (1)
  b = "bbb";
  console.log(b); // (2)
  console.log(b); // (3)
}
a();
```

a 함수를 실행하는 순간 a 함수의 실행 컨텍스트가 생성된다. 이때 변수명과 함수 선언의 정보를 수집한다. **다만 변수는 선언부와 할당부를 나누어 선언부만 끌어올리는 반면 함수 선언은 함수 전체를 끌어올린다.**

호이스팅을 마친 후 코드를 실행하면 예상[_(1)에러 또는 undefined, (2)'bbb', (3)b 함수_]과 다르게, **실제로는 (1)b 함수, (2)'bbb', (3)'bbb'가 출력된다.**

### 함수 선언문과 함수 표현식

함수 선언문과 함수 표현식은 모두 함수를 새롭게 정의할 때 쓰이는 방식이다.

```javascript
// 함수를 정의하는 세 가지 방식
function a() {
  /* ... */
} // 함수 선언문. 함수명 a가 곧 변수명.
a(); // 실행 OK.

var b = function () {
  /* ... */
}; // (익명) 함수 표현식. 변수명 b가 곧 함수명.
b(); // 실행 OK.

var c = function d() {
  /* ... */
}; // 기명 함수 표현식. 변수명은 c, 함수명은 d.
c(); // 실행 OK.
d(); // 에러!
```

> - 함수 선언문 : function 정의부만 존재하고 별도의 할당 명령이 없다. 반드시 함수명이 정의돼 있어야 한다.

- 함수 표현식 : 정의한 fuction을 별도의 변수에 할당한다. 함수명은 없어도 된다.

함수명을 정의한 함수 표현식을 '기명 함수 표현식', 정의하지 않은 것을 '익명 함수 표현식'이라고 부르기도 하는데, **일반적으로 함수 표현식은 익명 함수 표현식을 말한다.**

> _**참고로 화살표 함수 표현식은 함수 표현식에 대한 간결한 대안이다. 하지만 약간의 의미적 차이와 의도적인 사용상의 제한을 가지고 있다.**_

- 화살표 함수에는 자체 바인딩이 this에 없으며, 인수 또는 super로 사용해야 하며, 메서드로 사용하면 안 된다.
- 화살표 함수는 생성자로 사용할 수 없다. new로 호출하면 TypeError가 반환된다. new.target 키워드에 대한 액세스 권한도 없다.
- 화살표 함수는 함수 내부에서 yield를 사용할 수 없으며 제너레이터 함수로 생성할 수 없다.

```javascript
// 함수 선언문과 함수 표현식(1) - 원본 코드
console.log(sum(1, 2));
console.log(multiply(3, 4));

function sum(a, b) {
  // 함수 선언문 sum
  return a + b;
}

var multiply = function (a, b) {
  // 함수 표현식 multiply
  return a * b;
};
```

```javascript
// 함수 선언문과 함수 표현식(2) - 호이스팅을 마친 상태
var sum = function sum(a, b) {
  // 함수 선언문은 전체를 호이스팅합니다.
  return a + b;
};
var multiply; // 변수는 선언부만 끌어올립니다.
console.log(sum(1, 2)); // (1) 3
console.log(multiply(3, 4)); // (2) multiply is not a fuction

multiply = function (a, b) {
  // 변수의 할당부는 원래 자리에 남겨둡니다.
  return a * b;
};
```

코드를 호이스팅한 후 실행하면 (1)에서는 sum 함수가 정상적으로 실행되어 3이 출력되는 반면, (2)에서는 multiply에 값이 할당돼 있지 않기 때문에 'multiply is not a fuction'이라는 에러 메시지가 출력된다.

여기서 주목해야 할 점은 **함수 선언문은 전체를 호이스팅한 반면 함수 표현식은 변수 선언부만 호이스팅했다는 것**이다. 여기서 함수 선언문과 함수 표현식의 극단적인 차이가 발생한다.

```javascript
// 함수 선언문의 위험성
console.log(sum(3, 4));

function sum(x, y) {
  return x + y;
}

var a = sum(1, 2);

function sum(x, y) {
  return x + " + " + y + " = " + (x + y);
}

var c = sum(1, 2);
console.log(c);
```

전역 컨텍스트가 활성화될 때 전역공간에 선언된 함수들이 모두 가장 위로 끌어올려진다. **동일한 변수명에 서로 다른 값을 할당할 경우 나중에 할당한 값이 먼저 할당한 값을 덮어씌운다. 따라서 코드를 실행하는 중에 실제로 호출되는 함수는 오직 마지막에 할당한 함수, 즉 맨 마지막에 선언된 함수뿐이다.**

이처럼 **두 개의 sum 함수가 모두 함수 선언문으로 정의된 경우**, 처음에 선언된 sum 함수는 의도했던 것과 다른 값을 반환할 것이다. 그럼에도 처음에 선언된 sum 함수는 아무런 에러를 내지 않을 것이다. 따라서 **디버깅하기 어렵다.**

```javascript
// 상대적으로 함수 표현식이 안전하다.
console.log(sum(3, 4)); // Uncaught Type Error: sum is not a function

var sum = function (x, y) {
  return x + y;
};

var a = sum(1, 2);

var sum = function (x, y) {
  return x + " + " + y + " = " + (x + y);
};

var c = sum(1, 2);
console.log(c);
```

그렇다면 **두 개의 sum 함수가 모두 함수 표현식으로 정의**된다면?

두 개의 sum 함수는 각자의 의도대로 잘 동작할 것이고, 처음 선언한 100번째 줄 보다 이전에 sum 함수를 호출하는 경우 그 줄에서 바로 에러가 검출되므로 **더욱 손쉽게 디버깅 할 수 있다.**

## 스코프, 스코프 체인, outerEnvironmentReference

> **스코프란 식별자에 대한 유효범위이고, 스코프 체인이란 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것**이다. 이를 가능케 하는 것이 바로 LexicalEnvironment의 두 번째 수집 자료인 outerEnvironmentReference이다.

### 스코프

**ES5까지의 자바스크립트는** 특이하게도 전역공간을 제외하면 **오직 함수에 의해서만 스코프가 생성**되었다. 하지만 **ES6부터 블록에 의해서도 스코프 경계가 발생**하게 되었다. 다만, 이러한 블록은 var로 선언한 변수에 대해서는 작용하지 않고 오직 새로 생긴 **let, const, strict mode에서의 함수 선언 등에서만** 범위로서의 역할을 수행한다.

ES6에서는 이를 구분하기 위해 함수 스코프, 블록 스코프라는 용어를 사용한다.

- **함수 스코프**
  함수 스코프는 함수에서 선언한 변수는 해당 함수 내에서만 접근 가능하다. 한편, 변수가 함수 내부에 선언된 것이 아니면 이 변수의 스코프는 전역 스코프이므로 어디에서든 접근이 가능하다.

  ```javascript
  function abc() {
    var aa = "12"; // 함수 내부에서 선언
  }
  console.log(aa); // Uncaught ReferenceError: aa is not defined

  if (true) {
    var abc = "123"; // 함수 내부에서 선언 X
  }
  console.log(abc); // 123
  ```

- **블록 스코프**
  블록 스코프는 블록(`{ }`) 내부에서 선언된 변수는 해당 블록에서만 접근 가능하다.
  ```javascript
  // var로 선언 : 함수 스코프
  function loop() {
    for (var i = 0; i < 5; i++) {
      console.log(i);
    }
    console.log("final", i);
  }
  loop();
  /*
      0
      1
      2
      3
      4
      final 5
  */
  // let으로 선언 : 블록 스코프
  function loop() {
    for (let i = 0; i < 5; i++) {
      console.log(i);
    }
    console.log("final", i);
  }
  loop(); /* ReferenceError: i is not defined */
  ```

### 스코프 체인

outerEnvironmentReference는 현재 호출된 함수가 **선언될 당시**의 LexicalEnvironment를 참조한다. '선언하다'라는 행위가 실제로 일어날 수 있는 시점이란 콜 스택 상에서 **어떤 실행 컨텍스트가 활성화된 상태일 때**뿐이다. 어떤 함수를 선언하는 행위 자체도 하나의 코드에 지니지 않으며, **모든 코드는 실행 컨텍스트가 활성화 상태일 때 실행**되기 때문이다.

예를 들어, A 함수 내부에 B 함수를 선언하고 다시 B 함수 내부에 C 함수를 선언한 경우,
함수 C의 outerEnvironmentReference는 함수 B의 LexicalEnvironment를 참조하고, 함수 B의 outerEnvironmentReference는 다시 함수 B가 선언되던 때(A)의 LexicalEnvironment를 참조한다.

이처럼 outerEnvironmentReference는 연결리스트 형태를 띈다. 이런 구조적 특성 덕분에 **여러 스코프에서 동일한 식별자를 선언한 경우에는 무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능**하게 된다.

```javascript
// 스코프 체인
var a = 1;
var outer = function () {
  var inner = function () {
    console.log(a);
    var a = 3;
  };
  inner();
  console.log(a);
};
outer();
console.log(a);
```

![](https://velog.velcdn.com/images/skdbsqls/post/dc9630f2-23f8-4826-8d9f-be05967d3a41/image.png)

전역 공간에서는 전역 스코프에서 생성된 변수에만 접근할 수 있다. outer 함수 내부에서는 outer 및 전역 스코프에서 생성된 변수에 접근할 수 있지만, inner 내부에서 생성된 변수에는 접근하지 못 한다. inner 함수 내부에서는 inner, outer, 전역 스코프 모두에 접근할 수 있다.

> 다만, 스코프 체인 상에 있는 변수라고 해서 무조건 접근 가능한 것은 아니다. inner 함수 내부에서 a 변수를 선언했기 때문에 전역 공간에서 선언한 동일한 이름의 a 변수에는 접근할 수 없는 셈이다. 이는 **변수 은닉화**라고 한다.

### 전역변수와 지역변수

**전역 공간에서 선언한 변수는 전역변수이고, 함수 내부에서 선언한 변수는 무조건 지역변수이다.**
코드의 안전성을 위해서는 가급적 전역변수 사용을 최소화하고자 노력하는 것이 좋다.

## 4. this

실행 컨텍스트의 thisBinding에는 this로 지정된 객체가 저장된다. 실행 컨텍스트 활성화 당시에 this가 지정되지 않은 경우 this에는 전역 객체가 저장된다. 그밖에는 함수를 호출하는 방법에 따라 this에 저장되는 대상이 다르다.

> ## 📌 정리

- 실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체로, 전역 공간에서 자동으로 생성되는 전역 컨텍스트와 eval 및 함수 실행에 의한 컨텍스트 등이 있다.
- 실행 컨텍스트 객체는 활성화되는 시점에 VariableEnvironment, LexicalEnvironment, Thisbinding의 세 가지 정보를 수집한다.
- 실행 컨텍스트를 생성할 때는 VariableEnvironment와 LexicalEnvironment는 동일한 내용으로 구성되지만 LexicalEnvironment는 함수 실행 도중에 변경되는 사항이 즉시 반영되는 반면 VariableEnvironment는 초기 상태를 유지한다.
- VariableEnvironment와 LexicalEnvironment는 매개변수명, 변수의 식별자, 선언한 함수의 함수명 등을 수집하는 envrionmentRecord와 바로 직전 컨텍스트의 LexicalEnvironment 정보를 참조하는 outerEnvironmentReference로 구성돼 있다.
- 호이스팅은 코드 해석을 좀 더 수월하게 하기 위해 envrionmentRecord의 수집 과정을 추상화한 개념으로, 실행 컨텍스트가 관여하는 코드 집단의 최상단으로 이들을 '끌어올린다'고 해석하는 것이다.
- 변수 선언과 값 할당이 동시에 이뤄진 문장은 '선언부'만을 호이스팅하고, 할당 과정은 원래 자리에 남아있게 되는데, 여기서 함수 선언문과 함수 표현식의 차이가 발생한다.
- 스코프는 변수의 유효범위를 말한다.
- outerEnvironmentReference는 해당 함수가 선언된 위치의 LexicalEnvironment를 참조한다.
- 코드 상에서 어떤 변수에 접근하려고 하면 현재 컨텍스트의 LexicalEnvironment를 탐색해서 발견되면 그 값을 반환하고, 발견하지 못할 경우 다시 outerEnvironmentReference에 담긴 LexicalEnvironment를 탐색하는 과정을 거친다.
- 전역 컨텍스트의 LexicalEnvironment에 담긴 변수를 전역변수, 그 밖의 함수에 의해 생성된 실행 컨텍스트의 변수들은 모두 지역 변수이다.
- this에는 실행 컨텍스트를 활성화하는 당시에 지정된 this가 저장되는데, 지정되지 않은 경우에는 전역 객체가 저장된다.
