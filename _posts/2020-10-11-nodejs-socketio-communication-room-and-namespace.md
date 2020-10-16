---
published: true
layout: single
title: "Node.js 서버에서 Socket.IO로 실시간 통신하기 - 룸과 네임스페이스"
classes: wide   
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js 
  - socket.io
---

스마트폰이 보급된 이후로 카카오톡을 일상에서 많이 사용하는데, 만약 방이라는 개념이 없고 카카오톡을 사용하는 모든 사용자들에게만 보낼 수 있다면 매우 불편하지 않을까? 그래서 이번엔 Socket.IO의 룸과 네임스페이스에 대해 알아볼 것이다.

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/node.js/nodejs-socketio-communication-basic/) 부터 읽어주세요.

## 네임스페이스

네임스페이스란 Express의 라우팅처럼 url에 지정된 위치에 따라 신호의 처리를 다르게 하는 기술이다.

지금까지는 기본 네임스페이스인 / 에 신호를 전송하고 수신했지만, 다른 네임스페이스를 만들어서 신호를 처리할 수도 있다.

디버깅 신호를 처리하는 네임스페이스를 만들어서 연습을 해보겠다.

먼저 서버에서 디버그 신호를 네임스페이스를 만들어야 한다.

~~~js
// 디버그 신호를 주고받는 네임스페이스
const debug = io.of('/debug');

// 네임스페이스의 연결 처리는 제각각이다. 그러므로 연결 콜백을 다시 만들어야 한다.
debug.on('connection', (socket) => {
  // 룸의 목록 요청시 / 네임스페이스의 룸 목록 반환
  socket.on('getRooms', () => { 
    // 다른 네임스페이스의 객체에도 접근할 수 있다.
    socket.emit('rooms', io.sockets.adapter.rooms);
  });
});
~~~

HTML에서는 네임스페이스를 지정해야 하는 것 말고는 일반적으로 소켓을 사용하는 것처럼 쓰면 된다.

~~~diff
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

+    <button onclick="getRooms()">방 목록 가져오기</button>
+
+    <p id="rooms"></p>

    <script src="/socket.io/socket.io.js"></script>
    <script>
        var socket = io();
        var msgform = document.getElementById('msgform');
+       var roomsText = document.getElementById('rooms');
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

+       function getRooms() { // 방 목록 가져오기 버튼 클릭시
+           // url을 지정해서 특정 네임스페이스를 들어갈 수 있다.
+           var debug = io.connect('http://localhost:3000/debug');
+
+           debug.emit('getRooms');  // getRooms 이벤트 호출
+
+           debug.on('rooms', (rooms) => { // rooms 이벤트 발생
+               // 룸 목록 업데이트
+               roomsText.textContent = "";
+               for (var room in rooms) {
+                   roomsText.innerHTML += room + "<br>";
+               }
+           });
+       }
    </script>
</body>

</html>
~~~

서버를 실행하고 테스트 해보았을 때 디버그 네임스페이스에 접속해서 정상적으로 룸의 목록을 가져오는 모습을 볼 수 있다.

