---
title: "FastAPI 백엔드 기초 정리"
date: 2026-07-18
categories: [Backend]
tags: [FastAPI, HTTP, REST API, ASGI, Uvicorn]
---

# FastAPI 백엔드 기초 정리

FastAPI 공부 전에 알아야 할 백엔드 기초 개념 정리

- 클라이언트와 서버
- 2티어와 3티어
- HTTP 요청과 응답
- HTTP 메서드와 상태 코드
- REST API
- FastAPI와 Uvicorn
- 동기와 비동기
- WSGI와 ASGI
- Path Parameter와 Query Parameter

---

## 1. 전체 요청 처리 흐름

클라이언트가 다음 주소로 요청을 보낸다고 가정하면

```text
http://127.0.0.1:8000/items/10?q=apple
```

FastAPI 서버에서는 다음과 같은 코드가 실행됨

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {
        "item_id": item_id,
        "q": q,
    }
```

전체 흐름

```text
클라이언트
    ↓ HTTP 요청
Uvicorn
    ↓
FastAPI
    ↓
요청 URL과 메서드에 맞는 함수 실행
    ↓
Python 객체 반환
    ↓
JSON 응답 변환
    ↓
클라이언트
```

요청

```http
GET /items/10?q=apple HTTP/1.1
Host: 127.0.0.1:8000
```

응답

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "item_id": 10,
  "q": "apple"
}
```

핵심 구조

```text
클라이언트 → HTTP → Uvicorn → FastAPI → Python 함수 → JSON 응답
```

---

## 2. 클라이언트와 서버

### 클라이언트

서버에 요청을 보내는 프로그램

예시

- 웹 브라우저
- 모바일 앱
- React, Next.js
- Postman
- 다른 백엔드 서버

### 서버

클라이언트의 요청을 받아 처리하고 응답을 반환하는 프로그램

일반적인 처리 과정

```text
요청 수신
→ 입력값 검증
→ 비즈니스 로직 실행
→ 데이터베이스 조회 또는 수정
→ 응답 반환
```

일반 컴퓨터에서도 서버 실행 가능

```bash
uvicorn main:app --reload
```

위 명령어 실행 시 현재 컴퓨터의 특정 포트에서 요청 대기

---

## 3. IP, 도메인, 포트

### IP 주소

네트워크에서 컴퓨터를 구분하는 주소

```text
127.0.0.1
192.168.0.10
```

`127.0.0.1`은 현재 컴퓨터를 의미

```text
http://127.0.0.1:8000
http://localhost:8000
```

두 주소는 일반적으로 같은 서버를 가리킴

### 도메인

IP 주소를 사람이 기억하기 쉬운 이름으로 표현

```text
google.com
naver.com
api.example.com
```

DNS가 도메인을 IP 주소로 변환

### 포트

한 컴퓨터에서 실행 중인 프로그램을 구분하는 번호

```text
localhost:3000  → Next.js
localhost:8000  → FastAPI
localhost:5432  → PostgreSQL
```

---

## 4. 2티어와 3티어

### 2티어 구조

클라이언트가 데이터베이스 또는 데이터 서버에 직접 접근하는 구조

```text
클라이언트
    ↓
데이터베이스
```

장점

- 구조가 단순함
- 빠른 개발 가능

단점

- 데이터베이스가 클라이언트에 노출될 수 있음
- 비즈니스 로직이 클라이언트에 분산될 수 있음
- 규모가 커질수록 유지보수가 어려움

### 3티어 구조

클라이언트, 백엔드, 데이터베이스를 분리한 구조

```text
클라이언트
    ↓ HTTP
백엔드 서버
    ↓ SQL
데이터베이스
```

예시

```text
React
    ↓
FastAPI
    ↓
PostgreSQL
```

각 계층의 역할

```text
클라이언트
화면 표시와 사용자 입력 처리

백엔드
요청 검증과 비즈니스 로직 처리

데이터베이스
데이터 저장과 조회
```

일반적인 웹 서비스에서 주로 사용하는 구조

---

## 5. HTTP

HTTP는 클라이언트와 서버가 통신하기 위한 규칙

