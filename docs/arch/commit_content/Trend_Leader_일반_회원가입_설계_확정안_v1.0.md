# Trend Leader 일반 회원가입 설계 확정안 v1.0

- 문서명: 일반 회원가입 설계 확정안
- 프로젝트: Trend Leader
- 기준 브랜치: `dev`
- 기준 커밋: `208560db397e73fb0d94e1ba96e25eda88be33ce`
- 기준 시점: 2026-08-24
- 상태: 설계 확정
- 범위: 일반 회원가입 기능 설계
- 제외 범위:
  - Google OAuth 회원가입/로그인
  - Refresh Token
  - Token Revocation / Blocklist
  - 회원 정보 조회/수정
  - 비밀번호 변경
  - 회원 탈퇴
  - 관심사 조회/수정
  - Main 화면 실제 트렌드 기능
  - 트렌드 수집/추천/검색/북마크

---

# 1. 문서 목적

이 문서는 Trend Leader의 현재 `dev` 브랜치에 반영된 일반 로그인/JWT 인증 구조와 인증 세션 관리/로그아웃 구조를 기준으로, **일반 회원가입 기능의 최종 설계**를 확정한다.

이번 설계의 핵심 원칙은 다음과 같다.

1. 기존 인증 구조를 변경하지 않고 회원가입 기능을 그 위에 연결한다.
2. 회원가입 성공 후에는 별도의 Login 재입력을 요구하지 않고 자동 로그인한다.
3. 회원가입 화면이 직접 관심사 화면으로 Navigation하지 않는다.
4. 회원가입 성공 시 `AuthProvider`에 인증 세션을 등록하고, `RootNavigator`가 `next_step`을 기준으로 `INTEREST_SELECTION` 화면을 선택한다.
5. 사전 중복확인은 UX를 위한 기능이며, 최종 데이터 정합성은 DB `UNIQUE` 제약으로 보장한다.
6. 회원가입 Transaction의 경계는 Service 계층이 소유한다.
7. 일반 회원가입 요청의 Validation 정책은 DB nullable 여부보다 더 강한 비즈니스 규칙으로 적용한다.

---

# 2. 현재 dev 기준 확인 사항

## 2.1 User ORM Model

현재 `users` 테이블 관련 핵심 필드는 다음과 같다.

| 필드 | ORM 타입/정책 | DB 제약 |
|---|---|---|
| `user_id` | `int` | PK |
| `login_id` | `str \| None` | nullable, UNIQUE |
| `password_hash` | `str \| None` | nullable |
| `name` | `str` | NOT NULL |
| `email` | `str \| None` | nullable, UNIQUE |
| `status` | `UserStatus` | NOT NULL, 기본값 `ACTIVE` |
| `created_at` | datetime | DB 기본값 |
| `updated_at` | datetime \| None | nullable |
| `withdrawn_at` | datetime \| None | nullable |
| `withdraw_reason` | str \| None | nullable |

### 설계 해석

`login_id`, `password_hash`, `email`이 DB에서 nullable인 이유는 향후 OAuth 계정 구조까지 포괄하기 위함이다.

따라서:

- DB 수준: `login_id`, `password_hash`, `email` nullable 허용
- 일반 회원가입 Use Case 수준:
  - `login_id`: 필수
  - `password`: 필수
  - `name`: 필수
  - `email`: 선택

으로 적용한다.

---

## 2.2 UserRepository

현재 존재하는 메서드는 다음과 같다.

```text
find_by_id()
find_by_login_id()
```

회원가입 구현 시 아래 메서드는 새로 추가 대상이다.

```text
find_by_email()
save()
```

단, 현재 존재하는 메서드로 간주하지 않는다.

Repository의 저장 책임은 현재 프로젝트의 기존 write 구조에 맞추어:

```text
add / flush
```

까지만 담당한다.

최종 `commit / rollback`은 Service에서 처리한다.

---

## 2.3 AuthService

현재 `AuthService`는 다음 기능을 담당한다.

```text
login()
get_session()
```

로그인 성공 시 현재 세션 계약은 다음 세 값으로 구성된다.

```text
user
has_selected_interests
next_step
```

현재 `next_step`은 다음 규칙을 따른다.

```text
has_selected_interests == true
→ MAIN

has_selected_interests == false
→ INTEREST_SELECTION
```

일반 회원가입 역시 이 세션 계약을 그대로 재사용한다.

