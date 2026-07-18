# FastAPI 백엔드 기초 정리

이번 글에서는 FastAPI를 공부하기 전에 알아야 할 백엔드 기초 개념을 정리한다.

다루는 내용은 다음과 같다.

* 클라이언트와 서버
* 2티어와 3티어 구조
* HTTP 요청과 응답
* HTTP 메서드와 상태 코드
* REST API 설계 원칙
* FastAPI
* 동기와 비동기 처리
* WSGI와 ASGI
* Uvicorn
* Path Parameter와 Query Parameter

---

## 1. 전체 요청 처리 흐름

클라이언트가 다음 주소로 요청을 보낸다고 가정해 보자.

```text
http://127.0.0.1:8000/items/10?q=apple
```

FastAPI 서버에서는 다음과 같은 코드가 실행될 수 있다.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    return {
        "item_id": item_id,
        "q": q,
    }
```

전체 요청 처리 흐름은 다음과 같다.

```text
클라이언트
    ↓ HTTP 요청
Uvicorn
    ↓ ASGI 규격으로 요청 전달
FastAPI
    ↓ URL과 HTTP 메서드에 맞는 함수 탐색
read_item(item_id=10, q="apple")
    ↓
Python 객체 반환
    ↓
JSON 응답으로 변환
    ↓ HTTP 응답
클라이언트
```

클라이언트가 실제로 보내는 요청은 개념적으로 다음과 같다.

```http
GET /items/10?q=apple HTTP/1.1
Host: 127.0.0.1:8000
```

서버는 다음과 같은 응답을 반환한다.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "item_id": 10,
  "q": "apple"
}
```

---

# 2. 클라이언트와 서버

## 2.1 클라이언트

클라이언트는 서버에 요청을 보내는 프로그램이다.

대표적인 클라이언트는 다음과 같다.

* 웹 브라우저
* 스마트폰 애플리케이션
* React, Next.js와 같은 프론트엔드
* Postman
* curl
* 다른 백엔드 서버

클라이언트는 서버에 다음과 같은 작업을 요청한다.

```text
사용자 정보를 조회해 주세요.
회원가입을 처리해 주세요.
게시글을 저장해 주세요.
상품을 삭제해 주세요.
```

## 2.2 서버

서버는 클라이언트의 요청을 받아 처리하고 결과를 반환하는 프로그램이다.

일반적인 서버의 처리 과정은 다음과 같다.

```text
1. 클라이언트의 요청을 받는다.
2. 요청값이 올바른지 검사한다.
3. 비즈니스 로직을 실행한다.
4. 데이터베이스를 조회하거나 수정한다.
5. 처리 결과를 클라이언트에 반환한다.
```

예를 들어 로그인 요청은 다음과 같이 처리할 수 있다.

```text
이메일과 비밀번호 입력
    ↓
사용자 존재 여부 확인
    ↓
비밀번호 비교
    ↓
인증 토큰 생성
    ↓
로그인 결과 반환
```

## 2.3 일반 컴퓨터도 서버가 될 수 있다

서버는 특별한 종류의 컴퓨터를 의미하는 것이 아니다.

현재 사용 중인 컴퓨터에서 서버 프로그램을 실행하면 그 컴퓨터가 서버 역할을 한다.

```bash
uvicorn main:app --reload
```

위 명령어를 실행하면 Uvicorn이 현재 컴퓨터의 특정 포트를 열고 클라이언트의 요청을 기다린다.

---

# 3. IP 주소, 도메인, 포트

## 3.1 IP 주소

IP 주소는 네트워크에서 컴퓨터를 식별하기 위한 주소다.

```text
127.0.0.1
192.168.0.10
```

`127.0.0.1`은 현재 사용 중인 자기 자신의 컴퓨터를 의미한다.

다음 두 주소는 일반적으로 같은 대상을 가리킨다.

```text
http://127.0.0.1:8000
http://localhost:8000
```

## 3.2 도메인

도메인은 IP 주소를 사람이 기억하기 쉬운 이름으로 표현한 것이다.

```text
google.com
naver.com
api.example.com
```

DNS는 도메인을 실제 IP 주소로 변환한다.

```text
api.example.com
    ↓ DNS 조회
203.0.113.10
```

## 3.3 포트

포트는 한 컴퓨터 안에서 어떤 프로그램과 통신할지 구분하기 위한 번호다.

한 컴퓨터에서 여러 서버 프로그램이 동시에 실행될 수 있다.

```text
127.0.0.1:3000  → Next.js
127.0.0.1:8000  → FastAPI
127.0.0.1:5432  → PostgreSQL
127.0.0.1:6379  → Redis
```

IP 주소를 건물 주소라고 생각하면 포트는 건물 안의 호수와 비슷하다.

---

# 4. URL 구조

다음 URL을 구성 요소별로 나누어 보자.

```text
https://api.example.com:443/users/10/orders?status=paid&page=2
```

각 부분은 다음 의미를 가진다.

```text
https
프로토콜

api.example.com
도메인

443
포트

/users/10/orders
경로

status=paid&page=2
쿼리 문자열
```

HTTPS의 기본 포트는 443이기 때문에 일반적으로 생략한다.

```text
https://api.example.com/users
```

HTTP의 기본 포트는 80이다.

---

# 5. 2티어와 3티어 구조

## 5.1 Tier와 Layer

Tier와 Layer는 비슷하게 사용되지만 엄밀히는 차이가 있다.

* Layer는 코드의 논리적인 역할 분리를 의미한다.
* Tier는 실제 실행 환경이나 배포 단위의 분리를 의미한다.

예를 들어 다음 코드는 세 개의 Layer로 나눌 수 있다.

```text
Router Layer
Service Layer
Repository Layer
```

하지만 세 Layer가 모두 하나의 서버에서 실행된다면 물리적으로는 하나의 Tier일 수 있다.

---

## 5.2 2티어 구조

2티어 구조에서는 클라이언트가 데이터베이스나 데이터가 포함된 서버에 직접 접근한다.

```text
클라이언트
    ↓
데이터베이스 서버
```

