---
published: true
layout: single
title: "C# 유용한 라이브러리 - Json.NET"
classes: wide
category: .NET
sidebar:
    nav: "dotnet" 
tags: 
  - .net
  - 라이브러리 소개
---

자바스크립트에서는 객체 자신을 json이라는 형식의 텍스트 파일로 출력할 수 있다. 그렇다면 다른 언어에서는 객체를 파일로 추출할 수 없는걸까? 당연히 방법은 있다. 그래서 이번에 소개할 라이브러리는 .NET 언어들의 객체를 json 형식으로 출력해주는 Json.NET이라는 라이브러리다.

## 설치하기

설치는 NuGet 패키지 관리자를 통해 진행할 수 있다.

NuGet 명령줄로 설치를 하거나,

~~~
PM> Install-Package Newtonsoft.Json
~~~

IDE에서 제공하는 NuGet 패키지 관리자를 통해 설치할 수도 있다.

![nuget](https://imgur.com/en94WXT.png)

## 사용법

보통 json 형식의 데이터는 다음과 같은 용도에 사용된다.

- 웹에서 폼 데이터 전송
- API의 요청과 응답
- 설정 파일

듣기만 하면 다재다능하지만 json 파일을 만드는 작업이 .NET 환경에서 매우 번거롭기 때문에 Json.NET이 있는 것이다.

Json.NET은 json 파일과 관련된 여러 기능을 제공한다. 이번 포스트에서 다룰 기능들은

- 객체와 json간 변환 
- json과 xml간 변환

이다.

그리고 이번에 테스트 대상이 될 객체는 간단한 유저의 정보다.

~~~cs
class User
{
  string Name;        //이름
  DateTime CreatedAt; //가입시각
  string[] Friends;   //친구목록
}
~~~

### 객체에서 json 텍스트로 변환하기

JsonConvert 클래스의 SerializeObject 메소드를 사용하면 자동으로 json 텍스트로 변환된다.

~~~cs
//객체 생성
User user = new User();
user.Name = "강현구";
user.CreatedAt = new DateTime(2020, 10, 18);
user.Friends = new string[] { "아싸라서", "친구가", "없다" };

//변환
string json = JsonConvert.SerializeObject(user);
Console.WriteLine(json);
~~~

출력해보면 다음과 같이 정상적인 json 텍스트가 나온다.

~~~json
{"Name":"강현구","CreatedAt":"2020-10-18T00:00:00","Friends":["아싸라서","친구가","없다"]}
~~~

하지만 자동으로 변환되는게 믿음직스럽지 않거나 객체의 몇몇 변수를 제외해야 한다면 수동으로 변환하는 방법도 있다.

JObject 클래스의 객체는 Linq를 통해 json을 생성할 수 있기 때문에 JObject를 활용하면 수동으로 변환할 수 있다.

~~~cs
JObject obj = new JObject();
obj.Add("Name", user.Name);
obj.Add("CreatedAt", user.CreatedAt);

//배열을 담을 땐 JArray를 사용할 수 있다.
JArray arr = new JArray();
foreach(string friend in user.Friends)
{
    arr.Add(friend);
}
obj.Add("Friends", arr);

Console.WriteLine(obj.ToString());
~~~

출력해보면 정상적으로 json 텍스트가 나온다.

~~~json
{
  "Name": "강현구",
  "CreatedAt": "2020-10-18T00:00:00",
  "Friends": [
    "아싸라서",
    "친구가",
    "없다"
  ]
}
~~~

### json에서 객체로 변환하기

json에서 객체로 변환할 때도 JsonConvert의 DeserializeObject 메소드로 자동 변환할 수 있다. 대신 객체에서 json으로 변환할 때랑의 차이점은 어떤 객체로 변환할 지 타입을 지정해야 한다는 것이다. 

~~~cs
//변환할 json 텍스트
string json = "{\"Name\":\"강현구\",\"CreatedAt\":\"2020-10-18T00:00:00\",\"Friends\":[\"아싸라서\",\"친구가\",\"없다\"]}";

//객체로 변환
User user = JsonConvert.DeserializeObject<User>(json);

Console.WriteLine(user.Name);
Console.WriteLine(user.CreatedAt);
foreach(string friend in user.Friends)
{
  Console.WriteLine(friend);
}
~~~

출력하면 객체에 정상적으로 값이 들어가 있는걸 볼 수 있다.

~~~
강현구
2020-10-18 오전 12:00:00
아싸라서
친구가
없다
~~~

객체에서 json으로 변환할 떄 처럼 당연히 수동으로 변환하는 방법도 있다.

~~~cs
//변환할 json 텍스트
string json = "{\"Name\":\"강현구\",\"CreatedAt\":\"2020-10-18T00:00:00\",\"Friends\":[\"아싸라서\",\"친구가\",\"없다\"]}";
//json 텍스트에서 JObject 생성
JObject obj = JObject.Parse(json);
//데이터를 담을 객체
User user = new User();

//JObject에서 데이터 가져오기
user.Name = obj.SelectToken("Name").ToString();
//객체인경우 ToObject로 변환할 수 있음
user.CreatedAt = obj.SelectToken("CreatedAt").ToObject<DateTime>();
//LINQ 기능들을 모두 사용할 수 있음
user.Friends = obj.SelectToken("Friends").Select(t => (string)t).ToArray();

Console.WriteLine(user.Name);
Console.WriteLine(user.CreatedAt);
foreach(string friend in user.Friends)
{
    Console.WriteLine(friend);
}
~~~

출력하면 객체에 정상적으로 값이 들어가 있는걸 볼 수 있다.

~~~
강현구
2020-10-18 오전 12:00:00
아싸라서
친구가
없다
~~~