---

## 2.4 Auth Router / Schema / Dependency

현재 구현된 주요 Auth API는 다음과 같다.

```text
POST /api/auth/login
GET  /api/auth/session
```

현재 일반 회원가입용 Schema, Router Endpoint는 존재하지 않는다.

추가 대상:

```text
POST /api/auth/signup
GET  /api/auth/check-login-id
```

회원가입 및 로그인 ID 중복확인 API는 인증 전 단계이므로 Bearer 인증 Dependency를 적용하지 않는다.

---

## 2.5 비밀번호 Hashing 구조

현재 `core.security`는:

```text
PasswordHash.recommended()
```

를 사용하고 있으며 로그인에서는 `verify_password()`로 검증한다.

회원가입 구현 시 같은 Hasher를 사용하는:

```text
hash_password()
```

진입점을 추가한다.

회원가입 Service가 별도의 `PasswordHash()` 인스턴스나 별도의 Hash 알고리즘을 생성하지 않는다.

최종 구조:

```text
core.security

PasswordHash.recommended()
       │
       ├─ hash_password()    ← 회원가입
       └─ verify_password()  ← 로그인
```

---

## 2.6 Frontend 인증 구조

현재 Frontend는 다음 구조를 사용한다.

```text
QueryProvider
  → AuthProvider
    → RootNavigator
```

`AuthProvider` 주요 상태:

```text
RESTORING
UNAUTHENTICATED
AUTHENTICATED
RESTORE_ERROR
```

주요 메서드:

```text
establishSession()
restoreSession()
revalidateSession()
completeInterestSelection()
logout()
```

`RootNavigator`는 인증 상태와 `next_step`을 기준으로 화면을 선언적으로 분기한다.

```text
UNAUTHENTICATED
→ AuthNavigator

AUTHENTICATED + INTEREST_SELECTION
→ OnboardingNavigator

AUTHENTICATED + MAIN
→ AppNavigator
```

현재 `AuthNavigator`에는 Login 화면만 존재한다.

회원가입 구현 시:

```text
AuthNavigator
├─ Login
└─ Signup
```

구조로 확장한다.

---

# 3. 최종 회원가입 API

## 3.1 Endpoint

```http
POST /api/auth/signup
```

### 인증 여부

불필요

### Frontend Client

```text
publicApiClient
```

### 책임

```text
입력 검증
→ login_id / email 사전 중복 조회
→ 비밀번호 Hash
→ User 생성
→ Transaction Commit
→ Access Token 발급
→ Session 정보 구성
→ 응답 반환
```

회원가입 Transaction 안에서 관심사를 생성하거나 저장하지 않는다.

---

# 4. 회원가입 Request 구조

최종 Request Body:

```json
{
  "login_id": "trend_user01",
  "password": "example-long-password",
  "password_confirm": "example-long-password",
  "name": "김트렌드",
  "email": null
}
```

## 4.1 필드 정책

| 필드 | 필수 여부 | Validation 정책 |
|---|---:|---|
| `login_id` | 필수 | 4~50자, 영문 소문자/숫자/`_`, 첫 글자 영문 소문자 |
| `password` | 필수 | 15~128자, trim 금지 |
| `password_confirm` | 필수 | `password`와 정확히 일치 |
| `name` | 필수 | trim 후 1~50자, Unicode 허용 |
| `email` | 선택 | 누락/NULL/공백은 `None`, 값이 있으면 이메일 형식 검증 |

---

# 5. login_id Validation 정책

최종 정책:

```text
최소 길이: 4
최대 길이: 50
허용 문자: 영문 소문자, 숫자, _
첫 문자: 영문 소문자
```

예시:

```text
trend_user01   O
user123        O

TrendUser      X
trend-user     X
한재123         X
```

## 목적

- 로그인 ID를 DB collation의 대소문자 처리에 과도하게 의존하지 않게 한다.
- Frontend/Backend 양쪽에서 동일한 형식 규칙을 적용하기 쉽게 한다.
- 향후 사용자 검색이나 식별 규칙을 단순화한다.

---

# 6. password Validation 정책

최종 정책:

```text
최소 15자
최대 128자
대문자 필수 X
숫자 필수 X
특수문자 필수 X
입력값 trim X
```

`password_confirm`은 요청 검증 전용 필드이며 DB에 저장하지 않는다.

### 처리 원칙