예를 들어 데스크톱 프로그램이 데이터베이스에 직접 SQL을 실행할 수 있다.

```text
데스크톱 프로그램
    ↓ SQL 실행
데이터베이스
```

### 장점

* 구조가 단순하다.
* 개발 속도가 빠르다.
* 작은 내부 시스템에 적합하다.

### 단점

* 클라이언트가 데이터베이스 구조를 알아야 할 수 있다.
* 데이터베이스 접속 정보가 노출될 위험이 있다.
* 비즈니스 로직이 여러 클라이언트에 중복될 수 있다.
* 데이터베이스 구조 변경 시 클라이언트도 함께 수정해야 할 수 있다.
* 규모가 커질수록 유지보수가 어려워진다.

---

## 5.3 3티어 구조

일반적인 웹 서비스에서는 3티어 구조를 많이 사용한다.

```text
Presentation Tier
클라이언트 또는 프론트엔드
        ↓
Application Tier
백엔드 서버
        ↓
Data Tier
데이터베이스
```

예를 들면 다음과 같다.

```text
React 또는 모바일 앱
        ↓ HTTP
FastAPI 서버
        ↓ SQL
PostgreSQL
```

### Presentation Tier

사용자와 상호작용하는 계층이다.

주요 역할은 다음과 같다.

* 화면 출력
* 버튼 입력 처리
* 사용자 입력 수집
* 서버에 요청 전송
* 응답 결과 표시

### Application Tier

서비스의 핵심 로직을 처리한다.

주요 역할은 다음과 같다.

* 로그인 검증
* 권한 검사
* 주문 금액 계산
* 결제 처리
* 요청 데이터 검증
* 응답 데이터 가공

### Data Tier

데이터를 저장하고 조회하는 계층이다.

주요 역할은 다음과 같다.

* 사용자 정보 저장
* 게시글 조회
* 주문 상태 수정
* 상품 정보 삭제

### 3티어 구조의 장점

* 계층별 역할이 명확하다.
* 데이터베이스가 클라이언트에 직접 노출되지 않는다.
* 여러 클라이언트가 하나의 백엔드 API를 사용할 수 있다.
* 프론트엔드, 백엔드, 데이터베이스를 독립적으로 수정하거나 확장할 수 있다.

```text
웹 프론트엔드
모바일 앱
관리자 페이지
        ↓
FastAPI
        ↓
PostgreSQL
```

---

# 6. HTTP

HTTP는 클라이언트와 서버가 데이터를 주고받기 위해 사용하는 통신 규칙이다.

```text
HTTP = HyperText Transfer Protocol
```

HTTP는 특정 프로그래밍 언어나 프레임워크에 종속되지 않는다.

따라서 서로 다른 언어로 작성된 프로그램도 HTTP 규칙을 따르면 통신할 수 있다.

```text
React 또는 Next.js
        ↓ HTTP
FastAPI
```

```text
Spring
        ↓ HTTP
FastAPI
```

---

# 7. HTTP 요청 구조

HTTP 요청은 크게 네 부분으로 구성된다.

```text
1. 요청 시작 줄
2. 헤더
3. 빈 줄
4. 본문
```

예시는 다음과 같다.

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer abcdef1234

{
  "name": "Kim",
  "age": 25
}
```

## 7.1 요청 시작 줄

```http
POST /users HTTP/1.1
```

다음 세 가지 정보가 포함된다.

```text
HTTP 메서드
요청 경로
HTTP 버전
```

* `POST`는 수행하려는 동작을 나타낸다.
* `/users`는 요청 대상 자원을 나타낸다.
* `HTTP/1.1`은 사용하는 HTTP 버전을 나타낸다.

## 7.2 요청 헤더

헤더에는 요청에 대한 부가 정보가 들어간다.

```http
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token
Accept: application/json
```

### Content-Type

요청 본문의 데이터 형식을 나타낸다.

```http
Content-Type: application/json
```

요청 본문이 JSON 형식이라는 뜻이다.

### Accept

클라이언트가 원하는 응답 형식을 나타낸다.

```http
Accept: application/json
```

### Authorization

인증 정보를 전달한다.

```http
Authorization: Bearer access-token
```

### Cookie

브라우저에 저장된 쿠키를 서버에 전달한다.

```http
Cookie: session_id=abc123
```

## 7.3 요청 본문

요청 본문에는 서버에 전달할 실제 데이터가 들어간다.

```json
{
  "name": "Kim",
  "age": 25
}
```

주로 POST, PUT, PATCH 요청에서 사용한다.

---

# 8. HTTP 응답 구조

HTTP 응답도 크게 네 부분으로 구성된다.

```text
1. 상태 줄
2. 헤더
3. 빈 줄
4. 본문
```

예시는 다음과 같다.

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 10,
  "name": "Kim"
}
```

## 8.1 상태 줄

```http
HTTP/1.1 201 Created
```

다음 정보가 포함된다.

```text
HTTP 버전
상태 코드
상태 메시지
```

## 8.2 응답 헤더

응답 데이터에 대한 부가 정보가 들어간다.

```http
Content-Type: application/json
```

응답 본문이 JSON 형식이라는 뜻이다.

## 8.3 응답 본문

서버가 클라이언트에 전달하는 실제 데이터다.

```json
{
  "id": 10,
  "name": "Kim"
}
```

---

# 9. JSON

현대 웹 API에서는 데이터를 주고받을 때 JSON 형식을 많이 사용한다.

```json
{
  "id": 1,
  "name": "Kim",
  "active": true,
  "skills": ["Python", "FastAPI"],
  "profile": {
    "age": 25
  }
}
```

JSON에서 사용할 수 있는 주요 타입은 다음과 같다.

```text
문자열
숫자
참과 거짓
null
배열
객체
```

Python 타입과 비교하면 다음과 같다.

```text
JSON object  ↔ Python dict
JSON array   ↔ Python list
JSON null    ↔ Python None
JSON true    ↔ Python True
JSON false   ↔ Python False
```

FastAPI에서는 Python 딕셔너리를 반환하면 자동으로 JSON 응답으로 변환한다.

