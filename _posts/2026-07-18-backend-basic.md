---

title: "백엔드 기초: 클라이언트, 서버, API, REST API"
date: 2026-07-18
categories: [Backend]
tags: [Backend, Client, Server, API, HTTP, REST]
---

백엔드 공부 전에 알아야 할 클라이언트, 서버, API, HTTP, REST API 개념 정리

## 1. 클라이언트와 서버

웹 서비스는 일반적으로 클라이언트가 서버에 요청을 보내고, 서버가 결과를 응답하는 구조로 동작

| 구분     | 역할                 | 예시                      |
| ------ | ------------------ | ----------------------- |
| 클라이언트  | 서버에 요청을 보내고 결과를 사용 | 브라우저, 모바일 앱, React      |
| 서버     | 요청을 처리하고 응답을 반환    | FastAPI, Spring, Django |
| 데이터베이스 | 데이터를 저장하고 조회       | PostgreSQL, MySQL       |

전체 흐름

```text
클라이언트 → 요청 → 서버에서 처리 → 응답
```

사용자가 브라우저에서 상품 목록을 확인하는 경우

```text
브라우저에서 상품 목록 요청
→ 서버가 요청 확인
→ 데이터베이스에서 상품 조회
→ 조회 결과 반환
```

## 2. 프론트엔드와 백엔드

| 구분     | 역할                     | 대표 기술                        |
| ------ | ---------------------- | ---------------------------- |
| 프론트엔드  | 사용자가 보는 화면과 입력 처리      | HTML, CSS, JavaScript, React |
| 백엔드    | 요청 처리, 비즈니스 로직, 데이터 관리 | FastAPI, Spring, Django      |
| 데이터베이스 | 데이터 저장과 조회             | PostgreSQL, MySQL            |

### React

웹 화면을 만들기 위한 JavaScript 라이브러리

버튼, 입력창, 목록 등의 화면을 구성하고 백엔드 API를 호출하는 역할

```text
화면 표시
→ 사용자 입력 처리
→ 백엔드 API 호출
→ 응답 결과 표시
```

## 3. 2티어와 3티어 구조

### 2티어 구조

클라이언트가 데이터베이스에 직접 접근하는 구조

```text
클라이언트 → 데이터베이스
```

구조가 단순하지만 데이터베이스가 클라이언트에 직접 노출될 수 있고, 서비스가 커질수록 보안과 관리가 어려움

### 3티어 구조

클라이언트와 데이터베이스 사이에 백엔드 서버를 두는 구조

```text
클라이언트 → 백엔드 서버 → 데이터베이스
```

일반적인 웹 서비스 구조

```text
React → FastAPI → PostgreSQL
```

| 계층           | 역할                |
| ------------ | ----------------- |
| Presentation | 화면 표시와 사용자 입력 처리  |
| Application  | 요청 검증과 비즈니스 로직 처리 |
| Data         | 데이터 저장과 조회        |

## 4. API (Application Programming Interface)

프로그램끼리 정해진 방식으로 요청과 응답을 주고받기 위한 인터페이스

식당에 비유

| 웹 서비스  | 식당         |
| ------ | ---------- |
| 클라이언트  | 손님         |
| API    | 메뉴판과 주문 방식 |
| 서버     | 주방         |
| 응답 데이터 | 완성된 음식     |

클라이언트는 서버 내부 코드나 데이터베이스 구조를 알 필요 없이 API 규칙에 맞춰 요청

```text
클라이언트: 10번 사용자 정보를 요청
서버: 10번 사용자 정보를 응답
```

API 주소 예시

```text
/users       # 사용자 목록
/users/10    # 10번 사용자
/products    # 상품 목록
/orders      # 주문 목록
```

## 5. HTTP (HyperText Transfer Protocol)

클라이언트와 서버가 요청과 응답을 주고받기 위한 통신 규칙

서로 다른 기술로 개발된 프로그램도 HTTP 규칙을 따르면 통신 가능

```text
React
  ↓ HTTP
FastAPI
```

### HTTP 요청

| 구성     | 설명                     |
| ------ | ---------------------- |
| Method | 서버에 요청할 동작             |
| URL    | 요청할 대상                 |
| Header | 데이터 형식, 인증 정보 등의 부가 정보 |
| Body   | 생성하거나 수정할 실제 데이터       |

요청 예시

```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Kim",
  "age": 25
}
```

* `POST`: 사용자 생성 요청
* `/users`: 요청 대상
* `Content-Type`: 요청 데이터 형식
* Body: 서버에 전달할 실제 데이터

### HTTP 응답

