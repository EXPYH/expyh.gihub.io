---
title: "Spring Boot로 Exception 정의하기" 
date: 2017-10-20 23:24:28 +0900 
categories: Spring boot
---

## 발단

REST API에서는 에러가 발생할 경우 404 같은 일반적인 status code가 아닌, 서버 쪽에서 정의한 error code를 별도로 리턴하는 경우가 있다. 서버쪽에서 error code를 별도로 정의하면 같은 404 에러라도 세분화하여 에러를 구분할 수 있고, 클라이언트 측에서도 상황별로 에러 처리를 하기가 편해진다.

보통은 @ExceptionHandler 및 @ControllerAdvice를 사용하는 듯 하지만, 이번에는 ErrorAttribute를 재정의하는 방법을 사용해본다.

## DefaultErrorAttributes

Spring boot 기본 설정에서는 @RestController에서 예외가 발생하면 클라이언트에게 다음과 같은 형식으로 기본 error response가 전달된다.

<pre><code>{
    "timestamp": "2020-10-20T11:57:49.947+00:00",
    "status": 404,
    "error": "Not found",
    "message": "blahblah",
    "path": "/user/abc"
}</code></pre>

이 에러형식은 ErrorAttribute를 구현한 DefaultErrorAttributes를 따른다. 따라서 이 DefaultErrorAttribute를 상속하여 구현하고 빈으로 등록해주면 다음과 같이 error response를 커스텀할 수 있다. 

<pre><code>{
    "errorCode": "TEST-40001",
    "message": "Missing header - user" 
}</code></pre>
/

## 1. Exception을 상속받는 커스텀 Exception 클래스 정의

그 전에 먼저 커스텀 Exception을 정의하자. Exception을 상속받은 커스텀 클래스를 만들어준다.

<pre><code>package com.expyh.exceptiontest.exception;

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
}</code></pre>

이제 CustomException을 던질 때 매개변수로 넣어줄 Error를 정의하자. Error는 enum형태로 정의하면 편하다.

<pre><code>public enum CustomError{
    //java의 에러가 아닌 erorCode로 표현되는 그 에러를 뜻하는 것임.
    
    NO_HEADER("TEST-40001", HttpStatus.BAD_REQUEST.value(), "Missing header - {}"),
    NO_PATH_VARIABLE("TEST-40002", HttpStatus.BAD_REQUEST.value(), "Missing path variable - {}");

    String errorCode;
    int status;
    String message;

    CustomError(String errorCode, int status, String message) {
        this.errorCode = errorCode;
        this.status = status;
        this.message = message;
    }

    public String getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(String errorCode) {
        this.errorCode = errorCode;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}</code></pre>


이제 모든 준비가 끝났고 DefaultErrorAttributes만 재정의해주면 된다. getErrorAttributes()를 이용해서 error response의 항목들을 가져온 다음 여기에 새로 추가하거나 제거함으로써 error response를 조정할 수 있다. 

<pre><code>@Component
public class CustomErrorAttribute extends DefaultErrorAttributes {

    @Override
    public Map&ltString, Object&gt getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
        Map &ltString, Object&gt errorAttributes = super.getErrorAttributes(webRequest, options);
        Throwable error = super.getError(webRequest);
        if (error instanceof CustomException) {
            errorAttributes.put("errorCode", ((CustomException) error).getError().getErrorCode() );
        }
        else {
            errorAttributes.put("errorCode", "TEST-50001");
        }
        errorAttributes.remove("timestamp");
        errorAttributes.remove("status");
        errorAttributes.remove("error");
        errorAttributes.remove("path");

        return errorAttributes;
    }

}</code></pre>

실제 컨트롤러에서 Exception을 던질 때는 CustomException의 매개변수로 enum으로 선언한 CustomError를 넣어주면 된다. 이렇게 되면 CustomErrorAttribute가 error response를 조작하여 우리가 원하는 형태로 전달해준다.

<pre><code>@RestController
public class HelloController {

    @RequestMapping("/")
    public String index() {
        return "Greetings from Spring Boot!";
    }

    @RequestMapping("/test")
    public String index(@RequestHeader("user") String user) throws Exception {
        if (user.isEmpty()){
            throw new CustomException(CustomError.NO_HEADER, "user");
        }
        return "Hello ! : " + user ;
    }
}</code></pre>