```python
@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

응답은 다음과 같다.

```json
{
  "message": "Hello World"
}
```

---

# 10. HTTP 메서드

HTTP 메서드는 서버의 자원에 어떤 동작을 수행할지 표현한다.

## 10.1 GET

데이터를 조회할 때 사용한다.

```http
GET /users
GET /users/10
```

FastAPI에서는 다음과 같이 작성한다.

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

GET 요청은 일반적으로 서버의 데이터를 변경하지 않아야 한다.

다음과 같은 설계는 피하는 것이 좋다.

```http
GET /users/10/delete
```

GET 요청을 실행했는데 데이터가 삭제된다면 브라우저 캐시나 검색 엔진의 접근으로 데이터가 변경될 위험이 있다.

---

## 10.2 POST

새로운 자원을 생성하거나 특정 작업을 실행할 때 사용한다.

```http
POST /users
```

요청 본문은 다음과 같다.

```json
{
  "name": "Kim"
}
```

새로운 자원이 생성되었다면 `201 Created`를 반환할 수 있다.

```python
@app.post("/users", status_code=201)
def create_user():
    return {
        "id": 1,
        "name": "Kim",
    }
```

---

## 10.3 PUT

기존 자원 전체를 새로운 데이터로 교체할 때 주로 사용한다.

```http
PUT /users/10
```

기존 데이터가 다음과 같다고 가정하자.

```json
{
  "name": "Kim",
  "age": 25,
  "city": "Seoul"
}
```

다음 PUT 요청은 전체 사용자 정보를 새로운 값으로 교체한다는 의미를 가진다.

```json
{
  "name": "Lee",
  "age": 30,
  "city": "Busan"
}
```

---

## 10.4 PATCH

기존 자원의 일부만 수정할 때 사용한다.

```http
PATCH /users/10
```

```json
{
  "city": "Busan"
}
```

기존의 `name`, `age` 값은 유지하고 `city`만 변경한다.

---

## 10.5 DELETE

자원을 삭제할 때 사용한다.

```http
DELETE /users/10
```

삭제에 성공했고 반환할 데이터가 없다면 다음 상태 코드를 사용할 수 있다.

```http
204 No Content
```

---

# 11. HTTP 메서드의 안전성과 멱등성

## 11.1 안전한 메서드

안전한 메서드는 서버의 상태를 변경하지 않는 메서드다.

대표적인 예는 GET이다.

```http
GET /users/10
```

같은 요청을 여러 번 보내도 데이터를 조회만 해야 한다.

## 11.2 멱등성

같은 요청을 여러 번 수행해도 최종 서버 상태가 동일한 성질을 멱등성이라고 한다.

### PUT

사용자 이름을 `Kim`으로 변경하는 요청을 여러 번 보내도 최종 이름은 `Kim`이다.

따라서 PUT은 일반적으로 멱등성을 가진다.

### DELETE

같은 사용자를 여러 번 삭제해도 최종 상태는 사용자가 존재하지 않는 상태다.

따라서 DELETE도 일반적으로 멱등성을 가진다.

### POST

```http
POST /orders
```

같은 요청을 두 번 보내면 주문이 두 개 생성될 수 있다.

따라서 POST는 일반적으로 멱등성을 가지지 않는다.

---

# 12. HTTP 상태 코드

HTTP 상태 코드는 서버가 요청을 어떻게 처리했는지 나타낸다.

## 12.1 2xx 성공

### 200 OK

일반적인 요청 처리 성공이다.

```http
GET /users/10
```

```http
200 OK
```

### 201 Created

새로운 자원이 생성되었음을 나타낸다.

```http
POST /users
```

```http
201 Created
```

### 204 No Content

요청 처리에는 성공했지만 응답 본문이 없음을 나타낸다.

```http
DELETE /users/10
```

```http
204 No Content
```

---

## 12.2 3xx 리다이렉션

### 301 Moved Permanently

요청한 주소가 영구적으로 변경되었다.

### 302 Found

요청한 주소가 임시로 다른 위치로 이동했다.

### 307 Temporary Redirect

HTTP 메서드를 유지하면서 임시로 다른 주소로 이동한다.

FastAPI에서는 `/items`와 `/items/`가 다를 때 리다이렉션이 발생할 수 있다.

---

## 12.3 4xx 클라이언트 오류

### 400 Bad Request

클라이언트가 보낸 요청의 형식이나 내용이 잘못되었다.

```text
JSON 문법 오류
잘못된 요청 데이터
```

### 401 Unauthorized

인증되지 않은 요청이다.

```text
토큰이 없음
토큰이 만료됨
로그인 정보가 올바르지 않음
```

### 403 Forbidden

사용자가 누구인지는 확인되었지만 해당 작업을 수행할 권한이 없다.

```text
일반 사용자가 관리자 API에 접근한 경우
```

### 404 Not Found

요청한 경로나 자원을 찾을 수 없다.

```http
GET /users/99999
```

FastAPI에 해당 URL이 등록되지 않은 경우에도 발생한다.

```json
{
  "detail": "Not Found"
}
```

### 405 Method Not Allowed

URL은 존재하지만 HTTP 메서드가 잘못되었다.

다음과 같이 GET만 등록된 경우를 생각해 보자.

```python
@app.get("/users")
def get_users():
    return []
```

다음 POST 요청을 보내면 405 오류가 발생할 수 있다.

```http
POST /users
```

### 409 Conflict

현재 서버 상태와 요청이 충돌한다.

```text
이미 존재하는 이메일로 회원가입
중복된 사용자 이름 생성
```

### 422 Unprocessable Content

요청을 읽을 수는 있지만 입력값 검증에 실패했다.

```python
@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id}
```

다음 요청에서는 `abc`를 정수로 변환할 수 없다.

```http
GET /items/abc
```

FastAPI는 입력값 검증 실패 응답을 반환한다.

### 429 Too Many Requests

짧은 시간 동안 너무 많은 요청을 보냈다.

---

## 12.4 5xx 서버 오류

### 500 Internal Server Error

서버 코드를 실행하는 중 예상하지 못한 오류가 발생했다.

```python
result = 10 / 0
```

### 502 Bad Gateway

앞단 서버가 뒤쪽 서버에서 정상적인 응답을 받지 못했다.

```text
클라이언트
    ↓
