---
published: true
layout: single
title: "[DApp] web3와 Vue.js를 이용한 첫 이더리움 DApp 만들기 (3)"
category: post
tags:
comments: true
---



Part.1: <https://makehoney.github.io/post/2018/08/13/dapp-with-vuejs-1/>

Part.2: <https://makehoney.github.io/post/2018/08/15/dapp-with-vuejs-2/>

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

   이번 과정에서는 먼저 컨트랙트 초기화의 뼈대가 되는 코드를 작성한 뒤에 스마트 컨트랙트를 배포하고 ABI와 address 값을 저희 앱에 넣을 것입니다.

  * 유저가 베팅할 금액을 써 넣을 input field가 필요합니다.
  * 유저가 베팅할 번호를 선택할 수 있는 버튼이 필요합니다.
  * on click 함수는 컨트랙트의 bet() 함수를 호출해야합니다.
  * 트랜잭션 중(완료되지 않음)임을 표시해주는 로딩 스피너가 필요합니다.
  * 트랜잭션이 완료됐을 때 게임의 결과를 표시해줘야 합니다.

   먼저 우리 앱과 컨트랙트를 연결해주는 작업이 필요합니다. util 아래에 getContract.js를 생성하여 아래와 같이 작성해 줍니다.

  ``````javascript
  /* util/getContract.js */
  
  import Web3 from 'web3';
  import { address, ABI } from './constants/casinoContract';
  
  let getContract = new Promise((resolve, reject) => {
    let web3 = new Web3(window.web3.currentProvider);
    let casinoContract = web3.eth.contract(ABI);
    let casinoContractInstance = casinoContract.at(address);
    resolve(casinoContractInstance);
  });
  
  export default getContract;
  ``````

   우선 주목할 부분은 두번째라인의 import하는 파일이 현재는 존재하지 않는다는 점입니다. casinoContract파일은 컨트랙트를 remix를 통해 배포한 뒤에 작성하도록 하겠습니다.

   다음으로 casino-component.vue 파일을 아래와 같이 작성해줍니다. 아래 코드는 아시겠지만 dispatch의 대상이 되는 action과 commit의 대상인 mutation이 없으면 불완전한 코드입니다.

  ``````vue
  <!-- casino-component.vue -->
  
  export default {
   name: ‘casino’,
   mounted () {
   console.log(‘dispatching getContractInstance’)
   this.$store.dispatch(‘getContractInstance’)
   }
  }
  ``````

   이제 store/index.js에서 getContract를 import한 뒤에 여기에 상응하는 action과 mutation을 작성하겠습니다.

  ``````javascript
  /* action */
  
  async getContractInstance({ commit }) {
    try {
      let result = await getContract;
      commit('registerContractInstance', result);
    } catch (err) {
      console.log('error in action getContractInstance', err);
    }
  }
  ``````

  ``````javascript
  /* mutation */
  
  registerContractInstance(state, payload) {
    console.log('Casino contract instance: ', payload);
    state.contractInstance = () => payload;
  }
  ``````

   이 작업을 통해서 우리의 컨트랙트 인스턴스가 컴포넌트로부터 store의 state에 저장될 것입니다.

  <br />

