---
published: true
layout: single
title: "[Vue.js] computed, watcher"
category: post
tags:
comments: true
---

- ### Computed

  *computed* 프로퍼티는 기본적으로 *methods* 프로퍼티와 비슷해보입니다. 하지만 *computed* 프로퍼티의 메소드는 메소드내에 선언된 변수(*data*)에 의존적입니다.

  <br />

  **index.html**

  ```html
  <!DOCTYPE html>
  <html lang="ko" dir="ltr">
    <head>
      <meta charset="utf-8">
      <title>VueJS</title>
      <script src="vue.js"></script>
    </head>
    <body>
      <div id="app">
        <button @click="counter++">increase</button>
        <button @click="counter--">decrease</button>
        <button @click="secondCounter++">Increase Second</button>
        <p>Counter: {{counter}} | {{ secondCounter}}</p>
        <p>Result: {{ result() }} | {{ output }}</p>
      </div>
    </body>
  </html>

  <script src="app.js">

  </script>
  ```

  <br />

  **app.js**

  ```javascript
  new Vue({
    el: "#app",
    data: {
      counter: 0,
      secondCounter: 0
    },
    computed: {
      output() {
        console.log('Computed!')
        return this.counter > 5 ? 'Greater than 5' : 'Smaller than 5'
      }
    },
    methods: {
      result() {
        console.log('Method!')
        return this.counter > 5 ? 'Greater than 5' : 'Smaller than 5'
      }
    }
  })

  ```

  위 예제에서 increase 또는 decrease 버튼을 누르면 `result()` 메소드가 실행되고 `output()` 메소드 또한 실행됩니다.
  <br />

  **콘솔**

  ![1](https://user-images.githubusercontent.com/31656287/43925701-f53f72a6-9c62-11e8-90c4-cc2e4ee9de59.png)

  콘솔에 Method!와 Computed!가 출력된 것을 확인할 수 있습니다.

  다음으로 Increase Second 버튼을 클릭할 시 `result()` 메소드는 실행되는 반면, `output()` 메소드는 실행되지 않는 모습을 볼 수 있습니다.

  ![2](https://user-images.githubusercontent.com/31656287/43925850-72865630-9c63-11e8-97e6-0dd928e1077f.png)

   이 점이 *methods* 프로퍼티와 *computed* 프로퍼티의 주요 차이점입니다.  `output()` 메소드는 내부적으로 `this.counter`를 사용합니다. 이에 따라 Vue는 `output()`이 `this.counter`에 의존적인 것을 알고 있습니다. 따라서 Vue는 Increase Second 버튼을 누를 시 사용되는 변수가 secondCounter 밖에 없으므로 이에 의존하지 않는 `output()`을 실행하지 않고, `result()`는 이와 관계없이 실행이 됩니다.

   다시말해서 methods는 매번 재 렌더링시마다 불필요한 코드까지 실행할 위험(?)이 있으나 computed은 그렇지 않기 때문에 이 프로퍼티를 잘 이용하면 불필요한 코드실행을 줄여 리소스 낭비를 줄일 수 있습니다. 이러한 특징을 Vue에서는 캐싱한다고 표현하기도 합니다.

  <br />

- ### Watcher

  *watch* 프로퍼티는 인스턴스 내 데이터 변경을 감지하는 감시자이다.

  <br />

  **index.html**

  ```html
  <script src="https://npmcdn.com/vue/dist/vue.js"></script>

  <div id="exercise">
      <div>
          <p>Current Value: {{ value }}</p>
          <button @click="value += 5">Add 5</button>
          <button @click="value += 1">Add 1</button>
          <p>{{ result }}</p>
      </div>
      <div>
          <input type="text">
          <p>{{ value }}</p>
      </div>
  </div>

  <script src="app.js"></script>

  ```

  <br />

  **app.js**

  ```javascript
  new Vue({
          el: '#exercise',
          data: {
              value: 0
          },
          computed: {
            result() {
              return this.value === 37 ? 'done' : 'not there yet'
            }
          },
          watch: {
            result(val) {
              var vm = this
              setTimeout(() => {
                vm.value = 0;
              }, 5000)
            }
          }
      });

  ```



  다음 예제에서 *watch*는 `result()`(computed 메소드의 결과값은 하나의 데이터로 취급된다.)의 변화를 감지하여 `result()`가 변경된 경우 (= `value`가 37이 된 경우) 5초 뒤 value의 값을 0으로 초기화한다.  이러한 특징을 이용하여 `value` 값 변화에 무한 루프를 형성할 수 있다. 0 -> 37 -> 0 -> 37...

  <br />

  1. 버튼을 통해 `value` 값이 변화한다. (*data*)
  2. `result()`는 `value` 값을 37과 대조하여 이에 반응한다. (*computed*)
  3. *watch*는 `result` 값의 변화를 감지하여 `value` 값을 변경한다. (*watch*)