Nginx
    ↓ 연결 실패
FastAPI
```

### 503 Service Unavailable

서버가 일시적으로 요청을 처리할 수 없다.

```text
서버 점검
트래픽 과부하
```

### 504 Gateway Timeout

앞단 서버가 뒤쪽 서버의 응답을 기다리다가 시간 초과가 발생했다.

---

# 13. REST API

REST는 웹 자원을 일관된 방식으로 표현하고 조작하기 위한 아키텍처 스타일이다.

실무에서 가장 중요한 원칙은 다음과 같다.

```text
URL은 자원을 표현한다.
HTTP 메서드는 동작을 표현한다.
```

사용자 자원은 다음과 같이 표현할 수 있다.

```text
/users
/users/10
```

동작은 HTTP 메서드로 구분한다.

```http
GET    /users
POST   /users
GET    /users/10
PATCH  /users/10
DELETE /users/10
```

---

# 14. REST API URL 설계 원칙

## 14.1 URL에는 명사를 사용한다

권장되는 형태는 다음과 같다.

```http
GET /users
GET /users/10
POST /orders
```

다음처럼 동사를 URL에 포함하는 방식은 일반적으로 권장되지 않는다.

```http
GET /getUsers
POST /createUser
POST /deleteUser
```

동작은 HTTP 메서드가 이미 나타내고 있기 때문이다.

```text
GET + /users
사용자 조회
```

## 14.2 자원 이름은 복수형을 사용한다

```text
/users
/products
/orders
/posts
```

특정 자원을 조회할 때는 식별자를 추가한다.

```text
/users/10
```

## 14.3 계층 관계를 URL로 표현한다

특정 사용자의 주문 목록은 다음과 같이 표현할 수 있다.

```http
GET /users/10/orders
```

특정 사용자의 특정 주문은 다음과 같다.

```http
GET /users/10/orders/50
```

계층 구조는 다음과 같이 해석할 수 있다.

```text
users
└── user 10
    └── orders
        └── order 50
```

다만 URL 계층이 지나치게 깊어지지 않도록 주의해야 한다.

```text
/companies/1/departments/2/teams/3/users/4/orders/5
```

주문 자체를 식별할 수 있다면 다음처럼 단순화할 수 있다.

```text
/orders/5
```

## 14.4 필터링은 Query Parameter로 표현한다

```http
GET /products?category=book
GET /products?min_price=10000&max_price=30000
```

## 14.5 정렬

```http
GET /products?sort=price
GET /products?sort=-price
```

API 규칙에 따라 `-price`를 가격 내림차순으로 정의할 수 있다.

## 14.6 페이지네이션

```http
GET /products?page=2&size=20
```

또는 다음과 같이 표현할 수도 있다.

```http
GET /products?offset=20&limit=20
```

---

# 15. REST의 무상태성

REST의 중요한 원칙 중 하나는 무상태성이다.

무상태성은 서버가 이전 요청을 기억하고 있다고 전제하지 않는다는 의미다.

각 요청은 처리에 필요한 정보를 자체적으로 포함해야 한다.

예를 들어 인증이 필요한 API를 호출할 때마다 인증 토큰을 전달할 수 있다.

```http
GET /users/me
Authorization: Bearer access-token
```

서버는 현재 요청에 포함된 토큰을 이용해 사용자를 확인한다.

---

# 16. CRUD로 표현하기 어려운 동작

모든 비즈니스 동작이 단순한 생성, 조회, 수정, 삭제로 표현되지는 않는다.

예를 들어 주문 취소는 다음과 같이 표현할 수 있다.

```http
POST /orders/10/cancel
```

또는 주문의 상태를 수정하는 방법도 있다.

```http
PATCH /orders/10
```

```json
{
  "status": "cancelled"
}
```

두 방식 중 하나만 절대적으로 올바른 것은 아니다.

API 전체에서 의미와 설계 방식이 일관적인지가 더 중요하다.

---

# 17. FastAPI

FastAPI는 Python으로 API 서버를 만들기 위한 웹 프레임워크다.

FastAPI는 다음과 같은 기능을 제공한다.

```text
URL과 Python 함수 연결
HTTP 메서드 처리
입력값 변환
입력값 검증
JSON 응답 생성
오류 응답 처리
API 문서 자동 생성
의존성 주입
미들웨어
인증 기능 구현 지원
```

기본 코드는 다음과 같다.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

## 17.1 `app = FastAPI()`

FastAPI 애플리케이션 객체를 생성한다.

이 객체에는 다음과 같은 정보가 등록된다.

```text
URL
HTTP 메서드
실행할 함수
입력값 규칙
응답 형식
미들웨어
예외 처리기
```

## 17.2 `@app.get("/")`

`@app.get("/")`은 Python 데코레이터다.

다음 의미를 가진다.

```text
GET 방식으로 "/" 요청이 들어오면
아래에 있는 함수를 실행한다.
```

```python
@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

내부적으로는 다음 관계가 등록된다고 이해할 수 있다.

```text
GET /
    ↓
read_root 함수
```

## 17.3 반환값

```python
return {"message": "Hello World"}
```

FastAPI는 Python 딕셔너리를 JSON 응답으로 변환한다.

```json
{
  "message": "Hello World"
}
```

---

# 18. FastAPI의 기반 기술

FastAPI는 내부적으로 Starlette와 Pydantic을 활용한다.

## 18.1 Starlette

Starlette는 웹 서버 기능을 담당하는 ASGI 프레임워크다.

주요 역할은 다음과 같다.

```text
라우팅
요청과 응답 처리
미들웨어
WebSocket
ASGI 처리
```

## 18.2 Pydantic

Pydantic은 데이터 검증과 변환을 담당한다.

주요 역할은 다음과 같다.

