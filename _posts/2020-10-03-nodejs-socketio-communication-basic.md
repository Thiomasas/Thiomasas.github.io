---
published: true
layout: single
title: "Node.js 서버에서 Socket.IO로 실시간 통신하기 - 기초"
classes: wide
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js
  - electron 
---

Node.js로 사용자와 HTTP로 통신하는 것은 Express나 내장된 http 모듈로 해본 적이 있을 것이다. 하지만 채팅같은 실시간으로 데이터를 주고받아야 하는 서비스는 어떻게 구현해야 할 것인지 막막할 때가 있다. 그래서 이번 포스팅에서는 서버와 클라이언트가 실시간 통신을 할 수 있는 Socket.IO 라이브러리에 대해 알아볼 것이다.

## 프로젝트 세팅

Socket.IO는 기본적으로 HTTP 서버 위에서 돌아가야 한다. 그래서 http나 Express같은 모듈이 필요한데, http 모듈로만 서버를 구축하는 일은 일반적으로 없으니 Express와 Socket.IO를 설치하고 진행하자. 

덤으로 html을 렌더링하려면 ejs 모듈이 필요하다.

~~~
npm i --save express socket.io ejs
~~~

## 간단한 예제

Socket.IO의 통신 방식은 이벤트를 호출하고 수신하면서 그에 따라 작업을 수행하는 방식으로 이루어져 있다.

Socket.IO를 사용하려면 http 서버 객체를 Socket.IO 생성자의 매개변수로 주면 Socket.IO 객체가 하나 생성되는데, 이 객체를 통해 소켓의 연결을 관리하고, 소켓끼리의 이벤트를 정의할 수 있다.

예제는 클라이언트 부분과 서버 부분으로 나뉘어있다. 먼저 서버 코드를 짜보자.

~~~js
var app = require('express')();
var server = require('http').createServer(app);
var io = require('socket.io')(server);

app.set('view engine', 'ejs'); // 렌더링 엔진 모드를 ejs로 설정
app.set('views',  __dirname + '/views');    // ejs이 있는 폴더를 지정

app.get('/', (req, res) => {
    res.render('index');    // index.ejs을 사용자에게 전달
})

io.on('connection', (socket) => {   //연결이 들어오면 실행되는 이벤트
    // socket 변수에는 실행 시점에 연결한 상대와 연결된 소켓의 객체가 들어있다.
    
    //socket.emit으로 현재 연결한 상대에게 신호를 보낼 수 있다.
    socket.emit('usercount', io.engine.clientsCount);

    // on 함수로 이벤트를 정의해 신호를 수신할 수 있다.
    socket.on('message', (msg) => {
        //msg에는 클라이언트에서 전송한 매개변수가 들어온다. 이러한 매개변수의 수에는 제한이 없다.
        console.log('Message received: ' + msg);

        // io.emit으로 연결된 모든 소켓들에 신호를 보낼 수 있다.
        io.emit('message', msg);
    });
});

server.listen(3000, function() {
  console.log('Listening on http://localhost:3000/');
});
~~~

이제 서버 부분이 완성되었으니 클라이언트 코드를 짜보자.

~~~html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Socket.IO 예제</title>
</head>
<body>
    <ul id="messages" type="none">
        <li id="usercount"></li>
    </ul>

    <form id="msgform">
        <input id="msginput" autocomplete="off" type="text">
        <button type="submit">전송</button>
    </form>

    <script src="/socket.io/socket.io.js"></script>
    <script>
        var socket = io();
        var msgform = document.getElementById('msgform');
        // socket.on 함수로 서버에서 전달하는 신호를 수신
        socket.on('usercount', (count) => {
            var userCounter = document.getElementById('usercount');
            userCounter.innerText = "현재 " + count + "명이 서버에 접속해있습니다.";
        });

        // 메시지 수신시 HTML에 메시지 내용 작성
        socket.on('message', (msg) => {
            var messageList = document.getElementById('messages');
            var messageTag = document.createElement("li");
            messageTag.innerText = msg;
            messageList.appendChild(messageTag);
        });

        msgform.onsubmit = (e) => {
            e.preventDefault();
            var msginput = document.getElementById('msginput');

            // socket.emit으로 서버에 신호를 전달
            socket.emit('message', msginput.value);

            msginput.value = "";
        };
    </script>
</body>
</html>
~~~

실행해보면 UI가 많이 허접하긴 해도 잘 작동하는 모습을 볼 수 있다.

![socket](https://imgur.com/AXR2XQT.png)