---
published: true
layout: single
title: "Express로 Node.js 서버 구축하기 - 라우팅"
classes: wide
category: Node.js
tags: 
  - node.js
  - express
---

Express 서버에서 요청을 받기 위해서는 요청을 수신하는 경로가 필요하다. 이번 포스트에서는 라우팅을 통해 요청을 받는 경로를 정의해볼 것이다.

이어지는 포스팅입니다. 처음부터 따라하려면 [첫 포스팅](https://fred16157.github.io/node.js/nodejs-express-server-basic/)부터 시작해주세요.

## 라우팅이란?

Express에서 라우팅은 특정 경로로 요청이 들어오면 경로에 따라 응답을 보내주는 방식이다.

라우팅이라고 말하면 생소할 수도 있는데 사실 [저번 포스팅](https://fred16157.github.io/node.js/nodejs-express-server-basic/)에서 라우팅을 이미 활용해 본적이 있다.

~~~js
app.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  return res.render('index', {title: "제목", max: 5});  //res 객체로 응답을 보낼 수 있다.
});
~~~

저번 포스팅에서 작성한 이 코드는 / 경로로 요청이 들어오면 Hello World! 라는 메시지를 반환했으므로 라우팅이라고 볼 수 있다. 이런 코드를 바로 라우터라고 한다.

만약 저 코드에서 경로를 / 에서 /hello 로 바꾼다면 예전 경로에서는 404 에러가, http://localhost:3000/hello 로 접속했을 때 결과를 볼 수 있을 것이다.

![hello](https://imgur.com/VrUtJjr.png)

그래서 이번 포스팅에서는 적극적으로 라우팅을 활용해 볼 것이다.

## 라우터 파일 만들기

하나의 소스파일에서 계속 라우팅을 한다면 결국엔 소스코드의 길이가 매우 길어져서 관리가 어려울 것이다. 그래서 라우터를 파일로 나누어볼 것이다.

프로젝트 폴더 아래에 routes 폴더를 만들고 그 안에 router.js라는 라우터 파일을 만든다음, router.js에 다음과 같이 코드를 작성하면 라우터로서 활용할 수 있다.

~~~js
const express = require('express');
const router = express.Router();

// 여기에 라우터 작성
router.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  return res.render('index', {title: "제목", max: 5});  //res 객체로 응답을 보낼 수 있다.
});

router.get('/hello', (req, res) => {    // /hello 경로로 요청을 받았을 때
    return res.send('Hello World!');    // 마찬가지로 res 객체로 응답을 보낼 수 있다.
})

module.exports = router;    //라우터를 메인 소스코드에서 활용할 수 있게 export
~~~

그런 다음 메인 소스코드인 index.js 에서 라우터를 작성하는 대신 라우터 파일을 임포트해 활용할 수 있게 된다.

~~~js
app.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  return res.render('index', {title: "제목", max: 5});  //res 객체로 응답을 보낼 수 있다.
});
~~~

~~~js
const mainRouter = require('./routes/router');  //라우터 파일을 임포트

app.use('/', mainRouter);   // / 경로에 router.js 파일의 라우트들을 할당
~~~

참고로 / 는 라우터의 경로가 시작되는 경로다. 만약 router 안에 경로가 /와 /hello가 있다면

- http://localhost:3000/
- http://localhost:3000/hello

의 경로에서 응답을 받을 수 있을 것이다.

그래서 시작되는 경로를 바꾸면 전체 경로가 달라진다.

~~~js
const mainRouter = require('./routes/router');  //라우터 파일을 임포트

app.use('/hello', mainRouter);   // / 경로에 router.js 파일의 라우트들을 할당
~~~

이럴 땐 / 에서 경로가 시작되는게 아니라 /hello에서 경로가 시작된다.

- http://localhost:3000/hello/
- http://localhost:3000/hello/hello


라우터 파일은 얼마든지 추가적으로 할당할 수 있다.

예를 들어 라우터 파일 이름이 hello.js라면

~~~js
const mainRouter = require('./routes/router');  //라우터 파일을 임포트
const helloRouter = require('./routes/hello');  //다른 라우터 파일을 임포트

app.use('/', mainRouter);   // / 경로에 router.js 파일의 라우트들을 할당
app.use('/hello', helloRouter)  // /hello 경로에 hello.js 파일의 라우트들을 할당
~~~

와 같이 사용할 수 있다.