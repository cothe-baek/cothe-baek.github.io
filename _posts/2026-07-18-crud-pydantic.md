---

title: "FastAPI CRUD와 Pydantic: Task API 구현"
date: 2026-07-18 17:26:00 +0900
categories: [Backend]
tags: [FastAPI, CRUD, Pydantic, REST API, Request Body]
---

FastAPI와 Pydantic을 이용한 Task CRUD API 구현하기
(데이터베이스 대신 Python 딕셔너리를 임시 저장소로 사용)

## 1. CRUD란

CRUD는 데이터를 관리할 때 필요한 네 가지 기본 작업을 의미함

| 구분     | 전체 이름 | 의미         |
| ------ | ----- | ---------- |
| Create | 생성    | 새로운 데이터 추가 |
| Read   | 조회    | 저장된 데이터 확인 |
| Update | 수정    | 기존 데이터 변경  |
| Delete | 삭제    | 저장된 데이터 제거 |

웹 API에서는 HTTP 메서드와 URL을 조합해 CRUD 동작 표현함

### 1.1 HTTP 요청

REST API에서는 URL로 리소스를 표현하고 HTTP 메서드로 동작을 표현

요청 예시

```text
POST /tasks HTTP/1.1            → Request Line
Content-Type: application/json  → Header
                                → Header와 Body 구분하는 빈 줄
{                               → Body
  "title": "FastAPI 공부하기"
}
```


| 구성     | 역할            | 예시             |
| ------ | ------------- | -------------- |
| Method | 서버에 요청할 동작    | `GET`, `POST`  |
| URL    | 요청할 대상 (리소스)        | `/tasks/1`     |
| Header | 데이터 형식, 인증 정보 | `Content-Type` |
| Body   | 생성하거나 수정할 데이터 | JSON 데이터       |


### 1.3 Path, Query

#### Path Parameter

특정 리소스를 식별할 때 사용

```text
GET /tasks/10   → ID가 10인 task를 조회
```

```python
@app.get("/tasks/{task_id}")
def read_task(task_id: int):
    ...
```

#### Query Parameter

조회 조건, 필터, 정렬 등에 사용

```text
GET /tasks?completed=true   → 완료된 task만 조회
```

```python
@app.get("/tasks")
def read_tasks(completed: bool | None = None):
    ...
```


### 1.4 CRUD와 HTTP 메서드

| CRUD   | HTTP 메서드       | URL 예시            |
| ------ | -------------- | ----------------- |
| Create | `POST`         | `POST /tasks`     |
| Read   | `GET`          | `GET /tasks/1`    |
| Update | `PUT`, `PATCH` | `PATCH /tasks/1`  |
| Delete | `DELETE`       | `DELETE /tasks/1` |

### 1.5 PUT과 PATCH

두 메서드 모두 데이터 수정에 사용

| 메서드     | 의미        | 요청 방식      |
| ------- | --------- | ---------- |
| `PUT`   | 리소스 전체 교체 | 전체 필드 전달   |
| `PATCH` | 리소스 일부 수정 | 변경할 필드만 전달 |

전체 Task 교체 예시

```text
PUT /tasks/1

{
  "title": "수정된 제목",
  "description": "수정된 설명",
  "completed": true
}
```

완료 여부만 수정하는 예시

```text
PATCH /tasks/1

{
  "completed": true
}
```

이번 실습에서는 일부 값만 수정할 수 있도록 `PATCH` 사용

### 1.6 HTTP 상태 코드

서버는 처리 결과를 HTTP 상태 코드로 표현

| 상태 코드                       | 의미                |
| --------------------------- | ----------------- |
| `200 OK`                    | 조회 또는 수정 성공       |
| `201 Created`               | 데이터 생성 성공         |
| `204 No Content`            | 삭제 성공, 응답 Body 없음 |
| `404 Not Found`             | 요청한 데이터 없음        |
| `422 Unprocessable Content` | 요청 데이터 검증 실패      |
| `500 Internal Server Error` | 서버 내부 오류          |

