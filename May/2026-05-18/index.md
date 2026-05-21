2026-05-18

# TIL

배운 것
- Web
    - Client-Server 구조
- HTTP (Message, Method, Status Code)
- RESTful API
- Spring

# Web

`DEF`

네트워크를 통해 사용자 간의 데이터를 주고받는 컴퓨터 시스템

**Web의 발전**

Web 1.0 단방향 정보 제곰 → Web 2.0 양방향 정보 공유 → Web 3.0 개인 맞춤 정보 제공&탈중앙화

## Client-Server

**Client**

`DEF`

Server에게 data나 service를 요청하는 주체

e.g. Web browser, postman, 모바일 application


**Server**

`DEF`

네트워크를 통해 클라이언트의 요청을 처리해 서비스/자원을 제공하는 컴퓨터 시스템/프로그램

Server 예시
- Web Server: 클라이언트 정적 요청(HTML, 이미지) 처리 (Nginix, Apache)
- Web Application Server(WAS): 동적 요청(로그인,회원가입 등) 처리
    - DB를 조회해 매번 다른 동적 행위 처리
    - e.g. Tomcat, Jetty, Node.js, SpringBoot(Tomcat)
- DB Server: 데이터 관리, (MySQL)

**WHY Client-Server 구조**

과거 중앙 집중 메인컴퓨터 형식일 때에는 Client + Server 역할을 모두 갖고 있었으므로, 비싸고 병목 현상이 심했다.

80년대 PC가 보급된 후 사용자 경험을 향상하기 위해 입출력 등의 작업은 Client가, 비즈니스 로직과 데이터 저장은 중앙 Server가 맡았다.

# HTTP

`DEF`

Hypertext Teansfer Protocol, Hypertext 전송 규약

`WHY`

플랫폼에 관계 없이 동일한 규칙으로 통신하기 위함

> HyperText: 글을 뛰어넘는 것 = 이미지, 링크, 목록 등을 구현하는 글(e.g. HTML, CSS, JS)

**HTTP 구성**
- HTTP Message
- HTTP Method
- Status Code
- URL

## HTTP Message

`DEF`

HTTP를 통해 Server-Client 간 데이터를 교환(요청/응답)하고자 작성하는 텍스트

`WHY`

규격화된 메시지를 통해 통신 효율을 높이기 위함

### [HTTP Message 구조](https://developer.mozilla.org/ko/docs/Web/HTTP/Guides/Messages)

**공통**
1. 시작줄: 요청/응답 개요
2. HTTP Headers: Message 정보(메타데이터)
3. 빈 줄: Header-Body 구분
4. Body/Payload: 전송할 데이터

**HTTP Request** 
1. 시작줄
- HTTP Method
- 요청 타겟(URL/프로토콜/포트, e.g. `POST / HTTP/1.1`)
2. Header: User-Agent 등
3. 빈 줄
4. Body: 보통 POST 요청인 경우 요청에 데이터 포함

**HTTP Response**
1. status line(상태줄)
- 요청 성공 여부를 나타내는 status code 포함 (e.g. `HTTP/1.1 404 Not Found.`)
2. Header
3. 빈 줄
4. Body

## HTTP Method

`DEF`

HTTP Message를 통해 Client가 Server에 요청하는 작업을 정의한 명령어

`WHY`

1. 리소스에 대한 행위를 구분해 서버가 수행해야 하는 작업을 이해하기 쉽다
2. GET 요청만 알면 Cache 서버로 조회 요청을 넘길 수 있어 본 서버의 서버의 부담을 줄임
3. 권한을 확인해야하는 데이터 수정/삭제하는 특정 작업에 대해 보안 확인을 수행할 수 있어 자원 효율적

**HTTP Method**

|HTTP Method	|기능	|CRUD|
|-|-|-|
GET	|조회	|Read|
|POST	|생성|	Create|
|PUT	|전체 수정|	Update|
|PATCH	|일부 수정|	Update|
|DELETE	|삭제|	Delete|

### [HTTP Method의 멱등성](https://developer.mozilla.org/ko/docs/Glossary/Idempotent)

> 멱등성 <br>
> 연산을 여러 번 적용하더라도 결괏값이 달라지지 않는 성질 <br>
> e.g. 1에 1을 곱하는 것, 같은 GET 요청을 반복하는 것

**GET의 멱등성(멱등 O)**

같은 조회를 수행해도 결과는 변하지 않음(멱등 O)

**DELTE의 멱등성(멱등 O)**

같은 요소를 삭제 요청이 들어오면 처음에 삭제한 뒤 계속 삭제된 상태 유지(멱등 O)
- 주의점: 블로그의 마지막 글을 삭제하는 `DELETE /posts/last` 같은 것은 멱등적이지 않음

