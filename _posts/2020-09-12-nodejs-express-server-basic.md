---
published: true
layout: single
title: "Express로 Node.js 서버 구축하기 - 기초"
classes: wide
category: Node.js
tags: 
  - node.js
  - express
---

Node.js는 JavaScript로 네트워크 애플리케이션을 개발하기 위해 설계된 런타임이다. 그래서 서버를 구축하는 예제들과 라이브러리들도 넘쳐나는데, 이번 포스트에서는 Express 라이브러리를 다룰 것이다.

## Express 설치

Express를 설치하기 전에 프로젝트 폴더로 사용할 폴더 하나를 생성하도록 하자.

Express를 사용하면서 설치할 패키지는 다음과 같다.

- express
- ejs
- morgan

Express는 왜 까는지 말 안해도 될 것 같고, ejs는 HTML을 직접 만드는 대신 ejs라는 템플릿 엔진을 사용해 HTML을 생성해서 제공하기 위해 사용할 것이다. morgan은 서버가 받은 요청을 콘솔로 출력하기 위해 사용한다.

프로젝트 폴더에서 다음 명령어를 입력하면 패키지가 설치된다.
~~~sh
npm install --save express ejs morgan
~~~

## 기본적인 Express 서버

프로젝트 폴더 아래에 index.js라는 파일을 하나 만들어서 기본적인 서버의 코드를 입력해보자.

~~~js
const express = require('express'); // Express 임포트
const app = express();              // Express 앱 생성

app.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  res.send('Hello World!');         // res 객체로 응답을 보낼 수 있다.
});

app.listen(3000, () => {            // 3000번 포트로 요청 수신
  console.log('Server started at http://localhost:3000');
});
~~~

코드를 실행하면 웹 브라우저에서 http://localhost:3000/ 으로 접속했을 떄 Hello World가 나오는 걸 볼 수 있다.
~~~
node index.js
~~~

![hello_world](https://imgur.com/2OSM2lY.png)

## 로그 미들웨어 설정하기

Express에서 미들웨어란 요청을 처리할 때마다 자동으로 실행되는 함수들이다. 이번에 우리가 사용할 미들웨어는 morgan으로, Express에서 사용하려면 위의 index.js에 두줄만 추가하면 된다.

~~~js
const morgan = require('morgan');   // morgan 임포트

app.use(morgan('common'));          // morgan 미들웨어의 로그 출력을 common 포맷으로 설정해서 Express에 넘겨준다.
~~~

다시 서버를 실행하면 브라우저로 접속할 떄마다 다음과 같이 로그가 뜬다.

![morgan_log](https://imgur.com/PKJfgMM.png)

## ejs로 웹페이지 전송하기

일반적으로 웹 브라우저에 컨텐츠를 보여줄 목적으로 웹 서버를 구축한다면 html 파일을 웹 서버에서 전달하는 경우가 많다. 하지만 HTML의 내용을 실시간으로 변경해서 전달하는 작업은 어렵다. 그래서 템플릿 엔진이라는 것이 나왔고, 우리는 이번 포스트에서 템플릿 엔진 중 ejs를 써볼 것이다.

일단 프로젝트 폴더 안에 views 폴더를 하나 더 생성해야 한다.
이 폴더에는 ejs 파일들이 들어갈 것이다.

index.js에 다음 두 줄을 추가하면 ejs를 사용할 준비가 된 것이다.

~~~js
app.set('views', __dirname + '/views'); // views 폴더에서 ejs 템플릿을 가져오게 설정
app.set('view engine', 'ejs');          // 템플릿 엔진을 ejs로 설정
~~~

이제 ejs 파일을 하나 만들 것인데, views 폴더 안에 index.ejs라는 파일을 하나 만들어보자.

~~~html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><%= title %></title> <!--title 변수를 문서에 포함-->
</head>
<body>
  <h1><%= title %></h1> <!--title 변수를 문서에 포함-->
    <% for(var i = 0; i < max; i++) { %> <!--for문으로 반복 렌더링-->
        <!--여기에 반복해서 렌더링 할 내용 입력-->
        <p>Hello World <%= i %>트</p>
    <% } %>   
    <% if(max % 2 == 0) { %>             <!--if문으로 조건부 렌더링-->
        <!--아래 내용은 if문의 조건이 충족할때만 출력된다.-->
        <p>max는 짝수입니다.</p>
    <% } else { %>
        <!--아래 내용은 if문의 조건이 충족할때만 출력된다.-->
        <p>max는 홀수입니다.</p>   
    <% } %>   
</body>
</html>
~~~

ejs 파일이 준비되었으니 index.js 파일에서 요청을 받을 때 ejs 파일이 렌더링 되게 수정할 것이다.

~~~js
app.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  return res.send('Hello World!');         // res 객체로 응답을 보낼 수 있다.
});
~~~
이 부분을 아래 코드로 바꾸면 ejs 파일이 렌더링 된다.
~~~js
app.get('/', (req, res) => {        // / 경로로 get 요청을 받았을 때
  return res.render('index', {title: "제목", max: 5});  //두번쨰 매개변수로 객체에 데이터를 담아 ejs에 전달할 수 있다.
});
~~~

다시 서버를 실행해보면 아래 사진처럼 정상적으로 작동한다.

![ejs_render](https://imgur.com/11eiwXA.png)

ejs 내에서 사용할 변수는 얼마든지 만들 수 있고, 넘기는 데이터의 종류도 상관없다. 값을 변경하거나 추가하면서 익혀보도록 하자.