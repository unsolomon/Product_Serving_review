# 🚀 FastAPI (1) — 논리 중심 정리

## **1️⃣ FastAPI 개요**

### ✅ 개념
- **FastAPI**는 Python 기반의 최신 웹 프레임워크로,  
  **고성능(High Performance)**, **생산성(Productivity)**, **자동 문서화(Easy Docs)**를 목표로 함.
- **Starlette(비동기 서버)**와 **Pydantic(데이터 검증)** 위에서 동작하며,  
  API 서버 구축과 데이터 유효성 검증을 효율적으로 처리.

### ✅ 주요 특징
| 구분 | 설명 |
|------|------|
| **High Performance** | Node.js, Go와 대등한 성능 |
| **Easy** | Flask와 유사한 구조, 마이크로서비스에 적합 |
| **Productivity** | Swagger / Redoc 자동 생성 |
| **Data Validation** | Pydantic을 이용한 직관적 검증 |

---

## **2️⃣ 학습 및 설계 접근법**

- 처음에는 **웹서버를 직접 띄우고 간단한 기능을 추가**하며 구조를 익힘.  
- 익숙해지면 **자신의 모델을 FastAPI 서버에 통합**.  
- 특정 기능을 구현하기 전, **어떻게 설계해야 하는지 사고 중심의 접근**이 중요.

---

## **3️⃣ 프로젝트 구조 설계**

| 구성 요소 | 역할 |
|------------|------|
| `__main__.py` | 애플리케이션의 진입점(entry point) |
| `main.py` or `app.py` | FastAPI 인스턴스 및 Router 설정 |
| `model.py` | ML 모델 클래스 및 함수 정의 |
| `app/` | 전체 코드가 위치할 모듈 디렉토리 |

> ⚙️ **Entry Point란**  
> 프로그램 실행 시 제일 먼저 실행되는 코드의 시작 지점을 의미.

---

## **4️⃣ Poetry — 개발 환경 및 의존성 관리 도구**

### ✅ 개념
- Python의 **패키지 및 의존성 관리 도구**로,  
  `pip`과 `virtualenv`를 대체하며 프로젝트 단위로 격리된 환경을 제공.
- 복잡한 버전 충돌 방지 및 프로젝트 설정을 `pyproject.toml`로 명시적으로 관리.

### ✅ Poetry 주요 명령 흐름
| 명령어 | 설명 |
|--------|------|
| `poetry init` | 프로젝트 초기화 |
| `poetry shell` | 가상환경 진입 |
| `poetry install` | 명시된 의존성 설치 |
| `poetry add 패키지명` | 패키지 추가 |
| `poetry remove 패키지명` | 패키지 제거 |

> ✅ **poetry.lock**  
> 동일한 의존성을 보장하기 위한 고정 버전 파일.  
> → 반드시 GitHub Repository에 함께 커밋 필요.

---

## **5️⃣ FastAPI 동작의 논리적 구조**

### 💡 웹 서버 흐름
```mermaid
graph TD
    A[사용자 요청 (Client)] --> B[FastAPI 서버]
    B --> C[요청 데이터 검증 - Pydantic]
    C --> D[비즈니스 로직 및 모델 처리]
    D --> E[Response 생성 및 반환]
```

- **GET/POST 요청**에 따라 데이터가 요청되고 응답이 생성됨.
- 클라이언트는 API 요청을 통해 **데이터 조회(GET)** 또는 **생성/수정(POST)** 수행.

---

## **6️⃣ 문서화 — Swagger & Redoc**

- FastAPI는 `/docs`, `/redoc` 경로에서 **자동화된 API 문서**를 생성.
- API 요청 형식과 응답 구조를 시각적으로 확인 가능.
- 팀 협업 및 클라이언트-서버 통신 명세 관리에 매우 유용.

| 도구 | 역할 |
|------|------|
| **Swagger UI** | 실시간 API 테스트 및 요청 파라미터 확인 |
| **Redoc** | 보다 깔끔하고 정적 형태의 API 문서 제공 |

---

## **7️⃣ URL Parameters**

| 구분 | 설명 | 예시 |
|------|------|------|
| **Path Parameter** | 특정 리소스 식별 | `/users/402` |
| **Query Parameter** | 필터링 / 정렬 목적 | `/users?id=402` |

- Path는 고유 리소스 접근에, Query는 검색/필터링에 적합.  
- 두 방식을 구분해 설계함으로써 API의 **의도 명확성** 확보.

---

## **8️⃣ Request & Response 논리 구조**

### 🧭 Request Body
- 클라이언트 → 서버로 데이터 전달 시 사용.  
- 주로 POST 방식에서 활용.
- 데이터 타입은 `Content-Type`으로 명시:
  - `application/json`
  - `application/x-www-form-urlencoded`
  - `multipart/form-data` (파일 포함 시)

### 🧩 Response Body
- 서버 → 클라이언트로 응답 데이터 반환.
- `response_model`을 통해 명시적 검증 및 자동 문서화 수행.

---

## **9️⃣ Form과 File 업로드 처리**

| 기능 | 설명 |
|------|------|
| **Form** | 입력 필드 데이터를 수신 (`python-multipart` 필요) |
| **File Upload** | 파일을 `bytes` 또는 `UploadFile` 형태로 처리 |
| **Jinja Template** | HTML 프론트엔드 연동 시 데이터 표현용 템플릿 엔진 |

> 💡 예를 들어 로그인 폼에서는 GET 요청이 아닌 POST 요청을 통해  
> 사용자 정보를 전달하고 인증 수행.

---

## **🔟 FastAPI와 Poetry의 결합적 가치**

| 측면 | 설명 |
|------|------|
| **FastAPI** | 서버와 API 설계의 효율성 향상 |
| **Poetry** | 패키지/환경 관리의 안정성 확보 |
| **결합 효과** | ML 모델 Serving, API 운영, 환경 재현성 확보에 최적 |

---

## **🔍 장단점 요약**

| 구분 | FastAPI | Poetry |
|------|----------|---------|
| **장점** | 고성능, 비동기 지원, 자동 문서화 | 의존성 관리 용이, 재현성 높은 환경 |
| **단점** | 비동기 구조에 대한 학습 필요 | 초기 설정 및 pip 호환성 문제 가능 |

---

## **📘 요약**
- **FastAPI**는 실시간 Online Serving 환경에서 핵심적인 역할을 수행.  
- **Poetry**는 효율적인 개발 환경과 의존성 관리를 지원.  
- 두 기술의 결합은 **ML 모델 서비스 운영 자동화**의 기반이 됨.

---

> 📚 참고: NAVER Connect Foundation — *[Product Serving] FastAPI (1)* 강의자료
