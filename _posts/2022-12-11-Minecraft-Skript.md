---
published: true
layout: single
title: "마인크래프트 스크립트 개발 - 시작 및 튜토리얼"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

드디어 중요한 일들이 거의 다 끝나고 연말이 다가왔다. 맞다. 우리는 추운 겨울 집안에서 귤을 맛있게 먹으며 놀 것이 필요하다. 우리들의 게임하면 무엇이 떠오르는 가. 바로 마인크래프트이다. 그래서 마인크래프트 서버를 열어 친구들과 놀기 위해 스크립트를 배워 스크립트를 만들어 보고자 한다.

## 미리 알아둬야 할 것

스피곳 버킷의 플러그인에서 사용할 API가 모두 자바로 짜여져 있고, 자바로만 접근할 수 있기 때문에 어쩔 수 없이 자바를 어떻게 쓰는지 알고 있어야 한다. 또한 코딩의 기초적인 매커니즘과 알고리즘을 알고 있다면, 스크립트를 배우고 만드는 데에 조금더 큰 도움이 될 것이다.

## 설치해야 할 것들

개발환경

- IntelliJ IDEA (선택) [링크](https://www.jetbrains.com/ko-kr/idea/download/#section=windows)
- 자바 JDK 1.8 이상 (필수) [링크](https://www.oracle.com/kr/java/technologies/javase-downloads.html)
- 스크립트 플러그인 [링크](https://skunity.com/downloads)

테스트 환경

- 마인크래프트 자바 에디션 (작성 기준 1.16.4)
- 스피곳 버킷 (마찬가지로 1.16.4) [링크](https://drive.google.com/file/d/1y9a1KZMHx1XY0lIM-8mFIxmkriQAAgVF/view?usp=sharing)

마인크래프트 클라이언트는 알아서 구하도록 하고(정품 추천), 스피곳 버킷은 위의 링크를 타고 들어가 압축을 푼 다음 start.bat를 실행하면 열린다.

## 프로젝트 준비

일단 위에서 다운로드 받은 서버 파일에 " start.bat " 파일을 실행시켜 서버를 켜준다. 또한 서버를 킬때 " eula.txt " 의 맨 아랫줄, eula=false 의 값을 true 로 바꾸어 준다. 그런다면 서버의 준비는 완료 되었고, 위에 첨부한 스크립트 플러그인 사이트에서 "가장 최신버전" 의 스크립트를 다운로드 받아, " plugins " 폴더에 넣어준다. 그렇게 했다면, 다시 한번 " start.bat " 파일을 실행시킨다. 그렇다면 모든 준비는 끝났다.

~~~gradle
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT' 

repositories {
    mavenCentral()
    maven {
        //API를 어디서 받을지 선언해준다.
        url "https://hub.spigotmc.org/nexus/content/repositories/snapshots/"
    }
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    implementation "org.spigotmc:spigot-api:1.16.4-R0.1-SNAPSHOT"   //스피곳 API
}
~~~

끝났으면 Ctrl + F9를 눌러 빌드해보자. 빨간줄(에러)이 뜨지 않았으면 성공이다.

그리고 이제 이 파일이 플러그인이고 어떤 플러그인인지 소개해주는 파일인 plugin.yml이 필요하다. 이 파일이 없으면 스피곳 버킷은 절대로 이 파일을 플러그인으로 인식하지 않는다.

![경로](https://imgur.com/iL6nHWW.png)

프로젝트의 src -> main -> resources 폴더에 plugin.yml이라는 파일을 하나 생성하자.

열어보면 내용물이 없을텐데, 우리가 채워넣어줘야 한다.

~~~yml
name: ZizonPlugin   #플러그인 이름
version: 1.0        #플러그인 버전
main: com.example.ZizonPlugin.ZizonPlugin   #플러그인 패키지 이름.진입점 클래스 이름
~~~

오른쪽의 설명을 보고 자신의 프로젝트에 맞게 적어넣도록 하자. main은 뭘 적을지 모르겠다면 
~~~
com.example.플러그인이름.플러그인이름
~~~ 
과 같이 적어주면 된다.

그리고 이제 개발할 준비는 끝났다.

## 진입점 만들기

위의 plugin.yml을 보면 진입점 클래스라는 개념이 나온다. 자바를 다뤄봤다면 알겠지만 모든 자바 프로그램은 main과 같은 진입점이 필요하다. 플러그인도 예외는 아니다. 그래서 이번에는 플러그인의 진입점 클래스를 만들 것이다.

![패키지 생성](https://imgur.com/fQUT3Ku.png)

일단 패키지가 필요하다. 위에서 적어준 패키지 이름으로 하나 만들어주자.

![클래스 생성](https://imgur.com/x8LuJFV.png)

그리고 자바 클래스 하나를 생성해야 한다. 위의 plugin.yml에서 선언한 진입점 클래스 이름으로 만들어주면 된다.

진입점 클래스는 반드시 JavaPlugin이라는 추상 클래스를 상속해야 한다. 그렇게 되면 이 진입점 클래스에서 버킷과 상호작용을 할 수 있게 되고, 우리가 구현하고 싶은 기능을 만들 수 있게 된다.

일단 간단하게 플러그인이 활성화되고 비활성화될 때 로그를 남기는 코드를 짜보자.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.plugin.java.JavaPlugin;

public class ZizonPlugin extends JavaPlugin {
    @Override
    public void onEnable() {    //플러그인 활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 활성화되었습니다!");  //서버의 로그에 출력
    }
    @Override
    public void onDisable() {   //플러그인 비활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 비활성화되었습니다."); //서버의 로그에 출력
    }
}
~~~

위의 코드에서 클래스 이름과 패키지 이름만 바꾸면 작동할 것이다.

## Jar 파일로 만들기

이제 개발은 끝났고, 플러그인을 Jar 파일로 만들 차례이다.

오른쪽의 Gradle 바를 클릭해보자.
![여기 숨어있다](https://imgur.com/fWiBW4d.png)

클릭하면 우리가 수행할 수 있는 작업들이 표시되는데, 우리는 Jar 파일로 만들어야 하니 jar를 더블클릭한다.

![jar 빌드](https://imgur.com/NG0y6rO.png)

그리고 프로젝트 경로 -> build -> libs 폴더에 가보면 플러그인 Jar가 생성된 것을 볼 수 있다.

![생성된 Jar](https://imgur.com/qGMD3jZ.png)
## 테스트하기

이제 플러그인 파일도 있으니 테스트를 할 차례이다.

![플러그인 폴더](https://imgur.com/zCyjc6Z.png)

스피곳 버킷의 경로 -> plugin 폴더(없다면 start.bat 실행했을 때 만들어짐) 안에 플러그인 Jar를 넣고, start.bat를 실행해보자.

![start.bat](https://imgur.com/lWMTkzk.png)

저 로그가 보인다면 성공적으로 플러그인이 적용된 것이다.
아직은 실질적인 기능따윈 없지만 다음 포스트부터는 플러그인에 기능을 추가해나가는 방법에 대해 작성해 볼 것이다.
