---
published: true
layout: single
title: "web3와 Vue.js를 이용한 첫 이더리움 DApp 만들기 (1)"
category: post
tags:
comments: true
---

* ### DApp description

  > 저희가 만들 DApp은 간단합니다. 유저는 1에서 10 사이의 수에 일정 금액을 베팅합니다. 만약 유저가 선택한 숫자가 당첨될 경우 베팅 금액의 10배를 받는 심플한 Casino DApp입니다.

  <br />

  * Part.1: 프로젝트 셋업 및 스마트 컨트랙트 생성
  * Part.2: web3.js 및 Vue.js/Vuex 소개 및 싱글페이지 앱 구현
  * Part.3: Vue.js와 스마트 컨트랙트 연결

  <br />

* ### Prerequisites

  저희는 Remix를 이용하여 스마트 컨트랙트를 MetaMask Ropsten 테스트넷에 배포할 예정입니다.  (https://remix.ethereum.org) 프로젝트에 앞서 node.js와 npm은 설치되어 있다고 가정하겠습니다.

  <br />

  아래 명령어를 통해서 vue-cli를 설치합니다.	

    ``````
    npm i vue-cli -g
    ``````

  덧붙여 스마트 컨트랙트를 테스트넷에 배포하기 위한 MetaMask가 설치되어 있어야 합니다.  MetaMask는 현재 크롬과 파이어폭스를 지원합니다. (https://metamask.io)

  <br />

* ### Project set-up

  저희가 구현할 프론트엔드 애플리케이션과 MetaMask의 테스트넷을 연결하기 위해서는 스마트 컨트랙트 address와 ABI가 필요합니다. 또한 MetaMask는 브라우저 상에서 돌아가기 때문에 web3와 Vue.js를 이용하면 프론트엔드 상에서도 충분한 상호작용이 가능합니다.

  <br />

  1. 터미널을 열고 프로젝트를 생성할 디렉토리로 이동합니다.
  2. 터미널 창에 아래 명령어를 입력하여 vue 프로젝트를 생성합니다.

  ``````
  vue init webpack betting-dapp
  ``````

  3. 해당 디렉토리로 이동하여 web3, vuex font-awesome npm을 설치합니다.

  ``````
  cd betting-dapp
  npm install web3@^0.20.0 vuex font-awesome
  ``````

  > MetaMask와의 호환성 문제로 web3 1.0.0 beta 버전은 사용하지 않습니다.