```text
문자열을 정수로 변환
필수값 검사
입력값 범위 검사
요청 본문 검증
응답 데이터 구조화
```

예시는 다음과 같다.

```python
from pydantic import BaseModel


class UserCreate(BaseModel):
    name: str
    age: int
```

```python
@app.post("/users")
def create_user(user: UserCreate):
    return user
```

정상적인 요청은 다음과 같다.

```json
{
  "name": "Kim",
  "age": 25
}
```

다음 요청은 `age`를 정수로 변환할 수 없기 때문에 검증에 실패한다.

```json
{
  "name": "Kim",
  "age": "abc"
}
```

---

# 19. FastAPI와 Uvicorn의 차이

FastAPI와 Uvicorn은 서로 다른 역할을 한다.

## 19.1 FastAPI

FastAPI는 요청을 어떻게 처리할지 정의한다.

```text
어떤 URL을 제공할 것인가
어떤 HTTP 메서드를 사용할 것인가
어떤 함수를 실행할 것인가
입력값을 어떻게 검증할 것인가
무엇을 응답할 것인가
```

## 19.2 Uvicorn

Uvicorn은 실제로 네트워크 요청을 받는 ASGI 서버다.

```text
특정 포트를 연다.
클라이언트 연결을 받는다.
HTTP 요청을 해석한다.
요청을 FastAPI에 전달한다.
FastAPI의 응답을 클라이언트에 전달한다.
```

FastAPI 애플리케이션 객체를 생성하는 것만으로는 네트워크 포트가 열리지 않는다.

```python
app = FastAPI()
```

Uvicorn을 실행해야 실제 요청을 받을 수 있다.

```bash
uvicorn main:app --reload
```

---

# 20. Uvicorn 실행 명령어

다음 명령어를 구성 요소별로 살펴보자.

```bash
uvicorn main:app --reload
```

```text
uvicorn
실행할 ASGI 서버 프로그램

main
main.py 파일 또는 main 모듈

app
main.py 안에 있는 FastAPI 객체

--reload
코드가 수정되면 서버를 자동으로 다시 실행
```

다음과 같은 파일이 있다고 가정한다.

```text
main.py
```

```python
from fastapi import FastAPI

app = FastAPI()
```

파일명이 `server.py`이고 FastAPI 객체 이름이 `api`라면 다음과 같이 실행한다.

```python
from fastapi import FastAPI

api = FastAPI()
```

```bash
uvicorn server:api --reload
```

## 20.1 호스트와 포트 지정

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

`127.0.0.1`은 현재 컴퓨터 내부에서만 접근할 수 있다.

`0.0.0.0`은 현재 컴퓨터의 모든 네트워크 인터페이스에서 요청을 받겠다는 의미다.

Docker 컨테이너에서 FastAPI를 실행할 때는 일반적으로 `0.0.0.0`을 사용한다.

---

# 21. WSGI와 ASGI

WSGI와 ASGI는 웹 서버와 Python 웹 애플리케이션 사이의 통신 규격이다.

```text
웹 서버
    ↓ WSGI 또는 ASGI
Python 웹 애플리케이션
```

## 21.1 WSGI

WSGI는 전통적인 동기 Python 웹 애플리케이션을 위한 규격이다.

```text
WSGI = Web Server Gateway Interface
```

대표적인 조합은 다음과 같다.

```text
Gunicorn
    ↓ WSGI
Flask 또는 Django
```

기본적인 요청 처리 흐름은 다음과 같다.

```text
요청이 들어온다.
    ↓
Python 함수가 실행된다.
    ↓
함수가 응답을 반환한다.
```

WSGI는 일반적인 동기 HTTP 요청과 응답을 처리하는 데 적합하다.

다만 WebSocket, 장시간 연결, 비동기 스트리밍 같은 기능은 자연스럽게 처리하기 어렵다.

## 21.2 ASGI

ASGI는 비동기 처리를 지원하는 Python 웹 애플리케이션 규격이다.

```text
ASGI = Asynchronous Server Gateway Interface
```

대표적인 조합은 다음과 같다.

```text
Uvicorn
    ↓ ASGI
FastAPI
```

ASGI는 다음과 같은 기능을 지원하기 적합하다.

```text
일반 HTTP 요청
비동기 요청 처리
WebSocket
실시간 통신
스트리밍
애플리케이션 시작과 종료 이벤트
```

## 21.3 관계 정리

```text
WSGI 서버
    ↓
WSGI 인터페이스
    ↓
Flask 또는 전통적인 Django
```

```text
ASGI 서버
    ↓
ASGI 인터페이스
    ↓
FastAPI 또는 Starlette
```

FastAPI는 ASGI 애플리케이션이고 Uvicorn은 ASGI 서버다.

---

# 22. 동기와 비동기 처리

## 22.1 동기 처리

동기 처리는 하나의 작업이 끝날 때까지 기다린 후 다음 작업을 처리하는 방식이다.

```text
작업 A 시작
작업 A 완료
작업 B 시작
작업 B 완료
```

Python에서는 일반 함수로 표현할 수 있다.

```python
def get_user():
    result = database_query()
    return result
```

`database_query()`가 끝날 때까지 함수 실행은 멈춰 있다.

## 22.2 비동기 처리

비동기 처리는 한 작업이 외부 응답을 기다리는 동안 다른 작업을 처리할 수 있는 방식이다.

```text
작업 A가 데이터베이스 응답을 기다림
    ↓
작업 B 처리
    ↓
작업 A의 데이터베이스 응답 도착
    ↓
작업 A 처리 재개
```

Python에서는 `async`와 `await`를 사용한다.

```python
async def get_user():
    result = await database_query()
    return result
```

비동기는 하나의 요청 자체를 무조건 더 빠르게 만드는 기능이 아니다.

데이터베이스 응답을 받는 데 1초가 걸린다면 해당 요청은 여전히 약 1초를 기다려야 한다.

다만 그 1초 동안 서버가 다른 요청을 처리할 수 있다.

---

# 23. 동시성과 병렬성

동시성과 병렬성은 서로 다른 개념이다.

## 23.1 동시성

