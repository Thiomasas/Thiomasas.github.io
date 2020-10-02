---
published: true
layout: single
title: ".NET 프레임워크 프로젝트에서 .NET 코어 프로젝트로 전환하기"
classes: wide
category: .NET
sidebar:
    nav: "dotnet" 
tags: 
  - .net
---

.NET 프레임워크 프로젝트 에서 .NET 코어 프로젝트로 전환하는 작업을 해볼 것이다.

## 왜 굳이?

잘쓰던 .NET 프레임워크에서 .NET 코어로 바꾸는건 크게 두가지 목적이 있다.

- [ML.NET](https://dotnet.microsoft.com/apps/machinelearning-ai/ml-dotnet) 등 .NET 코어가 필수적인 라이브러리를 사용하기 위해
- 윈도우 뿐만 아니라 리눅스와 맥에서 사용할 수 있게 하기 위해

요즘 마소가 오픈소스 친화적인 모습을 많이 보여주면서 .NET 프레임워크를 버리고 크로스 플랫폼이면서 오픈소스인 .NET 코어에 많이 투자하는 모습도 무시못하는 이유들 중 하나이다.

## 필요한 것

마소 공식문서의 .NET 프레임워크에서 .NET 코어로 포팅하는 방법을 알려주는 [포스팅](https://docs.microsoft.com/ko-kr/dotnet/core/porting/)에서는 [try-convert](https://github.com/dotnet/try-convert)라는 비공식 툴을 사용하는 방법과 비주얼 스튜디오 2019에 [.NET Portability Analyzer](https://docs.microsoft.com/ko-kr/dotnet/standard/analyzers/portability-analyzer) 라는 확장을 사용해서 포팅하는 방법을 알려준다. 이 포스팅에서는 작업이 간단한 편인 try-convert 툴을 사용하는 방법을 쓸 것이다.

그래서 필요한 건 다음과 같다.

- 윈도우가 깔린 컴퓨터
- .NET 코어 SDK [설치 링크](https://dotnet.microsoft.com/download)
- 바꿀 .NET 프레임워크 프로젝트 (또는 프로젝트를 포함하는 .sln 파일)

모두 준비되어 있으면 powershell이나 cmd를 켜서 

~~~
dotnet tool install -g try-convert
~~~

를 입력해서 try-convert를 설치한다.

## 전환하기

일단 이 툴으로 완전히 작동되는 프로젝트를 뽑아줄 것이라는 보장은 없다. 그래서 일단 프로젝트 전체를 다른 곳에 복사해 따로 백업본을 두는것을 추천한다.

명령어는 간단하다. 이 명령어를 치면 try-convert가 프로젝트를 .NET 코어 플랫폼으로 바꿔준다.

~~~
try-convert -w <프로젝트 파일 또는 솔루션 파일 경로>
~~~

제대로 된다면 좋겠지만, 스케일이 큰 프로젝트 같은 경우에는 포팅에 실패할 수 있다.

## 경험해본 실패한 경우와 대처법

~~~
프로젝트 이름.csproj contains a reference to 라이브러리 이름, which is not supported on .NET Core. You may have significant work ahead of you to fully port this project.
~~~

프로젝트가 .NET 코어에서 지원하지 않는 라이브러리를 참조한 경우이다. 지원하지 않는 라이브러리 대신 다른 라이브러리를 사용하여 참조를 제거해야 한다.

~~~
System.NotSupportedException: This project has custom imports in a manner that's not supported.
~~~

이 경우는 프로젝트 파일을 뜯어야 한다. 프로젝트 파일에서 이 줄을 제외하고 NuGet.targets로 끝나는 값이 있는 태그를 지워야 한다.

예시:
~~~
<Import Project="$(SolutionDir)\.nuget\NuGet.targets" Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')" />
~~~