```text
password
→ hash_password()
→ users.password_hash
```

원문 비밀번호는 로그, DB, Response에 포함하지 않는다.

---

# 7. name Validation 정책

최종 정책:

```text
trim 후 최소 1자
최대 50자
한글 등 Unicode 허용
```

빈 문자열 또는 공백만 입력된 값은 유효하지 않은 입력으로 처리한다.

---

# 8. email 정책

## 8.1 필수 여부

선택값으로 유지한다.

DB의 nullable 구조와 기존 API 명세를 그대로 존중한다.

## 8.2 정규화

다음 입력은 모두 DB에 `NULL`로 저장한다.

```text
필드 누락
null
""
"   "
```

실제 이메일 문자열이 있을 경우:

```text
trim
→ 형식 검증
→ canonical form 적용
→ 저장
```

MVP 기준 canonical form은 소문자 통일을 권고한다.

## 8.3 별도 이메일 중복확인 API

이번 일반 회원가입 범위에서는 별도 `/check-email` API를 추가하지 않는다.

회원가입 요청 시 Backend에서 최종 확인한다.

---

# 9. 로그인 ID 중복확인 API

## 9.1 Endpoint

```http
GET /api/auth/check-login-id?login_id=trend_user01
```

### 인증 여부

불필요

### Frontend Client

```text
publicApiClient
```

## 9.2 Response

사용 가능한 경우:

```json
{
  "success": true,
  "statusCode": 200,
  "message": "사용 가능한 아이디입니다.",
  "data": {
    "login_id": "trend_user01",
    "is_available": true,
    "reason": null
  }
}
```

중복인 경우:

```json
{
  "success": true,
  "statusCode": 200,
  "message": "이미 사용 중인 아이디입니다.",
  "data": {
    "login_id": "trend_user01",
    "is_available": false,
    "reason": "DUPLICATED_LOGIN_ID"
  }
}
```

### 설계 원칙

중복 여부 확인 자체는 정상적인 조회 요청이므로 HTTP `200 OK`를 유지한다.

Frontend에서는 사용자가 `login_id`를 수정하면 기존 중복확인 결과를 즉시 무효화한다.

---

# 10. 중복 처리와 Race Condition

## 10.1 사전 중복확인의 역할

사전 중복확인은 UX 개선을 위한 기능이다.

```text
/check-login-id
→ 사용 가능 여부를 사용자에게 빠르게 안내
```

하지만 데이터 정합성을 보장하지 않는다.

## 10.2 Race Condition 예시

```text
A → abc 사용 가능 확인
B → abc 사용 가능 확인

A → signup
B → signup
```

두 요청 모두 사전 조회 시점에는 사용 가능할 수 있다.

따라서 최종 구조는 다음 3단계로 한다.

```text
1. 사전 중복확인 API
2. signup Service에서 중복 재조회
3. DB UNIQUE Constraint가 최종 방어
```

### 핵심 원칙

```text
사전 조회
→ 친절한 UX / 빠른 오류 안내

DB UNIQUE Constraint
→ 최종 데이터 무결성 보장
```

---

# 11. DB Unique Constraint 위반 처리

현재 DB에는 다음 UNIQUE 제약이 존재한다.

```text
uq_users_login_id
uq_users_email
```

회원가입 Service는 사전 조회를 수행하더라도 DB 저장 시 `IntegrityError`를 처리해야 한다.

## 최종 처리

```text
uq_users_login_id 위반
→ rollback
→ ConflictException
→ 409 Conflict
→ reason = DUPLICATED_LOGIN_ID

uq_users_email 위반
→ rollback
→ ConflictException
→ 409 Conflict
→ reason = DUPLICATED_EMAIL
```

알 수 없는 `IntegrityError`를 전부 409로 변환하지 않는다.

원인이 명확하지 않은 DB 오류는 다시 상위로 전달하여 공통 500 처리로 보낸다.

---

# 12. Validation / HTTP Status 정책

최종 정책:

| 상황 | HTTP Status |
|---|---:|
| 필수값 누락 | 422 |
| login_id 형식 오류 | 422 |
| password 길이 오류 | 422 |
| password_confirm 불일치 | 422 |
| name 형식 오류 | 422 |
| email 형식 오류 | 422 |
| login_id 중복 | 409 |
| email 중복 | 409 |
| 알 수 없는 DB/서버 오류 | 500 |

