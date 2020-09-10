---
published: true
layout: single
title: "C#에서 암호화 사용하기"
classes: wide
category: .NET
tags: 
  - .net
  - 암호화
---

작년에 친구가 컴퓨터 전원을 꺼버려서 내 파일에 대한 복수의 의미로 랜섬웨어를 만들었던 기억이 나서 C#으로 암호화 하는 법을 적어볼 것이다.

암호화는 ASP.NET 프로젝트를 진행하면서 비밀번호 저장 등 민감한 정보를 다룰 때 자주 쓰이니까 라이브러리로 만들어두고 편하게 쓰자.

## 필요한 것

- C# 개발환경 하나면 충분하다.

## 암호화 하기 전 만들어야 하는 것

우리가 사용할 알고리즘이 AES이라는 알고리즘인데, AES 알고리즘은 암호화 작업을 수행할 때 키와 초기화 벡터라는 값을 요구한다. 그래서 키와 초기화 벡터를 생성하는 메소드를 미리 만들 것이다.

키 생성 메소드
~~~cs
public static Rfc2898DeriveBytes CreateKey(string password) {
    byte[] keyBytes = Encoding.UTF8.GetBytes(password);         //키값 생성
    byte[] saltBytes = SHA512.Create().ComputeHash(keyBytes);   //솔트값(원본 키값을 알지 어렵게 하는 값)

    Rfc2898DeriveBytes result = new Rfc2898DeriveBytes(keyBytes, saltBytes, 100000);    //키값에 솔트값을 사용해 새로운 키 생성, 마지막에 들어가는 수는 해시 생성의 반복 횟수이다.

    return result;  //키값 반환
}
~~~

초기화 벡터 생성 메소드

~~~cs
public static Rfc2898DeriveBytes CreateVector(string vector) {
    byte[] vectorBytes = Encoding.UTF8.GetBytes(vector);        //벡터 생성
    byte[] saltBytes = SHA512.Create().ComputeHash(vectorBytes);   //솔트값(원본 벡터를 알지 어렵게 하는 값)

    Rfc2898DeriveBytes result = new Rfc2898DeriveBytes(vectorBytes, saltBytes, 100000);    //벡터에 솔트값을 사용해 새로운 키 생성, 마지막에 들어가는 수는 해시 생성의 반복 횟수이다.

    return result;  //벡터 반환
}
~~~

## 암호화 메소드 생성

벡터를 생성할때 사용한 문자열은 [여기](https://randomkeygen.com/)에서 가져왔다. 유추하기 어려운 문자열을 쓰고 싶을때 유용한 사이트다.

이 예제에서는 AES-256으로 암호화를 진행한다.

~~~cs
public static byte[] Encrypt(byte[] origin, string password) {
    RijndaelManaged aes = new RijndaelManaged();       //AES 알고리즘
    Rfc2898DeriveBytes key = CreateKey(password);            //키값 생성
    Rfc2898DeriveBytes vector = CreateVector("ZaWmAcu1C2fbgJa4cPuZrT6MhuWmx6GE");   //벡터 생성 

    aes.BlockSize = 128;            //AES의 블록 크기는 128 고정이다.
    aes.KeySize = 256;              //AES의 키 크기는 128, 192, 256을 지원한다.
    aes.Mode = CipherMode.CBC;      
    aes.Padding = PaddingMode.PKCS7;
    aes.Key = key.GetBytes(32);     //AES-256을 사용하므로 키값의 길이는 32여야 한다.
    aes.IV = vector.GetBytes(16);   //초기화 벡터는 언제나 길이가 16이어야 한다.

    //키값과 초기화 벡터를 기반으로 암호화 작업을 하는 클래스 변수를 생성
    ICryptoTransform encryptor = aes.CreateEncryptor(aes.Key, aes.IV);  

    //using블록으로 변수를 사용하면 블록에서 나올때 자동으로 변수가 가비지컬렉팅 된다. 
    using(MemoryStream ms = new MemoryStream()) //결과를 담을 스트림 
    {
        //encryptor 변수에서 암호화된 데이터를 결과에 쓰는 스트림
        using(CryptoStream cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))   
        {
            cs.Write(origin, 0, origin.Length);
        }
        return ms.ToArray();    //암호화된 바이트 배열 반환
    }
}
~~~

## 복호화 메소드 생성

복호화할때 주의할 점은 벡터값이 암호화할 때랑 다르면 반드시 실패한다는 점이다. 위에서 사용한 원본 벡터 문자열을 그대로 사용해야 복호화를 할 수 있다.

마찬가지로 AES-256으로 복호화를 진행한다.

~~~cs
public static byte[] Decrypt(byte[] origin, string password) {
    RijndaelManaged aes = new RijndaelManaged();       //AES 알고리즘
    Rfc2898DeriveBytes key = CreateKey(password);            //키값 생성
    Rfc2898DeriveBytes vector = CreateVector("ZaWmAcu1C2fbgJa4cPuZrT6MhuWmx6GE");   //벡터 생성 

    aes.BlockSize = 128;            //AES의 블록 크기는 128 고정이다.
    aes.KeySize = 256;              //AES의 키 크기는 128, 192, 256을 지원한다.
    aes.Mode = CipherMode.CBC;      
    aes.Padding = PaddingMode.PKCS7;
    aes.Key = key.GetBytes(32);     //AES-256을 사용하므로 키값의 길이는 32여야 한다.
    aes.IV = vector.GetBytes(16);   //초기화 벡터는 언제나 길이가 16이어야 한다.

    //키값과 초기화 벡터를 기반으로 복호화 작업을 하는 클래스 변수를 생성
    ICryptoTransform decryptor = aes.CreateDecryptor(aes.Key, aes.IV);  

    //using블록으로 변수를 사용하면 블록에서 나올때 자동으로 변수가 가비지컬렉팅 된다. 
    using(MemoryStream ms = new MemoryStream()) //결과를 담을 스트림 
    {
        //encryptor 변수에서 복호화된 데이터를 결과에 쓰는 스트림
        using(CryptoStream cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Write))   
        {
            cs.Write(origin, 0, origin.Length);
        }
        return ms.ToArray();    //복호화된 바이트 배열 반환
    }
}
~~~

사실 복호화 예제랑 암호화 예제에서 변화한 것은 encryptor를 생성할때 사용한 메소드 이름의 단 두글자밖에 없다. 알고리즘의 기반이 같고 배열에 변화한 값을 쓰는 건 같기 때문이다.