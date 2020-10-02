---
published: true
layout: single
title: "Inno Setup으로 설치 프로그램 만들기"
classes: wide
sidebar:
    nav: "etc" 
category: etc
tags: 
  - Inno Setup
---

대부분의 앱은 설치하면서 약간의 작업이 따로 필요한 경우가 있다. 하지만 그런 작업을 사용자에게 시키기엔 부담이 있기 때문에, 그런 경우는 간편하게 설치 작업을 수행하는 프로그램을 만들어준다. 이번엔 Inno Setup을 통해 설치 프로그램을 만들어보자.

## Inno Setup 설치

[이 링크](https://jrsoftware.org/isdl.php)에서 Inno Setup 설치 프로그램을 다운받아 설치하도록 하자.

이 포스트 작성 기준으로 최신 버전은 6.0.5이다.

## Inno Setup Compiler로 제작하기

설치가 끝나면 컴퓨터에 Inno Setup Compiler 라는 프로그램이 생겼을 것이다.

이 프로그램으로 Inno Setup 기반 설치 프로그램을 제작할 수 있다.

![welcome](https://imgur.com/6Ghvmpb.png)

실행해보면 위 사진 처럼 파일을 만들거나 여는 창이 나오는데, 프로그램을 새로 만들 것이니까 Create a new script file using the Script Wizard를 체크하고 OK를 눌러준다.

![create](https://imgur.com/F2QdpAK.png)

OK를 누르면 프로그램의 이름과 버전, 회사 이름, 웹사이트 주소를 입력하는 창이 나오는데, 이름이랑 버전 빼고 입력할 필요는 없다. 이름과 버전을 입력하고 Next를 눌러주자.

![folder](https://imgur.com/p6cl1iQ.png)

이 창에서는 기본적으로 설치 프로그램이 설치할 프로그램의 위치를 정한다. 기본값은 Program Files 아래 앱 이름의 폴더가 하나 생성되어 그곳에 설치된다.

바꿀 필요가 없다면 Next를 눌러주자.

![executable](https://imgur.com/sy0ba9O.png)

이 창에서 설치 프로그램이 설치할 파일을을 정의해줘야 한다. 사실 이 작업들 중 제일 중요할지도 모른다.

Application main executable file에는 설치할 파일들 중 실행이 가능한 파일의 경로를 넣어줘야 한다. 

하지만 Node.js같은 플랫폼 위에서 돌아가는 스크립트(Electron같은 패키징된 앱 제외)들을 설치할 것이라면 위의 항목은 무시하고 The application doesn't have a main executable file을 체크해주면 된다.

Other application files에는 설치할 프로그램이 필요로 하는 파일들의 경로를 넣어줘야 한다. Add files를 누르면 파일을 하나하나 포함할 수 있고, Add folder를 누르면 폴더 내 모든 항목을 포함할 수 있다.

파일 정의가 끝났으면 Next 버튼을 눌러주자.

![shortcut](https://imgur.com/QpPgGzI.png)

이 창에서는 설치할 프로그램에 대한 바로가기를 만드는 옵션들을 정의한다. 잘 읽어보고 프로그램에 필요한 바로가기에 대한 옵션을 체크하도록 하자.

바로가기 설정이 끝났다면 Next 버튼을 누른다.

![license](https://imgur.com/vet2sYT.png)

License file은 이용약관 파일의 경로, Information File은 사용자가 설치할 때 보여줘야 하는 파일의 경로를 입력해주면 된다. 세 칸 모두 빈칸으로 둬도 된다.

설정이 끝나면 Next 버튼을 눌러주자.

![mode](https://imgur.com/DHnWXv8.png)

Administrative install mode는 관리자 권한으로 모든 유저를 위해 설치할지, Non administrative install mode는 일반 유저 권한으로 설치 프로그램을 실행한 유저만 설치할지 정할 수 있다.

Ask the user to choose the install mode at startup을 체크하면 설치 프로그램이 시작될 때 따로 물어보고, 유저의 선택에 따라 달라진다.

![langs](https://imgur.com/SPl9mEm.png)

설치 프로그램이 사용할 언어를 정할 수 있다.

참고로 한국어파일은 내장되어 있진 않지만 [이 링크](https://github.com/jrsoftware/issrc/blob/main/Files/Languages/Unofficial/Korean.isl)에서 한국어 번역 파일을 구할 수 있다. 번역 파일은 Inno Setup 설치 경로 안의 Languages 폴더 안에 넣으면 인식된다.

사용할 언어를 체크하고 넘어가자.

![compiler](https://imgur.com/lR8dRaU.png)

이 창에서는 컴파일러에 대한 설정을 할 수 있다.

Custom compiler output folder는 설치 프로그램이 생성될 위치를 정할 수 있다.

Compiler output base file name은 설치 프로그램이 생성될 때 붙여질 이름이다.

Custom Setup icon file은 설치 프로그램의 아이콘을 정할 수 있다.

Setup Password는 설치 프로그램을 실행할 때 입력해야 하는 비밀번호를 설정할 수 있다. 빈칸으로 두면 비밀번호를 묻지 않고 설치를 진행한다.

![preprocessor](https://imgur.com/kIBxAbQ.png)

마지막 창으로, 스크립트에 #define으로 된 전처리기를 사용하겠냐는 질문이다. 코드 수정이 쉬워지기 때문에 개인적으로 체크해놓지만, 체크를 안해도 된다.

이렇게 파일이 만들어지면, 처음 시작할 때 컴파일해보겠냐는 질문이 나온다. 컴파일해보면 다음과 같이 파일이 만들어지는 것을 볼 수 있다.

![output](https://imgur.com/aefNFn6.png)
