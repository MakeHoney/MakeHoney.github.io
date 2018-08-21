---
published: true
layout: single
title: "[DApp] web3와 Vue.js를 이용한 첫 이더리움 DApp 만들기 (3)"
category: post
tags:
comments: true
---



Part.1: <https://makehoney.github.io/post/2018/08/13/dapp-with-vuejs-1/>

Part.2: <https://makehoney.github.io/post/2018/08/13/dapp-with-vuejs-2/>

<br />

* ## 데이터 폴링

  데이터 폴링: <https://ko.wikipedia.org/wiki/%ED%8F%B4%EB%A7%81_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)>

  <br />

   지금까지의 저희 앱은 MetaMask로부터 데이터를 불러와 브라우저에 표시를 할 수 있습니다.  하지만 유저가 MetaMask의 계정을 변경하는 경우, 저희 앱은 자동으로 변경된 데이터를 로드하지 않고 페이지를 리프레시해야 변경된 데이터가 화면에 표시됩니다. 저희 앱이 reactive하지 못하다고 할 수 있는 부분입니다. 따라서 이부분을 구현해보겠습니다.

   현재 MetaMask는 웹소켓을 지원하지 않으므로 저희가 interval을 설정하여 데이터를 폴링하는 방식으로 구현을 해나갈 생각입니다. 먼저 util 아래에 pollWeb3.js 파일을 생성합니다.

  * MetaMask 인스턴스에 의존하지 않기 위해서 Web3를 import해줍니다.
  * 우리의 store를 import해줍니다. 이를 통해 저희는 값을 비교하고 mutation에 commit할 수 있습니다.
  * web3 인스턴스를 생성합니다.
  * 계정이 변경되었는지 매번 확인할 interval을 세팅합니다. 계정이 바뀐 것이 아니라면 잔액을 비교하여 잔액 변화도 반영할 수 있도록 합니다.
  * 현재 hello-metamask 컴포넌트가 computed 프로퍼티로 web3를 가지고 있으므로 위 과정에 의한 데이터 변경은 즉각적으로 반영될 것입니다. (reactive)

  ``````javascript
  /* util/pollWeb3.js */
  
  import Web3 from 'web3';
  import { store } from '../store';
  
  let pollWeb3 = state => {
    let web3 = window.web3;
    web3 = new Web3(web3.currentProvider);
  
    setInterval(() => {
      if(web3 && store.state.web3.web3Instance) {
        if(web3.eth.coinbase !== store.state.web3.coinbase) {
          let newCoinbase = web3.eth.coinbase;
          web3.eth.getBalance(newCoinbase, (err, newBalance) => {
            if (err) {
              console.log(err);
            } else {
              store.commit('pollWeb3Instance', {
                coinbase: newCoinbase,
                balance: parseInt(newBalance, 10)
              });
            }
          });
        } else {
          web3.eth.getBalance(store.state.web3.coinbase, (err, polledBalance) => {
            if (err) {
              console.log(err);
            } else if (parseInt(polledBalance, 10) !== store.state.web3.balance) {
              store.commit('pollWeb3Instance', {
                coinbase: store.state.web3.coinbase,
                balance: polledBalance
              });
            }
          });
        }
      }
    }, 500);
  }
  
  export default pollWeb3;
  ``````

   이제 우리는 web3Instance가 등록되면 데이터 폴링을 시작하도록 해야합니다. web3Instance 등록은 casino-dapp 컴포넌트의 beforeCreate 단계에서 실행됩니다. (Part.2) 따라서 store/index.js에서 pollWeb3.js를 import한 뒤에 mutation 맨 마지막 라인에 pollWeb3()를 실행하여 web3Instance가 등록되면 백그라운드에서 pollWeb3함수가 돌도록 만들어 줍니다.

  ``````javascript
  /* store/index.js */
  
  mutations: {
     registerWeb3Instance (state, payload) {
       console.log('registerWeb3instance Mutation being executed', payload);
       let result = payload;
       let web3Copy = state.web3;
       web3Copy.coinbase = result.coinbase;
       web3Copy.networkId = result.networkId;
       web3Copy.balance = parseInt(result.balance, 10);
       web3Copy.isInjected = result.injectedWeb3;
       web3Copy.web3Instance = result.web3;
       state.web3 = web3Copy;
       /* 추가 */
       pollWeb3();
     }
  ``````

   또한 pollWeb3 함수에서 commit의 대상이 되는 pollWeb3Instance 함수를 mutation에 등록해줍니다.

  ``````javascript
  pollWeb3Instance(state, payload) {
      console.log('pollWeb3Instance mutation being executed', payload);
      state.web3.coinbase = payload.coinbase;
      state.web3.balance = parseInt(payload.balance, 10);
  }
  ``````

   여기까지 마치셨으면 이제 저희 앱은 잔액의 변화 또는 계정의 변화를 즉각적으로 반영하여 화면에 표시해 줄겁니다.

<br />



* ## 스마트 컨트랙트 초기화하기

  