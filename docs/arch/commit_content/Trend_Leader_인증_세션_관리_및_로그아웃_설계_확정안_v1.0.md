# Trend Leader 인증 세션 관리 및 로그아웃 설계 확정안 v1.0

## 0. 문서 목적

본 문서는 Trend Leader의 일반 로그인/JWT 인증 구현 이후 진행할 **인증 세션 복원, 공통 401 처리, 로그아웃, 인증 기반 Root Navigation** 구조를 확정하기 위한 문서이다.

이 문서는 이후 별도 구현 채팅에서 이전 설계 대화를 다시 읽지 않아도 작업을 시작할 수 있도록 다음 내용을 포함한다.

- 현재 인증 구조
- Frontend Auth State 구조
- 앱 시작 시 세션 복원
- Backend Session API
- Public / Authenticated API Client 분리
- Access Token 만료 및 401 처리
- 로그아웃
- 관심사 선택과 세션 상태 연계
- Root Navigation
- 보안 규칙
- 테스트 범위
- 구현 순서
- 이번 작업 범위 밖의 후속 과제

### 설계 기준 Repository

```text
Repository:
Trend-Leader-capstoneProject/feature

기준 Branch:
dev

설계 확정 당시 HEAD:
a9829fd74f77e8fb6e29e1fd6bfc066ae6353aa8
```

구현 시작 시에는 **위 SHA를 고정해서 작업하지 말고 반드시 `dev` 최신 HEAD를 다시 확인한 뒤 차이가 있으면 인증 관련 변경사항을 먼저 재검증한다.**

---

## 1. 현재 인증 구조

현재 Backend 일반 로그인은 다음 구조까지 구현되어 있다.

```text
POST /api/auth/login

Request
├─ login_id
└─ password

Response
├─ access_token
├─ token_type = "Bearer"
├─ user
│   ├─ user_id
│   ├─ login_id
│   ├─ name
│   └─ status
├─ has_selected_interests
└─ next_step
    ├─ MAIN
    └─ INTEREST_SELECTION
```

현재 JWT Access Token은 다음 정보를 가진다.

```text
sub = str(user_id)
exp = Access Token 만료 시각
```

Refresh Token은 사용하지 않는다.

Backend 보호 API에서는 `CurrentUserDep / get_current_user()`를 통하여 다음을 검증한다.

```text
Authorization: Bearer <token>
        ↓
JWT 서명 검증
        ↓
exp 검증
        ↓
sub 검증
        ↓
user_id 변환
        ↓
DB 사용자 조회
        ↓
사용자 존재 여부 검증
        ↓
UserStatus.ACTIVE 검증
        ↓
인증 성공
```

Frontend에는 다음 기반 코드가 존재한다.

```text
SecureStore
├─ saveAccessToken()
├─ getAccessToken()
└─ deleteAccessToken()

apiClient
└─ Request Interceptor
      └─ SecureStore Token 조회
            ↓
         Authorization Bearer 자동 주입

useLogin
└─ 로그인 성공 시 Access Token SecureStore 저장
```

그러나 현재는 앱 전체 인증 상태를 관리하는 계층과 세션 복원 기능이 존재하지 않으며 `RootNavigator`도 항상 Login 화면에서 시작한다.

---

## 2. 이번 설계의 핵심 원칙

최종적으로 다음 구조를 사용한다.

```text
SecureStore
    │
    │ Access Token 영속 저장
    ↓
AuthProvider
    │
    │ 현재 앱 인증 상태 관리
    ↓
RootNavigator
    │
    │ 인증 상태 기반 화면 Tree 결정
    ↓
Screen
```

그리고 서버 인증의 최종 판단 기준은 항상 Backend이다.

```text
Frontend Auth State
= 현재 앱 실행 상태

SecureStore Token
= 인증 정보를 복원하기 위한 영속 데이터

Backend JWT Validation
= 실제 인증의 최종 보안 경계
```

Frontend Navigation은 보안 장치가 아니라 **UX 제어 장치**로만 취급한다.

---

## 3. Frontend 인증 상태 관리

### 확정안

React Context 기반의 `AuthProvider`를 사용한다.

Zustand 등 새로운 전역 상태관리 라이브러리는 이번 기능만을 위해 추가하지 않는다.