## 2. Pydantic이란

Pydantic은 Python 타입 힌트를 기반으로 **데이터를 검증**하고 **변환**하는 라이브러리임

FastAPI에서 주로 **Request Body**와 **Response Body**의 **데이터 구조 정의**에 사용

Pydantic의 주요 역할

| 구분 | 주요 역할 |
| :--- | :--- |
| **구조 및 설정** | 데이터 구조 정의, 필수 필드 확인, 기본값 적용 |
| **검증 및 변환** | 데이터 타입 검증, 자동 타입 변환, 문자열 길이 및 숫자 범위 검증 |
| **출력 및 문서화** | 검증된 Python 객체 생성, Dict/JSON 직렬화(변환), JSON Schema 생성 |

### 2.1 Pydantic을 사용하지 않으면

일반 딕셔너리로 요청 데이터를 직접 처리해야 함

```python
def create_task(data: dict):
    title = data["title"]
    completed = data.get("completed", False)
```

* `title` 존재 여부 확인
* `title` 타입 확인
* 빈 문자열 확인
* ....

등을 검증하는 코드를 직접 작성해야 함

### 2.2 Pydantic을 사용하면

```python
from pydantic import BaseModel

class TaskCreate(BaseModel):
    title: str
    completed: bool = False

@app.post("/tasks")
def create_task(task: TaskCreate):
    return task
```

[1] 클라이언트가 JSON 요청 전송

```json
{
  "title": "FastAPI 공부하기"
}
```

[2] Pydantic 검증 실행 및 **Basemodel** 객체 생성

```python
TaskCreate(
    title="FastAPI 공부하기",
    completed=False,
)
```

[3] 함수 내부에서는 **검증이 끝난 객체** 사용 가능

```python
task.title
task.completed
```

### 2.3 `BaseModel`

`BaseModel`: 데이터 모델을 정의하기 위한 기본 기능이 구현된 클래스

Pydantic 모델은 `BaseModel`을 **상속**해 정의함

```python
from pydantic import BaseModel

class TaskCreate(BaseModel):
    title: str
    description: str | None = None
    completed: bool = False
```

`BaseModel`을 상속하면 추가되는 주요 기능

| 기능     | 설명                     |
| ------ | ---------------------- |
| 필드 정의  | 타입 힌트로 데이터 구조 정의       |
| 필수값 확인 | 기본값이 없는 필드 입력 여부 확인    |
| 타입 검증  | 선언한 타입에 맞는지 확인         |
| 타입 변환  | 변환 가능한 입력을 선언한 타입으로 변환 |
| 기본값 적용 | 생략된 필드에 기본값 적용         |
| 객체 생성  | 검증된 모델 인스턴스 생성         |
| 직렬화    | 모델을 딕셔너리나 JSON 형태로 변환  |
| 스키마 생성 | 모델 구조를 JSON Schema로 생성 |

일반 Python 클래스에 **데이터 검증**과 **변환 기능**을 추가한 형태

### 2.4 필드 정의 및 의미

```python
class TaskCreate(BaseModel):
    title: str
    description: str | None = None
    completed: bool = False
```

| 선언                                | 의미                   |
| --------------------------------- | -------------------- |
| `title: str`                      | 문자열 타입의 필수값          |
| `description: str \| None = None` | 문자열 또는 `None`, 생략 가능 |
| `completed: bool = False`         | 불리언 타입, 생략 시 `False` |

```python
title: str              # null값 불가능, 기본값 없으니 필드 필수
description: str | None # 문자열 또는 `None`을 허용하지만 기본값이 없으므로 필드 필수
memo: str | None = None # 기본값이 `None`으로 존재하므로 필드 생략 가능
```

선택 필드를 만들 때 중요한 요소는 `| None`만이 아니라 기본값 존재 여부


### 2.5 타입 검증과 변환

