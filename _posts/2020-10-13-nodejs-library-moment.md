---
published: true
layout: single
title: "Node.js 유용한 라이브러리 - Moment.js"
classes: wide
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js 
  - 라이브러리 소개
---

웹사이트에서 시간 표현을 하기 위해 포맷팅을 하려면 매우 귀찮아진다. 만약 포맷팅을 해주는 함수같은게 있다면 다행이겠지만, 그걸 만드는 작업도 귀찮은건 사실이다. 그래서 이번 포스팅에서는 Moment.js라는 라이브러리를 써볼 것이다.

## 설치

npm을 사용한다면 위의 명령어를, yarn을 사용한다면 아래의 명령어를 사용하자.

~~~sh
# npm
npm install moment

# yarn
yarn add moment
~~~

## 시간 포맷팅

포맷 관련 문서는 [여기](https://momentjs.com/docs/#/parsing/string-format/)에서 가져올 수 있다.

~~~js
const moment = require('moment');

//현재 시간을 기본 포맷으로 출력
console.log(moment().format());

//현재 시간을 YYYY-MM-DD 포맷으로 출력
console.log(moment().format("YYYY-MM-DD"));

//지정된 시간을 기본 포맷으로 출력
console.log(moment("2020-10-13").format());
~~~

## 상대적 시간 출력

~~~js
const moment = require('moment');

//지정된 시간으로부터 얼마나 지났는지 출력
console.log(moment("2020-10-13").fromNow());

//지정된 시간까지 얼마나 남았는지 출력
console.log(moment("2025-10-13").fromNow());

//오늘의 처음을 LLLL 포맷으로 출력
console.log(moment().startOf('day').format('LLLL'));

//오늘의 끝을 LLLL 포맷으로 출력
console.log(moment().endOf('day').format('LLLL'));

//현재 시간으로부터 7일 전
console.log(moment().subtract(7, 'days').format('LLLL'));

//현재 시간으로부터 7일 후
console.log(moment().add(7, 'days').format('LLLL'));
~~~

## 달력 표기

~~~js
const moment = require('moment');

//현재 시간을 달력 표기
console.log(moment().calendar());

//5일 전 시간을 달력 표기
console.log(moment().subtract(7, 'days').calendar());

//5일 후 시간을 달력 표기
console.log(moment().add(7, 'days').calendar());
~~~

## 로케일 변경

~~~js
const moment = require('moment');

//현재 로케일을 한국으로 변경
moment.locale('ko');

//현재 시간으로부터 5일 전
console.log(moment().subtract(5, 'days').format('LLLL'));

//5일 전 시간을 달력 표기
console.log(moment().subtract(5, 'days').calendar());
~~~