TanStack Query 역시 Auth State 자체의 주 관리자로 사용하지 않는다.

Provider 구조는 다음과 같이 변경한다.

```text
SafeAreaProvider
    ↓
QueryProvider
    ↓
AuthProvider
    ↓
RootNavigator
```

`AuthProvider`가 QueryProvider 내부에 위치하는 이유는 로그아웃과 강제 세션 종료 시 사용자 관련 Query Cache를 정리해야 하기 때문이다.

---

## 4. Auth State

단순한:

```text
isLoggedIn: boolean
```

구조는 사용하지 않는다.

앱 시작 직후에는 인증 여부를 아직 판단할 수 없기 때문이다.

최소 상태는 다음과 같다.

```text
RESTORING
UNAUTHENTICATED
AUTHENTICATED
RESTORE_ERROR
```

개념적인 State 구조는 다음과 같다.

```text
RESTORING
└─ 앱 시작 후 SecureStore / Backend 검증 중


UNAUTHENTICATED
└─ 로그인되지 않은 상태


AUTHENTICATED
├─ user
├─ hasSelectedInterests
└─ nextStep
    ├─ INTEREST_SELECTION
    └─ MAIN


RESTORE_ERROR
└─ Token은 존재하지만
   네트워크 / 서버 문제 때문에
   인증 상태 확인에 실패한 상태
```

### 중요한 원칙

Access Token 문자열 자체를 일반 React State에 다시 복제해 장기간 보관하지 않는다.

영속 Token의 Source of Truth는 계속 SecureStore로 유지한다.

---

## 5. Backend Session Restore API

### 확정

다음 API를 추가한다.

```text
GET /api/auth/session
```

Authorization:

```text
Authorization: Bearer <access_token>
```

목적은:

> 현재 저장된 Access Token이 아직 유효한지 검증하고, 해당 사용자가 앱의 어느 진입 상태에 있어야 하는지를 반환한다.

서버 세션을 새로 생성하는 API가 아니다.

다음 기능도 수행하지 않는다.

```text
Access Token 재발급 X
Refresh Token 발급 X
Token 만료시간 연장 X
서버 Session 생성 X
```

---

## 6. Session API 응답

성공:

```text
200 OK
```

응답 Data:

```text
user
├─ user_id
├─ login_id
├─ name
└─ status

has_selected_interests

next_step
├─ MAIN
└─ INTEREST_SELECTION
```

Access Token은 Session API 응답에서 다시 반환하지 않는다.

기존 Login Response와 공유할 데이터 구조는 재사용할 수 있지만, Token까지 포함된 `LoginData` 자체를 그대로 Session Response로 재사용하지 않는다.

별도의 Session Response Data Schema를 두는 것이 권장된다.

---

## 7. Session API Backend 처리 구조

기존 인증 구조를 최대한 재사용한다.

```text
GET /auth/session
        ↓
CurrentUserDep
        ↓
JWT 검증
        ↓
ACTIVE User 확인
        ↓
AuthService
        ↓
InterestRepository.exists_by_user_id()
        ↓
has_selected_interests 계산
        ↓
next_step 계산
```

별도의 `SessionRepository`나 Server-side Session Table을 만들지 않는다.

현재 인증은 Stateless Access Token 구조를 유지한다.

---

## 8. 앱 실행 시 Session Restore

앱 시작 Flow를 다음과 같이 확정한다.

```text
App Start
    ↓
AuthState = RESTORING
    ↓
SecureStore.getAccessToken()
    │
    ├─ Token 없음
    │      ↓
    │  UNAUTHENTICATED
    │      ↓
    │    Login
    │
    └─ Token 존재
           ↓
       GET /auth/session
           │
           ├─ 200
           │    ↓
           │ AUTHENTICATED
           │    ↓
           │ next_step
           │  ├─ INTEREST_SELECTION
           │  └─ MAIN
           │
           ├─ 401
           │    ↓
           │ Session Cleanup
           │    ↓
           │ Token 삭제
           │    ↓
           │ UNAUTHENTICATED
           │
           └─ Network / 5xx
                ↓
           Token 유지
                ↓
           RESTORE_ERROR
```

---

## 9. Restore 실패 정책

### 401