```text
HTTP = HyperText Transfer Protocol
```

서로 다른 언어로 개발된 프로그램도 HTTP를 통해 통신 가능

```text
Next.js
    ↓ HTTP
FastAPI
```

---

## 6. HTTP 요청과 응답

### HTTP 요청 구조

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token

{
  "name": "Kim",
  "age": 25
}
```

구성 요소

```text
요청 시작 줄
헤더
빈 줄
본문
```

요청 시작 줄

```http
POST /users HTTP/1.1
```

- `POST`: HTTP 메서드
- `/users`: 요청 경로
- `HTTP/1.1`: HTTP 버전

주요 헤더

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer token
```

- `Content-Type`: 요청 본문의 데이터 형식
- `Accept`: 원하는 응답 데이터 형식
- `Authorization`: 인증 정보

요청 본문

```json
{
  "name": "Kim",
  "age": 25
}
```

서버에 전달할 실제 데이터

### HTTP 응답 구조

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "name": "Kim"
}
```

구성 요소

```text
상태 줄
헤더
빈 줄
본문
```

상태 줄

```http
HTTP/1.1 201 Created
```

- `201`: 상태 코드
- `Created`: 상태 메시지

---

## 7. HTTP 메서드

### GET

데이터 조회

```http
GET /users
GET /users/10
```

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

### POST

새로운 데이터 생성 또는 작업 실행

```http
POST /users
```

```python
@app.post("/users", status_code=201)
def create_user():
    return {"id": 1, "name": "Kim"}
```

### PUT

기존 데이터 전체 수정

```http
PUT /users/10
```

```json
{
  "name": "Lee",
  "age": 30,
  "city": "Seoul"
}
```

### PATCH

기존 데이터 일부 수정

```http
PATCH /users/10
```

```json
{
  "age": 30
}
```

### DELETE

데이터 삭제

```http
DELETE /users/10
```

정리

```text
GET     조회
POST    생성
PUT     전체 수정
PATCH   일부 수정
DELETE  삭제
```

---

## 8. HTTP 상태 코드

### 2xx: 성공

```text
200 OK
일반적인 요청 성공

201 Created
새로운 데이터 생성 성공

204 No Content
성공했지만 응답 본문 없음
```

### 3xx: 리다이렉션

```text
301 Moved Permanently
주소가 영구적으로 변경됨

307 Temporary Redirect
HTTP 메서드를 유지하며 임시 이동
```

FastAPI에서 `/items`와 `/items/` 차이로 리다이렉션이 발생할 수 있음

### 4xx: 클라이언트 요청 오류

```text
400 Bad Request
잘못된 요청

401 Unauthorized
인증 정보 없음 또는 인증 실패

403 Forbidden
접근 권한 없음

404 Not Found
경로나 데이터를 찾을 수 없음

405 Method Not Allowed
지원하지 않는 HTTP 메서드

409 Conflict
현재 데이터 상태와 요청이 충돌

422 Unprocessable Content
입력값 검증 실패
```

FastAPI 예시

```python
@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id}
```

다음 요청은 `abc`를 정수로 변환할 수 없으므로 422 응답

```http
GET /items/abc
```

### 5xx: 서버 오류

```text
500 Internal Server Error
서버 코드 실행 중 오류

502 Bad Gateway
앞단 서버가 뒤쪽 서버의 정상 응답을 받지 못함

503 Service Unavailable
서버가 일시적으로 요청을 처리할 수 없음

