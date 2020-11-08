---
title: "Spring Boot로 Exception 정의하기" 
date: 2017-10-20 23:24:28 +0900 
categories: Spring boot
---

## 발단

최근에 새 서비스를 위해 REST API 개발을 하는데 클라이언트가 error code를 정의해달라고 요청이 왔다. 단순히 status code 정도만 리턴하던 API였는데 임의로 error code를 정의 하다보니 코드 상으로나 로직 상으로나 고민해야할 것이 한 두가지가 아니었다.

여기서는 내가 코드 상으로 custom error code를 정의했던 방법을 다룬다.

## 목표

Spring boot 기본 설정인 상태에서 예외가 발생하면 클라이언트에게 대충 다음과 같은 형식으로 Response body가 전달된다.

<pre><code>
{
    "timestamp": "2020-10-20T11:57:49.947+00:00",
    "status": 400,
    "error": "Bad Request",
    "message": "Missing request header 'user' for method parameter of type String",
    "path": "/login"
}
</code></pre>

이를 다음과 같이 errorCode와 message 만 남기도록 하고, 클라이언트는 errorCode와 message를 통해서 예외처리를 하도록 하려고 한다.

<pre><code>
{
    "errorCode": "TEST-40001",
    "message": "Missing header - user" 
}
</code></pre>

## 순서

다음과 같은 순서로 이뤄진다.

1. Exception을 상속받는 커스텀 Exception 클래스 정의
2. enum 형태로 에러 클래스 정의
3. @ControllerAdvice 정의 
4. @ExceptionHandler 정의
5. DefaultErrorAttributes 를 상속받는 커스텀 ErrorAttribute 클래스 정의




## 1. Exception을 상속받는 커스텀 Exception 클래스 정의

내 맘대로 Exception을 정의하자. Exception을 상속받은 커스텀 클래스를 만들어준다.
String.format()을 사용하면 인자를 에러메시지의 {} 안에 넣어줄 수 있다.

<pre><code>
package com.expyh.exceptiontest.exception;

public class CustomException extends Exception{
    CustomError error;
    String message;

    public CustomException(CustomError error, String message, String... args) {
        this.error = error;
        this.message = message;
        if (args.length > 0){
            message = String.format(message, args);
        }
    }

    public CustomError getError() {
        return error;
    }

    public void setError(CustomError error) {
        this.error = error;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

</code></pre>

이제 CustomException을 던질 때 넣어줄 Error를 정의하자. Error는 enum형태로 정의하면 편하다.

<pre><code>
package com.expyh.exceptiontest.exception;

public class CustomException extends Exception{
    CustomError error;
    String message;

    public CustomException(CustomError error, String message, String... args) {
        this.error = error;
        this.message = message;
        if (args.length > 0){
            message = String.format(message, args);
        }
    }

    public CustomError getError() {
        return error;
    }

    public void setError(CustomError error) {
        this.error = error;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

</code></pre>