401은 인증 자체가 유효하지 않다는 의미로 처리한다.

원인이 반드시 Token 만료라고 단정하지 않는다.

가능한 원인은 다음과 같다.

```text
Token 만료
Token 위조 / 손상
잘못된 서명
사용자가 DB에 없음
WITHDRAWN
SUSPENDED
기타 인증 실패
```

따라서 사용자 메시지는:

```text
인증 정보가 유효하지 않습니다.
다시 로그인해 주세요.
```

정도의 일반적인 표현을 사용한다.

처리:

```text
Token 삭제
↓
사용자 Query Cache 제거
↓
Auth State 제거
↓
UNAUTHENTICATED
```

### Network Error / 5xx

인증이 잘못되었다는 증거가 아니므로 Token을 삭제하지 않는다.

```text
Token 유지
↓
RESTORE_ERROR
```

사용자는 최소한 다음 동작을 할 수 있어야 한다.

```text
다시 시도
로그인 화면으로 이동
```

사용자가 직접 Login으로 이동하기로 선택하면 기존 Token을 제거한 뒤 `UNAUTHENTICATED`로 변경한다.

---

## 10. Access Token 만료 정책

Frontend에서 JWT를 직접 decode하여 만료 Timer를 운영하지 않는다.

Backend를 인증 Source of Truth로 사용한다.

앱이 종료된 동안 Token이 만료된 경우:

```text
App Start
→ /auth/session
→ 401
→ 세션 종료
→ Login
```

앱 사용 중 Token이 만료된 경우:

```text
보호 API 요청
→ Backend 401
→ 공통 인증 실패 처리
→ 세션 종료
→ Login
```

현재 Refresh Token이 없으므로 만료된 Access Token은 자동 갱신하지 않는다.

---

## 11. API Client 구조

기존 단일 `apiClient`에서 다음 두 Client로 책임을 분리한다.

```text
publicApiClient
authenticatedApiClient
```

실제 파일명은 구현 단계에서 프로젝트 컨벤션에 맞춰 조정할 수 있다.

### Public Client

다음과 같은 인증 이전 요청에 사용한다.

```text
POST /auth/login

향후:
회원가입
비밀번호 복구
Google OAuth 시작 등
```

특징:

```text
Authorization 자동 주입 X
Global Session 401 Handling X
```

로그인 ID/Password 오류에 따른 401은 로그인 화면이 정상적으로 처리한다.

### Authenticated Client

다음 요청에 사용한다.

```text
GET /auth/session
POST /users/me/interests
향후 사용자/북마크/맞춤 트렌드 보호 API 등
```

특징:

```text
Request Interceptor
└─ Access Token 자동 주입

Response Interceptor
└─ 인증된 요청의 401 공통 처리
```

---

## 12. apiClient와 React 계층의 의존성 규칙

다음 구조는 금지한다.

```text
apiClient
→ Navigation 직접 호출

apiClient
→ LoginScreen 직접 import

apiClient
→ React Hook 직접 호출
```

API 계층은 UI 계층을 알아서는 안 된다.

필요하면 작은 인증 이벤트/콜백 연결 계층을 둔다.

개념:

```text
Authenticated API Client
        ↓
401 감지
        ↓
Auth Failure Event / Callback
        ↓
AuthProvider
        ↓
Session Cleanup
        ↓
AuthState 변경
        ↓
RootNavigator가 자동 변경
```

구현 방식의 이름은 확정하지 않지만 **의존성 방향은 반드시 유지한다.**

---

## 13. 공통 401 처리 규칙

단순히:

```text
status === 401
→ logout
```

으로 구현하지 않는다.

최종 조건은 다음 개념을 만족해야 한다.

```text
① Authenticated Client 요청인가?

② 해당 요청에 Authorization Bearer Token이 실제로 있었는가?

③ 응답이 401인가?

④ 해당 Request에서 사용한 Token이
   현재 SecureStore Token과 동일한가?
```

모두 만족할 경우에만 현재 인증 세션을 무효화한다.

---

## 14. 오래된 401이 새 세션을 종료하는 Race Condition 방지

다음 상황을 반드시 방어한다.

```text
TOKEN_A로 Request 전송
        ↓
사용자 로그아웃
        ↓
새 로그인
        ↓
TOKEN_B 발급
        ↓
예전 TOKEN_A Request가 늦게 401 반환
```

