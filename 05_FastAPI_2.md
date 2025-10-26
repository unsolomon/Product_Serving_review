# 🚀 **FastAPI (2) — 심화 및 확장 기능**

## **1. Pydantic 사용법**

### 🔹 1.1 Pydantic 소개 & 기능
- **Pydantic**은 FastAPI의 핵심 데이터 모델 검증 도구로,  
  타입 힌트를 **런타임에서 강제**하여 안전한 데이터 처리를 지원합니다.
- 주요 특징:
  - Python 기본 타입 (`str`, `int`, `float`, `bool` 등) + `List`, `Dict`, `Tuple` 검증 가능  
  - 기존 Validation 라이브러리보다 빠른 성능 (Benchmark 확인됨)
  - **ML Feature 데이터 검증**에도 활용 가능  
    예) Feature A는 `int`, 값 범위는 0~100 사이 등

---

### 🔹 1.2 Validation

#### ✅ 검증 사례
- 머신러닝 모델 입력값 검증
- Online Serving에서 Request 데이터 Validation

**검증 조건 예시**
1. 올바른 URL 입력 (`url`)
2. 1~10 사이의 정수 (`rate`)
3. 유효한 디렉토리 경로 (`target_dir`)

#### 🧩 Validation 방법 비교
| 방법 | 설명 | 단점 |
|------|------|------|
| 일반 Python Class | 직접 검증 로직 작성 | 코드冗長, 예외 처리 복잡 |
| Dataclass | `__post_init__`로 생성 시점 Validation | 여전히 수동 검증 필요 |
| Pydantic | 생성 시점 자동 Validation | 코드 6줄 내외로 간결, 타입 기반 에러 발생 |

#### 💡 Pydantic의 장점
- Validation 및 Config 기능을 하나의 모델로 관리 가능  
- HTTP URL, DB URL, Enum 등 기본 제공 타입 외에도 **Custom Type** 쉽게 정의 가능  
- 에러 발생 시 **위치, 타입, 메시지**를 상세히 반환  
- 참고: [Pydantic 공식 문서](https://docs.pydantic.dev/latest/concepts/types/)

---

### 🔹 1.3 Config 관리

#### ⚙️ Config 정의
- 앱 실행에 필요한 **환경 설정 정보 (예: DB 연결, API 키 등)**  
- 한 곳에서 관리할 수 있도록 구성 필요

#### 📦 Config 관리 방법
| 방법 | 특징 | 단점 |
|------|------|------|
| 코드 내 상수로 관리 | 가장 단순 | 보안 노출 위험, 환경 분리 어려움 |
| YAML 등 외부 파일 관리 | 환경별 파일 분리 (dev/prod 등) | 여전히 파일 노출 위험 존재 |
| 환경변수 + `Pydantic.BaseSettings` | 타입 기반 검증 + `.env` 활용 | 실무 추천 방식 |

> ✅ **추천:**  
> FastAPI 사용 시 `Pydantic.BaseSettings`를 활용하여 Config 관리  
> → 보안 정보 노출 방지 + 환경별 유연한 설정 가능

---

## **2. FastAPI 확장 기능**

### 🔹 2.1 Lifespan Function
- FastAPI 앱의 **시작(Startup)** 과 **종료(Shutdown)** 시 실행되는 로직을 정의
- 예)  
  - 시작 시 모델 로드  
  - 종료 시 DB 연결 종료
- 구현 예시:
  ```python
  async def lifespan(app: FastAPI):
      # 앱 시작 전
      yield
      # 앱 종료 전
  app = FastAPI(lifespan=lifespan)
  ```