* ## 스마트 컨트랙트와 상호작용하기

   스마트 컨트랙트와의 상호작용을 위해서는 먼저 브라우저 상에서 보여질 템플릿을 작성한 뒤에 템플릿에 상응하는 data와 methods 프로퍼티를 추가해야 합니다.

  ``````vue
  <!-- casino-component.vue --> 
  
  data () {
     return {
       amount: null,
       pending: false,
       winEvent: null
     }
   }
  ``````

   다음으로 숫자가 클릭되었을 때 컨트랙트의 bet() 함수를 트리거해주는 함수를 methods 프로퍼티에 추가해줍니다.

  ``````vue
  <!-- casino-component -->
  
  methods: {
    clickNumber (event) {
      console.log(event.target.innerHTML, this.amount)
      this.winEvent = null
      this.pending = true
      this.$store.state.contractInstance().bet(event.target.innerHTML, {
        gas: 300000,
        value: this.$store.state.web3.web3Instance().toWei(this.amount, 'ether'),
        from: this.$store.state.web3.coinbase
      }, (err, result) => {
        if (err) {
          console.log(err)
          this.pending = false
        } else {
          let bettingResult = this.$store.state.contractInstance().bettingResult()
          /* .watch => solidity event를 감시 */
          bettingResult.watch((err, result) => {
            if (err) {
              console.log('could not get event Won()')
            } else {
              this.winEvent = result.args
              this.winEvent.rewards = parseInt(result.args.rewards, 10)
              console.log(`winEvent: ${result.args}`)
              this.pending = false
            }
          });
        }
      });
    }
  }
  ``````

   bet() 함수의 첫번째 파라미터인 event.tartget.innerHTML은 템플릿의 li 태그 내부 값(숫자 1-10)을 가리킵니다. 그 다음 파라미터는 트랜잭션 파라미터로서 가스, 유저가 거는 금액 및 베팅하는 사람의 address 등을 받고, 마지막 파라미터는 bet() 함수의 콜백으로 동작합니다. watch 메소드는 컨트랙트 코드 상에서의 event를 감시하는 감시자입니다.

   다음은 템플릿 및 스타일 코드입니다. casino-component의 템플릿과 스타일 시트에 그대로 작성합니다.

  ``````vue
  <!-- casino-component.vue -->
  
  <template>
   <div class="casino">
     <h1>Welcome to the Casino</h1>
     <h4>Please pick a number between 1 and 10</h4>
     Amount to bet: <input v-model="amount" placeholder="0 Ether">
     <ul>
       <li v-on:click='clickNumber'>1</li>
       <li v-on:click='clickNumber'>2</li>
       <li v-on:click='clickNumber'>3</li>
       <li v-on:click='clickNumber'>4</li>
       <li v-on:click='clickNumber'>5</li>
       <li v-on:click='clickNumber'>6</li>
       <li v-on:click='clickNumber'>7</li>
       <li v-on:click='clickNumber'>8</li>
       <li v-on:click='clickNumber'>9</li>
       <li v-on:click='clickNumber'>10</li>
    </ul>
    <img v-if="pending" id="loader" src="https://loading.io/spinners/double-ring/lg.double-ring-spinner.gif">
    <div class="event" v-if="winEvent">
      <p>Won: {{ winEvent.userWin }}</p>
      <p>Winning Number: {{ winEvent.winningNumber }}</p>
      <p>Amount: {{ winEvent.rewards }} Wei</p>
    </div>
    <div class="event" v-if="winEvent">
     <p v-if="winEvent.userWin" id="has-won"><i aria-hidden="true" class="fa fa-check"></i> Congragulations, you have won {{winEvent.rewards}} wei</p>
     <p v-else id="has-lost"><i aria-hidden="true" class="fa fa-check"></i> Sorry you lost, please try again.</p>
    </div>
   </div>
  </template>
  
  <style scoped>
  .casino {
   margin-top: 50px;
   text-align:center;
  }
  #loader {
   width:150px;
  }
  ul {
   margin: 25px;
   list-style-type: none;
   display: grid;
   grid-template-columns: repeat(5, 1fr);
   grid-column-gap:25px;
   grid-row-gap:25px;
  }
  li{
   padding: 20px;
   margin-right: 5px;
   border-radius: 50%;
   cursor: pointer;
   background-color:#fff;
   border: -2px solid #bf0d9b;
   color: #bf0d9b;
   box-shadow:3px 5px #bf0d9b;
  }
  li:hover{
   background-color:#bf0d9b;
   color:white;
   box-shadow:0px 0px #bf0d9b;
  }
  li:active{
   opacity: 0.7;
  }
  *{
   color: #444444;
  }
  #has-won {
    color: green;
  }
  #has-lost {
    color:red;
  }
  </style>
  ``````

  <br />