잘못된 구현:

```text
401
→ 무조건 Logout
→ TOKEN_B 세션까지 종료
```

금지한다.

올바른 처리:

```text
401 Request에서 사용된 Token
             ↓
현재 SecureStore Token과 비교
             │
             ├─ 동일
             │    ↓
             │ 현재 세션 인증 실패
             │    ↓
             │ Session Cleanup
             │
             └─ 다름
                  ↓
              오래된 응답
                  ↓
                 무시
```

이 규칙은 **필수 보안 요구사항**으로 취급한다.

---

## 15. 중복 401 처리

Main 화면 등에서는 여러 요청이 동시에 발생할 수 있다.

```text
recommended trends → 401
bookmarks          → 401
profile            → 401
```

이때 세 번의 Logout/Alert/Cache Clear를 수행해서는 안 된다.

Session Cleanup은:

```text
idempotent
+
single-flight
```

성격을 갖도록 설계한다.

쉽게 표현하면:

> 같은 세션 종료 요청이 여러 번 들어와도 실제 정리 작업은 한 번만 수행한다.

수동 Logout과 401 강제 로그아웃도 동일한 Session Cleanup primitive를 사용한다.

---

## 16. 로그인 성공 처리

현재는:

```text
useLogin
→ SecureStore 저장
→ LoginScreen
→ next_step 검사
→ navigation.replace(...)
```

형태이다.

이를 다음 책임 구조로 변경한다.

```text
LoginScreen
└─ 입력 / UI / 오류 표시


useLogin
└─ Login API 호출 상태


AuthProvider
├─ 로그인 성공 결과 수신
├─ Access Token 저장
└─ Authenticated Session 확립


RootNavigator
└─ next_step 기반 화면 결정
```

LoginScreen은 로그인 성공 후 어느 화면으로 갈지 직접 결정하지 않는다.

---

## 17. 관심사 선택과 Auth Session

`has_selected_interests`와 `next_step`은 앱의 인증 세션 상태 일부로 취급한다.

로그인 직후 관심사가 없다면:

```text
AUTHENTICATED
next_step = INTEREST_SELECTION
        ↓
InterestSelect
```

관심사 저장 성공 시:

```text
POST /users/me/interests
        ↓
201
        ↓
AuthProvider에 관심사 완료 통지
        ↓
has_selected_interests = true
next_step = MAIN
        ↓
RootNavigator 재평가
        ↓
Main
```

InterestSelectScreen이 직접 Main으로 Navigation하지 않는다.

---

## 18. 관심사 409 상태 불일치 처리

다음 상황이 발생할 수 있다.

Frontend:

```text
next_step = INTEREST_SELECTION
```

Backend:

```text
이미 관심사가 존재함
```

POST 결과:

```text
409 Conflict
```

이 경우 UI를 단순히 잠그고 끝내지 않는다.

권장 흐름:

```text
409
 ↓
/auth/session 재검증
 ↓
서버 상태 재동기화
 ↓
next_step = MAIN 확인
 ↓
RootNavigator
 ↓
Main
```

서버 상태를 최종 기준으로 사용한다.

---

## 19. Root Navigation 구조

현재처럼 Login과 InterestSelect를 동일한 Stack에서 자유롭게 이동시키는 구조를 종료한다.

최종 구조:

```text
NavigationContainer
        ↓
RootNavigator
        │
        ├─ RESTORING
        │      ↓
        │ Session Restore / Splash
        │
        ├─ RESTORE_ERROR
        │      ↓
        │ Session Restore Error
        │
        ├─ UNAUTHENTICATED
        │      ↓
        │ AuthNavigator
        │      ↓
        │ Login
        │
        └─ AUTHENTICATED
               │
               ├─ INTEREST_SELECTION
               │      ↓
               │ Onboarding / Interest Navigator
               │      ↓
               │ InterestSelect
               │
               └─ MAIN
                      ↓
                   AppNavigator
                      ↓
                     Main
```

Main 기능의 실제 Trend 화면 구현은 이번 범위에 포함하지 않는다.

필요하면 임시 Main Placeholder를 둘 수 있다.

---

## 20. Navigation Reset 정책