Pydantic은 타입만 확인하는 것이 아니라 **가능한 경우 타입 변환도 수행**

```python
class Example(BaseModel):
    count: int
```

입력

```python
Example(count="10")
```

변환 가능한 문자열이므로 정수로 변환될 수 있음

```python
Example(count=10)
```

변환 불가능한 값 입력

```python
Example(count="hello")
```

검증 오류 발생


### 2.6 Field

`Field`를 사용하면 타입 외의 추가 검증 조건 정의 가능

```python
from pydantic import BaseModel, Field


class TaskCreate(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )
```

Field를 활용해 최소 길이: 1, 최대 길이: 100 의 검증 조건 추가

잘못된 요청 예시: 빈 문자열이므로 (길이 0) 검증 실패

```json
{
  "title": ""
}
```

이 외에도 `gt`, `ge`, `lt`, `le` 등의 조건도 존재함

### 2.7 Pydantic 모델 객체

검증에 성공하면 **Pydantic 모델 객체를 생성**함

```python
task = TaskCreate(
    title="FastAPI 공부하기",
    completed=False,
)
```

성공적으로 생성되면 **필드 접근** 가능

```python
task.title      # Pydantic 객체
data["title"]   # 딕셔너리
```

### 2.8 주요 메서드

```python
# 1. model_dump(): Pydantic 객체를 Python 딕셔너리로 변환
task.model_dump()
# ➔ {"title": "FastAPI 공부하기", "completed": False}

# 2. model_validate(): 딕셔너리 데이터를 검증하고 Pydantic 객체로 생성
data = {"title": "FastAPI 공부하기", "completed": False}
task = TaskCreate.model_validate(data)

# 3. model_json_schema(): 모델의 JSON Schema 생성 (FastAPI Swagger UI 구조 생성에 활용)
TaskCreate.model_json_schema()
```

### 2.9 Pydantic 모델 상속

여러 모델에서 공통 필드를 반복하지 않기 위한 상속 사용

```python
class TaskBase(BaseModel):
    title: str
    description: str | None = None

class TaskCreate(TaskBase):
    completed: bool = False
```

`TaskCreate`의 필드

```text
TaskBase에서 상속받은: title, description
TaskCreate에서 추가된: completed
```


### 2.10 FastAPI와 Pydantic

FastAPI 경로 함수에서 Pydantic 모델 타입 선언

```python
@app.post("/tasks")
def create_task(task: TaskCreate):
    return task
```

처리 과정

```text
JSON Request Body → Pydantic 검증 → TaskCreate 객체 → create_task() 실행
```

검증 성공시

```text
Pydantic 객체 생성 → 경로 함수 실행
```

검증 실패시

```text
경로 함수 실행 안 됨 → 422 오류 응답
```

## 3. Task 기반 CRUD 구현

이번 실습에서 사용하는 Task는 진짜 그냥 일반적의 의미의 **사용자가 해야 할 일**을 의미함

### 3.1 구현 기능

* 새로운 Task 생성
* 전체 Task 조회
* 완료 여부에 따른 필터링
* 특정 Task 조회
* Task 일부 수정
* Task 삭제

### 3.2 Task 구조

```json
{
  "id": 1,
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현",
  "completed": false
}
```

| 필드            | 타입            | 설명         |
| ------------- | ------------- | ---------- |
| `id`          | `int`         | Task 고유 번호 |
| `title`       | `str`         | 할 일 제목     |
| `description` | `str`, `null` | 할 일 상세 설명  |
| `completed`   | `bool`        | 완료 여부      |

`id`는 클라이언트가 입력하는 값이 아니고 서버에서 객체 생성시 부여함

### 3.3 API 설계

| 기능         | API                       | 입력 방식           |
| ---------- | ------------------------- | --------------- |
| Task 생성    | `POST /tasks`             | Request Body    |
| 전체 Task 조회 | `GET /tasks`              | Query Parameter |
| 특정 Task 조회 | `GET /tasks/{task_id}`    | Path Parameter  |
| Task 수정    | `PATCH /tasks/{task_id}`  | Path + Body     |
| Task 삭제    | `DELETE /tasks/{task_id}` | Path Parameter  |