기존 Notion 명세의 일부 `400` Validation 항목은 현재 프로젝트의 공통 `RequestValidationError → 422` 정책에 맞추어 `422`로 통일한다.

---

# 13. 회원가입 Transaction 책임

최종 결정:

```text
AuthService가 Transaction 경계를 소유한다.
```

권고 흐름:

```text
AuthService
  ↓
UserRepository.save()
  ↓
flush
  ↓
commit
```

오류 발생 시:

```text
rollback
→ 예상 가능한 중복이면 409
→ 그 외 오류는 상위로 전달
```

Repository는 저장 방식만 알고, Service가 하나의 회원가입 Use Case를 최종 확정할지 결정한다.

회원가입 구현 시 `AuthService`에 현재 없는 `db: Session` 의존성을 추가하는 방향을 권고한다.

---

# 14. 신규 사용자 기본값

회원가입 성공 시 신규 User는 다음 상태로 생성한다.

```text
login_id        = 요청값
password_hash   = hash(password)
name            = 요청값
email           = 요청값 또는 NULL

status          = ACTIVE
created_at      = DB default
updated_at      = NULL
withdrawn_at    = NULL
withdraw_reason = NULL
```

회원가입 시 아래 관련 Row는 생성하지 않는다.

```text
user_interest_categories  생성 X
user_profiles              생성 X
oauth_accounts             생성 X
```

가입 직후 관심사가 없다는 사실 자체가 다음 Session 상태를 의미한다.

```text
has_selected_interests = false
next_step = INTEREST_SELECTION
```

---

# 15. 회원가입 성공 후 처리 방식 비교

## 15.1 방식 1 — 가입 후 Login 화면으로 이동

흐름:

```text
Signup
→ 201
→ UNAUTHENTICATED 유지
→ Login
→ 사용자가 ID/PW 재입력
→ POST /auth/login
→ establishSession()
→ INTEREST_SELECTION
```

### 장점

- Signup API 책임이 단순하다.
- 사용자 생성과 인증 과정을 명확히 분리할 수 있다.

### 단점

- 사용자가 동일한 ID/PW를 다시 입력해야 한다.
- 로그인 API 호출이 한 번 더 필요하다.
- 현재 `RootNavigator` 구조에서 UNAUTHENTICATED 사용자는 Onboarding으로 진입할 수 없다.
- 가입 직후 관심사 선택이라는 기존 UX와 단절된다.

---

## 15.2 방식 2 — 가입 성공 즉시 자동 로그인

흐름:

```text
Signup
→ User 생성
→ Access Token 발급
→ Token + Session 응답
→ AuthProvider.establishSession()
→ AUTHENTICATED
→ INTEREST_SELECTION
→ OnboardingNavigator
```

### 장점

- 현재 로그인 성공 흐름을 거의 그대로 재사용할 수 있다.
- 사용자가 ID/PW를 다시 입력할 필요가 없다.
- `RootNavigator`가 기존 방식대로 화면을 결정한다.
- Android Back으로 Login/Signup 화면에 복귀하는 문제를 구조적으로 방지할 수 있다.
- 회원가입 화면에 별도 Navigation 분기 로직을 넣지 않아도 된다.

### 단점

- Signup API가 사용자 생성 외에 Access Token 발급까지 담당한다.
- 가입 성공과 Session 저장 실패를 구분해서 처리해야 한다.

---

# 16. 최종 선택 — 자동 로그인 방식

Trend Leader는 **방식 2를 최종 채택한다.**

회원가입 성공 후:

```text
POST /api/auth/signup
→ Access Token + Session 응답
→ AuthProvider.establishSession()
→ AUTHENTICATED 상태 등록
→ RootNavigator가 next_step 평가
→ OnboardingNavigator
→ INTEREST_SELECTION
```

회원가입 화면에서 직접 다음 코드를 호출하는 형태는 사용하지 않는다.

```text
navigation.replace("InterestSelection")
```

대신 인증 상태 변경이 Navigation 결과를 유도한다.

---

# 17. 회원가입 Response 구조

회원가입 성공 응답은 로그인 API와 동일한 Token + Session 계약을 사용한다.

최종 `data` 구조:

```json
{
  "access_token": "<jwt>",
  "token_type": "Bearer",
  "user": {
    "user_id": 123,
    "login_id": "trend_user01",
    "name": "김트렌드",
    "status": "ACTIVE"
  },
  "has_selected_interests": false,
  "next_step": "INTEREST_SELECTION"
}
```