* ## Ropsten 테스트넷에 컨트랙트 배포하기

   Ropsten 테스트넷에 컨트랙트를 배포하기 위해서는 Part.1의 과정에서 environment를 javascriptVM로 설정했던 것을 injected Web3로 설정한 뒤에 동일하게 배포를 하시면 됩니다. 

   배포를 성공적으로 마치면 remix 콘솔창에 EtherScan의 링크가 나올 것이고 들어가면 다음과 같이 컨트랙트 배포가 된 것을 확인해 보실 수 있습니다.

  ![3](https://user-images.githubusercontent.com/31656287/44467547-18528d00-a65e-11e8-8ed6-8fd90e110169.png)

   다음으로 Remix의 Compile 탭의 Detail 버튼을 누르면 ABI를 알아낼 수 있습니다.

  ![1](https://user-images.githubusercontent.com/31656287/44467841-dece5180-a65e-11e8-8eb1-f44b6b225667.png)

   그 다음 Run 탭의 Deployed Contracts에서 배포된 컨트랙트의 address를 알 수 있습니다.

  ![2](https://user-images.githubusercontent.com/31656287/44467843-e0981500-a65e-11e8-80ba-bfda109f35c0.png)

  

   마지막으로 이렇게 알아낸 정보들을 util/constants 아래에 casinoContract.js를 생성하여 다음과 같이 작성하면 마침내 저희 DApp이 완성됩니다!

  ``````
  const address = ‘0x…………..’
  const ABI = […]
  export {address, ABI}
  ``````

  <br />

* ## 프론트엔드

   이번 주제는 필수적인 부분이 아닙니다. 위 과정에서 만족하셨으면 넘어가셔도 무방합니다!

  여기서는 hello-metamask 컴포넌트를 수정할 계획입니다. 먼저 Vuex의 mapState 헬퍼를 시용하여 템플릿에 분기문을 작성하고 HTML이 그에 따라 다르게 렌더링되도록 할 것입니다. 

   이에 앞서 아이콘 사용을 위해서 main.js에서 다음과 같이 css를 import해줍니다.

  ``````javascript
  /* main.js */
  
  import 'font-awesome/css/font-awesome.css'
  ``````

  

   최종 hello-metamask.vue의 코드는 다음과 같습니다.

  ``````vue
  <!-- hello-metamask.vue -->
  
  <template lang="html">
    <div class='metamask-info'>
      <p v-if="isInjected" id="has-metamask"><i aria-hidden="true" class="fa fa-check"></i> Metamask installed</p>
      <p v-else id="no-metamask"><i aria-hidden="true" class="fa fa-times"></i> Metamask not found</p>
      <p>Network: {{ network }}</p>
      <p>Account: {{ coinbase }}</p>
      <p>Balance: {{ balance }} Wei </p>
    </div>
  </template>
  
  <script>
    import {NETWORKS} from '../util/constants/networks'
    import {mapState} from 'vuex'
    export default {
      name: 'hello-metamask',
      computed: mapState({
        isInjected: state => state.web3.isInjected,
        network: state => NETWORKS[state.web3.networkId],
        coinbase: state => state.web3.coinbase,
        balance: state => state.web3.balance
      })
    }
  </script>
  
  <style scoped>
  #has-metamask {
    color: green;
  }
  #no-metamask {
    color:red;
  }</style>
  ``````

  <br />

* ## 마무리

   저희가 여태까지 만든 DApp의 전체 코드는  <https://github.com/MakeHoney/DApp_with_vue> 이곳에서 확인해 보실 수 있습니다! 여기까지 잘 따라오셨다면 터미널에서 앱을 실행 시킨 뒤에 localhost:8080으로 접속하시어 베팅을 해보시면 아래와 같은 결과를 보실 수 있으실 겁니다.

  ![1](https://user-images.githubusercontent.com/31656287/44472583-6d47d080-a669-11e8-9954-84862a9e507d.png)

  고생하셨습니다!

   <br />

    ***\* 이 튜토리얼은 아래 참조 링크를 바탕으로 코드 상의 경고 또는 에러를 수정하여 작성되었음을 알려드립니다***

* ## References

  - <https://itnext.io/create-your-first-ethereum-dapp-with-web3-and-vue-js-part-3-dc4f82fba4b4>

  