### 3.4 Pydantic 모델 설계

```python
# 공통 필드 정의
class TaskBase(BaseModel):
    title: str
    description: str | None = None

# 생성 요청 검증
class TaskCreate(TaskBase):
    completed: bool = False

# 수정 요청 검증
class TaskUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    completed: bool | None = None

# 응답 데이터 구조 정의
class TaskResponse(TaskBase):
    id: int
    completed: bool
```


### 3.5 전체 코드

`main.py`

```python
from fastapi import FastAPI, HTTPException, Response, status
from pydantic import BaseModel, Field

app = FastAPI()


class TaskBase(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )
    description: str | None = Field(
        default=None,
        max_length=500,
    )


class TaskCreate(TaskBase):
    completed: bool = False


class TaskUpdate(BaseModel):
    title: str | None = Field(
        default=None,
        min_length=1,
        max_length=100,
    )
    description: str | None = Field(
        default=None,
        max_length=500,
    )
    completed: bool | None = None


class TaskResponse(TaskBase):
    id: int
    completed: bool


tasks: dict[int, TaskResponse] = {}
next_task_id = 1


def get_task_or_404(task_id: int) -> TaskResponse:
    task = tasks.get(task_id)

    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found",
        )

    return task


@app.post(
    "/tasks",
    response_model=TaskResponse,
    status_code=status.HTTP_201_CREATED,
)
def create_task(task_create: TaskCreate):
    global next_task_id

    task = TaskResponse(
        id=next_task_id,
        title=task_create.title,
        description=task_create.description,
        completed=task_create.completed,
    )

    tasks[next_task_id] = task
    next_task_id += 1

    return task


@app.get(
    "/tasks",
    response_model=list[TaskResponse],
)
def read_tasks(completed: bool | None = None):
    task_list = list(tasks.values())

    if completed is None:
        return task_list

    return [
        task
        for task in task_list
        if task.completed == completed
    ]


@app.get(
    "/tasks/{task_id}",
    response_model=TaskResponse,
)
def read_task(task_id: int):
    return get_task_or_404(task_id)


@app.patch(
    "/tasks/{task_id}",
    response_model=TaskResponse,
)
def update_task(
    task_id: int,
    task_update: TaskUpdate,
):
    stored_task = get_task_or_404(task_id)

    update_data = task_update.model_dump(
        exclude_unset=True,
    )

    if "title" in update_data and update_data["title"] is None:
        raise HTTPException(
            status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
            detail="title cannot be null",
        )

    updated_data = {
        **stored_task.model_dump(),
        **update_data,
    }

    updated_task = TaskResponse(**updated_data)
    tasks[task_id] = updated_task

    return updated_task


@app.delete(
    "/tasks/{task_id}",
    status_code=status.HTTP_204_NO_CONTENT,
)
def delete_task(task_id: int):
    get_task_or_404(task_id)
    del tasks[task_id]

    return Response(
        status_code=status.HTTP_204_NO_CONTENT,
    )
```

서버 실행

```bash
uvicorn main:app --reload
```

Swagger UI

```text
http://127.0.0.1:8000/docs
```

### 3.6 Pydantic 모델

#### `TaskBase`

```python
class TaskBase(BaseModel):
    title: str = Field(
        min_length=1,
        max_length=100,
    )
    description: str | None = Field(
        default=None,
        max_length=500,
    )
```

여러 Task 모델에서 공통으로 사용하는 필드 정의

* `title`: 필수값, 1자 이상 100자 이하
* `description`: 선택값, 최대 500자

#### `TaskCreate`

```python
class TaskCreate(TaskBase):
    completed: bool = False
```

Task 생성 요청에 사용

`TaskBase`의 `title`, `description` 상속