전체 응답:

```json
{
  "success": true,
  "statusCode": 201,
  "message": "회원가입이 완료되었습니다.",
  "data": {
    "access_token": "<jwt>",
    "token_type": "Bearer",
    "user": {
      "user_id": 123,
      "login_id": "trend_user01",
      "name": "김트렌드",
      "status": "ACTIVE"
    },
    "has_selected_interests": false,
    "next_step": "INTEREST_SELECTION"
  }
}
```

## 결정 사항

기존 Signup API 명세에 있던 `email`은 회원가입 성공 Session Response에서는 제외한다.

이유:

- 현재 로그인 Session의 `user`에도 email이 없다.
- 인증 세션 및 Navigation 결정에 email이 필요하지 않다.
- Login / Signup 성공 Response 구조를 일관되게 유지할 수 있다.

---

# 18. 별도 Session 조회 여부

회원가입 성공 후:

```text
GET /api/auth/session
```

을 추가 호출하지 않는다.

최종 결정:

```text
Signup Response 자체에
user
has_selected_interests
next_step
포함
```

Backend 내부에서는 기존 `get_session(user)` 로직을 재사용하여 로그인과 동일한 Session 계산 규칙을 유지한다.

---

# 19. AuthProvider 연결

현재:

```text
establishSession(loginResponse: LoginResponse)
```

처럼 로그인 응답 타입에 직접 결합되어 있다.

회원가입 자동 로그인 도입 시 로그인/회원가입 성공 응답 모두 동일한 Token + Session 구조를 갖게 되므로, 구현 단계에서 `establishSession()`의 인자 타입을 로그인 전용 이름에서 일반적인 인증 Session 응답 타입으로 일반화하는 것을 권고한다.

개념적 구조:

```text
LoginResponse
SignupResponse
      ↓
Token + Session Contract
      ↓
AuthProvider.establishSession()
```

이는 AuthProvider의 책임을 변경하는 것이 아니라 현재 책임을 타입에도 정확히 반영하는 소규모 리팩터링이다.

---

# 20. Public / Authenticated API Client 정책

다음 API는 모두 `publicApiClient`를 사용한다.

```text
POST /api/auth/login
POST /api/auth/signup
GET  /api/auth/check-login-id
```

이유:

- 세 API 모두 인증을 획득하기 전에 호출한다.
- 인증 Client의 Authorization 자동 주입 및 401 Session Termination 정책을 적용할 이유가 없다.

회원가입 및 중복확인 API에 `authenticatedApiClient`를 사용하지 않는다.

---

# 21. 회원가입 화면 Navigation 위치

최종 구조:

```text
AuthNavigator
├─ Login
└─ Signup
```

Login 화면에서 회원가입 버튼 클릭 시:

```text
Login
→ Signup
```

만 수행한다.

회원가입 성공 후에는 SignupScreen에서 Interest 화면으로 직접 이동하지 않는다.

실제 화면 전환은:

```text
establishSession()
→ AuthState = AUTHENTICATED
→ RootNavigator 재평가
→ AuthNavigator unmount
→ OnboardingNavigator mount
```

로 이루어진다.

---

# 22. Frontend Error 처리 정책

최종 정책:

| 상황 | 처리 |
|---|---|
| 로컬 입력 오류 | API 호출 전 필드 하단 표시 |
| 422 | Backend ValidationErrorData를 필드별 표시 |
| 409 login_id | login_id 필드 강조, 중복확인 상태 무효화 |
| 409 email | email 필드 강조 |
| Network Error | 입력값 유지, 재시도 안내 |
| 5xx | 입력값 유지, 일반 서버 오류 안내 |
| Signup 성공 | password/password_confirm 메모리 상태 정리 후 Session 확립 |

## 22.1 Machine-readable 409 응답

Frontend가 한글 메시지를 파싱하여 오류 필드를 판단하지 않도록 한다.

권고 예시:

```json
{
  "success": false,
  "statusCode": 409,
  "message": "이미 사용 중인 아이디입니다.",
  "data": {
    "field": "login_id",
    "reason": "DUPLICATED_LOGIN_ID"
  }
}
```

이메일 중복:

