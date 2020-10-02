---
published: true
layout: single
title: "Flutter에서 HTTP 요청하기"
classes: wide
category: Flutter 
sidebar:
    nav: "flutter" 
tags: 
  - flutter
  - http
  - 비동기
---

Flutter에서는 HTTP 요청에 필요한 비동기 작업을 Future와 await 키워드로 할 수 있다.

## Future의 역할

Future는 리턴값이 있는 비동기 함수에서 사용된다.

Future는 JavaScript를 할 줄 안다면 들어본 적 있을 Promise와 비슷하다. 

Promise처럼 변수값으로 전달될 수 있고, await 키워드로 호출하여 작업이 끝나는 걸 대기하거나 작업이 끝났을 때 then 함수의 매개변수로 전달된 콜백함수를 실행하여 작업을 이어서 할 수 있다는 것을 비슷한 점으로 들 수 있을 것 같다.


이 Future를 사용하는 이유가 무엇이냐면, Flutter 특성 상 메인 스레드인 UI 스레드에서는 화면에 보여주는 데 필요한 작업 말고 다른 작업을 처리한다면 비효율적이고 사용자에게 버벅이는 화면을 보여줄 수 있다.

그래서 HTTP 연결과 같이 화면 구현에 불필요하고, 언제 끝날지도 모르는 작업을 UI 스레드에서 처리한다면 비효율적이기 때문에 화면이 계속 움직이게 다른 스레드에서 HTTP 작업을 실행하고, Future로 작업이 끝났을 때 메인 스레드로 작업의 결과값을 보내는 방식을 제공한다.

## 설치해야 할 라이브러리

pubspec.yml에 http 라이브러리를 추가해야 HTTP 기능을 사용할 수 있다.

~~~yml
dependencies:
  http: 0.12.2
~~~

## HTTP 요청 보내기

### GET방식으로 데이터 가져오기

~~~dart
import 'package:http/http.dart' as http;

Future<String> getData() async {
  //http.get은 리턴값이 Future이기 때문에 async 함수 내에서 await로 호출할 수 있다.
  http.Response res = await http.get('https://jsonplaceholder.typicode.com/posts/1');

  //여기서는 응답이 객체로 변환된 res 변수를 사용할 수 있다.
  //여기서 res.body를 jsonDecode 함수로 객체로 만들어서 데이터를 처리할 수 있다.

  return res.body; //가져올 데이터인 String 값을 리턴
}
~~~

### FutureBuilder로 가져온 데이터를 화면에 보여주기

Future에서 데이터를 받아서 화면에 표시하려면 FutureBuilder라는 위젯을 사용해야 한다.

~~~dart
class DataWidget extends StatefulWidget { 
  @override
  DataWidgetState createState() => DataWidgetState(); 
}

class DataWidgetState extends State<DataWidget> {
  FutureBuilder (
    future: getData() //위에서 만든 비동기 함수의 Future를 FutureBuilder에 전달한다.
    builder: (context, snapshot) {
      if(snapshot.hasData) {  //Future 작업이 끝나고 데이터가 반환될 때
        return Text(snapshot.data); //받아온 데이터를 화면에 표시
      }
      else if(snapshot.hasError) { //Future 작업중 에러가 발생했을 때
        return Text('에러: ' + snapshot.error);
      }
      //아직 데이터가 반환되지 않았을 때 표시되는 위젯
      return CircularProgressIndicator();
    }
  )
}
~~~

### POST방식으로 데이터 전송하기

~~~dart
import 'package:http/http.dart' as http;

Future<void> sendData(String data) async {
  //http.post는 리턴값이 Future이기 떄문에 async 함수 내에서 await로 호출할 수 있다.
  http.Response res = await http.post('https://ptsv2.com/t/8eqmo-1599568603/post', 
    body: { //여기에 전송할 데이터를 json 형식으로 포함해서 전송한다.
      'data': data
  });

  //여기서는 응답이 객체로 변환된 res 변수를 사용할 수 있다.
  //여기서 res.body를 jsonDecode 함수로 객체로 만들어서 데이터를 처리할 수 있다.

  return; //작업이 끝났기 때문에 리턴
}
~~~

참고로 이 예제 코드로 전송된 데이터는 [여기](https://ptsv2.com/t/8eqmo-1599568603)에서 확인할 수 있다.