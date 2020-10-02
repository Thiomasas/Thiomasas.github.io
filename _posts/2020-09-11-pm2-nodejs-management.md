---
published: true
layout: single
title: "pm2로 Node.js 서버 관리하기"
classes: wide
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js
  - pm2
---

Node.js 기반 서버를 ssh로 접속한 컴퓨터에서 구동하려면 nohup, pm2 등 여러 방법이 있다. 오늘은 pm2로 원격 컴퓨터에서 Node.js 서버를 구동해볼 것이다.

## 준비물

- NPM이 포함된 Node.js 환경

원격 컴퓨터에서 작업해도 되고 로컬 컴퓨터에서 작업해도 상관없다. pm2는 사용 가능한 Node.js 환경만 있으면 동작한다.

## pm2 설치하기

터미널을 열어서 다음 명령어를 입력해서 pm2를 설치한다.

~~~
npm install -g pm2
~~~

## 사용하기

pm2에서 서버를 실행하는 모드는 fork 모드와 cluster 모드로 나뉘는데, fork 모드는 일반적인 Node.js의 특징인 싱글 스레드로 실행되어 서버의 사양에 따라 최대 성능을 내지 못할 수 있고, cluster 모드는 여러 코어를 활용해 사용할 수 있는 자원을 최대로 활용할 수 있다는 특징이 있다.

fork 모드로 실행하려면 다음 명령어를 입력하면 된다.

~~~sh
pm2 start <실행할 코드 파일>
~~~

cluster 모드로 실행하려면 다음 명령어를 입력하면 된다.

프로세스 갯수에는 max 키워드로 사용 가능한 최대 프로세스 갯수를 할당할 수도 있다.

~~~sh
pm2 start <실행할 코드 파일> -i <프로세스 갯수>
~~~

어느쪽이든 pm2 list를 입력했을 떄 아래 사진처럼 나오면 프로세스가 실행중이라는 뜻이다.

![pm2_list](https://i.imgur.com/oEBONT7.png)

pm2 monit을 입력하면 아래 사진처럼 현재 실행중인 프로세스들의 로그와 업타임 등 여러 정보를 볼 수 있다.

![pm2_monit](https://i.imgur.com/xo0LDb7.png)

pm2의 실행중인 프로세스에 적용 가능한 명령어는 4가지다.

~~~sh
pm2 restart <앱 이름>   # 프로세스 재시작(클러스터 선택시 모두 프로세스 종료 후 재시작)
pm2 reload <앱 이름>    # 프로세스 재시작(클러스터 선택시 최소 프로세스 하나를 남겨서 다운되지 않게 함)
pm2 stop <앱 이름>      # 프로세스 정지
pm2 delete <앱 이름>    # 프로세스 제거
~~~

위의 명령어에서 앱 이름 대신 프로세스의 id로 특정하거나, 또는 all 키워드로 모두 선택할 수 있다.

## 자동 시작 설정

컴퓨터를 재부팅할 때마다 일일이 pm2의 세팅을 다시 해주는 것이 귀찮다면 pm2의 상태를 저장한 후 컴퓨터 부팅 시 pm2가 시작하게 세팅하는 방법도 있다. 

### 유닉스 계열에 설정

pm2의 공식 문서에 따르면 유닉스 계열의 경우 다음의 프로세스 초기화 시스템이 설정되어 있어야 한다.

- systemd: Ubuntu >= 16, CentOS >= 7, Arch, Debian >= 7
- upstart: Ubuntu <= 14
- launchd: Darwin, MacOSx
- openrc: Gentoo Linux, Arch Linux
- rcd: FreeBSD
- systemv: Centos 6, Amazon Linux

만약 설정되어 있다면 다음 두 명령어로 세팅을 완료할 수 있다.

~~~sh
pm2 startup # 이 명령어로 설정된 프로세스 초기화 시스템을 자동 탐지해서 시작 스크립트를 생성한다.
pm2 save    # 현재 실행중인 프로세스 목록을 저장해서 다음 부팅에 사용할 수 있게 만든다.
~~~

### 윈도우에서 설정

명령어 3줄로 세팅을 완료할 수 있다.

~~~sh
npm install pm2-windows-startup -g  # npm에서 시작 스크립트 다운로드
pm2-startup install                 # 시작 스크립트 설치
pm2 save    # 현재 실행중인 프로세스 목록을 저장해서 다음 부팅에 사용할 수 있게 만든다.
~~~