```json
{
  "success": false,
  "statusCode": 409,
  "message": "이미 사용 중인 이메일입니다.",
  "data": {
    "field": "email",
    "reason": "DUPLICATED_EMAIL"
  }
}
```

---

# 23. 가입 성공 후 SecureStore 저장 실패

자동 로그인에서는 다음 실패 경로를 별도로 처리해야 한다.

```text
POST /signup 성공
→ DB User 생성 완료
→ Access Token 수신
→ establishSession()
→ SecureStore 저장 실패
```

이 경우 회원가입 API를 다시 호출하지 않는다.

이미 DB 가입은 완료되었기 때문이다.

최종 처리:

```text
회원가입은 완료됨
→ Session 등록 실패
→ UNAUTHENTICATED 상태로 복구
→ Login 화면에서 다시 로그인하도록 안내
```

권고 메시지:

```text
회원가입은 완료되었지만 로그인 정보를 저장하지 못했습니다.
로그인 화면에서 다시 로그인해 주세요.
```

---

# 24. Backend ↔ Frontend 최종 API 계약

## 24.1 회원가입

```text
POST /api/auth/signup

Authentication:
없음

Client:
publicApiClient

Success:
201 Created

data:
- access_token
- token_type
- user
- has_selected_interests
- next_step
```

오류:

```text
422
→ Request Validation

409
→ DUPLICATED_LOGIN_ID
→ DUPLICATED_EMAIL

500
→ 예상하지 못한 서버/DB 오류
```

---

## 24.2 로그인 ID 중복 확인

```text
GET /api/auth/check-login-id?login_id=...

Authentication:
없음

Client:
publicApiClient
```

성공:

```text
200 OK
```

응답 데이터:

```text
login_id
is_available
reason
```

오류:

```text
422
→ login_id 형식 오류

500
→ 서버/DB 오류
```

---

# 25. 필요한 Backend 테스트 범위

## 25.1 Schema / Router

- 정상 Request → 201
- login_id 누락 → 422
- login_id 형식 오류 → 422
- password 길이 오류 → 422
- password_confirm 불일치 → 422
- name 누락/형식 오류 → 422
- email 형식 오류 → 422
- 인증 Header 없이 정상 호출 가능
- 성공 Response Contract 검증

## 25.2 AuthService

- 정상 가입 → `ACTIVE` User 생성
- email 미입력 → `None`
- 비밀번호 원문 미저장
- 생성된 hash가 `verify_password()`로 검증 가능
- `has_selected_interests == false`
- `next_step == INTEREST_SELECTION`
- Access Token `sub == user_id`
- 기존 login_id → 409
- 기존 email → 409
- WITHDRAWN/SUSPENDED 계정의 login_id도 사용 불가
- login_id UNIQUE Race → rollback + 409
- email UNIQUE Race → rollback + 409
- 알려지지 않은 DB 오류 → 500 경로

## 25.3 UserRepository Integration

- `find_by_email()` 정상 조회
- `find_by_email()` 미존재 → None
- `save()` flush 후 PK 확보
- login_id UNIQUE 검증
- email UNIQUE 검증
- 여러 NULL email 저장 가능

## 25.4 check-login-id

- 없는 ID → `is_available = true`
- 존재하는 ID → `is_available = false`
- WITHDRAWN 계정 ID도 false
- 잘못된 형식 → 422

---

# 26. 필요한 Frontend 테스트 범위

- Signup API가 `publicApiClient` 사용
- check-login-id가 `publicApiClient` 사용
- 중복확인 성공 → signup 가능 상태
- loginId 변경 → 기존 중복확인 상태 초기화
- Signup 201 → `establishSession()` 호출
- Signup 성공 → `AUTHENTICATED + INTEREST_SELECTION`
- 409 login_id → login_id 필드 오류
- 409 email → email 필드 오류
- 422 → 필드 Validation 오류
- Network / 500 → 입력값 유지
- Signup 성공 + SecureStore 실패 → Signup 재호출 없음
- Signup 성공 후 AuthNavigator 제거
- Android Back으로 Login/Signup 복귀 불가

---

# 27. 구현 순서

설계 확정 후 구현은 다음 순서를 따른다.

## Phase 1 — Backend

```text
1. Signup Request / Response Schema
2. login_id / password / name / email Validation
3. core.security hash_password()
4. UserRepository find_by_email()
5. UserRepository save()
6. AuthService signup Transaction
7. UNIQUE → 409 Race 처리
8. Signup Access Token + Session 생성
9. GET /auth/check-login-id
10. auth_router Endpoint 연결
```

