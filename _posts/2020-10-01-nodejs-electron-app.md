---
published: true
layout: single
title: "Electron으로 Node.js 데스크탑 앱 제작하기"
classes: wide
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js
  - electron 
---

Electron은 자바스크립트를 통해 데스크톱 앱을 제작하기 위해 만들어진 프레임워크이다. Chrome 브라우저의 오픈소스 버전인 Chromium을 통해 웹 페이지 기반 렌더링을 지원하고, 크로스 플랫폼으로 개발할 수 있다는 장점이 있다. 이번 포스트에서는 Electron으로 데스크탑 앱을 제작해 볼 것이다.

## Electron 프로젝트 생성

[이 링크](https://www.electronjs.org/community#boilerplates)로 들어가보면 Electron 프로젝트에 관한 프로젝트들이 많은데, 그 중 Boilerplate는 우리가 Electron 앱 개발을 바로 시작할 수 있게 도와주는 프로젝트들이다. 

하지만 저 링크에 있는건 참고용이나 필요한 툴을 찾는 용도로 사용하고, 이 포스트에선 [electron-forge](https://www.electronforge.io/) 라는 것을 사용해볼 것이다.

다음 명령어로 electron-forge 기반 프로젝트를 생성할 수 있다.

~~~sh
# yarn
yarn create electron-app 앱-이름

# npm
npx create-electron-app 앱-이름
~~~

명령어 실행이 끝나면 앱 이름으로 된 폴더가 하나 생성되었을 것이다.

생성된 폴더에서 npm start 또는 yarn start를 입력하면 예제 프로젝트가 빌드되어 실행된다.

![example](https://imgur.com/WpwpzYs.png)

## 프로젝트 구조

Electron 프레임워크는 웹 기반 렌더링을 지원한다. 그렇기 때문에 프로젝트 속 HTML, CSS, JS 파일들로 앱을 제작할 수 있게 설계되어 있다.

위에서 만든 프로젝트 속에는 src 폴더가 있는데, 핵심적인 파일이 3개 있다.

- index.js
- index.html
- index.css

index.js 파일은 Electron 앱의 시작점이다. 이 파일 속에 Electron 앱이 시작하면서 html을 렌더링하도록 이미 electron-forge가 코드를 작성해놓았기 때문에 미리 작성된 코드는 Electron API에 대해 숙지한 후에 수정하도록 하고, 파일의 맨 끝에서부터 코드를 작성하도록 하자.

기본적인 웹에서 사용되는 바닐라 JS와는 다른 점이 있다면, Chromium이지만 Node.js 위에서 돌아가기 때문에 Electron과 Node.js API에 직접 접근할 수 있다는 점이다.

index.html와 index.css는 Electron 앱에서 화면을 표시할 떄 사용된다. 작성 방법은 일반적인 웹에서 사용하는 것 처럼 작성하면 된다. 

참고로 index.html의 script 태그 속의 스크립트들도 Electron과 Node.js API에 접근할 수 있다.

## 라우팅

Electron 내 html과 css, js는 모두 정적 파일로 취급되기 때문에 상대적인 파일 경로를 직접 지정해주면 바로 접근할 수 있다.

예를 들어 home.html이라는 html 파일을 하나 만든 다음 index.html의 a 태그에 링크한다면 a 태그를 클릭했을 때 home.html로 이동된다.

~~~html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    <link rel="stylesheet" href="index.css">
  </head>
  <body>
    <h1>💖 Hello World!</h1>
    <p>Welcome to your Electron application.</p>

    <p>이 페이지는 index.html 입니다.</p>

    <a href="home.html">home.html로 가기</a>
  </body>
</html>
~~~

![routing](https://imgur.com/22jtsIQ.gif)

## 패키징하기

Electron에는 [electron-builder](https://github.com/electron-userland/electron-builder)같은 패키징 툴이 여러가지 있다. 그렇지만 electron-forge로 프로젝트를 생성했다면 빌드 수단이 내장되어 있기 때문에 이 포스트에서는 그것을 사용할 것이다.

이 명령어를 사용하면 우리가 제작한 앱의 패키지를 생성할 수 있다.

~~~sh
# yarn
yarn make

# npm
npm run make
~~~

명령어 실행이 끝나면 out 폴더 속에 패키징된 앱이 생성된다.