**POST의 멱등성(멱등 X)**

POST는 리소스를 생성/처리하는 명령어로, POST를 N회 요청하면 데이터가 N회 생성되거나 변경된다(멱등 X)

**PUT의 멱등성(멱등 O)**

PUT은 리소스 전체를 대체하므로, PUT을 N회 요청하면 같은 데이터로 N번 바뀌는 것이므로 같은 결과를 갖는다.(멱등 O)

**PATCH의 멱등성(설계에 따라 다름)**

PATCH는 리소스 일부를 수정하는 HTTP Method로, 기본적으로는 멱등성을 갖지 않는다.

- PATCH의 멱등적인 설계
    - e.g. 나이를 20으로 바꾸는 요청을 N회 해도 결과는 동일(멱등 O)

- PATCH의 멱등적이지 않은 설계
    - e.g. 나이를 1 증가하는 요청을 N회하면 결과가 달라짐(멱등 X)

### PUT vs PATCH

1. 자원 낭비(+ null 위험성)

50개의 필드 중 하나만 수정할 때,<br>
PUT은 50개 필드 전부를 채워야함(그렇지 않으면 null)<br>
PATCH는 변경되는 1개의 필드만을 보내도 됨

2. 동시 수정 위험성

두 작업자가 동시에 다른 필드를 PUT으로 수정하는 경우 한 사람의 수정 결과가 덮어씌워져 무시될 수 있음

## HTTP Status Code

`DEF`

Client의 요청에 대한 Server의 응답 종류를 숫자 코드로 규격화한 것

`WHY`

요청의 성공 실패 여부를 세자리 숫자로 규격화하여 클라이언트가 응답 결과에 따라 다음 행동을 자동으로 판단하고 처리하기 위함

e.g. 200 OK, 404 Not Found, 500 Internal Server Error

## URL

Uniform Resource Locator 통합 리소스 주소

`WHY`

웹 상의 리소스를 식별하고 접근하기 위함

`DEF`

리소스의 위치와 종류를 나타내는 주소

**URL의 구성**

`http://www.example.co.kr:80/path/example.html?key1=value1&key2=value2`

1. Schme: URL의 첫 부분, 프로토콜
2. Domain, port
3. 리소스 경로
4. 매개변수

    `?` 뒤에 작성, &를 사용해 매개변수 구분<br>
    e.g. ?key1=value1&key2=value2

### Query String

`DEF`

클라이언트가 동일한 리소스에 대한 추가 정보를 서버에 요청하기 위해 사용하는 문자열
주로 정렬, 필터링, 검색에 사용

`WHY`

리소스를 바꾸지 않으면서 데이터의 조건을 첨부해 원하는 형태로 결과를 정제하기 위함

e.g. `https://shop.naver.com/shoplist?category=tshirt&size=100`

### Path Variable

`DEF`

리소스를 특정해야 하는 경우 URL에 리소스를 포함하는 변수

`WHY`

RESTful API 설계에서 자원의 식별자를 URL 구조 일부로 포함하여 직관적인 자원 엔드포인트를 표현하기 위함

e.g. `https://naver.com/users/135/profile-img`

# RESTful API

`DEF`

HTTP 프로토콜을 최대한 활용해 정보를 주고받기 위해, 
1) 자원을 포함하고, 
2) 자원에 대한 행위는 HTTP Method로 표현하는 
REST 소프트웨어 아키텍처를 따르는 API

`WHY`

1. HTTP 요청만 내용만 보고 무슨 자원이 어떤 행동을 하는지 직관적으로 알 수 있음
2. 플랫폼에 상관없이 HTTP를 사용하는 모든 사람들이 규격에 맞춰 데이터를 사용할 수 있음

## REST

Representational State Transfer

**REST 핵심 규칙**

1. URI는 정보의 자원을 표현(소문자, 하이픈, 명사, 복수형)

    e.g. /users, /users/{id}/posts
    부적절 e.g. /getUsers 

2. 자원의 행위는 HTTP 메서드로 한다

    e.g. 사용자 생성 `POST /users`

## RESTful API 설계 시 주의사항

**개체** 관점에서 설계하라

e.g. 회원 삭제 시 복구를 위해 실제 데이터는 삭제하지 않고 삭제까지 남은 일자를 수정한다면 PATCH인가 DELETE인가?

정답에 가까운 것은 DELETE이다. RESTful API는 HTTP 요청만 보고 자원에 대한 행위를 알 수 있어야 하는데, PATCH /users/id를 한다면 프론트엔드 개발자입장에서는 삭제가 아닌 정보 수정으로 인식하기 때문이다.

