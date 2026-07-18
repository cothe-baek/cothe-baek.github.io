---
title: "FastAPI 백엔드 기초 정리"
date: 2026-07-18
categories: [Backend]
tags: [FastAPI, HTTP, REST API, ASGI, Uvicorn]
---

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