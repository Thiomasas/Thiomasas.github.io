---
published: true
layout: single
title: "마인크래프트 플러그인 만들기 - 시작"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

![솔크](https://steamuserimages-a.akamaihd.net/ugc/948469551049176371/68310660388C2974738D6F07F2EC81037CE7B06E/?imw=637&imh=358&ima=fit&impolicy=Letterbox&imcolor=%23000000&letterbox=true)

드디어 중요한 일들이 거의 다 끝나고 연말이 다가왔다. 하지만 슬프게도 솔로인 채로 크리스마스를 지내야 하는 사실은 변함이 없었다. 게다가 코로나도 겹쳐 밖에 나가 놀지도 못해 더 슬퍼진다. 그런 슬픔을 달래고자 갓갓 띵작 마인크래프트에서 놀기 위해 서버 플러그인을 만들어보기로 했다.

## 미리 알아둬야 할 것

스피곳 버킷의 플러그인에서 사용할 API가 모두 자바로 짜여져 있고, 자바로만 접근할 수 있기 때문에 어쩔 수 없이 자바를 어떻게 쓰는지 알고 있어야 한다.

## 설치해야 할 것들

개발환경

- IntelliJ IDEA (다른 자바 IDE도 괜찮다) [링크](https://www.jetbrains.com/ko-kr/idea/download/#section=windows)
- 자바 JDK 1.8 이상 [링크](https://www.oracle.com/kr/java/technologies/javase-downloads.html)

테스트 환경

- 마인크래프트 자바 에디션 (작성 기준 1.16.4)
- 스피곳 버킷 (마찬가지로 1.16.4) [링크](https://drive.google.com/file/d/1y9a1KZMHx1XY0lIM-8mFIxmkriQAAgVF/view?usp=sharing)

마인크래프트 클라이언트는 알아서 구하도록 하고(정품 추천), 스피곳 버킷은 위의 링크를 타고 들어가 압축을 푼 다음 start.bat를 실행하면 열린다.

## 프로젝트 준비

IntelliJ IDEA를 켜고 New Project를 선택하면 다음과 같은 창이 뜬다.

![새 프로젝트](https://imgur.com/HRDeIOl.png)

여기서 Gradle을 선택하고 Java를 체크한 다음 Next 버튼을 누른다.

![이름](https://imgur.com/Hlue3Bm.png)

그럼 프로젝트 이름을 정하라고 하는데, 나는 ZizonPlugin으로 하고 Finish를 눌렀다.

![프로젝트 창](https://imgur.com/8rYRNDC.png)

모두 끝나면 이렇게 프로젝트 창이 생긴다.

그런데, 아직 개발할 준비가 되지 않았다. 플러그인을 제작하기 위해선 스피곳 API가 필요한데, 그게 아직 프로젝트에 추가되지 않았다.

build.gradle을 왼쪽의 프로젝트 탐색기에서 열어 다음의 내용으로 교체한다.

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