여러 작업을 번갈아 처리하는 방식이다.

```text
작업 A 일부 처리
작업 B 일부 처리
작업 A 다시 처리
작업 B 다시 처리
```

비동기 프로그래밍은 주로 동시성을 높이는 데 사용된다.

## 23.2 병렬성

여러 작업을 실제로 동시에 실행하는 방식이다.

```text
CPU 코어 1 → 작업 A
CPU 코어 2 → 작업 B
```

병렬성을 위해서는 여러 프로세스나 여러 CPU 코어를 활용할 수 있다.

---

# 24. 비동기가 효과적인 작업

비동기는 주로 I/O 대기가 많은 작업에 효과적이다.

대표적인 I/O 작업은 다음과 같다.

```text
데이터베이스 조회
외부 API 요청
파일 읽기
네트워크 통신
Redis 조회
```

예시는 다음과 같다.

```python
@app.get("/weather")
async def get_weather():
    result = await call_external_api()
    return result
```

외부 API 응답을 기다리는 동안 서버가 다른 요청을 처리할 수 있다.

---

# 25. 비동기가 크게 도움 되지 않는 작업

CPU를 계속 사용하는 작업은 `async`를 사용한다고 자동으로 빨라지지 않는다.

대표적인 CPU 중심 작업은 다음과 같다.

```text
대규모 행렬 연산
영상 변환
복잡한 암호화
대용량 모델 추론
매우 큰 반복문
```

다음 코드에서 `async`를 붙였더라도 CPU 계산이 진행되는 동안 이벤트 루프가 막힐 수 있다.

```python
@app.get("/calculate")
async def calculate():
    result = very_heavy_cpu_calculation()
    return result
```

CPU 중심 작업은 다음과 같은 방법을 고려할 수 있다.

```text
별도 프로세스
멀티프로세싱
작업 큐
Celery
별도 모델 서버
GPU 서버
```

---

# 26. FastAPI의 `def`와 `async def`

FastAPI에서는 일반 함수와 비동기 함수를 모두 사용할 수 있다.

## 일반 함수

```python
@app.get("/sync")
def sync_endpoint():
    return {"type": "sync"}
```

## 비동기 함수

```python
@app.get("/async")
async def async_endpoint():
    return {"type": "async"}
```

비동기 데이터베이스나 비동기 HTTP 클라이언트를 사용한다면 `async def`가 적합하다.

```python
@app.get("/users")
async def get_users():
    users = await async_database.find_all()
    return users
```

동기 라이브러리를 사용한다면 일반 `def`를 사용할 수 있다.

```python
@app.get("/users")
def get_users():
    users = sync_database.find_all()
    return users
```

---

# 27. 비동기 함수 안의 블로킹 코드

다음 코드는 비동기 함수 안에서 동기 대기 함수를 사용하고 있다.

```python
import time


@app.get("/bad")
async def bad_endpoint():
    time.sleep(5)
    return {"message": "done"}
```

`time.sleep(5)`가 실행되는 동안 이벤트 루프가 막힐 수 있다.

비동기 환경에서는 다음과 같이 작성한다.

```python
import asyncio


@app.get("/good")
async def good_endpoint():
    await asyncio.sleep(5)
    return {"message": "done"}
```

`await asyncio.sleep(5)`로 기다리는 동안 이벤트 루프는 다른 요청을 처리할 수 있다.

---

# 28. 이벤트 루프

이벤트 루프는 비동기 작업의 실행 상태를 관리한다.

```text
1. 요청 A 실행
2. 요청 A가 데이터베이스 응답을 기다림
3. 요청 B 실행
4. 요청 B가 외부 API 응답을 기다림
5. 요청 A의 데이터베이스 응답 도착
6. 요청 A 실행 재개
7. 요청 B의 API 응답 도착
8. 요청 B 실행 재개
```

`await`는 다음 의미로 이해할 수 있다.

```text
이 작업의 결과를 기다려야 하지만
기다리는 동안 다른 작업을 처리해도 된다.
```

모든 함수에 `await`를 붙일 수 있는 것은 아니다.

비동기를 지원하는 함수에만 `await`를 사용할 수 있다.

---

# 29. Path Parameter

Path Parameter는 URL 경로 안에 포함되는 값이다.

```text
/users/10
```

위 주소에서 `10`은 특정 사용자를 식별한다.

FastAPI에서는 다음과 같이 작성한다.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

다음 요청이 들어오면

```http
GET /users/10
```

FastAPI는 다음과 같이 함수를 호출한다.

```python
get_user(user_id=10)
```

## 29.1 타입 변환

URL을 통해 전달되는 값은 기본적으로 문자열이다.

```text
"10"
```

FastAPI는 타입 힌트를 보고 값을 정수로 변환한다.

```python
user_id: int
```

```text
"10" → 10
```

다음 요청은 정수로 변환할 수 없기 때문에 검증에 실패한다.

```http
GET /users/abc
```

---

# 30. Query Parameter

Query Parameter는 URL의 `?` 뒤에 전달되는 값이다.

```text
/products?category=book&page=2
```

FastAPI에서는 다음과 같이 작성한다.

```python
@app.get("/products")
def get_products(
    category: str | None = None,
    page: int = 1,
):
    return {
        "category": category,
        "page": page,
    }
```

다음 요청이 들어오면

```http
GET /products?category=book&page=2
```

함수는 개념적으로 다음과 같이 호출된다.

```python
get_products(category="book", page=2)
```

## 30.1 기본값

```python
page: int = 1
```

요청에 `page`가 없다면 기본값인 1을 사용한다.

```http
GET /products
```

응답은 다음과 같다.

```json
{
  "category": null,
  "page": 1
}
```

## 30.2 선택값

```python
category: str | None = None
```

`category` 값은 전달하지 않아도 된다는 의미다.

---

# 31. Path Parameter와 Query Parameter의 차이

Path Parameter는 특정 자원을 식별할 때 사용한다.

```text
/users/10
/posts/50
/orders/100
```

일반적으로 Path Parameter는 필수값이다.

