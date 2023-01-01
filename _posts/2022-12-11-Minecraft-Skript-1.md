---
published: true
layout: single
title: ""
classes: wide
category: Skript
sidebar:
    nav: "skript" 
tags: 
  - skript
---

드디어 중요한 일들이 거의 다 끝나고 연말이 다가왔다. 맞다. 우리는 추운 겨울 집안에서 귤을 맛있게 먹으며 놀 것이 필요하다. 우리들의 게임하면 무엇이 떠오르는 가. 바로 마인크래프트이다. 그래서 마인크래프트 서버를 열어 친구들과 놀기 위해 스크립트를 배워 스크립트를 만들어 보고자 한다.

![코딩](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTkQGMJ2O19q-fq4kI7U56TYJRDT9TSXnbl5w&usqp=CAU)

## 미리 알아둬야 할 것

스피곳 버킷의 플러그인에서 사용할 API가 모두 자바로 짜여져 있고, 자바로만 접근할 수 있기 때문에 어쩔 수 없이 자바를 어떻게 쓰는지 알고 있어야 한다. 또한 코딩의 기초적인 매커니즘과 알고리즘을 알고 있다면, 스크립트를 배우고 만드는 데에 조금더 큰 도움이 될 것이다.

## 설치해야 할 것들

개발환경

- IntelliJ IDEA (선택) [링크](https://www.jetbrains.com/ko-kr/idea/download/#section=windows)
- 자바 JDK 1.8 이상 (필수) [링크](https://www.oracle.com/kr/java/technologies/javase-downloads.html)
- 스크립트 플러그인 [링크](https://skunity.com/downloads)
- VS 코드 개발툴 (추천) [링크](https://code.visualstudio.com/download)

테스트 환경

- 마인크래프트 자바 에디션 (작성 기준 1.19.2)
- 퍼퍼 버킷 (버전 1.19.2) [링크](https://api.purpurmc.org/v2/purpur/1.19.2/latest/download)

마인크래프트 클라이언트는 알아서 구하도록 하고(정품 추천), 퍼퍼 버킷은 위의 링크를 타고 들어가 압축을 푼 다음 __start.bat__를 실행하면 열린다.

## 프로젝트 준비

일단 위에서 다운로드 받은 서버 파일에 `start.bat` 파일을 실행시켜 서버를 켜준다. 또한 서버를 킬때 `eula.txt` 의 맨 아랫줄, eula=false 의 값을 true 로 바꾸어 준다. 
그런다면 서버의 준비는 완료 되었고, 위에 첨부한 스크립트 플러그인 사이트에서 __가장최신버전__ 의 스크립트를 다운로드 받아, `plugins` 폴더에 넣어준다. 

![서버폴더](https://t1.daumcdn.net/cfile/tistory/2117B53C51A73D062E)

그렇게 했다면, 다시 한번 `start` 파일을 실행시킨다. 그렇다면 모든 준비는 끝났다. 본 포스팅은 스크립트의 중점을 두었기에, 자세한 서버 열기 방법은 다른 강좌를 찾아보길 바란다.

## 프로젝트 생성

자 이제 아래의 경로의 디렉토리로 들어가준다. 앞으로 많이 들어갈 폴더이다.
~~~
서버폴더/plugins/Skript/scripts
~~~

이제 위의 폴더에 `파일명.sk` 의 파일을 만들자. 그런다음 그 폴더를 클릭한채, 다시 우클릭을 해 유틸리티 툴이 뜨게하고, h키를 눌러 연결프로그램을 선택해주자. 그리고 연결프로그램을 위에서 다운로드 받은 Intelij IDE 또는 VS 코드로 설정하여 조금 더 편한 개발환경을 구축하자.

여기까지 잘 따라왔다면 스크립트 개발자가 되기 위한 모든 준비는 끝났다.

## 마치며..

이 포스트에선 마인크래프트 스크립트를 통해 놀기위한 첫 관문인, 스크립트 설치 및 개발환경 구축에 대해 다루었다. 다음강의 부턴 조금더 고급적인 예제 및 본격적 강좌에 들어갈 예정이다.
