---
published: true
layout: single
title: "Nginx로 Reverse Proxy 설정하기"
classes: wide
category: etc
sidebar:
    nav: "etc" 
tags: 
  - nginx
---

Node.js로 HTTP 서버 프로젝트를 개발하고 실제 서버에 배포할 땐 Apache나 Nginx 같은 프록시 서버도 같이 이용하는 것을 권장한다. 이유는 보안상의 이점, 캐시 기능으로 리소스 낭비 방지 등이 있다. 이 포스트에서는 가볍고 성능이 뛰어난 Nginx 프록시 서버를 사용해 볼 것이다.

## 구축 환경

- Ubuntu 20.04
- [간단한 Express 기반 HTTP 서버](https://github.com/fred16157/express-sample) (HTTP 서버라면 아무거나 상관 X)

위의 링크는 예시를 위해 만든 간단한 서버다. 다른 HTTP 서버로 대체해도 된다.

## Nginx 설치

Nginx는 Ubuntu 20.04 버전 기준으로 다음 명령어로 설치할 수 있다.

~~~bash
sudo apt install nginx
~~~

설치가 끝났다면 다음 명령어로 제대로 설치되었는지 확인할 수 있다.

~~~bash
sudo systemctl status nginx
~~~

아래처럼 메시지가 떴을 때, 왼쪽 위에 초록불이 들어온다면 정상이고 잘 돌아간다는 뜻이다.

![nginx 테스트](https://imgur.com/CKriMNp.png)

## Nginx 설정하기

Ubuntu 20.04 기준으로 Nginx가 설치되면 /etc/nginx/conf.d/ 폴더 속의 .conf 파일들로 설정을 저장할 수 있다.

여기서 아무 이름으로 .conf 파일을 하나 만들어서 설정을 해보자. (나는 proxy.conf로 만들었다)

~~~nginx
server {
  # 80번 포트로 nginx가 요청을 수신
  listen 80;
  listen [::]:80;

  # / 경로로 요청이 들어왔을 때 (하위 경로로 들어왔을 때도 포함)
  location / {
    # proxy_pass (실제 http 서버가 열려있는 url); 으로 프록시 설정이 끝난다.
    proxy_pass http://127.0.0.1:3000; 
  }
}
~~~

그런 다음 Nginx를 재시작해서 제대로 돌아가는지 확인해보자.

~~~bash
sudo systemctl restart nginx && sudo systemctl status nginx
~~~

## HTTP 서버 열기 (Node.js)

이제 프록시 설정을 했으니 Node.js 서버를 열어줄 차례다. 이미 서버가 열려있다면 건너뛰어도 된다.

Node.js 서버를 계속 열린 상태로 두기 위해 pm2로 서버를 열 예정인데, pm2에 대해서는 [이 포스트](https://fred16157.github.io/node.js/pm2-nodejs-management/)에서 다루었으니 자세하게 알고 싶으면 읽어보자.

서버 코드가 있는 폴더에서 다음 명령어로 실행하면 pm2로 Node.js 서버가 열리게 된다.

~~~bash
pm2 start (서버 진입점 파일)
~~~

Node.js 서버까지 열어주면 80번 포트로 접속했을 때 페이지가 뜨는것을 볼 수 있다.

![완성](https://imgur.com/Vha6wmK.png)