---
published: true
layout: single
title: "리눅스에서 systemd로 Node.js 프로젝트를 서비스로 등록하기"
classes: wide
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js
  - linux
  - systemd
---

학교 동아리 서버컴에 PM2를 깔고 여러 프로젝트를 돌리다보니 최근에 크게 데인것도 있고 불안정할 때가 많아져서 대체할 방법을 찾던 도중, 스택 오버플로우([링크](https://stackoverflow.com/questions/4018154/how-do-i-run-a-node-js-app-as-a-background-service))에서 아주 좋은 해결책을 발견했다. 이번 포스트는 그래서 systemd를 활용해 프로젝트를 서비스로 등록하는 방법을 다뤄볼 것이다.

## 테스트 환경

- Ubuntu 20.04
- Node.js 14
- Node.js 예제 프로젝트([링크](https://github.com/fred16157/express-sample))

Ubuntu가 아니더라도 systemd가 깔려있는 운영체제라면 무관하다.

## 서비스 파일 작성

(서비스이름).service 파일을 하나 만들어두고 아래 양식을 참고하여 서비스의 내용을 작성하자.

~~~conf
[Unit]
Description=(서비스 설명)

[Service]
ExecStart=(실행할 js 파일의 경로)
Restart=always
# RHEL/Fedora 계열은 Group=nogroup 대신 User=nobody를 사용한다고 한다
Group=nogroup
# 환경변수를 Environment=변수이름=변수값 처럼 정의할 수 있다
WorkingDirectory=(작업 환경 경로)

[Install]
WantedBy=multi-user.target
~~~

참고로 작성자는 sampleproject.service 파일을 만들고 아래처럼 작성했다.

~~~conf
[Unit]
Description=A simple express sample project

[Service]
ExecStart=/home/user/express-sample/index.js
Restart=always
Group=nogroup
WorkingDirectory=/home/user/express-sample/

[Install]
WantedBy=multi-user.target
~~~

## 실행할 js 파일 수정

실행할 js 파일은 맨 첫줄에 Node.js의 경로를 명시해두어야 한다. 

작성자는 경로가 /home/user/.nvm/versions/node/v14.15.3/bin/node인데, js 파일에 적용하려면

~~~
#!/home/user/.nvm/versions/node/v14.15.3/bin/node
~~~

처럼 작성하면 된다.

혹시 어디 설치되어 있는지 모르겠다면 which node 명령어를 사용해보자.

그리고 실행할 js 파일은 chmod +x 을 적용해주어야 한다.

~~~
chmod +x (실행할 js 파일)
~~~

## 심볼릭 링크 생성

이제 이 서비스 파일을 가지고 서비스를 등록하면 된다. 

등록하는 방법에는 두가지가 있는데, 하나는 /etc/systemd/system 폴더에 복붙을 하는 것이고, 하나는 심볼릭 링크를 생성하는 것이다.

심볼릭 링크를 생성하면 수정할 때마다 번거롭게 복붙을 하지 않아도 수정사항이 반영되기 때문에 심볼릭 링크를 사용할 것이다.

~~~
systemctl link (서비스파일 경로)
~~~

작성자의 경우는 서비스파일이 있는 경로에서 다음 명령어를 입력했다.

~~~
systemctl link /home/user/sampleproject.service
~~~

## 서비스 실행하기

서비스를 시작하려면 다음 명령어를 입력하면 된다.

~~~
systemctl start (서비스 이름)
~~~

껐다 켜질때도 자동으로 시작하도록 설정하고 싶다면 enable 명령어를 사용하면 된다.

~~~
systemctl enable (서비스 이름)
~~~

마지막으로, 로그는 journalctl으로 볼 수 있다.

~~~
journalctl -u (서비스 이름)
~~~