---

## Phase 2 — Backend 테스트

```text
1. Schema Validation
2. UserRepository Integration
3. AuthService Unit
4. Race / IntegrityError
5. Router Contract
6. 전체 pytest
```

Backend API Contract 검증이 완료된 뒤 Frontend 구현으로 넘어간다.

---

## Phase 3 — Frontend

```text
1. Signup / CheckLoginId Type
2. publicApiClient API Function
3. useSignup / 중복확인 Mutation
4. AuthProvider establishSession 타입 일반화
5. AuthNavigator Signup 추가
6. SignupScreen 구현
7. Login → Signup Navigation
8. Signup 성공 → establishSession 연결
9. 422 / 409 / Network Error 처리
10. Jest + typecheck + verify
```

---

## Phase 4 — 실제 기기 검증

```text
01. Login → Signup 이동
02. ID 중복확인 available
03. ID 중복확인 duplicate
04. ID 변경 후 중복확인 상태 초기화
05. email 없이 가입
06. email 포함 가입
07. 중복 email → 오류
08. 잘못된 password → 입력 오류
09. 정상 가입 → Token SecureStore 저장
10. 정상 가입 → Login 화면을 거치지 않음
11. 즉시 InterestSelection 표시
12. Android Back → Signup/Login으로 돌아가지 않음
13. 관심사 저장 → Main
14. 앱 완전 종료/재실행 → Session Restore → Main
15. Network 실패 → 입력 유지
```

---

# 28. 최종 확정 요약

| 항목 | 최종 확정안 |
|---|---|
| Signup Endpoint | `POST /api/auth/signup` |
| 인증 | 불필요 |
| Client | `publicApiClient` |
| `login_id` | 일반 가입 필수 |
| `password` | 필수 |
| `password_confirm` | 필수, 검증 전용 |
| `name` | 필수 |
| `email` | 선택, 빈 값은 NULL |
| 로그인 ID 중복확인 | `GET /api/auth/check-login-id` 유지 |
| Email 별도 중복확인 API | 추가하지 않음 |
| Signup에서도 중복 재조회 | 필수 |
| DB UNIQUE | 최종 무결성 방어 |
| Duplicate HTTP Status | `409 Conflict` |
| Schema Validation | `422` |
| Password Hash | 현재 `pwdlib PasswordHash.recommended()` 재사용 |
| Transaction | `AuthService` 책임 |
| 초기 status | `ACTIVE` |
| 초기 관심사 Row | 없음 |
| 가입 성공 | 자동 로그인 |
| Signup Token | Access Token 발급 |
| Signup Session 정보 | Login과 동일하게 포함 |
| `/auth/session` 추가 조회 | 하지 않음 |
| 초기 `has_selected_interests` | `false` |
| 초기 `next_step` | `INTEREST_SELECTION` |
| Session 등록 | `AuthProvider.establishSession()` 계열 재사용 |
| Signup Navigation | `AuthNavigator` 내부 |
| 관심사 화면 이동 | `RootNavigator`가 Auth State로 결정 |
| SignupScreen 직접 Interest Navigation | 사용하지 않음 |

---

# 29. 후속 작업으로 분리할 항목

아래 항목은 이번 설계에서 구현하지 않는다.

- Google OAuth 회원가입/로그인
- Refresh Token
- Token Revocation / Blocklist
- 회원 정보 조회/수정
- 비밀번호 변경
- 회원 탈퇴
- 탈퇴 사용자 login_id/email 재사용 정책
- 관심사 조회/수정
- Main 실제 트렌드 기능
- Password Blocklist
- 로그인/회원가입 Rate Limiting
- Email Verification
- OAuth 동일 이메일 계정 연결 정책

---

# 30. 설계 핵심 문장

이번 일반 회원가입 설계의 핵심은 다음 한 문장으로 정리한다.

> 회원가입 화면이 관심사 화면으로 이동하는 것이 아니라, 회원가입 성공이 인증 세션을 확립하고 기존 Root Navigation이 그 세션 상태를 보고 관심사 화면을 선택한다.

이 원칙을 유지하면 일반 로그인, 회원가입, 향후 OAuth 로그인까지 인증 이후의 화면 전환 규칙을 하나의 구조로 일관되게 유지할 수 있다.