504 Gateway Timeout
뒤쪽 서버의 응답 대기 시간 초과
```

---

## 9. REST API

REST API의 핵심 원칙

```text
URL은 자원을 표현
HTTP 메서드는 동작을 표현
```

사용자 API 예시

```http
GET    /users
POST   /users
GET    /users/10
PATCH  /users/10
DELETE /users/10
```

### URL에는 명사 사용

권장

```http
GET /users
POST /users
DELETE /users/10
```

비권장

```http
GET /getUsers
POST /createUser
POST /deleteUser
```

### 자원 이름은 복수형 사용

```text
/users
/products
/orders
/posts
```

### 계층 관계 표현

특정 사용자의 주문 목록

```http
GET /users/10/orders
```

특정 주문 조회

```http
GET /orders/50
```

URL 계층은 필요한 만큼만 사용

### 필터링과 정렬

```http
GET /products?category=book
GET /products?sort=price
GET /products?page=2&size=20
```

검색, 필터, 정렬, 페이지 정보는 Query Parameter로 표현

---

## 10. FastAPI

FastAPI는 Python으로 API 서버를 개발하기 위한 웹 프레임워크

주요 기능

- URL과 Python 함수 연결
- 요청값 변환과 검증
- JSON 응답 생성
- 상태 코드 처리
- API 문서 자동 생성
- 비동기 처리 지원

기본 코드

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

### `app = FastAPI()`

FastAPI 애플리케이션 객체 생성

### `@app.get("/")`

`GET /` 요청과 `read_root` 함수 연결

```text
GET /
    ↓
read_root()
```

### 반환값

```python
return {"message": "Hello World"}
```

Python 딕셔너리가 JSON 응답으로 변환됨

```json
{
  "message": "Hello World"
}
```

---

## 11. FastAPI와 Uvicorn

### FastAPI

요청 처리 방법 정의

```text
어떤 URL을 사용할지
어떤 HTTP 메서드를 사용할지
어떤 함수를 실행할지
어떤 데이터를 반환할지
```

### Uvicorn

네트워크 요청을 실제로 받는 ASGI 서버

```text
포트 열기
HTTP 요청 수신
FastAPI에 요청 전달
응답을 클라이언트에 전달
```

실행 명령어

```bash
uvicorn main:app --reload
```

의미

```text
uvicorn  → 실행할 서버
main     → main.py
app      → main.py 안의 FastAPI 객체
--reload → 코드 변경 시 자동 재시작
```

파일명이 `server.py`, 객체명이 `api`라면

```bash
uvicorn server:api --reload
```

호스트와 포트 지정

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

---

## 12. WSGI와 ASGI

WSGI와 ASGI는 웹 서버와 Python 웹 애플리케이션 사이의 통신 규격

### WSGI

전통적인 동기 웹 애플리케이션 규격

```text
Gunicorn
    ↓ WSGI
Flask 또는 Django
```

일반적인 HTTP 요청과 응답 처리에 적합

### ASGI

비동기 처리를 지원하는 웹 애플리케이션 규격

```text
Uvicorn
    ↓ ASGI
FastAPI
```

지원하기 적합한 기능

- 비동기 HTTP 요청
- WebSocket
- 실시간 통신
- 스트리밍

정리

```text
FastAPI = ASGI 애플리케이션
Uvicorn = ASGI 서버
```

---

## 13. 동기와 비동기

### 동기 처리

한 작업이 끝난 후 다음 작업 처리

```text
작업 A 시작
작업 A 완료
작업 B 시작
작업 B 완료
```

```python
def get_user():
    result = database_query()
    return result
```

`database_query()`가 끝날 때까지 대기

### 비동기 처리

한 작업이 응답을 기다리는 동안 다른 작업 처리 가능

```text
작업 A가 데이터베이스 응답 대기
    ↓
작업 B 처리
    ↓
작업 A의 응답 도착
    ↓
작업 A 처리 재개
```

```python
async def get_user():
    result = await database_query()
    return result
```

비동기가 효과적인 작업

- 데이터베이스 요청
- 외부 API 요청
- 파일 입출력
- 네트워크 통신

비동기는 한 요청 자체를 무조건 빠르게 만드는 방식이 아님

대기 시간 동안 다른 요청을 처리해 서버의 동시 처리 성능을 높이는 방식

### CPU 중심 작업

다음 작업은 `async`만으로 빨라지지 않음

- 대규모 계산
- 영상 처리
- 머신러닝 모델 추론
- 큰 반복문

CPU 중심 작업은 별도 프로세스, 작업 큐, 모델 서버 등을 고려

---

## 14. `def`와 `async def`

일반 함수

```python
@app.get("/sync")
def sync_endpoint():
    return {"type": "sync"}