Query Parameter는 검색, 필터링, 정렬, 페이지네이션과 같은 추가 조건을 전달할 때 사용한다.

```text
/users?age=20
/products?sort=price
/posts?page=2&size=10
```

다음 기준으로 구분할 수 있다.

```text
값이 없으면 어떤 자원을 요청하는지 알 수 없는가?
→ Path Parameter

검색, 필터, 정렬, 페이지와 같은 옵션인가?
→ Query Parameter
```

예를 들어 다음 요청에서 `10`은 특정 사용자를 식별한다.

```http
GET /users/10
```

다음 요청에서 `age=20`은 사용자 조회 조건이다.

```http
GET /users?age=20
```

---

# 32. Path Parameter, Query Parameter, Request Body

세 가지 입력 방식의 역할을 구분해야 한다.

## 32.1 Path Parameter

어떤 자원을 대상으로 할 것인지 나타낸다.

```http
PATCH /users/10
```

```text
10번 사용자를 수정한다.
```

## 32.2 Query Parameter

어떤 조건이나 옵션으로 조회할 것인지 나타낸다.

```http
GET /users?active=true
```

```text
활성 상태인 사용자를 조회한다.
```

## 32.3 Request Body

어떤 데이터로 생성하거나 수정할 것인지 나타낸다.

```http
PATCH /users/10
```

```json
{
  "name": "Lee",
  "age": 30
}
```

```text
10번 사용자의 이름과 나이를 수정한다.
```

---

# 33. FastAPI 실습 코드

다음 코드는 Path Parameter, Query Parameter, Request Body를 모두 포함한다.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class ItemCreate(BaseModel):
    name: str
    price: int
    description: str | None = None


@app.get("/")
def read_root():
    return {"message": "Hello World"}


@app.get("/items")
def read_items(
    keyword: str | None = None,
    page: int = 1,
    size: int = 10,
):
    return {
        "keyword": keyword,
        "page": page,
        "size": size,
    }


@app.get("/items/{item_id}")
def read_item(
    item_id: int,
    detail: bool = False,
):
    return {
        "item_id": item_id,
        "detail": detail,
    }


@app.post("/items", status_code=201)
def create_item(item: ItemCreate):
    return {
        "id": 1,
        "name": item.name,
        "price": item.price,
        "description": item.description,
    }
```

서버는 다음 명령어로 실행한다.

```bash
uvicorn main:app --reload
```

---

# 34. 실습 요청 예시

## 34.1 루트 경로

```http
GET http://127.0.0.1:8000/
```

응답은 다음과 같다.

```json
{
  "message": "Hello World"
}
```

## 34.2 Query Parameter

```http
GET http://127.0.0.1:8000/items?keyword=phone&page=2&size=5
```

응답은 다음과 같다.

```json
{
  "keyword": "phone",
  "page": 2,
  "size": 5
}
```

## 34.3 Path Parameter와 Query Parameter

```http
GET http://127.0.0.1:8000/items/10?detail=true
```

응답은 다음과 같다.

```json
{
  "item_id": 10,
  "detail": true
}
```

## 34.4 Request Body

```http
POST http://127.0.0.1:8000/items
Content-Type: application/json
```

요청 본문은 다음과 같다.

```json
{
  "name": "Keyboard",
  "price": 50000,
  "description": "Mechanical keyboard"
}
```

응답은 다음과 같다.

```json
{
  "id": 1,
  "name": "Keyboard",
  "price": 50000,
  "description": "Mechanical keyboard"
}
```

---

# 35. FastAPI 자동 API 문서

FastAPI 서버를 실행하면 자동으로 API 문서가 생성된다.

Swagger UI는 다음 주소에서 확인할 수 있다.

```text
http://127.0.0.1:8000/docs
```

ReDoc은 다음 주소에서 확인할 수 있다.

```text
http://127.0.0.1:8000/redoc
```

FastAPI는 다음 정보를 이용해 문서를 생성한다.

```text
라우터의 URL
HTTP 메서드
Path Parameter
Query Parameter
Pydantic 모델
응답 상태 코드
Python 타입 힌트
```

따라서 FastAPI에서 타입 힌트는 단순한 코드 설명이 아니다.

다음 기능에도 사용된다.

```text
입력값 변환
입력값 검증
에디터 자동 완성
API 문서 생성
```

---

# 36. `detail: Not Found`가 발생하는 이유

FastAPI는 등록된 HTTP 메서드와 URL을 기준으로 실행할 함수를 찾는다.

다음 코드에는 `GET /`만 등록되어 있다.

```python
@app.get("/")
def read_root():
    return {"message": "Hello"}
```

다음 요청은 성공한다.

```text
GET http://127.0.0.1:8000/
```

그러나 다음 URL은 등록되어 있지 않다.

```text
GET http://127.0.0.1:8000/test
```

따라서 FastAPI는 404 응답을 반환한다.

```json
{
  "detail": "Not Found"
}
```

또 다른 흔한 원인은 잘못된 파일을 실행하는 것이다.

프로젝트 구조가 다음과 같다고 가정하자.

```text
project/
├── main.py
└── old_main.py
```

`main.py`를 수정했지만 다음 명령어를 실행하면 `old_main.py`의 애플리케이션이 실행된다.

```bash
uvicorn old_main:app --reload
```

따라서 새로 작성한 API 경로가 나타나지 않을 수 있다.

---

# 37. 라우팅 순서

고정된 경로와 Path Parameter 경로가 겹칠 수 있다.

```python
@app.get("/users/{user_id}")
def get_user(user_id: str):
    return {"user_id": user_id}


@app.get("/users/me")
def get_me():
    return {"user": "current user"}
```

`/users/me`의 `me`가 `{user_id}` 값으로 처리될 가능성을 줄이려면 고정 경로를 먼저 선언하는 것이 좋다.

```python
@app.get("/users/me")
def get_me():
    return {"user": "current user"}


@app.get("/users/{user_id}")
def get_user(user_id: str):
    return {"user_id": user_id}