- 관련 문서: [FastAPI Lifespan Docs](https://fastapi.tiangolo.com/advanced/events/)

---

### 🔹 2.2 API Router
- 엔드포인트 수가 많아질수록 관리 어려움 → **Router 분리**
- `APIRouter`를 활용하면 모듈 단위로 관리 가능
  ```python
  router = APIRouter()
  @router.get("/users/{id}")
  def get_user(id: int):
      ...
  app.include_router(router)
  ```
- 일반적으로 `routers/` 폴더 아래에 API별 파일 생성  
  (예: `user.py`, `order.py`)

---

### 🔹 2.3 Project Structure (프로젝트 구조)
> 규모가 커질수록 구조화가 필수!

**예시 구조**
```
app/
 ├── __init__.py
 ├── main.py            # 메인 실행 파일
 ├── dependencies.py    # 의존성 관리
 ├── routers/
 │   ├── __init__.py
 │   ├── items.py
 │   └── users.py
 └── internal/
     ├── __init__.py
     └── admin.py
```

- `routers` 폴더: Sub FastAPI 모듈  
- `internal`: 내부 전용 기능 (관리자 등)

---

### 🔹 2.4 Error Handler
- 서버 안정성을 위한 필수 기능
- 기본적으로 예외 발생 시 500 Internal Server Error  
  → 클라이언트는 원인 파악 불가
- **해결:** `HTTPException` 활용
  ```python
  from fastapi import HTTPException

  @app.get("/items/{item_id}")
  def read_item(item_id: int):
      if item_id not in items:
          raise HTTPException(status_code=404, detail="Item not found")
  ```
- 에러 발생 시 상태 코드 + 상세 메시지 반환 가능

---

### 🔹 2.5 Background Tasks
- 오래 걸리는 작업을 **비동기(Asynchronous)** 로 처리  
- 클라이언트는 즉시 응답받고, 작업은 백그라운드에서 실행됨
  ```python
  from fastapi import BackgroundTasks

  def send_email(email: str):
      ...

  @app.post("/send")
  def send(background_tasks: BackgroundTasks, email: str):
      background_tasks.add_task(send_email, email)
      return {"message": "Email sending started"}
  ```
- 예시: 이메일 전송, 로그 기록, 대용량 파일 처리 등

---

## **3. FastAPI가 어렵다면**

### 🔹 3.1 어려운 이유
1. 프로젝트 구조 설계 미숙  
2. 객체지향 프로그래밍(OOP) 미숙  
3. 백엔드 경험 부족  
4. 명확한 목표 부재

---

### 🔹 3.2 프로젝트 구조 - Cookiecutter
- 프로젝트 템플릿 생성 도구  
  - CLI 기반으로 구조를 자동 생성  
  - 공통 구조를 팀 단위로 공유 가능
- 추천 템플릿:
  - [cookiecutter-data-science](https://github.com/drivendata/cookiecutter-data-science)
  - [cookiecutter-fastapi](https://github.com/arthurhenrique/cookiecutter-fastapi)

> ⚡ 처음엔 하나의 파일로 익숙해진 뒤  
> 점차 구조화 및 클린 아키텍처로 확장할 것.

---

### 🔹 3.3 객체지향
- 절차형 vs 객체지향 차이를 이해  
- 코드 재사용성 향상 및 유지보수 용이
- Pydantic 모델 구조를 통해 자연스럽게 OOP 학습 가능

---

### 🔹 3.4 Try & Error
- 낯선 것이 당연하므로 **실습 중심 학습** 권장  
  - “작성 → 실행 → 수정” 반복  
- 작게 시작해서 점진적으로 확장  
- **명확한 목표 설정**
  - 예: “FastAPI로 간단한 예측 API 만들기”

---

## ✅ **정리 요약**

| 구분 | 핵심 내용 |
|------|------------|
| **Pydantic** | Validation + Config 관리, 타입 기반 안전성 |
| **Lifespan** | 앱 시작/종료 시 자동 실행 로직 |
| **API Router** | 모듈 단위로 API 구조화 |
| **Error Handler** | 예외처리 및 상세 에러 응답 |
| **Background Task** | 비동기 백그라운드 작업 |
| **Cookiecutter** | 프로젝트 구조 템플릿 |
| **학습 팁** | 작게 시작 → 구조화 → 반복 실습 |

---

> 📚 참고: NAVER Connect Foundation — *[Product Serving] FastAPI (2)* 강의자료
