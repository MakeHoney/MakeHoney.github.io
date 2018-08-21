---
published: true
layout: single
title: "[DApp] web3와 Vue.js를 이용한 첫 이더리움 DApp 만들기 (2)"
category: post
tags:
comments: true
---

Part.1: <https://makehoney.github.io/post/2018/08/13/dapp-with-vuejs-1/>



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

  <br />

* ## 기본 컴포넌트 작성

   우리는 Part.1에서 vue-cli를 이용하여 Vue 앱을 생성하고 거기에 필요한 의존성들을 설치했습니다. 만약 이 과정을 잘 따라오셨다면 저희 프로젝트 디렉토리 구조는 다음과 같겠습니다.

  ![1](https://user-images.githubusercontent.com/31656287/44269155-aa2d5500-a26e-11e8-8a2c-b97e7d1b4594.png)

  ***\* 처음 vue init을 할 때 ESLint를 설치할 것이냐고 물어보는데 저희는 ESLint를 사용하지 않습니다. 이유는 ESLint를 설치할 경우 엄격한 syntax 작성 규칙이 적용되기 때문입니다. 저희는 간단한 개인 프로젝트를 진행하는 것이므로 이와 같은 툴은 필요하지 않습니다.***

  

  * App.vue 파일의 img-tag를 제거하고 style 태그에 있는 모든 내용을 지웁니다.

  * components/HelloWorld.vue 파일을 지운뒤에 해당 디렉토리에 casino-dapp.vue와 hello-metamask.vue 파일을 생성합니다.

    * casino-dapp.vue: 메인 컴포넌트
    * hello-metamask: MetaMask 데이터를 포함하는 컴포넌트

  * hello-metamask.vue를 다음과 같이 작성합니다.

    ``````vue
    <!-- components/hello-metamask.vue -->
    
    <template lang="html">
      <p>Hello</p>
    </template>
    
    <script>
      export default {
        name: 'hello-metamask'
      }
    </script>
    
    <style>
    </style>
    ``````

  * 이제 우리는 casino-dapp 컴포넌트에서 hello-metamask 컴포넌트를 불러와야합니다. 이를 위해서 casino-dapp 컴포넌트에서 hello-metamask 컴포넌트를 import 한 뒤 자식 컴포넌트로 등록을 하면 템플릿에서 태그처럼 사용할 수 있습니다.

    ``````vue
    <!-- components/casino-dapp.vue -->
    
    <template lang="html">
      <hello-metamask/>
    </template>
    
    <script>
      import HelloMetamask from './hello-metamask.vue'
      export default {
        name: 'casino-dapp',
        components: { HelloMetamask }
      }
    </script>
    
    <style>
    </style>
    ``````

  * 이제 router/index.js 파일을 열어봅니다. 보시면 현재 하나의 라우터가 존재하고 여전히 HelloWorld.vue 컴포넌트를 가리키고 있는 모습을 볼 수 있는데요. 저희는 이 부분을 casino-dapp.vue을 가리키도록 수정할 것입니다.

    ``````javascript
    /* router/index.js */
    
    import Vue from 'vue';
    import Router from 'vue-router';
    import CasinoDapp from '@/components/casino-dapp';
    
    Vue.use(Router);
    
    export default new Router({
      routes: [
        {
          path: '/',
          name: 'casino-dapp',
          component: CasinoDapp
        }
      ]
    });
    ``````

  * 마지막으로 src 디렉토리 밑에 util이라는 새로운 폴더를 생성합니다. 그리고 util 밑에 constants라는 폴더를 생성한 뒤에 그 안에 networks.js를 생성합니다. networks.js를 다음과 같이 채워줍니다.

    ``````javascript
    /* src/util/constants/networks.js */
    
    export const NETWORKS = {
     '1': 'Main Net',
     '2': 'Deprecated Morden test network',
     '3': 'Ropsten test network',
     '4': 'Rinkeby test network',
     '42': 'Kovan test network',
     '4447': 'Truffle Develop Network',
     '5777': 'Ganache Blockchain'
    }
    ``````

    이 코드는 저희 이더리움 네트워크의 'id'를 대신하여 '이름'을 표시해 줄 것입니다.

  * 마지막으로 src 밑에 store라는 폴더를 생성해줍니다. 이 부분은 바로 다음 주제에서 다루겠습니다!

    여기까지 오셨다면 root directory에서 'npm start'를 입력하여 서버를 켤 수 있습니다. 브라우저에 Hello라는 메시지가 나온다면 다음 단계를 진행하셔도 좋습니다!

<br />

* ## Vuex store 세팅하기

  * 이번 주제에서는 Vuex의 store를 세팅해보겠습니다. 

  * 일단 store안에 index.js와 state.js를 생성한 뒤에 state.js를 다음과 같이 작성합니다.

    ``````javascript
    /* store/state.js */
    
    let state = {
      web3: {
        isInjected: false,
        web3Instance: null,
        networkId: null,
        coinbase: null,
        balance: null,
        error: null
      },
      contractInstance: null
    };
    
    export default state;
    ``````

    

  * 다음으로 index.js를 다음과 같이 작성합니다. Vuex 라이브러리와 Vue를 사용하기 위해서 import하고 state(데이터)를 역시 사용해야하므로 import해줍니다.

    ``````javascript
    /* store/index.js */
    
    import Vue from 'vue';
    import Vuex from 'vuex';
    import state from './state';
    
    Vue.use(Vuex);
    
    export const store = new Vuex.Store({
     strict: true,
     state,
     mutations: {},
     actions: {}
    });
    ``````

  * 마지막으로 main.js에서 store를 import해 줍니다. 다음과 같이 폴더자체를 import시키면 Vuex에 의해서 안의 index.js가 자동으로 import됩니다.

    ``````javascript
    /* src/main.js */
    
    import Vue from 'vue';
    import App from './App';
    import router from './router';
    import { store } from './store';
    
    Vue.config.productionTip = false;
    
    /* eslint-disable no-new */
    new Vue({
      el: '#app',
      router,
      store,
      components: { App },
      template: '<App/>'
    });
    ``````

<br />

* ## web3와 Metamask 연동

   앞서 설명드린바와 같이 우리 Vue 앱에서 사용할 데이터를 web3로부터 얻어 오려면 비동기 API 콜을 실행시켜줄 action을 dispatch해야 합니다. 먼저, util 밑에 getWeb3.js 파일을 생성한 뒤에 다음과 같이 작성해 줍니다.

  ``````javascript
  /* util/getWeb3.js */
  
  import Web3 from 'web3';
  
  let getWeb3 = new Promise((resolve, reject) => {
    var web3js = window.web3;
    if(typeof web3js !== 'undefined') {
      let web3 = new Web3(web3js.currentProvider);
      resolve({
        injectedWeb3: web3.isConnected(),
        web3 () {
          return web3;
        }
      });
    } else {
      reject(new Error('Unable to connect to Metamask'));
    }
  }).then(result => {
    return new Promise((resolve, reject) => {
      result.web3().version.getNetwork((err, networkId) => {
        if(err) {
          reject(new Error('Unable to retrieve network ID'));
        } else {
          result = Object.assign({}, result, { networkId });
          resolve(result);
        }
      });
    });
  }).then(result => {
    return new Promise((resolve, reject) => {
      result.web3().eth.getCoinbase((err, coinbase) => {
        if(err) {
          reject(new Error('Unable to retrieve coinbase'));
        } else {
          result = Object.assign({}, result, { coinbase });
          resolve(result);
        }
      });
    });
  }).then(result => {
    return new Promise((resolve, reject) => {
      result.web3().eth.getBalance(result.coinbase, (err, balance) => {
        if(err) {
          reject(new Error(`Unable to retrieve balance for addres: ${result.coinbase}`))
        } else {
          result = Object.assign({}, result, { balance });
          resolve(result);
        }
      });
    });
  });
  
  export default getWeb3;
  ``````

   MetaMask는 브라우저 상에서 자신의 web3 인스턴스를 지니고 있습니다. 따라서 우리는 첫번째 분기문을 통해서 window.web3(브라우저 상의 web3 인스턴스)가 undefined 인지 확인합니다. 만약 undefined가 아니라면 web3 인스턴스를 currentProvider로 생성합니다. 구조를 보시면 아시겠지만 다음 Promise에서 이전의 web3 인스턴스가 포함된 객체를 전달받으며 비동기 API 콜을 실행하고 결과에 따라서 객체에 멤버를 추가해 나갑니다.

  * web3.version.getNetwork()는 현재 연결된 네트워크 ID를 반환합니다.
  * web3.eth.coinbase()는 현재 채굴중인 노드의 주소를 반환합니다. 메타마스크 사용시에는 선택된 계좌 주소에 해당됩니다.
  * web3.eth.getBalance(addr)는 인자로 전달된 주소의 잔액을 반환합니다.

   앞서 설명드린 비동기 API 콜이 일어난 뒤로 해당 결과 값이 Vuex store를 통해서 state에 저장되기 까지의 과정을 기억하실 겁니다. 이제 이 부분을 연결할건데요. 먼저 store/index.js 에서 getWeb3.js 파일을 import해준뒤 action단에서 mutation에 commit하게 되면 mutation이 store에 데이터를 저장시켜 줄 것입니다.

  ``````javascript
  /* store/index.js */
  
  import getWeb3 from '../util/getWeb3';
  ``````

   다음으로 action 객체에서 getWeb3를 불러온 뒤 해당 결과 값을 mutation에 commit하겠습니다. 또한 일련의 과정을 확인하기 쉽도록 console.log도 중간중간 삽입하겠습니다.

  ``````javascript
  /* store/index.js */
  
   actions: {
     async registerWeb3 ({ commit }) {
       console.log('registerWeb3 Action being executed');
       try {
         let result = await getWeb3;
         console.log('registerWeb3Instance', result);
         commit('registerWeb3Instance', result);
       } catch (err) {
         console.log('error in action registerWeb3', err);
       }
     }
   }
  ``````

   다음으로 우리의 데이터를 state에 저장시켜줄 mutation 객체를 채워넣겠습니다. 저희가 action에서 commit한 데이터는 두번째 인자인 payload를 통해서 접근할 수 있습니다.

  ``````javascript
  /* store/index.js */
  
  mutations: {
     registerWeb3Insctance (state, payload) {
       console.log('registerWeb3instance Mutation being executed', payload);
       let result = payload;
       let web3Copy = state.web3;
       web3Copy.coinbase = result.coinbase;
       web3Copy.networkId = result.networkId;
       web3Copy.balance = parseInt(result.balance, 10);
       web3Copy.isInjected = result.injectedWeb3;
       web3Copy.web3Instance = result.web3;
       state.web3 = web3Copy;
     }
   }
  ``````

   이제는 컴포넌트 단에서 위 요소를 실행 가능하도록 해줘야 합니다. 이를 위해서 casino-dapp 컴포넌트가 생성되기 이전에 이를 실행할 수 있도록 아래와 같이 코드를 작성해 줍니다. 뷰 인스턴스의 라이프 사이클에 대해서 잘 모르시는 분들은 [여기](https://kr.vuejs.org/v2/guide/instance.html#%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%ED%9B%85)에서 공식 문서를 보실 수 있습니다.

  ``````vue
  <!-- casino-dapp.vue -->
  
  export default {
    name: 'casino-dapp',
    beforeCreate () {
      console.log('registerWeb3 Action dispatched from casino-dapp.vue')
      this.$store.dispatch('registerWeb3')
    },
    components: {
      'hello-metamask': HelloMetamask
    }
  }
  ``````

   자 이제 마지막으로 수정된 데이터가 렌더링되어 브라우저를 통해 보여질 수 있도록 hello-MetaMask 컴포넌트의 템플릿과 스크립트를 아래와 같이 수정해 줍니다.

  ``````vue
  <!-- hello-metamask.vue -->
  
  <template>
   <div class='metamask-info'>
     <p>Metamask: {{ web3.isInjected }}</p>
     <p>Network: {{ web3.networkId }}</p>
     <p>Account: {{ web3.coinbase }}</p>
     <p>Balance: {{ web3.balance }}</p>
   </div>
  </template>
  
  <script>
  export default {
   name: 'hello-metamask',
   computed: {
     web3 () {
       return this.$store.state.web3
       }
     }
  }
  </script>
  
  <style scoped></style>
  ``````

   이로써 저희는 브라우저 상의 MetaMask와 저희 Vue 앱을 연동하는 과정까지 해보았습니다. 만약 성공적으로 따라오셨다면 터미널에 npm start를 입력한 뒤 localhost:8080에 접속해보시면 아래와 같은 화면을 보실 수 있으실 겁니다.

  ![5](https://user-images.githubusercontent.com/31656287/44309823-6d489600-a407-11e8-8d8c-8676e6831014.png)

   Vuex의 개념을 처음 접하신 분들도 있으실 것이기에 Part.2의 부분이 전체 파트 중에서 가장 어렵다고도 볼 수 있는 부분입니다. 다음 파트에서는 저희가 앞서 작성한 스마트 컨트랙트를 Vue 앱에 올리고 앱의 간단한 UI 작업을 하는 것으로 마무리하겠습니다. 고생하셨습니다.

  <br />

  ***\* 이 튜토리얼은 아래 참조 링크를 바탕으로 코드 상의 경고 또는 에러를 수정하여 작성되었음을 알려드립니다***

* ## References

  * <https://itnext.io/create-your-first-ethereum-dapp-with-web3-and-vue-js-part-2-52248a74d58a>