`completed`를 전달하지 않으면 `False` 적용

#### `TaskUpdate`

```python
class TaskUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    completed: bool | None = None
```

`PATCH` 요청에 사용

모든 필드가 생략 가능하므로 변경할 값만 전달 가능

```json
{
  "completed": true
}
```

#### `TaskResponse`

```python
class TaskResponse(TaskBase):
    id: int
    completed: bool
```

클라이언트 응답에 사용 (서버가 생성한 `id` 포함)

### 3.7 임시 저장소

```python
tasks: dict[int, TaskResponse] = {}
next_task_id = 1
```

`tasks` 딕셔너리 구조

```python
{
    1: TaskResponse(...),
    2: TaskResponse(...),
}
```

| 구성    | 의미              |
| ----- | --------------- |
| key   | Task ID         |
| value | TaskResponse 객체 |

`next_task_id`는 새로운 Task에 부여할 ID

현재 데이터베이스가 없으므로 메모리에 데이터 저장

서버를 종료하면 저장된 데이터 삭제

### 3.8 Task 존재 확인

```python
def get_task_or_404(task_id: int) -> TaskResponse:
    task = tasks.get(task_id)

    if task is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found",
        )

    return task
```

조회, 수정, 삭제에서 반복되는 **존재 여부 확인용 코드**

Task가 없으면 `404 Not Found` 응답, Task가 있으면 해당 객체 반환

### 3.9 Task 생성

```python
@app.post(
    "/tasks",
    response_model=TaskResponse,
    status_code=status.HTTP_201_CREATED,
)
def create_task(task_create: TaskCreate):
```

요청 예시

```text
POST /tasks
Content-Type: application/json

{
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현"
}
```

새로운 Task 객체 생성 예시

```python
task = TaskResponse(
    id=next_task_id,
    title=task_create.title,
    description=task_create.description,
    completed=task_create.completed,
)

# 딕셔너리에 저장
tasks[next_task_id] = task

# 다음 Task를 위해 ID 증가
next_task_id += 1
```


응답

```json
{
  "id": 1,
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현",
  "completed": false
}
```

상태 코드 `201 Created`

### 3.10 전체 Task 조회

```python
@app.get(
    "/tasks",
    response_model=list[TaskResponse],
)
def read_tasks(completed: bool | None = None):
```

딕셔너리의 value를 리스트로 변환

```python
task_list = list(tasks.values())
```

| 조회 유형 | HTTP 요청 |
| :--- | :--- |
| 전체 조회 | `GET /tasks` |
| 완료된 Task만 조회 | `GET /tasks?completed=true` |
| 미완료 Task만 조회 | `GET /tasks?completed=false` |

```python
# Query Parameter가 없으면 전체 반환
if completed is None:
    return task_list

# 조건이 있으면 일치하는 Task만 반환
return [
    task
    for task in task_list
    if task.completed == completed
]
```

### 3.11 특정 Task 조회

```python
@app.get(
    "/tasks/{task_id}",
    response_model=TaskResponse,
)
def read_task(task_id: int):
    return get_task_or_404(task_id)
```

요청

```text
GET /tasks/1
```

`1`은 Path Parameter\
FastAPI가 내부의 Pydantic을 이용해 문자열 "1"을 정수 1로 자동 변환\
Task가 존재하면 `TaskResponse` 반환\
Task가 없으면 `404 Not Found` 반환\

### 3.12 Task 수정

```python
@app.patch(
    "/tasks/{task_id}",
    response_model=TaskResponse,
)
def update_task(
    task_id: int,
    task_update: TaskUpdate,
):
```

요청

```text
PATCH /tasks/1
Content-Type: application/json

{
  "completed": true
}
```

#### 기존 Task 조회 및 수정할 내용 추출

```python
# 수정할 Task 존재 여부 확인 및 정의
stored_task = get_task_or_404(task_id)

# 실제 전달된 필드 추출
update_data = task_update.model_dump(
    exclude_unset=True,
)
```

