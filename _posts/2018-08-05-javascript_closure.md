---
published: true
layout: single
title: "[JavaScript] Closure (클로저)"
category: post
tags:
comments: true
---


# Closure (클로저)

 클로저는 자바스크립트의 여러 개념 중 어려운 축에 속하는 개념입니다. 또한 클로저 개념이 없는 언어도 존재하기 때문에 자바스크립트의 특징 중 하나라고도 볼 수 있습니다. 때문에 자바스크립트를 '잘' 다룰줄 알아야 하는 개발자들은 필수적으로 이해하고 있어야 하는 개념이라고 생각합니다.

 개인적으로 자바스크립트 실행 컨택스트에 관한 개념이 없으신 분들은 이 부분에 대한 개념을 먼저 익히신 다음 클로저에 대해서 배우신다면 굉장히 쉽게 클로저 개념을 이해하실 수 있을 것이라고 생각합니다.  

<br />

> [실행 컨택스트](https://poiemaweb.com/js-execution-context)란?



<br />

```
클로저 : 외부함수가 더 이상 실행되지 않을 때에도 내부함수가 외부함수의 프로퍼티에 접근이 가능한 속성
```

역시 예제를 통해서 이해하는게 좋을 것 같습니다.

<br />

### 예제_1 (기본 예제)

------

```javascript
1	let outter = () => {
2	  let foo = 'beautiful girls';
3	  return () => {
4	    console.log(foo);
5	  };
6	};

7	let inner = outter();
8	inner();

/* 출력 결과 : beautiful girls */
```

 자바스크립트에서는 함수 안에 또 다른 함수를 정의하는 것이 가능합니다. 자바를 배우신 분이라면 자바의 nested class와 비슷한 개념이라고 생각하셔도 됩니다.

 코드를 보시면 `outter` 함수는 이미 line 7 에서 생을 마감했음에도 불구하고 출력 결과를 통해서 `inner`가 `outter`의 변수인 `foo`를 불러오는 모습을 보실 수 있습니다. 이러한 속성을 자바스크립트에서 **Closure (클로저)**라고 부릅니다.  

 이러한 일이 가능한 이유는 `outter`가 종료되면서 실행 컨택스트 스택에서 사라지더라도 `outter`의 Activation Object (활성 객체)는 사라지지 않기 때문에 내부 함수의 scope chain을 통해서 외부 함수의 AO 프로퍼티에 접근이 가능한 것입니다. (이 부분은 실행 컨택스트 개념을 익히지 않은 분들은 굳이 이해하지 않으셔도 됩니다.)

<br />

### 예제_2 (클로저를 이용한 private 속성 흉내내기)

------

```javascript
1 	var makeCounter = function() {
2 	  var privateCounter = 0;
3	  function changeBy(val) {
4	    privateCounter += val;
5	  }
6	  return {
7	    increment () {
8	      changeBy(1);
9	    },
10	    decrement () {
11	      changeBy(-1);
12	    },
13	    value () {
14	      return privateCounter;
15	    }
16	  }
17	};
18
19	var counter1 = makeCounter();
20	var counter2 = makeCounter();
21	console.log(counter1.value()); /* 0 */
22	counter1.increment();
23	counter1.increment();
24	console.log(counter1.value()); /* 2 */
25	counter1.decrement();
26	console.log(counter1.value()); /* 1 */
27	console.log(counter2.value()); /* 0 */
```

 위 예제에서 `privateCounter` 변수에 접근할 수 있는 방법은 `makeCounter`의 반환 객체에 존재하는 `increment`, `decrement` 메소드를 이용하는 방법 외에는 없습니다.  또한 `counter1, counter2`는 독립적으로 동작합니다. 즉, 하나의 클로저에서 변수 값을 변경해도 다른 클로저의 값에는 영향을 주지 않습니다.

 여기에서도 마찬가지로 내부 함수 (`increment`, `decrement`)에서 외부 프로퍼티 (`privateCounter` 및 `changeBy`)에 접근하는 클로저 개념이 사용되었음을 알 수 있습니다.

<br />

### 예제_3 (클로저 사용 시 자주하는 실수 : 반복문)

------

```javascript
1	var arr = [];
2	for (var i = 0; i < 5; i++) {
3	  arr[i] = function () {
4	    return i;
5	  };
6	}
7
8	for (var index in arr) console.log(arr[index]());

/*
출력 결과 :
5
5
5
5
5
*/
```

 0 1 2 3 4의 결과를 예상하셨던 분들은 출력 결과를 보면 의아하실 겁니다. 이러한 결과를 보이는 이유는 우선, 함수 `arr[0] ~ arr[4]`의 실행 컨택스트가 동일한 외부 환경(전역 스코프)을 갖기 때문입니다 (for문의 `var i`는 전역 변수로 선언됩니다). 전역 실행 컨택스트의 `i` 값은 반복문이 끝날 때까지 증가하는데, 이는 모든 외부 요소가 함께 공유하고 있는 값이므로 실행 시점(line 8)에서 i = 5가 되어버리고 그 값이 출력되는 것이죠.

<br />

```javascript
var arr = [];
var i = 0;

arr[0] = function() {return i;}
i++;
arr[1] = function() {return i;}
i++;
arr[2] = function() {return i;}
i++;
arr[3] = function() {return i;}
i++;
arr[4] = function() {return i;}
i++;

for (var index in arr) console.log(arr[index]());

/*
출력 결과 :
5
5
5
5
5
*/
```

 이해를 돕기 위해서 반복문을 풀어서 쓴 예제입니다. 물론 출력 결과는 같습니다.

<br />

```javascript
1	var arr = [];
2	for (var i = 0; i < 5; i++) {
3	  arr[i] = (function (id) {
4	    return function() {
5	      return id;
6	    }
7	  })(i);
8	}
9
10	for (var index in arr) console.log(arr[index]());

/*
출력 결과 :
0
1
2
3
4
*/
```

 위 예제는 문제점을 클로저를 활용하여 해결한 코드입니다. `arr[0] ~ arr[1]`의 실행 컨택스트 각각이 자신만의 (서로다른) 외부 환경을 가지도록 즉시실행함수(IIFE)를 이용하여 id 값을 매 루프마다 고정해주었습니다. 이로써 내부 익명 함수의 외부 id 프로퍼티는 0, 1, 2, 3, 4로 각각 달라졌고, id 값은 다른 어느 곳에서도 변경할 수 없게 되었습니다.

<br />

```javascript
1	var arr = [];
2	for (let i = 0; i < 5; i++) {
3	  arr[i] = function() {
4	    return i;
5	  }
6	}
7
8	for (var index in arr) console.log(arr[index]());

/*
출력 결과 :
0
1
2
3
4
*/
```

 사실 이 문제는 ES6로 들어오면서 let 키워드를 쓰면 단번에 해결됩니다. var가 함수 스코프를 가지는 반면 let은 일반 다른 언어들과 마찬가지로 블록 스코프를 가지기 때문입니다.
