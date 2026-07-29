# Trend Leader

<div align="center">

## 관심사 기반 개인화 트렌드 큐레이션 앱

**Trend Leader**는 사용자가 선택한 관심 분야를 기준으로 최신 트렌드를 선별해 제공하고,  
AI 기반 요약과 유행 이유 분석을 통해 트렌드의 맥락을 빠르게 이해할 수 있도록 돕는 모바일 앱입니다.

</div>

---

### Backend, Frontend, Platform

<p align="center">
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=flat-square&logo=fastapi&logoColor=blue" />
  <img src="https://img.shields.io/badge/React%20Native-Frontend-61DAFB?style=flat-square&logo=react&logoColor=cyan" />
  <img src="https://img.shields.io/badge/Expo-Mobile-3DDC84?style=flat-square&logo=expo&logoColor=white" />
  <img src="https://img.shields.io/badge/Python-3.14-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-Strict-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL%20%2F%20MariaDB-Database-4479A1?style=flat-square&logo=mysql&logoColor=white" />
</p>



---

## 1. 프로젝트 개요

Trend Leader는 여러 플랫폼에 흩어진 트렌드 정보를 한곳에서 확인하고,  
사용자의 관심 카테고리에 맞는 트렌드를 우선적으로 탐색할 수 있도록 설계된 졸업작품 프로젝트입니다.

초기 목표는 단순한 최신 이슈 제공 앱이었으나, 기획을 구체화하면서 **사용자 관심사 기반 맞춤형 트렌드 제공 서비스**로 방향을 확장했습니다.

### 핵심 가치

- 관심 분야 중심의 트렌드 탐색
- 최신 이슈와 개인 관심 트렌드의 분리 제공
- AI 기반 트렌드 요약, 유행 이유, 관련 키워드 제공
- 저장/북마크를 통한 관심 트렌드 재확인
- 향후 사용자 행동 기반 추천 고도화 확장

---

## 2. 주요 기능

### 2.1 사용자 / 인증

- 회원가입
- 로그인
- 구글 OAuth 로그인
- 로그아웃
- 내 정보 조회
- 회원 정보 수정
- 비밀번호 변경
- 회원 탈퇴

### 2.2 관심사

- 카테고리 목록 조회
- 회원가입 후 관심사 선택
- 기존 관심사 조회
- 관심사 수정

### 2.3 트렌드

- 관심사 기반 맞춤 트렌드 목록 조회
- 전체 최신 이슈 목록 조회
- 트렌드 상세 조회
- AI 요약, 유행 이유, 관련 키워드 조회
- 트렌드 출처 정보 조회

### 2.4 북마크

- 트렌드 저장
- 트렌드 저장 해제
- 저장한 트렌드 목록 조회

### 2.5 검색

- 트렌드 검색
- 최근 검색어 조회
- 검색 기록 삭제

---

## 3. 기술 스택

### Frontend

| 구분 | 기술 |
|---|---|
| Framework | React Native |
| Runtime / Tooling | Expo |
| Language | TypeScript |
| Package Manager | npm |
| Target | Android 중심 모바일 앱 |

### Backend

| 구분 | 기술 |
|---|---|
| Framework | FastAPI |
| Language | Python |
| Server | Uvicorn |
| Validation | Pydantic |
| ORM | SQLAlchemy |
| Migration | Alembic |
| Auth | JWT Bearer Token |
| Password Hashing | passlib[bcrypt] |

### Database

| 구분 | 기술 |
|---|---|
| RDBMS | MySQL 또는 MariaDB |
| Driver | PyMySQL |
| Charset | utf8mb4 |

### AI / External

| 구분 | 내용 |
|---|---|
| AI Provider | 초기 mock 기반, 이후 OpenAI/Gemini 등 확장 가능 |
| OAuth | Google OAuth 연동 예정 |

---

## 4. Repository 구성

본 프로젝트는 기능 구현 레포지토리와 디자인 산출물 레포지토리를 분리해 관리합니다.

