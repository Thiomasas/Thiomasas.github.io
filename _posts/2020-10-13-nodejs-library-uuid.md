---
published: true
layout: single
title: "Node.js 유용한 라이브러리 - uuid"
category: Node.js
sidebar:
    nav: "nodejs" 
tags: 
  - node.js 
  - 라이브러리 소개
---

UUID는 Universally Unique IDentifier의 약자로, 전세계에 하나밖에 없는 ID라는 뜻이다. 이런 ID는 고유하기 때문에 서버에서 사용자들에 UUID를 붙여서 구분하는 등 여러 방면에서 유용하게 쓰일 수 있다. 이번 포스트에서는 Node.js에서 UUID를 생성해주는 라이브러리를 소개할 것이다.  

## 설치

npm을 사용한다면 위의 명령어를, yarn을 사용한다면 아래의 명령어를 사용하자.

~~~sh
# npm
npm install uuid

# yarn
yarn add uuid
~~~

## 사용법

UUID에는 생성 방법이 여러가지 있다.

- v1: 타임스탬프(시간) 기준으로 생성
- v3: MD5 해시 기준으로 생성
- v4: 랜덤값을 기반으로 생성
- v5: SHA-1 해시 기준으로 생성

위의 생성방법들 중 마음에 드는 것을 골라서 사용하도록 하자.

## v1(타임스탬프 기반) UUID 생성

~~~js
import { v1 } from 'uuid';

//기본 설정으로 생성
console.log(v1());

//설정을 붙여서 생성
let options = {
    node: // 바이트값 6개
    clockseq: // 클럭 시퀀스 (0 - 0x3fff 사이 값)
    msecs: // 밀리초
    nsecs: // 나노초
    random: // 16개의 랜덤 바이트값
    rng: // random 변수를 대체할 16개의 랜덤 바이트값을 반환하는 함수
}

console.log(v1(options));
~~~

## v3(MD5 기반) UUID 생성

참고로 RFC 표준에 따르면 하위호환같은 사유가 아니라면 v3보다는 v5를 사용하는걸 추천한다.

~~~js
import { v3 } from 'uuid';

//v5는 네임스페이스라는 값과 해싱할 값으로 UUID를 만든다.
//그래서 네임스페이스로 사용할 UUID 값이 필요하다.
console.log(v3('해싱할 값', '1a30bae5-e589-47b1-9e77-a7da2cdbc2b8'));

//해싱할 값이 URL이나 도메인인 경우 미리 만들어진 네임스페이스를 활용하면 된다.
//해싱할 값이 도메인인 경우
console.log(v3('www.google.com'), v3.DNS);

//해싱할 값이 URL인 경우
console.log(v3('https://www.google.com'), v3.URL);
~~~

## v4(랜덤값 기반) UUID 생성

~~~js
import { v4 } from 'uuid';

//기본 설정으로 생성
console.log(v4());

//설정을 붙여서 생성
let options = {
    random: // 16개의 랜덤 바이트값
    rng: // random 변수를 대체할 16개의 랜덤 바이트값을 반환하는 함수
}
console.log(v4(options));
~~~

## v5(SHA-1 기반) UUID 생성 

~~~js
import { v5 } from 'uuid';

//v5는 네임스페이스라는 값과 해싱할 값으로 UUID를 만든다.
//그래서 네임스페이스로 사용할 UUID 값이 필요하다.
console.log(v5('해싱할 값', '1a30bae5-e589-47b1-9e77-a7da2cdbc2b8'));

//해싱할 값이 URL이나 도메인인 경우 미리 만들어진 네임스페이스를 활용하면 된다.
//해싱할 값이 도메인인 경우
console.log(v5('www.google.com'), v5.DNS);

//해싱할 값이 URL인 경우
console.log(v5('https://www.google.com'), v5.URL);
~~~