로그아웃에서 다음과 같은 명령형 처리를 중심으로 사용하지 않는다.

```text
navigation.navigate("Login")
navigation.replace("Login")
navigation.reset(...)
```

Root Navigation Tree 자체를 Auth State에 따라 교체한다.

예:

```text
AUTHENTICATED
        ↓
Session Cleanup
        ↓
UNAUTHENTICATED
        ↓
AppNavigator Unmount
        ↓
AuthNavigator Mount
```

그 결과 기존:

```text
Main
Detail
Bookmark
...
```

History가 모두 제거된다.

Android Back 버튼으로 인증 이전 화면으로 돌아갈 수 없어야 한다.

---

## 21. Logout 정책

### Backend Logout API

현재는 구현하지 않는다.

현재 구조:

```text
Access Token
Stateless JWT

Refresh Token 없음
Revocation 없음
Blocklist 없음
Server Session 없음
```

이 상태에서 `/auth/logout`이 200을 반환해도 이미 발급된 JWT를 서버가 실제로 폐기할 수 없다.

따라서 현재 Logout은 **Client-side Session Termination**으로 정의한다.

API 명세의 기존 `/api/auth/logout` 계획 항목이 있다면:

```text
MVP 구현 보류
후속 Token Revocation 설계에서 재검토
```

로 변경한다.

---

## 22. Logout 처리 순서

개념적인 Session Cleanup:

```text
Session Termination Lock 획득
        ↓
새 인증 요청 발생 억제
        ↓
진행 중 사용자 Query 취소
        ↓
SecureStore Access Token 삭제
        ↓
사용자 Query Cache 제거
        ↓
Auth Session 정보 제거
        ↓
AuthState = UNAUTHENTICATED
        ↓
Authenticated Navigation Tree 제거
```

수동 로그아웃과 401에 의한 강제 로그아웃이 가능한 한 같은 Cleanup 경로를 사용한다.

---

## 23. SecureStore 삭제 실패

Access Token 삭제에 실패했는데 사용자에게 로그아웃 성공으로 표시하지 않는다.

잘못된 흐름:

```text
SecureStore 삭제 실패
↓
무시
↓
로그아웃 성공 표시
↓
앱 재실행
↓
기존 Token 발견
↓
자동 로그인
```

권장 정책:

```text
삭제 실패
↓
로그아웃 완료로 확정하지 않음
↓
사용자에게 재시도 안내
```

---

## 24. Query Cache 보안

로그아웃에서는 Token만 제거해서는 안 된다.

예:

```text
사용자 A 로그인
↓
A의 Profile / Bookmark / Trend Cache
↓
로그아웃
↓
사용자 B 로그인
```

A의 데이터가 B에게 잠깐이라도 보이지 않도록 사용자 관련 Query Cache를 정리한다.

세션 종료 시 최소한:

```text
진행 중 사용자 Query 취소
+
사용자 Cache 제거
```

를 수행한다.

향후 Query Key에 user identifier를 포함시키는 것도 추가적인 방어가 될 수 있으나 이번 기능의 필수 조건으로 확대하지 않는다.

---

## 25. Token Logging 금지

다음 값은 로그에 출력하지 않는다.

```text
Access Token
Authorization Header 전체
SecureStore 저장 Token
JWT 원문
Password
Axios Error 전체 객체의 무분별한 dump
```

개발 로그가 필요하면 다음과 같은 비민감 정보만 기록한다.

```text
HTTP Method
Endpoint
HTTP Status
오류 Category
Request ID
```

---

## 26. HTTPS 정책

로컬 Expo 개발 환경에서는 다음 형태가 허용될 수 있다.

```text
http://192.168.x.x:8000
```

그러나 Production에서는 반드시 HTTPS를 사용한다.

```text
Development
→ HTTP 허용 가능

Production
→ HTTPS 필수
```

Bearer Access Token을 Production HTTP 연결을 통해 전송하지 않는다.

---

## 27. JWT 보안 보강 정책

현재 반드시 유지해야 하는 조건:

```text
서명 검증
exp 필수
sub 필수
허용 Algorithm 서버 고정
JWT Secret 환경변수 관리
충분히 강한 JWT Secret
```

