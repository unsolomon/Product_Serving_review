# 🐳 06_Docker

## **1. Docker 소개**

### 1.1 가상화란?

- **가상화(Virtualization)**는 하나의 물리적 서버 위에 여러 개의 독립된 운영체제(OS)를 실행하는 기술입니다.  
  → 개발 환경(Local)과 운영 환경(Production)의 차이를 해결하기 위해 사용됩니다.
- 예시  
  - Local은 Windows, 서버는 Linux인 경우 패키지 설치 방식이 달라 문제 발생.  
  - OS가 같아도 환경 변수, 권한(permission), 사용자 그룹 등이 달라 동작이 달라질 수 있음.
- 서버 환경을 사람 손으로 맞추는 과정은 **Human Error**가 발생하기 쉬움.
- 여러 서버(예: 100대) 환경을 일일이 설정·업데이트하는 것은 비효율적.  
  ⇒ 이를 자동화하고 동일한 환경을 제공하기 위한 해법이 **소프트웨어 가상화(Container)**.

> **핵심:** 개발·운영·리서치 환경 간의 불일치를 해결하기 위한 공통 실행 템플릿 제공.

---

### 1.2 Docker 등장 이전

- 과거에는 **VM (Virtual Machine)** 기반의 가상화 기술 사용.  
  - 실제 컴퓨터(Host OS) 위에 가상의 OS(Guest OS)를 하나 더 실행.  
  - 예: Windows에서 Linux 실행, Mac에서 Windows 실행 등.
- 문제점:
  - VM은 OS를 중첩해 실행하기 때문에 매우 **무겁고 리소스를 많이 소모**함.
- 이후 등장한 **Container 기술**은 VM보다 훨씬 **경량화된 가상화 방식**을 제공.

---

### 1.3 Docker 소개

- **Docker**는 이러한 **컨테이너(Container)** 기술을 손쉽게 사용할 수 있게 해주는 **플랫폼/도구**입니다.
- 2013년 오픈소스로 등장했으며, 컨테이너 기반 개발·운영 환경 확산에 기여.
- **Docker Image**  
  - 컨테이너를 실행하기 위한 **템플릿(읽기 전용, Read-Only)**  
- **Docker Container**  
  - Image를 실행한 인스턴스 (쓰기 가능, Write 가능)
- 비유: PC방의 리셋 시스템 — 항상 동일한 초기 상태로 복원.

---

### 1.4 Docker로 할 수 있는 일

- 다른 사람이 만든 소프트웨어를 바로 실행 가능  
  - 예: `MySQL`, `Jupyter Notebook`을 Docker로 실행
- 자신이 만든 Docker 이미지를 만들어 **공유** 가능  
  - 원격 저장소(Container Registry)에 업로드  
  - DockerHub, GCR(Google), ECR(AWS) 등 활용

---

## **2. Docker 실습하며 배우기**

### 2.1 설치 및 실행

- Docker 공식 홈페이지에서 OS별 Docker Desktop 설치  
  👉 https://www.docker.com/get-started
- `docker` 명령어가 정상 동작하는지 확인.

#### 예시: MySQL 실행

```bash
docker pull mysql:8
docker run --name mysql-tutorial -e MYSQL_ROOT_PASSWORD=1234 -d -p 3306:3306 mysql:8
```

- `-e`: 환경 변수 설정 (예: MySQL root 비밀번호)
- `-d`: 백그라운드 실행
- `-p`: 포트 연결 (로컬포트:컨테이너포트)
- `docker ps`: 실행 중인 컨테이너 확인
- `docker exec -it 컨테이너명 /bin/bash`: 컨테이너 접속
- `docker stop`, `docker rm`: 컨테이너 중지 및 삭제

#### ⚙️ Volume Mount (파일 공유)

```bash
docker run -it -p 8888:8888   -v /some/host/folder:/home/jovyan/workspace   jupyter/minimal-notebook
```
- `-v Host폴더:Container폴더` 형태로 파일 공유 설정.

---

### 2.2 Docker Image 만들기

#### 1️⃣ 프로젝트 준비
```bash
mkdir 02-docker && cd 02-docker
poetry init
poetry add torch torchvision
```

#### 2️⃣ PyTorch 코드 작성
- [PyTorch Quickstart 예제](https://pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html)

#### 3️⃣ Dockerfile 작성 예시

```dockerfile
FROM pytorch/pytorch:1.13.1-cuda11.6-cudnn8-runtime
COPY . /app
WORKDIR /app
ENV PYTHONPATH=/app
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

#### 4️⃣ 이미지 빌드 및 실행

```bash
docker build -t myapp:latest .
docker images
docker run myapp:latest
```

---

### 2.3 Registry에 Docker Image Push

1. DockerHub 계정 생성 → [hub.docker.com](https://hub.docker.com)
2. 로그인  
   ```bash
   docker login
   ```
3. 태그 지정  
   ```bash
   docker tag myapp:latest yourID/myapp:latest
   ```
4. 업로드  
   ```bash
   docker push yourID/myapp:latest
   ```
5. 어디서든 pull 가능  
   ```bash
   docker pull yourID/myapp:latest
   ```

---

### 2.4 Docker Image Size 최적화

1️⃣ **작은 Base Image 사용**
- 예: `python:3.9-slim`, `python:3.9-alpine`

2️⃣ **Multi-Stage Build 활용**
```dockerfile
FROM python:3.9 as build
RUN pip install -r requirements.txt
FROM python:3.9-slim
COPY --from=build /app /app
CMD ["python", "main.py"]
```

3️⃣ **패키징 최적화**
- `.dockerignore`로 불필요한 파일 제외  
- 대용량 모델 파일은 컨테이너 외부에서 로드  
- 명령어 순서 최적화로 캐시 효율 극대화

---

## **3. Docker Compose**

### 3.1 소개

- 여러 **Docker Container**를 동시에 실행하고 관리하기 위한 도구.
- `docker-compose.yml` 파일에 설정을 정의.

#### 예시 구조
```yaml
version: "3"
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: 1234
    ports:
      - "3306:3306"

  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
```

---

### 3.2 실행 방법

```bash
docker-compose up
docker-compose up -d
docker-compose down
docker-compose logs app
```

- `docker ps` 또는 `docker-compose ps`로 실행 상태 확인.

---

## **4. Docker & Docker Compose 요약**

| 항목 | Docker | Docker Compose |
|------|---------|----------------|
| 목적 | 컨테이너 단일 실행 | 여러 컨테이너 동시 실행 |
| 구성 | Dockerfile | docker-compose.yml |
| 장점 | 가볍고 배포 용이 | 실행 순서, 의존성 관리 |
| 단점 | 복잡한 환경에 비효율적 | 대규모 환경엔 부적합 |

---

> ✅ **요약**
> - Docker는 **“환경의 표준화”**를,  
> - Docker Compose는 **“환경의 자동화와 확장성”**을 제공합니다.