응답에는 상태 코드, 헤더, 데이터 등이 포함됨

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 1,
  "name": "Kim",
  "age": 25
}
```

대표 상태 코드

| 상태 코드                       | 의미             |
| --------------------------- | -------------- |
| `200 OK`                    | 요청 성공          |
| `201 Created`               | 데이터 생성 성공      |
| `400 Bad Request`           | 잘못된 요청         |
| `401 Unauthorized`          | 인증 필요 또는 인증 실패 |
| `403 Forbidden`             | 접근 권한 없음       |
| `404 Not Found`             | 요청한 대상 없음      |
| `500 Internal Server Error` | 서버 내부 오류       |

## 6. HTTP 메서드

서버의 데이터에 어떤 작업을 요청할지 표현

| 메서드    | 용도        | 예시                 |
| ------ | --------- | ------------------ |
| GET    | 데이터 조회    | `GET /users/10`    |
| POST   | 데이터 생성    | `POST /users`      |
| PUT    | 데이터 전체 수정 | `PUT /users/10`    |
| PATCH  | 데이터 일부 수정 | `PATCH /users/10`  |
| DELETE | 데이터 삭제    | `DELETE /users/10` |

`PUT`은 기존 데이터를 전체적으로 교체하고, `PATCH`는 필요한 일부 값만 수정할 때 주로 사용

## 7. URL 구조

다음 주소를 기준으로 URL 구조 확인

```text
http://127.0.0.1:8000/users/10?detail=true
```

| 구분           | 값             | 의미               |
| ------------ | ------------- | ---------------- |
| Protocol     | `http`        | 통신 방식            |
| Host         | `127.0.0.1`   | 서버 주소            |
| Port         | `8000`        | 서버 프로그램을 구분하는 번호 |
| Path         | `/users/10`   | 요청할 리소스          |
| Query String | `detail=true` | 추가 조건            |

### IP 주소

네트워크에서 컴퓨터나 장치를 구분하는 주소

`127.0.0.1`과 `localhost`는 현재 사용 중인 컴퓨터를 가리킴

```text
http://127.0.0.1:8000
http://localhost:8000
```

### Port

한 컴퓨터에서 실행 중인 여러 프로그램을 구분하는 번호

| 주소               | 프로그램 예시          |
| ---------------- | ---------------- |
| `localhost:3000` | React 또는 Next.js |
| `localhost:8000` | FastAPI          |
| `localhost:5432` | PostgreSQL       |

같은 컴퓨터에서도 포트 번호가 다르면 서로 다른 프로그램에 연결

## 8. REST API (Representational State Transfer API)

리소스를 URL로 표현하고, 리소스에 수행할 동작을 HTTP 메서드로 표현하는 API 설계 방식

### 리소스(Resource)

API가 관리하는 대상

| 리소스    | URL 예시      |
| ------ | ----------- |
| 사용자 목록 | `/users`    |
| 특정 사용자 | `/users/10` |
| 상품 목록  | `/products` |
| 특정 주문  | `/orders/5` |

URL은 리소스를 식별하고, HTTP 메서드는 리소스에 수행할 동작을 표현

| 요청                 | 의미            |
| ------------------ | ------------- |
| `GET /users/10`    | 10번 사용자 조회    |
| `PATCH /users/10`  | 10번 사용자 일부 수정 |
| `DELETE /users/10` | 10번 사용자 삭제    |

핵심 원칙

```text
리소스: API가 관리하는 대상
URL: 리소스의 주소
HTTP 메서드: 리소스에 수행할 동작
```

사용자 API 예시

| 기능        | API                |
| --------- | ------------------ |
| 사용자 목록 조회 | `GET /users`       |
| 사용자 생성    | `POST /users`      |
| 특정 사용자 조회 | `GET /users/10`    |
| 특정 사용자 수정 | `PATCH /users/10`  |
| 특정 사용자 삭제 | `DELETE /users/10` |

### URL에는 명사 사용

권장: /users, /products, /orders, ...

비권장: /getUsers, /createUser, /deleteUser, ...

동작은 URL이 아니라 HTTP 메서드로 표현

### 리소스 이름은 주로 복수형 사용

```text
/users, /products, /orders, /posts, ...
```

### 리소스 간 관계 표현

10번 사용자의 주문 목록

```text
/users/10/orders
```

### 검색과 필터는 Query Parameter 사용

```text
/products?category=book   # 카테고리 필터
/products?sort=price      # 가격순 정렬
/products?page=2          # 2페이지 조회
```

## 9. JSON (JavaScript Object Notation)

클라이언트와 서버가 데이터를 주고받을 때 주로 사용하는 데이터 형식

```json
{
  "id": 1,
  "name": "Kim",
  "active": true
}
```

Python 타입과 비교

| JSON   | Python         |
| ------ | -------------- |
| Object | `dict`         |
| Array  | `list`         |
| String | `str`          |
| Number | `int`, `float` |
| null   | `None`         |
| true   | `True`         |
| false  | `False`        |

JSON의 key와 문자열은 큰따옴표 사용

```json
{
  "name": "Kim"
}
```

## 핵심 정리

| 개념       | 핵심                          |
| -------- | --------------------------- |
| 클라이언트    | 서버에 요청을 보내고 응답을 사용하는 프로그램   |
| 서버       | 요청을 처리하고 응답을 반환하는 프로그램      |
| 프론트엔드    | 사용자 화면과 입력 처리               |
| 백엔드      | 요청 처리와 비즈니스 로직 수행           |
| 데이터베이스   | 데이터 저장과 조회                  |
| API      | 프로그램끼리 통신하기 위한 인터페이스        |
| HTTP     | 클라이언트와 서버의 통신 규칙            |
| REST API | URL은 리소스, HTTP 메서드는 동작으로 설계 |
| JSON     | 요청과 응답에 주로 사용하는 데이터 형식      |

```text
사용자
→ React 화면
→ HTTP API 요청
→ 백엔드 서버
→ 데이터베이스 조회 또는 수정
→ HTTP 응답
→ React 화면에 결과 표시
```