현재 설정에서 JWT Secret은 최소 길이 검증을 받고 있으나 실제 운영 Secret은 단순한 문장보다는 충분한 난수값을 사용한다.

다음 Claim은 후속 JWT Hardening 후보로 둔다.

```text
iss
aud
```

현재 단일 Backend / 단일 Client MVP에서는 Session Management 구현을 막는 필수조건으로 만들지 않는다.

---

## 28. `jti` 정책

이번에 `jti`를 단독으로 추가하지 않는다.

```text
jti 추가
```

만으로는 Logout이나 Token Revocation이 구현되지 않는다.

향후:

```text
Token Blocklist
Token Revocation
Device Session
```

을 도입할 때:

```text
jti
+
Server-side Revocation State
```

를 함께 설계한다.

---

## 29. Session API 정보 최소화

`GET /auth/session`은 앱 부트스트랩에 필요한 최소 정보만 반환한다.

반환:

```text
user 기본정보
has_selected_interests
next_step
```

반환하지 않음:

```text
password_hash
JWT Secret
새 Access Token
불필요한 개인정보
```

인증 관련 응답에 대한 `Cache-Control: no-store` 적용은 구현 단계에서 보안 Hardening으로 검토한다.

---

## 30. Brute Force / Credential Stuffing

현재 로그인 기능과 관련하여 별도의 Rate Limiting은 현재 핵심 인증 구조에 포함되어 있지 않다.

이는 Session Management 작업과 직접 같은 책임은 아니므로 이번 작업에 섞지 않는다.

후속:

```text
인증 보안 Hardening
```

작업으로 분리한다.

후속 검토 항목:

```text
로그인 Rate Limiting
IP 기반 제한
계정 기반 제한
Credential Stuffing 방어
비정상 로그인 관측
```

현재 ID 없음/비밀번호 오류를 동일한 401 메시지로 반환하는 정책은 유지한다.

---

## 31. 현재 구조에서 의도적으로 남기는 보안 한계

현재 Stateless JWT에서는 사용자 로그아웃 후에도 **이미 탈취된 Access Token을 서버가 즉시 무효화할 수 없다.**

예:

```text
공격자가 TOKEN_A 탈취
        ↓
정상 사용자가 Logout
        ↓
정상 기기의 TOKEN_A 삭제
        ↓
공격자의 TOKEN_A는
exp까지 사용 가능
```

현재 방어:

```text
Access Token TTL
SecureStore
HTTPS
ACTIVE User DB 재검증
Token Logging 금지
```

완전한 해결:

```text
Refresh Token
Revocation / Blocklist
Token Rotation
Device Session
```

이지만 모두 이번 범위 밖이다.

---

## 32. Access Token TTL

현재 기본 Access Token 만료시간은 60분이다.

Refresh Token이 없는 현재 MVP에서 TTL을 지나치게 짧게 설정하면 사용자가 빈번하게 재로그인해야 한다.

따라서 이번 작업에서는 **현재 60분 정책을 유지**한다.

추후 Refresh Token 도입 시 Access Token TTL 단축을 함께 재검토한다.

---

## 33. Backend 테스트 범위

추가되는 `/auth/session`에서 최소한 다음을 검증한다.

```text
유효 Token + 관심사 없음
→ 200
→ INTEREST_SELECTION

유효 Token + 관심사 있음
→ 200
→ MAIN

Token 없음
→ 401

잘못된 Token
→ 401

만료 Token
→ 401

사용자 없음
→ 401

WITHDRAWN
→ 401

SUSPENDED
→ 401
```

기존 JWT / CurrentUser Dependency 테스트는 유지하고 중복 테스트를 불필요하게 늘리지 않는다.

---

## 34. Frontend 테스트 범위

최소 다음 시나리오를 검증한다.

```text
SecureStore Token 없음
→ UNAUTHENTICATED

Token + Session 200 + INTEREST_SELECTION
→ InterestSelect

Token + Session 200 + MAIN
→ Main

Session 401
→ Token 제거
→ Cache 제거
→ UNAUTHENTICATED

Session Network Error
→ Token 유지
→ RESTORE_ERROR

보호 API 401
→ Session 종료

Login API 401
→ Global Session 종료 발생하지 않음

동시 401
→ Cleanup 한 번

옛 Token Request 401
→ 새 세션 종료하지 않음

Logout
→ Token 제거
→ Cache 제거
→ UNAUTHENTICATED

Interest Save 성공
→ next_step MAIN

Interest 409 상태 불일치
→ Session 재검증
```