```

일반적으로 구체적인 고정 경로를 먼저 작성하고 동적인 경로를 나중에 작성한다.

---

# 38. 실무에서 사용하는 백엔드 계층 구조

프로젝트가 커지면 모든 로직을 하나의 FastAPI 함수에 작성하지 않는다.

일반적으로 다음과 같이 역할을 분리한다.

```text
클라이언트
    ↓
Router
    ↓
Service
    ↓
Repository
    ↓
Database
```

## 38.1 Router

Router는 HTTP 요청과 응답을 담당한다.

```python
@router.get("/users/{user_id}")
def get_user(user_id: int):
    return user_service.get_user(user_id)
```

Router의 주요 역할은 다음과 같다.

```text
URL 정의
HTTP 메서드 정의
입력값 수신
Service 호출
응답 반환
```

## 38.2 Service

Service는 비즈니스 로직을 담당한다.

```python
def get_user(user_id: int):
    user = user_repository.find_by_id(user_id)

    if user is None:
        raise UserNotFoundError()

    return user
```

Service의 주요 역할은 다음과 같다.

```text
업무 규칙 처리
권한 검사
데이터 조합
트랜잭션 처리
```

## 38.3 Repository

Repository는 데이터베이스 접근을 담당한다.

```python
def find_by_id(user_id: int):
    return db.query(User).filter(User.id == user_id).first()
```

Repository의 주요 역할은 다음과 같다.

```text
데이터 조회
데이터 저장
데이터 수정
데이터 삭제
```

이렇게 계층을 분리하면 HTTP 처리, 비즈니스 로직, 데이터베이스 접근 코드가 서로 섞이지 않는다.

---

# 39. 요청 하나가 처리되는 실제 과정

다음 요청이 들어왔다고 가정해 보자.

```http
GET /items/10?q=apple
```

## 39.1 클라이언트가 URL을 해석한다

```text
호스트: 127.0.0.1
포트: 8000
경로: /items/10
쿼리: q=apple
메서드: GET
```

## 39.2 운영체제가 요청을 Uvicorn에 전달한다

Uvicorn은 8000번 포트를 열고 요청을 기다리고 있다.

## 39.3 Uvicorn이 HTTP 요청을 해석한다

Uvicorn은 HTTP 요청을 ASGI 형식으로 변환한다.

개념적으로 다음과 같은 정보가 만들어진다.

```python
{
    "type": "http",
    "method": "GET",
    "path": "/items/10",
    "query_string": "q=apple",
}
```

## 39.4 FastAPI가 적절한 라우트를 찾는다

등록된 라우트는 다음과 같다.

```python
@app.get("/items/{item_id}")
```

요청 경로는 다음과 같다.

```text
GET /items/10
```

FastAPI는 두 경로가 일치한다고 판단한다.

## 39.5 입력값을 추출하고 검증한다

```text
item_id = "10"
q = "apple"
```

타입 힌트에 따라 `item_id`를 정수로 변환한다.

```text
"10" → 10
```

## 39.6 함수를 실행한다

```python
read_item(item_id=10, q="apple")
```

## 39.7 반환값을 JSON으로 변환한다

Python 딕셔너리는 다음과 같다.

```python
{
    "item_id": 10,
    "q": "apple",
}
```

JSON으로 변환하면 다음과 같다.

```json
{
  "item_id": 10,
  "q": "apple"
}
```

## 39.8 HTTP 응답을 생성한다

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "item_id": 10,
  "q": "apple"
}
```

## 39.9 Uvicorn이 응답을 클라이언트에 전달한다

클라이언트는 응답을 받아 화면에 표시하거나 다음 로직을 실행한다.

---

# 40. 직렬화와 역직렬화

## 40.1 역직렬화

역직렬화는 JSON과 같은 외부 데이터를 Python 객체로 변환하는 과정이다.

```json
{
  "name": "Kim",
  "age": 25
}
```

다음과 같은 Python 객체로 변환될 수 있다.

```python
UserCreate(name="Kim", age=25)
```

## 40.2 직렬화

직렬화는 Python 객체를 JSON과 같이 전송 가능한 데이터 형식으로 변환하는 과정이다.

```python
{
    "name": "Kim",
    "age": 25,
}
```

다음 JSON으로 변환된다.

```json
{
  "name": "Kim",
  "age": 25
}
```

FastAPI와 Pydantic이 이 과정을 대부분 자동으로 처리한다.

---

# 41. 핵심 정리

## 클라이언트와 서버

```text
클라이언트는 요청을 보낸다.
서버는 요청을 처리하고 응답한다.
```

## 2티어와 3티어

```text
2티어는 클라이언트와 데이터 계층이 직접 연결되는 단순한 구조다.

3티어는 프론트엔드, 백엔드, 데이터베이스를 분리한 구조다.
```

## HTTP

```text
클라이언트와 서버가 통신하기 위한 규칙이다.
```

## HTTP 메서드

```text
GET    조회
POST   생성 또는 작업 실행
PUT    전체 수정
PATCH  일부 수정
DELETE 삭제
```

## 상태 코드

```text
2xx 성공
3xx 리다이렉션
4xx 클라이언트 요청 오류
5xx 서버 오류
```

## REST API

```text
URL은 자원을 표현한다.
HTTP 메서드는 동작을 표현한다.
```

## FastAPI

```text
HTTP 요청을 Python 함수와 연결하는 ASGI 웹 프레임워크다.
```

## Uvicorn

```text
네트워크 요청을 실제로 받는 ASGI 서버다.
```

## WSGI와 ASGI

```text
WSGI는 전통적인 동기 웹 애플리케이션 인터페이스다.

ASGI는 비동기 처리, WebSocket, 스트리밍 등을 지원하는 인터페이스다.
```

## Path Parameter

```text
특정 자원을 식별하는 URL 경로 값이다.

/users/10
```

## Query Parameter

```text
필터, 검색, 정렬, 페이지 등의 옵션을 전달한다.

/users?age=20
```

전체 구조를 한 줄로 표현하면 다음과 같다.

```text
클라이언트 → HTTP → Uvicorn → ASGI → FastAPI → Python 함수 → JSON 응답
```
