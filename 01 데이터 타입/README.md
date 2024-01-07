## 1. 데이터 타입의 종류

> 자바스크립트의 데이터 타입은 **기본형**(원시형)과 **참조형**이 있다.

![](https://velog.velcdn.com/images/skdbsqls/post/da7051e5-9bc3-40d6-8f37-2afbb747273c/image.png)


일반적으로 기본형은 할당이나 연산 시 복제되고 참조형은 참조된다고 알려져 있으나, 엄밀히 말하면 둘 모두 복제를 하긴 한다. 다만 **기본형은 값이 담김 주솟값을 바로 복제하는 반면 참조형은 값이 담긴 주솟값들로 이루어진 묶음을 가리키는 주솟값을 복제한다**는 점이 다르다.

## 2. 데이터 타입에 관한 배경지식

### 메모리와 데이터

- **비트(Bit)**: 0 또는 1만 표현할 수 있는 하나의 메모리 조각을 비트라고 한다. 메모리는 매우 많은 비트들로 구성되어 있는데, 각 비트는 고유한 식별자를 통해 위치를 확인할 수 있다.

- **바이트(Byte)**: 매우 많은 비트를 한 단위로 묶으면 검색 시간을 줄일 수 있고 표현할 수 있는 데이터의 개수도 늘어나지만 동시에 낭비되는 비트가 생기기도 한다. 자주 사용하지 않을 데이터를 표현하기 위해 빈 공간을 남겨놓기보다는 표현 가능한 개수에 어느 정도 제약이 따르더라도 크게 문제가 되지 않을 적정한 공간을 묶는 편이 낫다. 이러한 고민의 결과 바이트라는 단위가 생겼다. 1바이트는 8개의 비트로 구성되어 있다.

> 비트는 고유한 식별자를 지니고, 바이트 역시 시작하는 비트의 식별자로 위치를 파악할 수 있다. **모든 데이터는 바이트 단위의 식별자, 메모리 주솟값을 통해 서로 구분하고 연결할 수 있다.**

### 식별자와 변수

- **변수**: 변할 수 있는 데이터를 말한다.
- **식별자**: 어떤 데이터를 식별하는 데 사용하는 이름, 즉 변수명을 말한다.

## 3. 변수 선언과 데이터 할당

### 변수 선언
```javascript
var a;
```

이를 말로 풀어쓰면 " 변할 수 있는 데이터를 만든다. 이 데이터의 식별자는 a로 한다."가 된다. 
**결국 변수란 변경 가능한 데이터가 담길 수 있는 공간 또는 그릇이라고 할 수 있다.**

### 데이터 할당
```javascript
var a; // 변수 a 선언
a = 'abc'; // 변수 a에 데이터 할당

var a = 'abc'; // 변수 선언과 할당을 한 문장으로 표현
```
이는 데이터 할당 과정이다. a라는 이름을 가진 주소를 검색해서 그곳에 문자열 'abc'를 할당하면 될 것 같지만, 실제로는 해당 위치에 문자열 'abc'를 직접 저장하지는 않는다.

![](https://velog.velcdn.com/images/skdbsqls/post/37e814f2-b01a-4988-8f8d-41109b843646/image.png)

**데이터를 저장하기 위한 별도의 메모리 공간을 다시 확보해서 문자열 'abc'를 저장하고, 그 주소를 변수 영역에 저장하는 식으로 이루어진다.**

### ❓ 왜 변수 영역에 값을 직접 대입하지 않고 굳이 번거롭게 한 단계를 더 거치는 걸까?

- 미리 확보한 공간 내에서만 데이터를 변환할 수 있다면 변환한 데이터를 다시 저장 하기 위해 '확보된 공간을 변환된 데이터 크기에 맞게 늘리는 작업'이 필요하다. 해당 공간이 메모리상의 가장 마지막에 있었다면 뒤쪽으로 늘리기만 하면 되지만, 중간에 있는 데이터를 늘려야 하는 상황이라면 해당 공간보다 뒤에 저장된 데이터들을 전부 뒤로 옮기고, 이동시킨 주소를 각 식별자에 다시 연결하는 작업을 해야한다. 
- 500개의 변수를 생성해서 모든 변수에 숫자 5를 할당할 때, 각 변수 공간마다 매번 숫자 5를 할당하는 대신 5를 별도의 공간에 한 번만 저장하고 해당 주소만 입력하는 것이 효율적이다.


> 이는 데이터 변환을 자유롭게 할 수 있게 함과 동시에 메모리를 더욱 효율적으로 관리하기 위함이다. 효율적으로 데이터의 변환과 중복된 데이터를 처리하기 위해서는 **변수와 데이터를 별도의 공간에 나누어 저장하는 것이 최적이다.**

## 4. 기본형 데이터와 참조형 데이터

### 불변값

> 변수와 상수를 구분 짓는 변경 가능성의 대상은 변수 영역 메모리인 반면 **불변성 여부를 구분할 때의 변경 가능성의 대상은 데이터 영역의 메모리이다.**

**👉🏻 기본형 데이터인 Number, String, Boolean, null, undefined, Symbol은 모두 불변값이다.**

```javascript
var a = 'abc';
a = a + 'def';
```
![](https://velog.velcdn.com/images/skdbsqls/post/3fe0d4e8-6e48-4acd-b7d8-425c20e7f3cd/image.png)

이처럼 변수 a에 문자열 'abc'를 할당했다가 뒤에 'def'을 추가하면 기존의 'abc'가 'abcdef'로 바뀌는 것이 아니라 **새로운 문자열 'abcdef'를 만들어 그 주소를 변수 a에 저장**한다. 또한 기존 @5004 데이터는 자신의 주소를 저장하는 변수가 하나도 없게 되면 가비지 컬렉터의 수거 대상이 된다. 'abc'와 'abcdef'는 완전히 별개의 데이터이며 다른 값으로 변경할 수 없다.

> **변경은 새로 만드는 동작을 통해서만 이루어지며, 이것이 바로 불변값의 성질이다.** 한 번 만들어진 값은 가비지 컬렉팅을 당하지 않는 한 영원히 변하지 않는다.

### 가변값

```javascript
var obj1 = {
  a: 1,
  b: 'bbb',
};
```
![](https://velog.velcdn.com/images/skdbsqls/post/bb98eaab-89f3-402d-b716-c02f743a711c/image.png)

참조형 데이터는 기본형 데이터와 다르게 '객체의 변수(프로퍼티) 영역'이 별도로 존재한다. 객체가 별도로 할애한 영역은 '변수 영역'으로 '데이터 영역'은 기존의 메모리 공간을 그대로 활용하고 있다.
**데이터 영역에 저장된 값은 모두 불변값이나 변수에는 다른 값을 얼마든지 대입할 수 있다. 바로 이 부분 때문에 흔히 참조형 데이터는 불변하지 않다, 즉 가변값이라고 한다.**

```javascript
var obj1 = {
  a: 1,
  b: 'bbb',
};
obj1.a = 2;
```
![](https://velog.velcdn.com/images/skdbsqls/post/70a21671-d333-409b-b11f-5fb844c6448c/image.png)

obj1의 a 프로퍼티에 숫자 2를 재할당 하고자 할 때, 데이터 영역에서 숫자 2를 검색한 결과가 없으므로 빈 공간인 @5005에 2를 저장하고 이 주소를 @7103에 저장한다. 재할당 이후로 obj1이 바라보고 있는 주소는 @5001로 변하지 않는다. 즉 새로운 객체가 만들어진 것이 아니라 기존의 객체 내부의 값만 바뀐 것이다.

### 변수 복사 비교
```javascript
var a = 10;
var b = a;

var obj1 = { c: 10, d: 'ddd' };
var obj2 = obj1;
```
![](https://velog.velcdn.com/images/skdbsqls/post/9ef37e34-734f-415e-b93d-135bc61fd1d9/image.png)

변수를 복사하는 과정은 기본형 데이터와 참조형 데이터 모두 같은 주소를 바라보게 되는 점에서 동일하다. 하지만 데이터 할당 과정에서 차이가 있기 때문에 변수 복수 이후의 동작에는 큰 차이가 발생한다.

- **변수 복사 이후 객체의 프로퍼티를 변경했을 때**

  ```javascript
  let a = 10;
  let b = a;
  b = 15;

  console.log(a); // 10
  console.log(b); // 15

  let obj1 = { c: 10, d: "ddd" };
  let obj2 = obj1;
  obj2.c = 20;

  console.log(obj1); // { c: 20, d: 'ddd' }
  console.log(obj2); // { c: 20, d: 'ddd' }
  ```

  ![](https://velog.velcdn.com/images/skdbsqls/post/f7c3c778-af55-43df-ab65-80b0c28f6ee5/image.png)

  기본형 데이터를 복사한 변수 b의 값을 바꾼 결과 @1002의 값이 달라진 반면, 참조형 데이터를 복사한 변수 obj2의 프로퍼티의 값을 바꾼 결과 @1004의 값은 달라지지 않았다.
  > **즉, 변수 a와 b는 서로 다른 주소를 바라보게 되었으나, 변수 obj1과 obj2는 여전히 같은 객체를 바라보고 있다. 이것이 바로 기본형과 참조형 데이터의 가장 큰 차이점이다.**

- **변수 복사 이후 객체 자체를 변경했을 때**

  ```javascript
  let a = 10;
  let b = a;
  b = 15;

  console.log(a); // 10
  console.log(b); // 15

  let obj1 = { c: 10, d: "ddd" };
  let obj2 = obj1;
  obj2 = { c: 20, d: "ddd" };

  console.log(obj1); // { c: 10, d: 'ddd' }
  console.log(obj2); // { c: 20, d: 'ddd' }
  ```

  ![](https://velog.velcdn.com/images/skdbsqls/post/a22addb2-4c7d-4754-820a-cb79ee4a5690/image.png)

  이번에는 b의 경우와 마찬가지로 obj2에 새로운 객체를 할당함으로써 값을 직접 변경한다면 메모리의 데이터 영역의 새로운 공간에 새 객체가 저장되고 그 주소를 변수 영역의 obj2 위치에 저장할 것이다.
> **  즉, 참조형 데이터가 '가변값'이라고 설명할 때의 '가변'은 참조형 데이터 자체를 변경할 경우가 아니라 그 내부의 프로퍼티를 변경할 때만 성립한다.**
> 
## 5. 불변 객체

### 불변 객체를 만드는 간단한 방법

> 참조형 데이터의 '가변'은 데이터 자체가 아닌 내부 프로퍼티를 변경할 때만 성립한다. 데이터 자체를 변경하고자 하면 기본형 데이터와 마찬가지로 기존 데이터는 변하지 않는다.

그렇다면 어떤 상황에서 불변 객체가 필요할까?

```javascript
// 객체의 가변성에 따른 문제점
var user = {
  name: 'Jaenam',
  gender: 'male',
};

var changeName = function(user, newName) {
  var newUser = user;
  newUser.name = newName;
  return newUser;
};

var user2 = changeName(user, 'Jung');

if (user !== user2) {
  console.log('유저 정보가 변경되었습니다.');
}
console.log(user.name, user2.name); // Jung Jung
console.log(user === user2); // true
```
이는 객체의 가변성으로 인한 문제점을 보여주는 간단한 예시이다. 단순히 새로운 객체를 생성하고 객체의 프로퍼티를 바꾸는 함수를 통해 객체를 복사한다면, 복사한 객체의 프로퍼티를 변경함과 동시에 원복 객체의 프로퍼티도 변경된다.

만약 정보가 바뀐 시점에 알림을 보내야 하거나, 바뀌기 전의 정보와 바뀐 후의 정보의 차이를 가시적으로 보여줘야 하는 경우 이러한 방법을 적절하지 않다. **변경 전과 후에 서로 다른 객체를 바라보게 만들어야만 한다.**

```javascript
// 객체의 가변성에 따른 문제점의 해결 방안
var user = {
  name: 'Jaenam',
  gender: 'male',
};

var changeName = function(user, newName) {
  return {
    name: newName,
    gender: user.gender,
  };
};

var user2 = changeName(user, 'Jung');

if (user !== user2) {
  console.log('유저 정보가 변경되었습니다.'); // 유저 정보가 변경되었습니다.
}
console.log(user.name, user2.name); // Jaenam Jung
console.log(user === user2); // false
```
이는 **새로운 객체를 반환**하는 방식의 함수로 변경하여 객체를 복사하는 방법이다. 이제 원본 객체와 사본 객체는 서로 다른 객체이기 때문에 변경 전과 변경 후를 비교할 수 있다.

다만 미흡한 점은 새로운 객체를 만들면서 **변경할 필요가 없는 기존 객체의 프로퍼티를 하드코딩으로 입력**했다는 것이다. 이 방법은 대상 객체의 정보가 많은수록 하드코딩도 늘어나므로 더 나은 방법이 필요하다.

### 얕은 복사

```javascript
// 기존 정보를 복사해서 새로운 객체를 반환하는 함수(얕은 복사)
var copyObject = function(target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
};
```
```javascript
var copyObject = function(target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
};

var user = {
  name: 'Jaenam',
  gender: 'male',
};

var user2 = copyObject(user);
user2.name = 'Jung';

if (user !== user2) {
  console.log('유저 정보가 변경되었습니다.'); // 유저 정보가 변경되었습니다.
}
console.log(user.name, user2.name); // Jaenam Jung
console.log(user === user2); // false
```
이는 for in 문법을 이용해 대상 객체의 프로퍼티들을 복사하는 방식으로, 이를 통해 간단하게 객체를 복사하고 내용을 수정할 수 있다. 다만 이러한 방법은 '얕은 복사만을 수행한다'는 점에서 아쉽다.

> **얕은 복사는 바로 아래 단계의 값만 복사하는 방식이다**. 이 말은 중첩된 객체에서 참조형 데이터가 저장된 프로퍼티를 복사할 때 그 주솟값만 복사한다는 의미 즉, 해당 프로퍼티에 대한 원본과 사본이 모두 동일한 참조형 데이터의 주소를 가리키게 된다는 것이다. 

```javascript
// 중첩된 객체에 대한 얕은 복사
var copyObject = function(target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
};

var user = {
  name: 'Jaenam',
  urls: {
    portfolio: 'http://github.com/abc',
    blog: 'http://blog.com',
    facebook: 'http://facebook.com/abc',
  },
};
var user2 = copyObject(user);
user2.name = 'Jung';

console.log(user.name === user2.name); // false

user.urls.portfolio = 'http://portfolio.com';
console.log(user.urls.portfolio === user2.urls.portfolio); // true

user2.urls.blog = '';
console.log(user.urls.blog === user2.urls.blog); // true
```
이는 중첩된 객체를 얕은 복사한 예시이다. 객체에 직접 속한 프로퍼티에 대해서는 복사를 통해 완전히 새로운 데이터가 만들어지는 반면, **한 단계 더 들어간 객체가 프로퍼티로 가지는 객체의 내부 프로퍼티들은 기존 데이터를 그대로 참조하고 있다.**

이런 현상이 발생하지 않게 하려면 객체[프로퍼티] 즉, 객체가 프로퍼티로 가지는 객체에 대해서도 불변 객체로 만들 필요가 있다.

### 깊은 복사

> **깊은 복사는 내부의 모든 값들을 하나하나 찾아서 전부 복사하는 방식이다.** 

```javascript
// 중첩된 객체에 대한 깊은 복사
var copyObject = function(target) {
  var result = {};
  for (var prop in target) {
    result[prop] = target[prop];
  }
  return result;
};

var user = {
  name: 'Jaenam',
  urls: {
    portfolio: 'http://github.com/abc',
    blog: 'http://blog.com',
    facebook: 'http://facebook.com/abc',
  },
};

var user2 = copyObject(user);
user2.urls = copyObject(user.urls);

user.urls.portfolio = 'http://portfolio.com';
console.log(user.urls.portfolio === user2.urls.portfolio); // false

user2.urls.blog = '';
console.log(user.urls.blog === user2.urls.blog); // false
```
이처럼 어떤 객체를 복사할 때 객체 내부의 모든 값을 복사해 완전히 새로운 데이터를 만들고자 한다면, 객체의 프로퍼티 중에서 그 값이 기본형 데이터일 경우에는 그대로 복사하면 되지만 참조형 데이터는 다시 그 내부의 프로퍼티들을 복사해야 한다.

```javascript
// 객체의 깊은 복사를 수행하는 범용 함수
var copyObjectDeep = function(target) {
  var result = {};
  if (typeof target === 'object' && target !== null) {
    for (var prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
};
```
```javascript
var copyObjectDeep = function(target) {
  var result = {};
  if (typeof target === 'object' && target !== null) {
    for (var prop in target) {
      result[prop] = copyObjectDeep(target[prop]);
    }
  } else {
    result = target;
  }
  return result;
};

var obj = {
  a: 1,
  b: {
    c: null,
    d: [1, 2],
  },
};
var obj2 = copyObjectDeep(obj);

obj2.a = 3;
obj2.b.c = 4;
obj.b.d[1] = 3;

console.log(obj); // { a: 1. b: { c: null, d: [1, 3] } }
console.log(obj2); // { a: 3. b: { c: 4, d: { 0: 1, 1: 2 } } }
```

> 이 과정을 참조형 데이터가 있을 때마다 재귀적으로 수행해야만 비로소 깊은 복사가 된다.

### 간단한 깊은 복사

간단하게 깊은 복사를 처리할 수 있는 방식으로 JSON을 활용하는 방법도 있다.
```javascript
//JSON을 활용한 간단한 깊은 복사
var copyObjectViaJSON = function(target) {
  return JSON.parse(JSON.stringify(target));
};
var obj = {
  a: 1,
  b: {
    c: null,
    d: [1, 2],
    func1: function() {
      console.log(3);
    },
  },
  func2: function() {
    console.log(4);
  },
};
var obj2 = copyObjectViaJSON(obj);

obj2.a = 3;
obj2.b.c = 4;
obj.b.d[1] = 3;

console.log(obj); // { a: 1. b: { c: null, d: [1, 3], func1: f() }, func2: f() }
console.log(obj2); // { a: 3. b: { c: 4,    d: [1, 2] } }
```
객체를 JSON 문법으로 표현된 문자열로 전환했다가 다시 JSON 객체로 바꾸는 방법이다. 다만 이 방법은 메서드(함수)나 숨겨진 프로퍼티처럼 JSON으로 변경할 수 없는 프로퍼티들은 무시한다.

> httpRequest로 받은 데이터를 저장한 객체를 복사할 때 등 순수한 정보만 다룰 때 활용하기 좋은 방법이다.

## 6. undefined와 null

> 자바스크립트에서 '없음'을 나타내는 값은 두 가지로 undefined와 null이 있다.

### undefined

undefined의 경우 사용자가 명시적으로 지정할 수도 있지만 값이 존재하지 않을 때 자바스크립트 엔진이 자동으로 부여하는 경우도 있다. 자바스크립트 엔진은 사용자가 응당 어떤 값을 지정할 것이라고 예상되는 상황임에도 실제로 그렇게 하지 않을 때 undefined를 반환한다.

- 값을 대입하지 않은 변수, 즉 데이터 영역의 메모리 주소를 지정하지 않은 식별자에 접근할 때
- 객체 내부의 존재하지 않는 프로퍼티에 접근하려고 할 때
- return 문이 없거나 호출되지 않는 함수의 실행 결과

그러나 값을 대입하지 않는 경우 중 배열의 경우에는 조금 특이한 동작을 확인할 수 있다.
```javascript
// undefined와 배열
var arr1 = [];
arr1.length = 3;
console.log(arr1); // [empty x 3]

var arr2 = new Array(3);
console.log(arr2); // [empty x 3]

var arr3 = [undefined, undefined, undefined];
console.log(arr3); // [undefined, undefined, undefined]
```
빈 배열을 만들고 배열의 크기를 정한 경우 각 요소에는 문자 그대로 undefined조차 할당돼 있지 않다. 이는 사용자가 배열을 생성하면서 각 요소에 undefined를 부여한 경우와 다른 결과를 보인다.

```javascript
// 빈 요소와 배열의 순회
var arr1 = [undefined, 1];
var arr2 = [];
arr2[1] = 1;

arr1.forEach(function(v, i) {
  console.log(v, i);
}); // undefined 0 / 1 1
arr2.forEach(function(v, i) {
  console.log(v, i);
}); // 1 1

arr1.map(function(v, i) {
  return v + i;
}); // [NaN, 2]
arr2.map(function(v, i) {
  return v + i;
}); // [empty, 2]

arr1.filter(function(v) {
  return !v;
}); // [undefined]
arr2.filter(function(v) {
  return !v;
}); // []

arr1.reduce(function(p, c, i) {
  return p + c + i;
}, ''); // undefined011
arr2.reduce(function(p, c, i) {
  return p + c + i;
}, ''); // 11
```
직접 undefined를 할당한 경우 일반적으로 알고 있듯이 배열의 모든 요소를 순회해서 결과를 출력하는 반면, 배열을 생성하고 값을 할당하지 않은 경우 각 메서드들이 비어 있는 요소에 대해서는 어떠한 처리도 하지 않고 건너뛰었음을 알 수 있다.

이는 배열에서만 발견할 수 있는 현상은 아닌, 배열도 객체임을 생각한다면 당연한 현상이다.

배열은 무조건 배열의 길이만큼 빈 공간을 확보하고 각 공간에 인덱스를 이름으로 지정할 것으로 생각할 수 있지만, 실제로는 객체와 마찬가지로 특정 인덱스에 값을 지정할 때 비로소 빈 공간을 확보하고 인덱스를 이름으로 지정하고 데이터의 주솟값을 저장하는 동작을 한다. 즉, **값이 지정되지 않은 인덱스는 '아직은 존재하지 않는 프로퍼티'에 지니지 않은 것**이다. 

이를 통해 사용자가 명시적으로 부여한 undefined와 비어있는 요소에 접근하려 할 때 반환되는 undefined를 구분할 수 있다.

> **사용자가 명시적으로 부여한 undefined 즉, 값으로써 어딘가에 할당된 undefined는 실존하는 데이터인 반면, 자바스크립트 엔진이 반환해주는 undefined는 문자 그대로 값이 없음을 나타낸다.**

### undefined와 null

**같은 의미를 가진 null이라는 값이 별도로 있는데 굳이 undefined를 써야할 필요는 없다. '비어있음'을 명시적으로 나타내고 싶을 때는 undefined가 아닌 null을 사용하면 된다.** 이러한 규칙을 따르면 undefined는 오직 '값을 대입하지 않은 변수에 접근하고자 할 때 자바스크립트 엔진이 반환해주는 값'으로서만 존재할 수 있다. 

```javascript
// undefined와 null 비교
var n = null;
console.log(typeof n); // object

console.log(n == undefined); // true
console.log(n == null); // true

console.log(n === undefined); // false
console.log(n === null); // true
```

한편, null은 한 가지 주의할 점이 있는데, 이는 typeof null이 object라는 점이다. 이는 자바스크립트 자체 버그이다. 따라서 변수의 값이 null인지 여부를 판별하지 위해서는 typeof가 아닌 undefined와 비교를 통해 판별할 수 있다.

> ## 📌 정리

- 자바스크립트 데이터 타입에는 기본형과 참조형이 있는데, 기본적으로 기본형은 불변값이고 참조형은 가변값이다.
- 변수는 변경 가능한 데이터가 담길 수 있는 공간이고, 식별자는 그 변수의 이름이다.
- 기본형 데이터의 경우 변수를 선언하면 컴퓨터는 우선 메모리의 빈 공간에 식별자를 저장하고, 그 공간에 자동으로 undefined를 할당한다. 이후 그 변수에 기본형 데이터를 할당하면 별도의 공간에 데이터를 저장하고, 그 공간의 주소를 변수의 값 영역에 할당한다.
- 참조형 데이터의 경우 컴퓨터는 참조형 데이터 내부 프로퍼티들을 위한 영역을 별도로 확보해서 확보된 주소를 변수에 연결하고, 다시 앞서 확보한 변수 영역에 각 프로퍼티의 식별자를 저장하고, 각 데이터를 별도의 공간에 저장해서 그 주소를 식별자들과 매칭시킨다.
- 참조형 데이터를 불변값으로 사용하고자 하면 내부 프로퍼티들을 일일이 복사하는 '깊은 복사'를 활용하면 된다.
- '없음'을 나타내는 값은 undefined와 null이 있는데, undefined는 어떤 변수에 값이 존재하지 않을 경우를 의미하고 null은 사용자가 명시적으로 '없음'을 표현하기 위해 대입한 값이다.