`exclude_unset=True`는 클라이언트가 실제로 전달한 필드만 포함함. 사용하지 않았는데 3.12의 요청처럼 completed만 True로 바꾸는 요청이 오면

```python
{
    "title": None,
    "description": None,
    "completed": True,
}
```

위와 같이 전달하지 않은 필드까지 `None`으로 포함될 수 있으므로 기존 값이 의도하지 않게 변경될 가능성이 있음

#### 기존 데이터와 수정 데이터 병합

```python
updated_data = {
    **stored_task.model_dump(),
    **update_data,
}

# Python 3.9+ 최신 방식 (Merge 연산자 활용)
updated_data = stored_task.model_dump() | update_data
```

기존 데이터

```python
{
    "id": 1,
    "title": "FastAPI 공부하기",
    "description": "CRUD API 구현",
    "completed": False,
}
```

수정 데이터

```python
{
    "completed": True,
}
```

병합 결과

```python
{
    "id": 1,
    "title": "FastAPI 공부하기",
    "description": "CRUD API 구현",
    "completed": True,
}
```

뒤에 위치한 `update_data`가 같은 key의 기존 값을 덮어씀

#### 수정 결과 저장

```python
updated_task = TaskResponse(**updated_data)
tasks[task_id] = updated_task
```

병합한 딕셔너리를 다시 `TaskResponse` 객체로 변환하고 기존 Task 위치에 수정된 객체 저장

### 3.13 Task 삭제

```python
@app.delete(
    "/tasks/{task_id}",
    status_code=status.HTTP_204_NO_CONTENT,
)
def delete_task(task_id: int):
```


```python
# Task 존재 여부 확인
get_task_or_404(task_id)

# 딕셔너리에서 제거
del tasks[task_id]
```

삭제 성공 시 `204 No Content` 응답

응답 Body 없음 (그래서 delete는 응답의 pydantic 검증 필요 x)

### 3.14 response_model

```python
response_model=TaskResponse
```

API가 반환할 데이터 구조 지정

주요 역할

* 반환 데이터 검증
* 반환할 필드 제한
* JSON 응답 변환
* Swagger UI 응답 스키마 생성

요청 모델은 서버가 받을 데이터 정의

응답 모델은 클라이언트에 반환할 데이터 정의

### 3.15 HTTPException

```python
raise HTTPException(
    status_code=status.HTTP_404_NOT_FOUND,
    detail="Task not found",
)
```

정상 응답 대신 HTTP 오류 응답 반환

다음 방식은 기본적으로 `200 OK`가 될 수 있으므로 부적절

```python
return {"message": "Task not found"}
```

존재하지 않는 데이터는 `404 Not Found` 사용

### 3.16 Swagger UI 실습

가상환경 활성화

**Windows:**
```powershell
.venv\Scripts\activate
```

**Mac:**
```bash
source .venv/bin/activate
```
서버 실행

```bash
uvicorn main:app --reload
```

정상 실행 시 다음과 비슷한 메시지 출력

```text
Uvicorn running on http://127.0.0.1:8000
```

Swagger UI 접속

```text
http://127.0.0.1:8000/docs
```

FastAPI가 현재 등록된 API를 자동으로 문서화한 화면 표시

각 API를 클릭한 뒤 다음 순서로 요청 실행

테스트 순서

| 순서 | 요청                          | 확인 내용       |
| -- | --------------------------- | ----------- |
| 1  | `POST /tasks`               | Task 생성     |
| 2  | `GET /tasks`                | 전체 목록 조회    |
| 3  | `GET /tasks/1`              | 특정 Task 조회  |
| 4  | `PATCH /tasks/1`            | 완료 여부 수정    |
| 5  | `GET /tasks?completed=true` | 필터링         |
| 6  | `DELETE /tasks/1`           | Task 삭제     |
| 7  | `GET /tasks/1`              | `404` 오류 확인 |