![namespace](https://imgur.com/Ay3nE00.gif)

## 룸

소켓들은 룸이라는 공간을 활용해 구분할 수 있다. 예를 들어 카톡 단톡방 두개가 있다고 치면 단톡방1에 메시지를 보내면 단톡방1의 사용자들은 메시지를 받지만 다른 단톡방의 사용자들은 그 메시지를 받지 못한다.

소켓들의 룸도 단톡방과 비슷하다. 특정 룸에 신호를 보내면 룸 안의 소켓들은 신호를 받지만, 다른 룸에 소속된 소켓들은 신호를 받지 못한다.

참고로 모든 소켓들은 기본적으로 소켓의 고유 ID가 이름인 룸에 들어가게 되고, 소켓들은 여러 룸을 들어가 있을 수 있다.

채팅방 두개를 만들어서 따로 채팅을 할 수 있게 구현해보자.

~~~diff
io.on('connection', (socket) => {   //연결이 들어오면 실행되는 이벤트
    // socket 변수에는 실행 시점에 연결한 상대와 연결된 소켓의 객체가 들어있다.
    
    //socket.emit으로 현재 연결한 상대에게 신호를 보낼 수 있다.
    socket.emit('usercount', io.engine.clientsCount);

+   //기본적으로 채팅방 하나에 접속시켜준다.
+   socket.join("채팅방 1");

    // on 함수로 이벤트를 정의해 신호를 수신할 수 있다.
-   socket.on('message', (msg) => {
+   socket.on('message', (msg, roomname) => {
        //msg에는 클라이언트에서 전송한 매개변수가 들어온다. 이러한 매개변수의 수에는 제한이 없다.
        console.log('Message received: ' + msg);

        // io.emit으로 연결된 모든 소켓들에 신호를 보낼 수 있다.
-       io.emit('message', msg);
+       // io.to(방이름).emit으로 특정 방의 소켓들에게 신호를 보낼 수 있다.
+       io.to(roomname).emit('message', msg);
    });

+   // 룸 전환 신호
+   socket.on('joinRoom', (roomname, roomToJoin) => {
+     socket.leave(roomname); // 기존의 룸을 나가고
+     socket.join(roomToJoin);  // 들어갈 룸에 들어간다.
+
+     // 룸을 성공적으로 전환했다는 신호 발송
+     socket.emit('roomChanged', roomToJoin);
+   });
});
~~~

~~~diff

<body>
+   <select id="roomoptions" onchange="joinRoom()">
+       <option value="채팅방 1" selected>채팅방 1</option>
+       <option value="채팅방 2">채팅방 2</option>
+   </select>
    <ul id="messages" type="none">
        <li id="usercount"></li>
    </ul>

    <form id="msgform">
        <input id="msginput" autocomplete="off" type="text">
        <button type="submit">전송</button>
    </form>

    <button onclick="enableDebug()">방 목록 가져오기</button>

    <p id="rooms"></p>

    <script src="/socket.io/socket.io.js"></script>
    <script>
+       var roomname = "채팅방 1"
        var socket = io();
        var msgform = document.getElementById('msgform');
        var getUserForm = document.getElementById('getUserForm');
        var roomsText = document.getElementById('rooms');
        var usersText = document.getElementById('users');
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

+       // 접속한 룸이 바뀌었을 때
+       socket.on('roomChanged', (joinedRoom) => { 
+           roomname = joinedRoom;
+           var messageList = document.getElementById('messages');
+           var messageTag = document.createElement("li");
+           messageTag.innerText = joinedRoom + "에 접속했습니다.";
+           messageList.appendChild(messageTag);
+       });

        msgform.onsubmit = (e) => {
            e.preventDefault();
            var msginput = document.getElementById('msginput');

            // socket.emit으로 서버에 신호를 전달
            // 특정 룸에 메시지를 보내기 위해 룸의 이름을 같이 전송
            socket.emit('message', msginput.value, roomname);

            msginput.value = "";
        };

        function enableDebug() { // 방 목록 가져오기 버튼 클릭시
            // url을 지정해서 특정 네임스페이스를 들어갈 수 있다.
            var debug = io.connect('http://localhost:3000/debug');

            debug.emit('getRooms'); // getRooms 이벤트 호출

            debug.on('rooms', (rooms) => {  // rooms 이벤트 발생
                // 룸 목록 업데이트
                roomsText.textContent = "";
                for (var room in rooms) {
                    roomsText.innerHTML += room + "<br>";
                }
            });
        }

+       function joinRoom() { // 방 접속 버튼 클릭시
+           var roomOptions = document.getElementById("roomoptions");
+           var roomToJoin = roomOptions.options[roomOptions.selectedIndex].value;
+
+           // 서버에 룸 전환 신호를 발신
+           socket.emit('joinRoom', roomname, roomToJoin);
+       }
    </script>
</body>
~~~

테스트해보면 방을 오가면서 채팅을 보낼 수 있게 된다.

![rooms](https://imgur.com/4wAj54g.gif)