# Spring

`DEF`

자바 엔터프라이즈급 애플리케이션 개발을 편하게 할 수 있도록 객체 관리, 트랜잭션, 보안 등 인프라적인 기술을 종합적으로 제공하는 오픈소스 애플리케이션 프레임워크

`WHY`

과거 JSP, EJB는 프레임워크 종속성이 강해 해당 프레임워크가 있어야만 실행이 가능하여 코드가 무겁고 테스트가 어려웠으며, 객체 관리가 어려워 객체지향적 프로그래밍을 할 수 없었음

Spring은 순수 자바 객체(POJO)만으로 엔터프라이즈급 기능을 구현할 수 있도록 하여 개발자가 인프라 설정이 아닌 비즈니스 문제를 해결하는데 집중하도록 하는 도구로 탄생

## Spring의 특징

1. IoC / DI
2. AOP
3. POJO

### IoC 

Inversion of Control 제어의 역전

`DEF`

객체의 생성, 생명주기 관리, 의존성 연결 등의 제어권을 개발자가 소스코드에 직접 기술하지 않고, 스프링 컨테이너에 위임하는 설계 원칙

`WHY`

객체 간 결합도 완화
<br>
개발자가 직접 new로 객체를 생성 시 클래스의 생성자가 변경되거나 다른 클래스로 대체될 때, 코드가 사용된 곳 전부를 직접 수정해야함(e.g. DB를 로컬 인메모리에서 DBMS로 변경할 때, repository를 new로 선언한 경우 해당 코드가 있는 곳들을 전부 직접 변경해야함)

제어권을 컨테이너로 넘기면 애플리케이션의 구성 구조가 바뀌어도 비즈니스 코드는 영향을 받지 않아 유연한 코드가 됨

### DI

Dependency Injection 의존성 주입

`DEF`

스프링 컨테이너에 등록된 빈(Bean)들 사이에서, 하나의 객체가 필요로 하는 다른 객체(의존성)를 컨테이너가 런타임에 동적으로 주입해 주는 기술로, IoC를 구현하는 핵심 방법

`WHY`

클래스 내부에서 구체적인 객체에 의존하지 않기 때문에 구현체 변경이 쉽다.

실제 구현체는 스프링이 빈 컨테이너에서 주입하므로 비즈니스 코드 수정 없이 애너테이션만으로 다른 구현체로 변경할 수 있다.

### AOP

Aspect-Oriented Programming 관점 지향 프로그래밍

`DEF

어플리케이션의 공통의 관심사(로그, 보안, 성능 측정 등)와 핵심 관심사(비즈니스 로직)를 분리해 모듈화하는 것

`WHY`

코드 중복 제거
    
API 성능 측정을 위한 시간 기록 코드 등은 수백 개의 메서드에 똑같이 반복된다. 이를 모듈화하여 한 곳에 정의하고 필요한 곳에 적용할 수 있다.

> ### POJO
>
> Plain Old Java Object 순수 자바 객체
> 
> Spring은 외부 라이브러리가 전혀 없는 순수 자바 객체로 구성되어 있으며, 이를 통해 타 프레임워크 종속성 없이 실행 가능

## Spring Container

`DEF`

애플리케이션에 필요한 자바 객체(Spring Bean)의 생성, 의존성 주입, 라이프사이클 관리를 전담하며, 객체 간의 관계를 얽어주는 런타임 엔진

**Spring Container 역할**

1. Annotation(@Component, @Service, @Repository 등)을 찾아 객체를 빈으로 등록
2. 빈 생성 + 의존성 주입
3. 라이프사이클 관리

### Spring Bean

`DEF`

스프링 컨테이너가 전체 어플리케이션의 라이프사이클 동안 생성, 의존성 주입, 소멸 등의 모든 과정을 직접 관리하는 자바 객체

`WHY`

싱글톤 적용으로 메모리 효율성 증대
<br>
수많은 요청이 들어오는 환경에서, 매 요청마다 new 키워드로 객체를 생성하는 경우 GC에 큰 부담이다. 컨테이너에 등록된 Bean은 기본적으로 하나만 생성되어 공유하므로 메모리와 자원을 효율적으로 사용할 수 있다.

# Ref

[HTTP Message 구조_Mozilla Docs](https://developer.mozilla.org/ko/docs/Web/HTTP/Guides/Messages)

[멱등성_Mozilla Docs](https://developer.mozilla.org/ko/docs/Glossary/Idempotent)

[HTTP의 멱등성_Inpa Dev](https://developer.mozilla.org/ko/docs/Glossary/Idempotent)