Frontend 테스트 환경은 구현 시작 시 현재 package 상태를 다시 확인한 뒤 결정한다.

---

## 35. 실제 Expo 기기 Acceptance Test

다음은 자동 테스트 외에 반드시 실제로 확인한다.

```text
01. Token 없이 앱 실행
    → Login

02. 로그인
    → SecureStore 저장

03. 관심사 없는 계정 로그인
    → InterestSelect

04. 앱 완전 종료 후 실행
    → Login이 아니라 InterestSelect

05. 관심사 저장
    → Main

06. 앱 완전 종료 후 실행
    → Main

07. 만료 Token 저장 후 실행
    → Login

08. 앱 사용 중 Token 만료 / 잘못된 Token
    → 다음 보호 API 401
    → Login

09. Logout
    → Login

10. Android Back
    → 이전 인증 화면으로 돌아가지 않음

11. 서버를 끈 상태에서 앱 실행
    → Token 삭제되지 않음
    → Restore Error

12. 서버 재시작 후 Retry
    → 기존 Session 정상 복원

13. 로그인 실패 401
    → 입력 오류 표시
    → Global Logout 동작 없음

14. 여러 API가 동시에 401
    → Logout 처리 한 번

15. 오래된 Token 요청의 401이 늦게 도착
    → 새 로그인 세션 유지
```

---

## 36. 구현 순서 최종 확정

### Step 1 — Backend Session Contract

```text
Session Response Schema
↓
AuthService Session 조회
↓
GET /api/auth/session
↓
Backend pytest
```

### Step 2 — Frontend API Client 구조

```text
공통 Axios 설정 정리
↓
Public Client
↓
Authenticated Client
↓
Session API Function
↓
Session Types
```

### Step 3 — AuthProvider

```text
AuthState
↓
restoreSession
↓
로그인 성공 Session 확립
↓
Session Cleanup
↓
logout
↓
interest completion
↓
session refresh/revalidation
```

### Step 4 — 공통 401 처리

반드시 다음을 함께 구현한다.

```text
Authenticated Client에만 적용
Authorization 존재 확인
Request Token 확인
현재 Token과 비교
동시 Cleanup 방지
오래된 401 무시
```

### Step 5 — Root Navigation

```text
RESTORING
RESTORE_ERROR
UNAUTHENTICATED
AUTHENTICATED + INTEREST_SELECTION
AUTHENTICATED + MAIN
```

분기를 구현한다.

### Step 6 — 기존 Login 리팩터링

제거:

```text
LoginScreen의 next_step Navigation 판단
LoginScreen의 navigation.replace()
```

AuthProvider 중심으로 이동한다.

### Step 7 — InterestSelect 연결

저장 성공:

```text
Auth Session State 갱신
↓
MAIN
```

409 발생:

```text
Session Revalidation
```

401 화면별 처리는 공통 인증 처리로 이동한다.

### Step 8 — Logout

```text
Session Termination Lock
↓
진행 중 Query 정리
↓
SecureStore Token 삭제
↓
Cache 제거
↓
Auth State 제거
↓
Root Navigation 전환
```

### Step 9 — Test / Device Validation

```text
Backend pytest
↓
Frontend 인증 상태 테스트
↓
401 Race Condition
↓
Expo 실제 기기 Acceptance Test
```

순서로 검증한다.

---

## 37. 이번 범위에서 구현하지 않는 항목

다음 사항은 필요성이 발견되더라도 이번 작업에 임의로 포함하지 않는다.

```text
Refresh Token

Access Token 자동 갱신

Token Rotation

Token Revocation

JWT Blocklist / Denylist

Server-side Session

Device Session 관리

Google OAuth

회원가입

비밀번호 찾기

로그인 Rate Limiting

Credential Stuffing 고급 방어

Main 화면 실제 Trend 기능

회원정보 수정

회원 탈퇴 기능 구현
```

각 항목은 별도의 후속 설계/구현 채팅으로 분리한다.

---

## 38. 이번 설계에서 반드시 지켜야 할 핵심 결정