| Repository | 역할 |
|---|---|
| [`feature`](https://github.com/Trend-Leader-capstoneProject/feature) | Frontend / Backend 실제 기능 구현 |
| [`design`](https://github.com/Trend-Leader-capstoneProject/design) | 문서, 발표자료, Figma, 로고, 스토리보드 등 디자인 산출물 관리 |

---

## 5. 현재 프로젝트 구조

```text
feature/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── router.py
│   │   │   └── routes/
│   │   │       └── health.py
│   │   ├── core/
│   │   │   └── config.py
│   │   ├── db/
│   │   ├── models/
│   │   ├── repositories/
│   │   ├── resources/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── utils/
│   │   │   └── response.py
│   │   └── main.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   └── navigation/
│   │   │       └── RootNavigator.tsx
│   │   └── assets/
│   ├── App.tsx
│   ├── index.ts
│   ├── app.json
│   ├── package.json
│   ├── package-lock.json
│   └── tsconfig.json
│
├── .gitignore
└── README.md
```

---

## 6. 권장 구조

Trend Leader는 백엔드와 프론트엔드의 책임을 분리하고,  
각 기능의 위치를 쉽게 찾을 수 있도록 계층형·기능 중심 구조를 사용합니다.

---

### Backend

```text
backend/
├── app/
│   ├── api/             # FastAPI Router
│   ├── core/            # 환경변수 및 애플리케이션 설정
│   ├── db/              # DB 연결 및 세션 관리
│   ├── models/          # SQLAlchemy ORM Model
│   ├── schemas/         # API 요청·응답 Schema
│   ├── services/        # 비즈니스 로직
│   ├── repositories/    # DB 접근 로직
│   ├── resources/       # 프롬프트, Seed, 샘플 데이터
│   ├── utils/           # 공통 유틸리티
│   └── main.py          # FastAPI 애플리케이션 진입점
├── alembic/             # DB Migration
├── tests/               # 테스트 코드
├── .env.example
└── README.md
```

백엔드의 기본 처리 흐름은 다음과 같습니다.

```text
Client
  ↓
Router
  ↓
Service
  ↓
Repository
  ↓
Database
```

- `Router`: HTTP 요청 및 응답 처리
- `Service`: 기능의 핵심 비즈니스 로직 처리
- `Repository`: 데이터베이스 조회·저장·수정·삭제
- `Model`: 데이터베이스 테이블 매핑
- `Schema`: API 요청 및 응답 데이터 검증

---

### Frontend

```text
frontend/
├── assets/                # 이미지, 아이콘, 폰트
├── src/
│   ├── app/
│   │   ├── navigation/    # 화면 이동 및 Navigation 타입
│   │   ├── providers/     # React Query 등 전역 Provider
│   │   └── config/        # 앱 환경 설정
│   │
│   ├── features/
│   │   ├── auth/          # 로그인 및 회원가입
│   │   ├── user/          # 사용자 정보 및 마이페이지
│   │   ├── interest/      # 관심사 선택 및 수정
│   │   ├── trend/         # 트렌드 목록 및 상세
│   │   ├── bookmark/      # 저장 트렌드
│   │   └── search/        # 트렌드 검색
│   │
│   └── shared/
│       ├── api/           # 공통 API Client
│       ├── components/    # 공통 UI Component
│       ├── constants/     # 색상, 간격 등 공통 상수
│       ├── hooks/         # 공통 Custom Hook
│       ├── storage/       # 토큰 등 로컬 저장소
│       ├── types/         # 공통 Type
│       └── utils/         # 공통 유틸리티
│
├── App.tsx
├── index.ts
├── .env.example
└── README.md
```

각 Feature는 필요한 범위에서 다음 구조를 사용합니다.

```text
feature/
├── api/          # FastAPI 서버 통신
├── components/   # 기능 전용 UI Component
├── hooks/        # API 요청 및 상태 관리
├── screens/      # 화면 구성
└── types/        # 요청·응답 및 기능 Type
```

프론트엔드의 기본 처리 흐름은 다음과 같습니다.

```text
User
  ↓
Screen
  ↓
Custom Hook
  ↓
API Function
  ↓
FastAPI Backend
```

- `Screen`: 화면 구성, 사용자 이벤트, Navigation 처리
- `Hook`: React Query 기반 API 상태 관리
- `API Function`: 서버 요청 및 응답 반환
- `Component`: 재사용 가능한 UI
- `Type`: API 요청·응답 데이터 구조 정의

---

## 구조 관리 원칙

- 백엔드는 `Router → Service → Repository` 책임을 분리합니다.
- 프론트엔드는 API를 Screen에서 직접 호출하지 않고 Hook을 사용합니다.
- 여러 기능에서 사용하는 코드는 `shared`에 배치합니다.
- 특정 기능에만 사용되는 코드는 해당 `features` 내부에 배치합니다.
- 빈 폴더를 모두 미리 생성하지 않고 실제 코드가 필요할 때 추가합니다.

---

## 상세 코딩 컨벤션

세부 파일명, 함수명, API 작성 규칙은 별도 컨벤션 문서를 따릅니다.

- Backend 코딩 컨벤션
- Frontend 코딩 컨벤션

---

## 환경변수 관리

환경변수는 프로젝트 루트의 `.env` 파일에서 관리합니다.

```env
EXPO_PUBLIC_API_BASE_URL=http://localhost:8000/api
```

실제 환경변수 파일은 Git에 포함하지 않습니다.

```text
.env          → 실제 개발 환경값, Git 제외
.env.example  → 팀 공유용 예시값, Git 포함
```

클라이언트에 포함되는 환경변수에는 JWT Secret, DB Password, 외부 API Secret 등 서버 비밀값을 작성하지 않습니다.

---

## 7. 개발 환경 준비

다음 도구가 설치되어 있어야 합니다.

- Git
- Node.js LTS
- npm
- Python 3.12 권장
- MySQL 또는 MariaDB
- Android Studio 또는 Expo Go
- VS Code

---

## 8. Frontend 실행 방법

```bash
cd frontend
npm install
npx expo start -c
```

Expo Go를 사용하는 경우 터미널에 표시되는 QR 코드를 모바일 기기로 스캔합니다.

Android Emulator 또는 개발 빌드를 사용하는 경우 다음 명령을 사용할 수 있습니다.

```bash
npm run android
```

### Expo 관련 주의사항

- Expo Go 앱 버전과 `package.json`의 Expo SDK 계열이 맞아야 합니다.
- `package.json`과 `package-lock.json`의 Expo 관련 버전이 어긋나면 `node_modules`와 `package-lock.json`을 정리한 뒤 다시 설치합니다.
- 캐시 문제 발생 시 `npx expo start -c`를 사용합니다.

Windows PowerShell에서 정리할 경우:

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item -Force package-lock.json
npm install
npx expo start -c
```

---

## 9. Backend 실행 방법

### 9.1 가상환경 생성

```bash
cd backend
python -m venv venv
```

Windows PowerShell:

```powershell
.\venv\Scripts\Activate.ps1
```

Windows CMD:

```cmd
venv\Scripts\activate
```

macOS / Linux:

```bash
source venv/bin/activate
```

### 9.2 의존성 설치

```bash
pip install -r requirements.txt
```

### 9.3 환경 변수 설정

```bash
cp .env.example .env
```

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

`.env` 예시:

```env
APP_NAME=Trend Leader API
APP_ENV=local
DEBUG=true
API_PREFIX=/api

DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=trend_leader
DB_USER=trend_user
DB_PASSWORD=trend_pass
DATABASE_URL=

JWT_SECRET_KEY=change_this_secret_key_min_16_chars
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

AI_PROVIDER=mock
OPENAI_API_KEY=
GEMINI_API_KEY=

CORS_ORIGINS=http://localhost:8081
```

> 실제 API Key, OAuth Secret, 운영 DB 비밀번호는 절대 Git에 커밋하지 않습니다.

### 9.4 서버 실행

```bash
uvicorn app.main:app --reload
```

또는 FastAPI CLI를 사용하는 경우:

```bash
fastapi dev app/main.py
```

### 9.5 서버 확인

```text
http://127.0.0.1:8000/
```

Swagger 문서:

```text
http://127.0.0.1:8000/docs
```

---

## 10. Docker 실행

Docker Compose를 사용하면 다음 서비스를 함께 실행할 수 있습니다.

| 서비스 | 컨테이너 이름 | 역할 | 기본 포트 |
|---|---|---|---|
| `db` | `trend-leader-db` | MariaDB 데이터베이스 | `3306` |
| `backend` | `trend-leader-api` | FastAPI 백엔드 서버 | `8000` |

백엔드 컨테이너는 MariaDB의 상태 확인이 성공한 이후 실행됩니다.

> 모든 Docker Compose 명령은 `docker-compose.yml`이 있는 **프로젝트 루트 디렉터리**에서 실행합니다.

### 10.1 사전 준비

Docker Desktop을 실행한 뒤 Docker Engine이 정상 동작하는지 확인합니다.

```powershell
docker version
docker compose version
```

명령 실행 중 Docker Engine 연결 오류가 발생하면 Docker Desktop이 실행 중인지 먼저 확인합니다.

---

### 10.2 환경변수 파일 생성

프로젝트에서는 환경변수 파일을 다음과 같이 구분합니다.

| 파일 | 역할 |
|---|---|
| `.env.compose` | Docker Compose와 MariaDB 실행 설정 |
| `backend/.env` | FastAPI 애플리케이션 설정 |
| `.env.compose.example` | Compose 환경변수 예시 |
| `backend/.env.example` | 백엔드 환경변수 예시 |

#### Windows PowerShell

```powershell
Copy-Item .env.compose.example .env.compose
Copy-Item backend/.env.example backend/.env
```

#### macOS / Linux

```bash
cp .env.compose.example .env.compose
cp backend/.env.example backend/.env
```

이미 실제 환경변수 파일이 존재한다면 다시 복사하지 않고 기존 파일을 사용합니다.

---

### 10.3 `.env.compose` 설정

프로젝트 루트의 `.env.compose` 파일을 확인합니다.

```env
# Docker Compose / MariaDB settings

MARIADB_IMAGE_TAG=latest

DB_NAME=trend_leader
DB_USER=trend_user
DB_PASSWORD=change-me
DB_ROOT_PASSWORD=change-me

DB_EXTERNAL_PORT=3306
BACKEND_EXTERNAL_PORT=8000
```

각 항목의 역할은 다음과 같습니다.

| 환경변수 | 설명 |
|---|---|
| `MARIADB_IMAGE_TAG` | 사용할 MariaDB 이미지 태그 |
| `DB_NAME` | 생성할 데이터베이스 이름 |
| `DB_USER` | 애플리케이션용 DB 사용자 |
| `DB_PASSWORD` | 애플리케이션용 DB 사용자 비밀번호 |
| `DB_ROOT_PASSWORD` | MariaDB root 계정 비밀번호 |
| `DB_EXTERNAL_PORT` | 호스트에서 MariaDB에 접근할 포트 |
| `BACKEND_EXTERNAL_PORT` | 호스트에서 FastAPI에 접근할 포트 |

`change-me` 값은 팀 로컬 개발 환경에 맞는 비밀번호로 변경합니다.

> `.env.compose`는 Docker Compose가 자동으로 읽는 기본 파일명이 아닙니다.  
> 따라서 실행할 때 반드시 `--env-file .env.compose` 옵션을 사용합니다.

---

### 10.4 `backend/.env` 설정 확인

`backend/.env`에서 최소한 다음 항목을 확인합니다.

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=trend_leader
DB_USER=trend_user
DB_PASSWORD=trend_pass

DATABASE_URL=

JWT_SECRET_KEY=change_this_secret_key_min_16_chars
```

Docker Compose 실행 시 `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`는 Compose 설정값으로 덮어씁니다.

`DATABASE_URL`은 다음과 같이 **빈 값으로 유지합니다.**

```env
DATABASE_URL=
```

`DATABASE_URL`에 `127.0.0.1` 기반 주소가 설정되어 있으면 백엔드 컨테이너가 MariaDB 컨테이너 대신 자기 자신에게 접속할 수 있습니다.

`JWT_SECRET_KEY`에는 최소 16자 이상의 값을 설정합니다.

---

### 10.5 Compose 설정 검증

컨테이너를 실행하기 전에 환경변수가 정상적으로 치환되는지 확인합니다.

```powershell
docker compose --env-file .env.compose config
```

완성된 Compose 설정이 출력되고 다음과 같은 경고가 없다면 정상입니다.

```text
variable is not set
Defaulting to a blank string
```

경고가 발생하면 다음 항목을 확인합니다.

1. 프로젝트 루트에 `.env.compose` 파일이 존재하는지 확인
2. `--env-file .env.compose` 옵션을 사용했는지 확인
3. `.env.compose`의 환경변수 이름이 누락되지 않았는지 확인

---

### 10.6 컨테이너 실행

MariaDB와 FastAPI 컨테이너를 빌드하고 백그라운드에서 실행합니다.

```powershell
docker compose --env-file .env.compose up --build -d
```

각 옵션의 의미는 다음과 같습니다.

| 옵션 | 설명 |
|---|---|
| `--env-file .env.compose` | Compose 치환 변수 파일 지정 |
| `up` | 서비스 생성 및 실행 |
| `--build` | 백엔드 Docker 이미지를 다시 빌드 |
| `-d` | 백그라운드 실행 |

최초 실행 시 MariaDB 이미지 다운로드와 백엔드 의존성 설치 때문에 로그 출력이 많을 수 있습니다.

---

### 10.7 컨테이너 상태 확인

```powershell
docker compose --env-file .env.compose ps
```

정상적인 경우 다음 컨테이너가 실행 상태로 표시됩니다.

```text
trend-leader-db    Up (healthy)
trend-leader-api   Up
```

MariaDB가 먼저 초기화되고 상태가 `healthy`가 된 이후 FastAPI 백엔드가 실행됩니다.

---

### 10.8 로그 확인

전체 서비스 로그를 확인합니다.

```powershell
docker compose --env-file .env.compose logs
```

MariaDB와 백엔드 로그를 함께 실시간으로 확인합니다.

```powershell
docker compose --env-file .env.compose logs -f db backend
```

최근 로그만 확인하려면 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose logs --tail=100 db backend
```

실시간 로그 확인을 종료할 때는 `Ctrl + C`를 누릅니다. 컨테이너 자체는 계속 실행됩니다.

---

### 10.9 FastAPI 및 DB 연결 확인

브라우저에서 다음 주소에 접속합니다.

| 주소 | 설명 |
|---|---|
| `http://localhost:8000/` | FastAPI 기본 실행 상태 |
| `http://localhost:8000/api/health` | 백엔드 프로세스 상태 |
| `http://localhost:8000/api/health/ready` | 백엔드와 MariaDB 연결 상태 |
| `http://localhost:8000/docs` | Swagger API 문서 |
| `http://localhost:8000/redoc` | ReDoc API 문서 |

Windows PowerShell에서는 다음 명령으로 확인할 수 있습니다.

```powershell
Invoke-RestMethod http://localhost:8000/
Invoke-RestMethod http://localhost:8000/api/health
Invoke-RestMethod http://localhost:8000/api/health/ready
```

`/api/health/ready` 응답의 `database` 값이 `available`이면 FastAPI와 MariaDB 연결이 정상입니다.

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Trend Leader API is ready",
  "data": {
    "service": "trend-leader-api",
    "status": "ready",
    "database": "available"
  }
}
```

---

### 10.10 컨테이너 중지 및 재실행

컨테이너를 중지하고 제거합니다.

```powershell
docker compose --env-file .env.compose down
```

MariaDB 데이터는 Docker Volume에 남아 있으므로 다음 실행에서도 유지됩니다.

다시 실행할 때는 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose up -d
```

백엔드 코드나 의존성이 변경되어 이미지를 다시 빌드해야 한다면 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose up --build -d
```

서비스만 재시작하려면 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose restart
```

백엔드 컨테이너만 재시작하려면 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose restart backend
```

---

### 10.11 MariaDB 데이터까지 초기화

개발 중 기존 MariaDB 데이터와 Volume을 모두 삭제해야 하는 경우에만 다음 명령을 사용합니다.

```powershell
docker compose --env-file .env.compose down -v
```

그다음 컨테이너를 다시 생성합니다.

```powershell
docker compose --env-file .env.compose up --build -d
```

> `down -v`는 MariaDB 데이터가 저장된 Docker Volume까지 삭제합니다.  
> 보존해야 할 데이터가 있다면 실행하지 않습니다.

MariaDB 계정이나 비밀번호를 변경했는데 기존 설정이 계속 적용되는 경우, 기존 Volume에 초기 계정 정보가 남아 있을 수 있습니다. 개발 데이터 삭제가 가능한 상황에서만 Volume 초기화를 수행합니다.

---

### 10.12 자주 발생하는 문제

#### 환경변수 미설정 경고

```text
The "DB_NAME" variable is not set.
Defaulting to a blank string.
```

실행 명령에 `--env-file .env.compose`가 포함되었는지 확인합니다.

```powershell
docker compose --env-file .env.compose up --build -d
```

#### 포트 충돌

```text
Ports are not available
address already in use
```

`.env.compose`에서 외부 포트를 변경합니다.

```env
DB_EXTERNAL_PORT=3307
BACKEND_EXTERNAL_PORT=8001
```

변경 후 접속 주소도 해당 포트를 사용합니다.

```text
http://localhost:8001/
http://localhost:8001/docs
```

컨테이너 내부의 DB 포트 `3306`과 FastAPI 포트 `8000`은 변경하지 않습니다.

#### 백엔드가 MariaDB에 연결하지 못함

다음 항목을 확인합니다.

1. `backend/.env`의 `DATABASE_URL`이 비어 있는지 확인
2. `.env.compose`의 DB 환경변수가 모두 설정되어 있는지 확인
3. MariaDB 컨테이너가 `healthy` 상태인지 확인
4. MariaDB와 백엔드 로그 확인

```powershell
docker compose --env-file .env.compose ps
docker compose --env-file .env.compose logs --tail=100 db backend
```

#### Docker Engine 연결 오류

```text
failed to connect to the docker API
open //./pipe/dockerDesktopLinuxEngine
```

Docker Desktop을 실행하고 Linux Container Engine이 정상적으로 시작될 때까지 확인한 뒤 명령을 다시 실행합니다.

---

### 10.13 빠른 실행 명령 모음

최초 실행:

```powershell
Copy-Item .env.compose.example .env.compose
Copy-Item backend/.env.example backend/.env
docker compose --env-file .env.compose config
docker compose --env-file .env.compose up --build -d
docker compose --env-file .env.compose ps
```

상태 확인:

```powershell
Invoke-RestMethod http://localhost:8000/api/health/ready
```

로그 확인:

```powershell
docker compose --env-file .env.compose logs -f db backend
```

종료:

```powershell
docker compose --env-file .env.compose down
```

---

## 11. 주요 API 설계

공통 응답 구조:

```json
{
  "success": true,
  "statusCode": 200,
  "message": "요청에 성공했습니다.",
  "data": {}
}
```

### Auth API

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/api/auth/signup` | 회원가입 |
| GET | `/api/auth/check-login-id` | 아이디 중복 확인 |
| POST | `/api/auth/login` | 로그인 |
| POST | `/api/auth/google-login` | 구글 로그인 |
| POST | `/api/auth/logout` | 로그아웃 |

### User API

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/users/me` | 내 정보 조회 |
| PATCH | `/api/users/me` | 회원 정보 수정 |
| PATCH | `/api/users/me/password` | 비밀번호 변경 |
| DELETE | `/api/users/me` | 회원 탈퇴 |

### Interest API

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/categories` | 카테고리 목록 조회 |
| POST | `/api/users/me/interests` | 관심사 저장 |
| GET | `/api/users/me/interests` | 기존 관심사 조회 |
| PUT | `/api/users/me/interests` | 관심사 수정 |

### Trend API

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/trends/recommended` | 맞춤 트렌드 목록 조회 |
| GET | `/api/trends` | 전체 트렌드 목록 조회 |
| GET | `/api/trends/{trend_id}` | 트렌드 상세 조회 |

### Bookmark API

| Method | Endpoint | 설명 |
|---|---|---|
| POST | `/api/trends/{trend_id}/bookmark` | 트렌드 저장 |
| DELETE | `/api/trends/{trend_id}/bookmark` | 트렌드 저장 해제 |
| GET | `/api/users/me/bookmarks` | 저장 트렌드 목록 조회 |

### Search API

| Method | Endpoint | 설명 |
|---|---|---|
| GET | `/api/trends/search` | 트렌드 검색 |
| GET | `/api/users/me/search-logs` | 최근 검색어 조회 |
| DELETE | `/api/users/me/search-logs/{search_log_id}` | 검색 기록 삭제 |

---

## 12. Database 주요 테이블

| 테이블 | 설명 |
|---|---|
| `users` | 사용자 계정 정보 |
| `user_profiles` | 사용자 프로필 |
| `oauth_accounts` | OAuth 계정 연동 정보 |
| `categories` | 관심 카테고리 |
| `user_interest_categories` | 사용자 관심 카테고리 |
| `user_interest_keywords` | 사용자 관심 키워드 |
| `trends` | 트렌드 기본 정보 |
| `trend_category_map` | 트렌드-카테고리 매핑 |
| `trend_rank_snapshots` | 플랫폼별 트렌드 순위 기록 |
| `trend_ai_analyses` | AI 요약, 유행 이유, 상세 설명 |
| `trend_sources` | 트렌드 출처 |
| `trend_related_keywords` | 관련 키워드 |
| `user_trend_bookmarks` | 사용자 저장 트렌드 |
| `search_logs` | 검색 기록 |

---

## 13. Design Repository 구조

디자인 레포지토리는 기능 구현 레포지토리와 분리되어 있으며, 발표자료와 UI/UX 산출물 관리에 사용합니다.

```text
design/
└── designs/
    ├── docs/
    │   ├── temp/
    │   ├── idea/
    │   ├── output/
    │   ├── presentation/
    │   └── UI_UX/
    ├── figma_UI_UX/
    │   ├── figma_img/
    │   └── source/
    ├── logos/
    ├── storyboard/
    └── README.md
```

### 산출물 관리 기준

| 폴더 | 용도 |
|---|---|
| `docs/idea` | 초기 아이디어, 기획 메모 |
| `docs/output` | 최종 산출 문서 |
| `docs/presentation` | 발표자료 |
| `docs/UI_UX` | UI/UX 문서 |
| `figma_UI_UX/figma_img` | Figma 화면 이미지 |
| `figma_UI_UX/source` | UI 제작에 필요한 원본 이미지 |
| `logos` | 로고 및 브랜드 이미지 |
| `storyboard` | 사용자 시나리오, 스토리보드 |

---

## 14. 개발 우선순위

현재 프로젝트는 초기 구조 정리 단계이므로 다음 순서로 구현합니다.

### 1단계: 기반 안정화

- Frontend Expo 실행 안정화
- Backend `/api` prefix 적용
- CORS 설정
- 공통 응답/예외 처리 구조 정리
- README와 `.env.example` 최신화

### 2단계: 관심사 기능

- `GET /api/categories`
- `POST /api/users/me/interests`
- `GET /api/users/me/interests`
- `PUT /api/users/me/interests`

### 3단계: 트렌드 조회

- `GET /api/trends/recommended`
- `GET /api/trends`
- `GET /api/trends/{trend_id}`

### 4단계: 북마크 / 검색

- 트렌드 저장/해제
- 저장 트렌드 목록 조회
- 트렌드 검색
- 최근 검색어 관리

### 5단계: AI 분석

- 초기 mock 응답
- AI 요약 프롬프트 정리
- 외부 AI API 연동
- 분석 결과 저장/조회

---

## 15. Git 협업 규칙

### Branch 전략

권장 브랜치 구조:

```text
main
└── dev
    ├── feature/frontend-init
    ├── feature/backend-init
    ├── feature/auth-api
    ├── feature/interest-api
    └── feature/trend-screen
```

### Commit Message 예시

```text
[init] 프로젝트 초기 구조 생성
[feat] 관심사 카테고리 조회 API 구현
[fix] Expo SDK 버전 불일치 수정
[docs] README 실행 방법 추가
[refactor] 라우터 구조 분리
[test] 관심사 저장 서비스 테스트 추가
```

### Pull Request 기준

PR 작성 시 다음 내용을 포함합니다. (예시이기에 참고만!!)

```text
## 작업 내용
-

## 확인 방법
-

## 관련 이슈
-

## 비고
-
```

---

## 16. 코드 작성 원칙

### Backend

- `main.py`에는 API 로직을 직접 작성하지 않습니다.
- Router는 요청과 응답만 담당합니다.
- 핵심 판단은 Service에서 처리합니다.
- DB 접근은 Repository에서 처리합니다.
- Request / Response는 Pydantic Schema로 정의합니다.
- 공통 응답 구조를 유지합니다.
- 민감 정보는 `.env`로 분리합니다.

### Frontend

- 화면 단위 코드는 `screens`에 둡니다.
- 재사용 UI는 `components`에 둡니다.
- API 호출 코드는 `services`에 둡니다.
- 타입은 `types`에 분리합니다.
- 화면 이동 구조는 `navigation`에 둡니다.
- API 응답 로딩, 실패, 빈 상태를 반드시 고려합니다.

---

## 17. 자주 발생할 수 있는 문제

### 17.1 Expo 앱이 강제 종료되는 경우

확인할 것:

- Expo Go 앱 버전과 프로젝트 Expo SDK 버전이 맞는지 확인
- `package.json`과 `package-lock.json` 버전이 일치하는지 확인
- `node_modules` 재설치
- Metro 캐시 삭제

```bash
npx expo start -c
```

### 17.2 npm install 중 패키지 버전 오류가 나는 경우

```bash
rm -rf node_modules package-lock.json
npm install
```

Windows PowerShell:

```powershell
Remove-Item -Recurse -Force node_modules
Remove-Item -Force package-lock.json
npm install
```

### 17.3 Backend에서 환경 변수 오류가 나는 경우

`backend/.env` 파일이 있는지 확인합니다.

```bash
cp .env.example .env
```

`JWT_SECRET_KEY`는 최소 16자 이상이어야 합니다.

### 17.4 DB 연결 오류가 나는 경우

확인할 것:

- MySQL/MariaDB 서버 실행 여부
- `.env`의 DB 계정 정보
- `DATABASE_URL`
- DB 이름 `trend_leader` 생성 여부
- DB 사용자 권한

---

## 18. 참고 문서

| 문서 | 설명 |
|---|---|
| API 구성표 | API Endpoint, 요청/응답, 화면 흐름 정리 |
| ERD / SQL | DB 테이블 구조 |
| 코드 컨벤션 | Backend / Frontend 폴더 구조 및 작성 규칙 |
| Design Repository | Figma, 발표자료, 로고, 스토리보드 |
| 기능 상세 명세서 | 화면/기능별 세부 구현 기준 |

---

## 19. 프로젝트 상태 메모

현재 프로젝트는 초기화 및 구조 설계 단계입니다.

우선 목표는 다음 세로 흐름을 완성하는 것입니다.

```text
회원가입/로그인
→ 관심사 선택
→ 맞춤 트렌드 목록 조회
→ 트렌드 상세 조회
→ 북마크 저장
```

이 흐름이 완성되면 Trend Leader의 핵심 서비스 구조가 작동하기 시작합니다.

---

## 20. License

본 프로젝트는 강남대학교 컴퓨터공학부 졸업작품 팀 프로젝트로 진행 중입니다.  
라이선스 정책은 추후 팀 내부 협의 후 결정합니다.