```text
API 선택
→ Try it out 클릭
→ 필요한 값 입력
→ Execute 클릭
→ 응답 확인
```

응답 영역에서 확인할 내용

| 항목            | 의미                 |
| ------------- | ------------------ |
| Request URL   | 실제 요청이 전송된 주소      |
| Response code | 서버가 반환한 HTTP 상태 코드 |
| Response body | 서버가 반환한 JSON 데이터   |

#### [1] Task 생성

Swagger UI에서 `POST /tasks` 선택 후 `Try it out` 클릭

Request Body에 다음 데이터 입력

```json
{
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현"
}
```

`Execute` 클릭

예상 상태 코드

```text
201 Created
```

예상 응답

```json
{
  "id": 1,
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현",
  "completed": false
}
```

`completed`를 전달하지 않았으므로 기본값 `false` 적용

#### [2] 전체 Task 조회

`GET /tasks` 선택 후 `Try it out`, `Execute` 클릭

예상 상태 코드

```text
200 OK
```

예상 응답

```json
[
  {
    "id": 1,
    "title": "FastAPI 공부하기",
    "description": "CRUD API 구현",
    "completed": false
  }
]
```

#### [3] 특정 Task 조회

`GET /tasks/{task_id}` 선택

`task_id` 입력란에 다음 값 입력

```text
1
```

`Execute` 클릭

예상 응답

```json
{
  "id": 1,
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현",
  "completed": false
}
```

#### [4] Task 수정

`PATCH /tasks/{task_id}` 선택

`task_id`에 `1` 입력

Request Body에 수정할 필드만 입력

```json
{
  "completed": true
}
```

`Execute` 클릭

예상 응답

```json
{
  "id": 1,
  "title": "FastAPI 공부하기",
  "description": "CRUD API 구현",
  "completed": true
}
```

요청에 포함되지 않은 `title`, `description`은 기존 값 유지

#### [5] 완료된 Task 필터링

`GET /tasks` 선택

Query Parameter의 `completed`에 `true` 선택 또는 입력

`Execute` 클릭

실제 요청 주소

```text
http://127.0.0.1:8000/tasks?completed=true
```

완료 상태가 `true`인 Task만 반환

```json
[
  {
    "id": 1,
    "title": "FastAPI 공부하기",
    "description": "CRUD API 구현",
    "completed": true
  }
]
```

#### [6] Task 삭제

`DELETE /tasks/{task_id}` 선택

`task_id`에 `1` 입력 후 `Execute` 클릭

예상 상태 코드

```text
204 No Content
```

삭제 성공 시 Response Body 없음

#### [7] 삭제 결과 확인

다시 `GET /tasks/{task_id}` 선택

`task_id`에 `1` 입력 후 요청

예상 상태 코드

```text
404 Not Found
```

예상 응답

```json
{
  "detail": "Task not found"
}
```

삭제된 Task가 저장소에 존재하지 않음을 확인


## 핵심 정리

| 개념               | 핵심                    |
| ---------------- | --------------------- |
| CRUD             | 데이터 생성, 조회, 수정, 삭제    |
| URL              | 작업 대상 리소스 표현          |
| HTTP 메서드         | 리소스에 수행할 동작 표현        |
| Pydantic         | 데이터 구조 정의, 검증, 변환     |
| `BaseModel`      | Pydantic 모델의 기본 기능 제공 |
| `Field`          | 길이와 범위 등 추가 검증        |
| `TaskCreate`     | 생성 요청 데이터             |
| `TaskUpdate`     | 일부 수정 요청 데이터          |
| `TaskResponse`   | 응답 데이터                |
| `model_dump()`   | 모델 객체를 딕셔너리로 변환       |
| `exclude_unset`  | 요청에 실제 포함된 필드만 추출     |
| `response_model` | 응답 데이터 구조 지정          |
| `HTTPException`  | HTTP 오류 응답 반환         |
| 인메모리 저장소         | 서버 실행 중에만 유지되는 임시 저장소 |