| 항목 | 최종 결정 |
|---|---|
| Access Token 저장 | Expo SecureStore |
| 저장 Token | Access Token만 |
| Auth State | React Context `AuthProvider` |
| Token React State 복제 | 하지 않음 |
| 앱 시작 | `RESTORING` |
| Session 검증 | Backend가 Source of Truth |
| Session API | `GET /api/auth/session` |
| Session Token 재발급 | 하지 않음 |
| JWT Client Decode | 세션 판단 목적으로 사용하지 않음 |
| 만료 Timer | 사용하지 않음 |
| 로그인 Client | Public Client |
| 보호 API Client | Authenticated Client |
| 공통 401 | Authenticated Client만 |
| Login 401 | 화면의 정상 인증 실패 |
| 401 현재 세션 확인 | **Request Token == Current Token 검증 필수** |
| 오래된 401 | 무시 |
| 동시 401 | Cleanup 1회 |
| API Client의 Navigation 호출 | 금지 |
| Root Navigation | Auth State 기반 선언적 분기 |
| LoginScreen 직접 화면 결정 | 제거 |
| Interest 완료 화면 전환 | Auth State 변경 |
| Restore 401 | Token 삭제 + Login |
| Restore Network/5xx | Token 유지 |
| Logout Backend API | 현재 구현하지 않음 |
| Logout Cache | 제거 |
| Token Logging | 금지 |
| Production HTTP | 금지 |
| Access Token TTL | 현재 60분 유지 |
| `jti` | Revocation 도입 전 보류 |
| Refresh / Revocation | 후속 |

---

## 39. 설계 완료 기준

이 문서에서 아래 사항은 **설계가 확정된 것으로 간주한다.**

```text
SecureStore 기반 Session Restore
Backend /auth/session 추가
AuthProvider
Public / Authenticated Client 분리
공통 401 정책
Stale 401 Race Condition 방어
동시 401 Cleanup 방어
Client-side Logout
Query Cache Cleanup
Auth State 기반 Root Navigation
Interest Selection ↔ MAIN Session 연동
보안 Logging 정책
Production HTTPS 정책
```

구현 중 특별한 기술적 문제가 발견되지 않는 한 이 구조를 임의로 다시 바꾸지 않는다.

구조 변경이 필요하면:

```text
문제 발견
↓
왜 기존 설계로 해결할 수 없는지 확인
↓
대안 비교
↓
설계 변경
↓
구현
```

순서를 따른다.

---

## 40. 새 구현 채팅 시작 프롬프트

```text
같은 Trend Leader 프로젝트의 `채팅 기능별 구분 방법`에서 정리한
채팅 분리 기준을 적용해주세요.

이번 채팅은 `인증 세션 관리 및 로그아웃 구현` 채팅입니다.

첨부한 `Trend Leader 인증 세션 관리 및 로그아웃 설계 확정안 v1.0`을
구현 기준으로 삼아주세요.

구현을 시작하기 전에 `Trend-Leader-capstoneProject/feature`
Repository의 dev 브랜치 최신 HEAD를 다시 조회하고,
설계 당시 기준 커밋
`a9829fd74f77e8fb6e29e1fd6bfc066ae6353aa8`
이후 인증 관련 Backend/Frontend 변경사항이 있는지 먼저 검증해주세요.

변경사항이 있다면 기존 확정안과 충돌하는 부분을 먼저 알려주시고,
충돌이 없다면 확정된 구현 순서에 따라 단계별로 진행해주세요.

이번 구현의 범위는 Session Restore, AuthProvider,
Public/Authenticated API Client 분리, 공통 401 처리,
Stale 401 방어, Root Navigation 분기,
Client-side Logout, Query Cache Cleanup,
Interest Selection과 Session 상태 연계 및 관련 테스트까지입니다.

Refresh Token, Token Rotation, Revocation/Blocklist,
Google OAuth, 회원가입, Rate Limiting,
Main 화면의 실제 Trend 기능은 구현하지 말고 후속 작업으로 유지해주세요.

한 단계씩 진행하고, 각 단계마다 기존 코드와의 책임 분리,
보안상 문제, 테스트 결과를 검증한 뒤 다음 단계로 넘어가 주세요.
```
