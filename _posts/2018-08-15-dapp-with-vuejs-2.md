---
published: true
layout: single
title: "[DApp] web3와 Vue.js를 이용한 첫 이더리움 DApp 만들기 (2)"
category: post
tags:
comments: true
---

 안녕하세요. 이번 포스팅은 Casino DApp 구축 튜토리얼의 2편입니다. 혹시 1편을 보지 못하신분들은 <https://makehoney.github.io/post/2018/08/13/dapp-with-vuejs/> 에서 1편을 보실 수 있습니다.



* ## Vue.js

   Vue.js(이하 Vue)는 웹 프론트엔드 구축을 담당하는 JavaScript 기반 웹 라이브러리입니다. Vue를 사용하면 브라우저와 유저간의 동적인 인터랙션이 가능합니다. MetaMask가 브라우저에서 돌아가기 때문에 Vue 또는 React와 같은 라이브러리를 활용하여 DApp을 구축하면 매우 손쉽게 web3를 통해서 웹앱과 이더리움 클라이언트(+ test net)를 연동할 수 있습니다. 

  ``````vue
  <div id=”app”>
   {{ message }}
  </div>
  
  var app = new Vue({
   el: '#app',
   data: {
   message: 'Hello Vue!'
   }
  })
  ``````

   다음 코드는 Vue의 가장 기본적인 코드 구조입니다. data 객체에 존재하는 `message` 프로퍼티는 app이라는 id와 함께 기본 HTML로 렌더링되어 화면에 표시됩니다. 만약 `message` 프로퍼티가 변경된다면 페이지 리프레싱 없이 변경된 데이터가 화면에 표시될 것입니다. 다음 jsfiddle 링크에 가시면 결과 값을 확인해 보실 수 있습니다.

  <https://jsfiddle.net/makehoney/jvwkqodt/2/>

  <br />

   또 다른 Vue의 주요 특징 중 하나는 Vue의 컴포넌트입니다. 컴포넌트는 작고 재사용이 가능한Vue 인스턴스로 생각하시면 됩니다. 실제로 웹 페이지는 다음 그림과 같이 Vue의 컴포넌트 간의 트리 구조로 추상화될 수 있습니다.

![19xltavitmhophmq634kfvg](https://user-images.githubusercontent.com/31656287/44133622-b0e2bad6-a09b-11e8-9c84-aa0c2d34a167.png)

<br />

* ## Vuex

   우리는 컴포넌트 간 공유되는 데이터, 즉 state를 관리하기 위해서 Vuex를 사용할 것입니다. Vuex를 통해서 우리는 손쉽게 데이터를 조작하고 애플리케이션에 예측 가능한 방식으로 데이터를 전달할 수 있습니다.

   Vuex가 동작하는 방식은 매우 직관적인데요. **컴포넌트**는 렌더링될 때 데이터를 필요로합니다. 해당 데이터를 얻기위해서 **컴포넌트**는 **action**을 dispatch합니다. 이후 **action**은 외부 API의 비동기 함수를 처리한 뒤 얻어진 데이터를 **mutation**에 commit합니다. 그러면 **mutation**은 해당 데이터를 이용하여 store의 state(저장되어 있는 데이터)를 새 데이터로 변경하고, 이렇게 변경된 새로운 데이터를 **컴포넌트**가 가져다 쓰는 구조입니다.

   사실 외부 API 등 비동기 처리를 통해 데이터를 얻을 필요가 없는 경우에는 action을 거칠 필요 없이 **컴포넌트** 자체적으로 **mutation**에 commit을 할 수 있습니다. 우리 앱의 경우 외부 API에 해당하는 부분이 web3입니다.

  ![vuex](https://user-images.githubusercontent.com/31656287/44134883-a23dd3f2-a0a1-11e8-9e06-c8dadcc2bd98.png)

   Vuex에 대한 자세한 설명은 아래 링크를 참조하시면 더 쉽게 이해하실 수 있습니다.

  <https://joshua1988.github.io/web-development/vuejs/vuex-getters-mutations/>

  