```

비동기 함수

```python
@app.get("/async")
async def async_endpoint():
    return {"type": "async"}
```

비동기 라이브러리 사용 시

```python
@app.get("/users")
async def get_users():
    users = await async_database.find_all()
    return users
```

동기 라이브러리 사용 시

```python
@app.get("/users")
def get_users():
    users = sync_database.find_all()
    return users
```

비동기 함수 안에서 동기 대기 함수를 사용하면 이벤트 루프가 막힐 수 있음

잘못된 예시

```python
import time


@app.get("/bad")
async def bad_endpoint():
    time.sleep(5)
    return {"message": "done"}
```

비동기 대기

```python
import asyncio


@app.get("/good")
async def good_endpoint():
    await asyncio.sleep(5)
    return {"message": "done"}
```

---

## 15. Path Parameter

URL 경로에 포함되는 값

```text
/users/10
```

`10`은 특정 사용자 식별값

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

요청

```http
GET /users/10
```

함수 실행

```python
get_user(user_id=10)
```

FastAPI가 문자열 `"10"`을 정수 `10`으로 변환

다음 요청은 정수 변환 실패

```http
GET /users/abc
```

Path Parameter는 특정 자원을 식별할 때 사용

---

## 16. Query Parameter

URL의 `?` 뒤에 전달되는 값

```text
/products?category=book&page=2
```

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

요청

```http
GET /products?category=book&page=2
```

함수 실행

```python
get_products(category="book", page=2)
```

기본값

```python
page: int = 1
```

`page`가 전달되지 않으면 1 사용

선택값

```python
category: str | None = None
```

`category`가 전달되지 않으면 `None` 사용

Query Parameter는 주로 다음 용도로 사용

- 검색
- 필터링
- 정렬
- 페이지네이션
- 선택 옵션

---

## 17. Path, Query, Body 차이

### Path Parameter

어떤 자원인지 식별

```http
GET /users/10
```

### Query Parameter

어떤 조건으로 조회할지 전달

```http
GET /users?active=true
```

### Request Body

어떤 데이터로 생성하거나 수정할지 전달

```http
PATCH /users/10
```

```json
{
  "name": "Lee",
  "age": 30
}
```

구분 기준

```text
자원 식별
→ Path Parameter

검색, 필터, 정렬, 페이지
→ Query Parameter

생성하거나 수정할 실제 데이터
→ Request Body
```

---

## 18. 실습 코드

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

서버 실행

```bash
uvicorn main:app --reload
```

API 문서

```text
http://127.0.0.1:8000/docs
```

---

## 19. 요청 예시

### Query Parameter

```http
GET http://127.0.0.1:8000/items?keyword=phone&page=2&size=5
```

응답

```json
{
  "keyword": "phone",
  "page": 2,
  "size": 5
}
```

### Path Parameter와 Query Parameter

```http
GET http://127.0.0.1:8000/items/10?detail=true
```

응답

```json
{
  "item_id": 10,
  "detail": true
}
```

### Request Body

```http
POST http://127.0.0.1:8000/items
Content-Type: application/json
```

```json
{
  "name": "Keyboard",
  "price": 50000,
  "description": "Mechanical keyboard"
}
```

---

## 20. 핵심 정리

```text
클라이언트: 서버에 요청을 보내는 프로그램
서버: 요청을 처리하고 응답을 반환하는 프로그램
HTTP: 클라이언트와 서버의 통신 규칙
REST API: URL은 자원, HTTP 메서드는 동작으로 표현
FastAPI: Python API 개발 프레임워크
Uvicorn: FastAPI 요청을 실제로 받는 ASGI 서버
WSGI: 전통적인 동기 Python 웹 규격
ASGI: 비동기 처리를 지원하는 Python 웹 규격
Path Parameter: 특정 자원을 식별하는 값
Query Parameter: 검색, 필터, 정렬 등의 조건값
```

전체 요청 흐름

```text
클라이언트
→ HTTP 요청
→ Uvicorn
→ FastAPI
→ Python 함수 실행
→ JSON 응답
→ 클라이언트
```