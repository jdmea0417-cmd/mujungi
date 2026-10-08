① 개정: A1.5 코드 등록·기기 자격·데모 브라우저 저장 예외와 원음 저장/삭제·세 통로의 인가 경계를 반영했다.
② 대체: 고정 ID 사전 시드·원음 미저장과 화면 충돌 미채택 설명을 이전 선택으로 보존했다. Google·Session·CSRF·1:1:1·내부 인증은 유지한다.
③ 남은 TBD: 코드/자격 형식·수명·등록 연결 대응·교체 경합, 운영 인증, 원음 저장 소유/완료/삭제와 푸시·전사 상세 및 기존 실연동/철회 경합.

# ADR-008 인증·접근 제어

- 상태: **Proposed**. Google만·코드 등록 데모·두 앱/서버 릴레이·1:1:1 방향은 최신 기준을 따른다. Spring Security OAuth2 Client + Spring Session JDBC와 자체 ID/Password 제외는 기존 Accepted 결정을 유지한다. 실제 구현·통합 검증은 남아 있다.
- 개정 기준: `ref/mujung-architecture-update-audit-v1.1.md`와 `ref/audit-addendum-ai-roles.md` A1.5 §9·§11·§12를 적용한다. 사용자 확정 > v0.2 수정 > 기존 비충돌 내용 보존 순서와 업무 소유권을 유지한다.
- 확정 조건: Google/Session/CSRF·현재 인가, 코드 등록/자격/교체·대기 WSS/할당/재생, 원음 저장/삭제 및 알림/전사 인가의 실제 시험. 데모 연결은 운영 인증 완료가 아니다.
- 관련: [ADR-002](ADR-002-technology-defaults.md), [ADR-003](ADR-003-child-device-ui.md), [ADR-004](ADR-004-communication-voice.md), [ADR-005](ADR-005-session-events-timeouts.md), [ADR-006](ADR-006-demo-deployment.md), [ADR-007](ADR-007-module-boundaries.md), [architecture/auth.md](architecture/auth.md).

---

# 1. Context

보호자와 화면 없는 아동용 음성 기기를 대신하는 `/device` 클라이언트는 역할과 신뢰 범위가 다르다. 두 Spring 앱 사이에도 별도 서비스 신뢰 경계가 생긴다.

| 대상 | 필요한 접근 | 책임 앱 |
| --- | --- | --- |
| 보호자 | Google 가입·로그인, 자기 아동/활동, 허용 결과·리포트, 동의·철회 | Core API Spring |
| `/device` | 코드 등록 후 기기 자격으로 대기 WSS, 입력 음성·승인 음성 재생, 제어 수신·상태/실제 재생 보고 | Realtime Activity Spring |
| 내부 호출자 | 허용 명령·조회·영속 이벤트 전달 | 수신하는 Core 또는 Realtime |

`/device`는 화면 없는 기기의 브라우저 대체 클라이언트다. 코드 등록으로 계정의 아이와 기기를 1:1로 데모 연결한다. 등록 Device 식별자는 기기 자격이 아니며 관계/코드 등록이 운영 페어링·물리 소유권 인증을 증명하지 않는다.

로그인·서비스 인증과 별도로 리소스 관계, 현재 동의, 허용 Origin, 현재 연결·기기 점유·활동 할당과 최신 작업 검사가 필요하다. 활동 상태·참여·아동 발화의 의미 최종 판정은 Realtime 서버 책임이다.

---

# 2. Decision

## 2.1 보호자·기기·내부 서비스 Context 분리

보호자 인증 정보를 기기 권한으로 재사용하지 않고, 기기 ID/연결로 보호자 API에 접근하지 못하게 한다. 내부 서비스 인증도 사용자 인가를 대신하지 않는다. 같은 Origin에서 보호자 Cookie가 기기 요청에 실려도 기기 권한으로 해석하지 않는다.

```text
Guardian Authentication (Core)
≠ Device credential / allow-list / connection / assignment (Realtime demo)
≠ Internal Service Authentication (receiving app)
```

현행 데모는 코드 등록과 기기 자격 검증을 추가하며 허용 Origin·현재 연결·할당 검사도 유지한다. 운영용 기기 인증/자격 보관 방식은 별도 TBD다.

---

## 2.2 MVP 보호자·아이·기기 관계

MVP는 **보호자 계정 1 : 아이 1 : 기기 1**이다. 기존 `GuardianChild` 관계를 유지하고 보호자–아이를 1:1로 제한한다. 기기는 그 계정의 아이와 1:1로 연결하며 A1.5의 코드 등록과 기기 자격을 사용한다. UNIQUE 등 제약 구현·관계 전달의 실제 필드와 API는 TBD다.

**아이 추가·삭제·선택 기능과 공동 보호자 기능은 두지 않는다.** 아이를 바꾸려면 계정 삭제 뒤 새로 가입한다. 가입 때 한 아이의 정보를 받는 일은 별도 아이 추가 기능이 아니며, 서비스 계정 삭제도 아이 단독 삭제 기능과 구분한다. 가입·계정 삭제의 구체 절차와 기존 자료의 보존·열람·삭제·철회 경합은 D-08/G-03의 확인 범위를 유지한다.

보호자 화면에는 아이나 기기를 고르는 단계가 없다. 상태 카드는 **계정의 아이에 연결된 기기**를 표시하고 시작 대상은 **계정의 아이 → 그 아이의 기기**로 유도한다. 기기가 연결되지 않았으면 **“연결된 기기 없음”**으로 표시하고 시작을 차단한다. **계정당 진행 세션은 1개**다. 관계 확인 뒤에도 Realtime의 허용·현재 연결·점유·원자 할당 검사를 생략하지 않는다.

기록·리포트·목표는 아이 기준이며 `activity_session`은 아이와 기기를 모두 기록한다. 여러 아이·공동 보호자·형제 기기 공유는 1:1 제약을 푸는 후속 확장 과제다. 데모 연결을 운영 페어링이나 물리 소유권 인증으로 해석하지 않는다.

---

# 3. 보호자 인증

## 3.1 기본 원칙

Core의 Spring Security가 보호자 인증을 관리한다. 보호자 브라우저의 서비스 로그인은 Session ID Cookie로 유지한다. `HttpOnly`를 기본으로 하고 HTTPS에서 `Secure`를 적용한다. `SameSite=Lax`는 기존 기본 후보이며 Cookie 이름·Path·Domain·TTL·동시 로그인과 정확한 설정은 TBD다. Cookie 속성만으로 CSRF 보호가 완료되지 않는다.

<a id="guardian-social-login"></a>

## 3.2 Google 소셜 로그인·서비스 Session — 방향 확정

**최신 사용자 확정: Google만 가입·로그인 수단으로 제공한다.** **Spring Security OAuth2 Client + Spring Session JDBC**는 2026-10-01의 기존 Accepted 선택을 유지한다. 자체 아이디·비밀번호 회원가입·로그인은 제공하지 않는다. Google로 외부 신원을 확인해도 내부 Guardian 계정과 가입 완료·접근 권한 확인은 필요하다.

| 구분 | 현행 선택 | 책임 |
| --- | --- | --- |
| 사용자 확인 | Google OAuth 2.0 기반 로그인, OIDC 사용 시 해당 검증 | 검증한 외부 계정과 내부 Guardian 연결 |
| Spring 통합 | Core의 Spring Security OAuth2 Client | 인가 요청·Callback·코드 교환·사용자 정보/ID Token 검증 |
| 서비스 로그인 유지 | Core의 Spring Session JDBC + Session ID Cookie | PostgreSQL에 인증 Context 저장, 보호자 API 인증 |
| 자체 비밀번호 | 제공하지 않음 | Password Hash·비밀번호 재설정 기능 추가 안 함 |

Google Access/Refresh Token·ID Token을 우리 보호자 API의 Credential로 재사용하지 않는다. OAuth/OIDC 로그인과 서비스 Session 인증은 별개이며 ID Token이 JWT라는 이유로 서비스 JWT 인증을 채택하지 않는다.

```text
Google 로그인 선택
→ Core가 인가 요청·state 저장 후 제공자 Redirect
→ Callback state 대조·인가 코드 서버 교환
→ 사용자 정보 / 사용한 OIDC ID Token·nonce 검증
→ 기존 Guardian 조회 또는 최초 가입 절차
→ 서비스 가입 완료·허용 권한 확인
→ 인증 Session 갱신·Session ID Cookie
→ Core 소유 PostgreSQL Spring Session 저장
```

`provider + 제공자 고유 회원번호`는 기존 계정 연결 구현 기준으로 보존한다. 이메일이 같다는 이유로 다른 계정을 자동 병합하지 않는다. Guardian 필드·최초 가입 완료 조건·추가 로그인 계정 연결/해제·서비스 계정 삭제/재가입의 구체 절차는 TBD다. 아이 변경은 계정 삭제 뒤 새 가입이라는 방향을 유지하며, 로그인 계정 연결 검토를 공동 보호자나 복수 아이 기능으로 확대하지 않는다. 이를 이유로 Google 외 로그인을 현행 범위에 추가하지 않는다. 제공자 개인정보 동의와 우리 서비스의 필수/선택 동의는 구분하고 활동 시 현재 DB Consent를 검사한다.

Spring 기본 경로 `GET /oauth2/authorization/{registrationId}`, `GET /login/oauth2/code/{registrationId}`와 `registrationId=google`은 **기존 구현 후보**다. 앱 등록·Scope·OAuth/OIDC 구성·속성 매핑·Client 인증·Redirect URI·성공/실패 응답·가입 미완료 Context는 실연동 검증 후 정한다. 기본 경로는 자체 ID/Password를 받던 `POST /api/auth/login` 후보를 대체한다. 실제 Endpoint 계약은 [protocol.md](architecture/protocol.md)에 연결한다.

Spring Session 테이블은 **이미 채택한 Flyway**로 관리하고 Core 소유 스키마·마이그레이션 책임에 맞춘다. 스키마명·권한·Migration 실행 단계는 TBD다. 재배포 후 로그인 유지에는 DB·Cookie와 OAuth Principal/SecurityContext 직렬화 호환성이 필요하다. Authorized Client의 실제 Token 저장 위치와 SecurityContext의 제공자 속성/ID Token 포함 여부를 시험한다. 내부 Guardian ID·필요 권한 중심의 최소 Principal과 불필요한 Token 정리를 검토한다. 기본 설정만으로 제공자 Token 미저장을 보장하지 않는다.

로그아웃은 **우리 서버 Session 무효화 + Cookie 폐기**다. Google 자체 로그아웃·계정 연결 해제·서비스 탈퇴와 다르다. 서비스 Session 삭제만으로 동의 철회 정책이 적용되지 않는다. 보호자 로그아웃·인증 만료 때 진행 활동 처리와 재로그인 후 확인은 D-03/ADR-005 연계 TBD이며 재로그인만으로 종료된 활동을 재개하지 않는다.

## 3.3 선택 이유와 미채택 대안

1. Core의 기존 Spring Security·PostgreSQL과 인증을 통합하고 Realtime의 기기/활동 수명과 분리한다.
2. Spring Security OAuth2 Client가 Google 인가 흐름을 처리하며 Guardian·Consent·접근 권한은 해당 도메인이 관리한다.
3. 자체 비밀번호 저장·재설정을 운영하지 않는다.
4. Spring Session JDBC로 서비스 로그아웃·강제 무효화·만료를 서버에서 관리한다.
5. 별도 인증 서버/관리형 인증 서비스는 이번 MVP에 추가하지 않는다.

**JWT + HttpOnly Cookie — 미채택 대안 이력:** ADR-002 v10에서 제안했으나 보호자 API는 Spring Session JDBC를 선택했다. 새 요구가 생기면 별도 결정으로 재검토하며 현재 Session과 JWT를 병행 발급하지 않는다.

**Auth0·Keycloak 등 — 미채택 대안:** 외부 회원 관리·중앙 SSO 장점이 있지만 Guardian/Consent 연결 책임은 남는다. 외부 서비스 계약·플랜 또는 별도 인증 서버/DB 운영을 추가하지 않는다.

**자체 ID/Password + 소셜 병행 — 미채택 대안:** 기존 일반 로그인 예시는 자체 비밀번호 가입 확정 근거가 아니었다. 자체 ID/Password 제외 정책을 유지한다.

Google 앱 등록·Redirect URI·Scope·공개 조건과 시험 계정을 검증한다. 선택 확정은 무제한 무료·심사 통과·실제 로그인 성공을 보장하지 않는다.

---

# 4. 보호자 리소스 접근 제어

인증된 보호자도 모든 리소스에 접근하지 못한다. 계정의 아이 정보 조회와 허용된 정보 수정, 활동 조회/종료, 결과 재시도, 리포트 조회에는 GuardianChild와 해당 리소스의 소유·관계 검사가 필요하다. 아이 단독 추가·삭제·선택과 공동 보호자 기능은 현행 범위에서 제외하고 서비스 계정 삭제는 별도 작업으로 검사한다. 경로·DTO·오류는 [protocol.md](architecture/protocol.md)의 후보 계약을 따른다.

```text
Authenticated Guardian
→ GuardianChild 1:1 관계로 계정의 아이 확인
→ 해당 Child / ActivitySession / Result / Report 접근 확인
→ 작업별 현재 권한·동의 검사
→ 허용 또는 거부
```

유효한 리소스 ID를 안다는 사실만으로 접근을 허용하지 않는다. ActivitySession은 아이와 기기를 모두 기록해야 한다. 원본의 `guardian_id`, `child_id` 등 관계 저장 후보와 실제 필드명·제약은 TBD이며 새 필드를 이 ADR에서 확정하지 않는다. 동등한 서버 조회 계약으로 현재 보호자의 접근과 계정의 아이–기기 관계를 확인할 수 있어야 한다.

Core는 공개 보호자 요청을 검증하고 Realtime 활동 계약에 위임한다. ActivitySession을 직접 UPDATE하지 않는다. Realtime은 검증된 요청 주체·대상·행위와 계정의 아이에 연결된 기기, 기존 활동/기기 결합을 다시 대조한다. 시작 시 계정당 진행 세션 1개와 해당 기기의 원자 할당 조건을 함께 확인하며 구체 동시성 계약은 TBD다. 다른 보호자의 활동 조회·종료·결과 확인·재시도를 거부한다.

---

# 5. 동의와 권한

인증과 동의는 서로 다르다. 오래된 Session/Token의 동의 주장으로 활동을 허용하지 않고 작업 실행 시 **현재 DB의 권위 있는 Consent**를 확인한다. Core가 Guardian/Child/Consent 변경을 소유하고 Realtime·Core Worker가 필요한 최소 현재 권한을 계약으로 확인한다.

철회 후 신규·지연 자료의 새 저장을 허용하지 않는 원칙을 유지한다. 새 처리·외부 요청·출력 승인·저장·결과 반영 전 이용 가능성을 확인하고 확인 불가이면 새 이용을 보류·차단한다. 이전 자료 보존, 새 분석, 열람·삭제는 별도 정책이다. 인증·소유권이 유효한 종료 요청·현재 동의 확인·최소 종료 상태 조회를 필수 동의 유지 조건으로 막지 않는 기존 구현안을 유지한다. 안전 정리는 동의 재확인 실패 때문에 지연시키지 않는다.

Core는 동의 변경과 철회 이벤트를 로컬 트랜잭션으로 기록해 우선 전달한다. Outbox 알림만으로 즉시 반영이 보장되지는 않는다. Realtime 이벤트 경로는 새 승인·진행을 막고 서버 버퍼·기기 대기열/입출력·공급자 자원을 정리한다. `consent`나 인증 모듈은 ActivitySession을 직접 변경하지 않는다. 철회 효력 시각·확인과 저장/외부 송신 사이 경합·권한 fence 또는 동등 원자성·진행 중 AI 취소의 실제 효과는 **G-03/D-08 연계 TBD**다. 최종 활동 상태·종료 사유도 임의로 새 매핑하지 않는다.

---

# 6. CSRF

보호자 Cookie로 상태를 바꾸는 REST(`POST`, `PUT`, `PATCH`, `DELETE`)와 로그아웃에 CSRF 보호를 유지한다. 활동 시작/종료·허용된 아이 정보 수정·동의 변경·서비스 계정 삭제·결과 재시도·별도 가입 완료 POST를 포함한다. 이 보호 범위를 아이 단독 추가/삭제 기능의 채택으로 해석하지 않는다. 개발 편의를 위해 전체 CSRF를 끄지 않는다.

OAuth 시작·GET Callback에는 일반 REST CSRF Header를 요구하지 않는다. 서버 인가 요청의 `state` 대조와 OIDC 사용 시 서명·issuer·audience·만료·nonce 검증으로 해당 흐름을 검증한다. OAuth 시작 전 REST CSRF 취득을 필수 선행으로 가정하지 않는다.

Token 취득 GET·응답 필드·전달 Header·가입 미완료 Context·저장소는 TBD다. 인증/로그아웃 뒤 새 Token 취득을 검증하며 `SameSite=Lax`만으로 방어 완료를 선언하지 않는다. CSRF Token 취득은 인증·활동 권한을 부여하지 않는다. WSS Handshake에 REST Header 방식을 그대로 적용하지 않고 추가 검증 필요성과 전달 방식은 별도 계약으로 정한다.

---

# 7. `/device` 허용 대상과 대기 연결

## 7.1 목적

MVP는 **보호자 계정 1 : 아이 1 : 기기 1**을 유지하며 기기 연결은 **서버 코드 발급 → 테스트용 기기 웹에 코드 표시 → 보호자 앱 입력 → 계정의 아이와 연결 → 기기 자격(토큰) 발급 → 자격을 사용한 대기 WSS**로 한다. 등록 코드·등록 후 유지되는 Device 식별자(deviceId)·기기 자격은 서로 다르다. 코드를 화면에 표시하는 것은 테스트용 기기 웹 한정이며 화면 없는 실제 기기의 스티커·음성 안내 등은 후속이다. 보호자 Cookie·Google 토큰·GPT-Live 공급자 자격을 기기 자격으로 쓰지 않는다. 데모 연결 ≠ 운영 페어링·물리 소유권 인증이다.

보호자의 **새 등록이 성공한 뒤** 이전 관계를 해제한다. 잘못된 코드·이미 다른 계정에 연결된 코드로 기존 정상 기기를 먼저 해제하지 않는다. 서버가 브라우저 데이터 삭제를 감지해 이전 기기를 해제한다고 가정하지 않는다. 기존 기기 자격의 사용을 차단해야 하며 열린 WSS·늦은 보고·활동 중 교체 요청의 처리/원자성·실패 복구는 TBD다. 코드와 대기 중 등록 연결의 대응, 자격을 실제로 수신하는 주체/전달 경로, 코드 형식·유효 시간·시도 제한·자격 수명/갱신·검증/폐기 구현도 TBD다. 등록 이후에도 허용·현재 연결·점유·계정당 진행 세션 1개·원자 할당 검사는 유지한다.

## 7.2 등록 이후 시작

```text
코드 등록 완료·기기 자격 검증
→ Realtime /ws 대기 WSS·heartbeat/상태 보고
→ 보호자 상태 카드에서 계정의 아이의 연결 기기 확인
→ Core 현재 관계/동의·시작 권한 확인
→ Realtime 허용·현재 연결·점유·계정당 진행1개 검사/원자 할당
→ 대기 기기에 START push
```

카드 조회는 예약이 아니고 접수·기기/공급자 준비·아동 참여도 별개다. 기기 미연결이면 “연결된 기기 없음”으로 시작을 차단한다. 대기 WSS만으로 유료 공급자 연결을 유지하지 않는다.

---

# 8. `/device` WebSocket 검증

Handshake에서 환경별 허용 Origin·등록 Device·현재 유효 기기 자격·연결의 결합을 검증한다. 코드나 ID만으로 인증을 대체하지 않는다. 자격 전달/검증·만료/폐기·재연결·WS 종료 코드/오류의 상세는 TBD이며 보호자 Cookie를 기기 자격으로 쓰지 않는다.

연결 후에도 입력 음성·상태/재생 보고·ACK를 계정의 아이에 연결된 해당 기기·현재 연결·할당된 ActivitySession·최신 작업 및 현재 이용 권한과 대조한다. 다른 활동이나 구 연결의 메시지를 현재 활동에 적용하지 않는다. 내부 논리 식별자·보고 필드는 [protocol.md](architecture/protocol.md)의 후보이며 이 ADR에서 새로 확정하지 않는다.

인증/연결 권한 만료·폐기가 감지되면 신규 작업과 승인을 막고 Available 후보에서 제외하며 활동 이벤트 경로에서 안전 정리한다. 이미 열린 WSS를 닫았다는 사실만으로 실제 음성이 정지됐다고 기록하지 않는다. 현재 활동 상태, 기기 실제 입력·출력 정리, 공급자 종료는 각각 확인한다. 지연 ACK·재연결·재인증으로 종료 활동을 부활시키지 않는다. 감지 주기·재연결·정리 보고 수신 순서·Timeout·최종 상태/사유는 ADR-005/D-03 연계 TBD다.

## 8.1 `playbackMark`와 음성 권한

기기 플레이어의 관측 실제 재생 범위는 Realtime이 수신·검증한다. 보고 대상은 해당 활동·현재 연결·승인 음성/재생 묶음과 결합한다. 생성·승인·송신·수신 ACK를 실제 재생이나 아동의 청취/이해로 바꾸지 않는다. 중복·역전·구 연결·중단 뒤 늦은 보고를 현재 재생 성공으로 덮지 않고 보고 불가이면 마지막 신뢰 관측과 미확인 범위를 유지한다. 필드명·단위·주기·늦은 보고 처리의 상세는 TBD다.

공급자 출력은 Realtime 임시 메모리에서 구간 분할→전사 대응→상태·권한·최신성/필요 경량 의미 검사→동일 음성 승인→송신 직전 재확인을 거친다. 실패·누락·시간초과는 폐기와 대체 안내 경로이며 늦은 승인으로 취소된 음성을 다시 열지 않는다. 끼어들기·쉬기·종료·철회는 ADR-004/005의 사유별 취소를 따른다. 끼어들기만으로 ENDING을 확정하지 않는다.

---

# 9. 데모 연결과 운영 인증

코드 등록은 아이–기기 1:1 데모 연결과 우리 기기 자격 발급이다. Device ID만 아는 것으로 인증/인가를 충족하지 않으며 운영 페어링·물리 소유권·Hardware Secure Credential·Device Certificate의 검증 완료를 뜻하지 않는다. 공개 `/device`·허용 Origin·관계만으로 모든 활동 접근을 허용하지 않는다.

---

# 10. 사용 가능한 VoiceClient와의 관계

허용 대상·아이와의 데모 관계·현재 연결·Available·Assigned는 서로 다르다. 계정의 아이에 연결된 기기가 코드 등록된 허용 대상이고 자격이 유효한지, 현재 유효 WSS/heartbeat가 있는지, 이미 점유됐는지, 이번 활동에 원자적으로 할당 가능한지를 확인한다. 기기가 없으면 시작을 차단하고 계정당 진행 세션은 1개로 제한한다. 전체 Available 수를 전역 다중 사용자 정책으로 삼지 않는다. 동시 시작·중복 ID 연결·연결 교체·할당 해제 계약의 상세는 TBD다.

할당한 기기에는 Realtime이 보호자 시작 push를 보낸다. 인증 Context·허용 대상 확인은 할당을 자동 성립시키지 않으며 시작 접수 ACK도 아동 참여/재생 완료가 아니다. 상세 통신·할당은 ADR-004와 protocol이 소유한다.

---

# 11. MVP 한계

합성·성인 역할극 데이터와 통제된 시연 환경을 사용하며, MVP 관계는 보호자 계정 1 : 아이 1 : 기기 1, 계정당 진행 세션 1개로 제한한다. 이 정책을 전체 계정에 걸친 전역 활동 수 제한이나 처리량 보장으로 확대하지 않는다. 코드 등록 데모·Origin만으로 위장 기기를 배제했다고 보장할 수 없으며 운영용 기기 인증 검증으로 기록하지 않는다.

---

# 12. 실제 서비스 전 재설계

현행 아이–기기 1:1 데모 연결은 유지하며, 실제 물리 기기 또는 다중 사용자 운영 전 Device Registration/Identity/Ownership, 운영 Guardian Pairing과 Child Assignment 검증, Credential Rotation/Revocation, 분실 대응을 설계한다. 여러 아이·공동 보호자·형제 기기 공유는 1:1 제약을 확장하는 후속 과제다. MVP 코드 등록/기기 자격은 §7을 따른다. 운영용 기기별 인증서·QR Pairing·물리 소유권 검증은 **검토 후보이며 미채택**이다. QR claim을 현행 데모에 추가하지 않는다.

---

# 13. API Key / Secret과 내부 서비스 인증

## 13.1 서버 비밀과 공급자 권한

GPT-Live API(`gpt-live-1`)의 주 WebSocket 연결은 Realtime이 서버 키로 소유한다. 브라우저는 공급자 키·세션 생성/제어 권한을 받지 않는다. Core의 사후 AI Adapter와 Realtime 음성 Adapter의 호출 권한·비밀 주입 범위를 분리하며 Java SDK/운영 버전·공급자 설정의 실제 계약은 TBD다.

Project API Key, Google OAuth Client Secret, DB Password, Session 관련 Secret과 내부 서비스 자격은 서버 환경변수 또는 적절한 Secret 관리 영역에 둔다. Next.js Client Bundle·브라우저 JavaScript 상수·Git·브라우저 저장·일반 로그에 넣지 않는다. JWT 서명 Secret도 미채택 대안을 재검토할 때 같은 원칙을 따른다.

## 13.2 Core ↔ Realtime 서비스 신뢰

즉시 명령/조회는 내부 HTTP, 유실되면 안 되는 요청·완료·정책 변경 사실은 Transactional Outbox + 내부 HTTP + 영속 수신/Inbox를 사용한다. Broker는 추가하지 않는다. 두 경로 모두 수신 앱이 서비스 자격·허용 호출·대상·요청 범위를 검증하며 내부 네트워크 자체를 신뢰 근거로 삼지 않는다. 구체 인증 방식·키 관리/회전·Endpoint·DTO·기한/한도는 TBD다.

Core가 검증한 주체·허용 행위·대상·기한을 전달하고 Realtime이 계정의 아이–기기 관계와 활동/기기/주체 결합, 현재 권한을 대조한다. 브라우저가 보낸 `X-Guardian-Id` 등의 값은 검증 없이 내부 인증 주체로 승격하지 않는다. 서비스 인증 성공이나 eventId 지식만으로 사용자 인가를 충족하지 않는다. 각 앱은 자신의 업무/스키마를 수정하고 상대 Repository를 직접 사용하지 않는다.

내부 경로는 공개 진입을 차단하고 수신 인증도 유지한다. 발신 업무와 Outbox의 로컬 커밋, 수신 업무 또는 Inbox 커밋 뒤 ACK, 중복/역전·최신성·정책 검사는 [protocol.md](architecture/protocol.md)를 따른다. 수신 ACK는 업무 완료가 아니다. STOP은 Core Worker 대기열 뒤에 넣지 않는다.

Core 장애 중 보호자 새 제어가 `보호자→Core→Realtime`으로 전달되지 않는 의존은 남는다. 현재 권한을 확인할 수 없으면 새 이용을 보류·차단하고 안전 정리는 수행한다. 보호자 직접 Realtime 종료 경로는 **G-02 미해결 대안**이며 이 ADR에서 채택하지 않는다.

---

# 14. Browser Storage

**기기 자격(토큰)의 브라우저 저장소 보관은 테스트 웹 데모 한정 예외**다. 예외 대상은 등록 후 발급된 우리 기기 자격과 이를 식별하는 등록 Device 참조이며 등록 코드·보호자 Session·Google 토큰·GPT-Live 키의 장기 보관 허용으로 확대하지 않는다. localStorage/sessionStorage 등 실제 저장 방식·저장 형식·수명·로그아웃/계정 삭제·성공한 새 등록/자격 폐기 때 무효화 및 정리 시점은 TBD다. 브라우저 데이터 삭제로 자격이 사라지면 재등록이 필요하지만 서버의 이전 관계 해제 근거는 성공한 새 등록이다. 스크립트 접근·탈취/재사용 위험이 있으며 실제 제품 보안이나 물리 소유권 인증으로 인정하지 않는다. 운영 기기 인증/보관 방식은 별도 TBD이고 보호자 Session Cookie의 HttpOnly 원칙과 서버 비밀 비노출을 유지한다.

---

# 15. 로그와 중계 원음

필요한 논리 식별자·인증 성공/실패·오류·시간·Origin만 최소로 기록한다. `guardianId`, `sessionId`, `deviceClientId`는 기존 로그 식별자 후보이며 채택·마스킹/Hash 범위는 TBD다. Cookie/JWT/Session ID 전체, Password, OAuth Code·state·nonce·Client Secret, Access/Refresh/ID Token, API Key·내부 비밀을 출력하지 않는다. 실제 아동 발화 원문도 일반 로그에 출력하지 않는다.

허용된 활동에서 기기로 들어온 **아이 입력 음성 원본만 지정 원음 저장소에 보관**한다. AI 출력 음성은 저장 대상이 아니며 게이트 출력 버퍼는 메모리 전용으로 사용·폐기한다. 원음 보관은 필수 동의 항목이고 계정 삭제까지 보관하며, 기존 계정 삭제 요청 경로가 기록·전사와 함께 원음 저장소 및 DB 메타데이터도 삭제하도록 연결한다. DB에는 저장 위치·구간·세션 참조 등 메타데이터만 두고 원음 바이트·Outbox/Inbox·일반 로그·임시 디스크로 우회 저장하지 않는다. 기존 승인 안내 음성 파일은 별도 콘텐츠다.

대기 WSS의 주변 음성을 상시 저장하지 않는다. 참여 확인 전 입력은 기존 미참여 원문 미저장·수집 허용 정책과 대조해야 하며 포함 여부는 **TBD**다. Realtime 수신에서 저장 경로를 연결하되 저장 담당 앱·메타데이터 쓰기 소유자·저장 완료 기준·부분 저장/실패·삭제 중 늦은 저장 차단의 원자성은 **TBD**다. 저장소 종류·형식·구간 단위·암호화·접근 권한·백업 삭제 상세도 TBD이며 다른 앱 Repository를 직접 수정하지 않는다. 저장 실패를 임시 디스크 우회나 무제한 메모리 적재로 보완하지 않는다.

---

# 16. Alternatives와 대체된 이전 선택

## Alternative A. Guardian과 기기에 동일 인증 — 미채택

보호자 사용자 권한과 데모 기기의 신뢰 수준·역할이 달라 동일 인증을 쓰면 기기에 불필요한 권한이 생긴다. 보호자 Cookie를 기기 인증으로 재사용하지 않는다.

## Alternative B. Origin만으로 기기 허용 — 미채택

Origin만으로 허용 ID·현재 연결·할당과 개별 Client 권한을 표현할 수 없다. 현행 통제 데모도 허용 대상·연결·점유·할당 검사를 함께 사용한다. 이것이 운영 인증 완성을 뜻하지 않는다.

## Alternative C. MVP부터 하드웨어 인증 — 미채택

실제 하드웨어가 없고 Device Identity 정책이 미정이므로 제품화 전 재설계를 남긴다. Browser 데모에 인증서·페어링 기술을 자동 추가하지 않는다.

## 대체된 이전 선택 — 현행 구현 지시 아님

- **3개 소셜 로그인:** 2026-10-01 원본은 카카오·구글·네이버를 채택했다. `google/kakao/naver` 등록값과 각각 `/oauth2/authorization/{registrationId}`·`/login/oauth2/code/{registrationId}` Callback, 카카오·네이버 사용자 속성 매핑·Client 설정, 세 제공자 시험 순서·계정·등록/공개 검수, 네이버 일반 공개 검수 및 각 제공자 쿼터 확인을 요구했다. 최신 Google만 결정이 이 범위를 대체한다. 기존 내부 Guardian·Session·Logout·CSRF·동의 검사는 보존한다.
- **기기 발급 후보:** 원본 `/device` 단기 인증 Cookie·운영자 일회용 발급 코드는 발급 자격·환경·Origin·CSRF 확인→코드 유효/미사용·원자 소모→Credential 발급→Handshake 검증을 제안했다. Cookie 이름/TTL/Path·코드 전달/보관·시도 제한·폐기/갱신은 미정이었다. 당시 고정 ID 사전 시드·대기 WSS가 이를 대체했다. A1.5는 별도 코드 등록/기기 자격 계약으로 다시 구체화하며 원본 Cookie·운영자 코드 계약을 채택하지 않는다.
- **전역 기기 수:** 원본의 Available 0개 차단·1개 사용·2개 이상 차단은 계정의 아이에 연결된 기기의 허용·연결·점유·원자 할당 검사로 대체했다. 원본의 한 번에 하나 시연 전제는 전역 운영 제한이 아니며 현행 동시 진행 제한은 계정당 1개다.
- **브라우저 공급자 권한:** 원본 직접 WebRTC·SDP·브라우저 Data Channel·Sideband와 공급자 세션 준비/종료 관측 후보는 Realtime 소유 GPT-Live 주 WebSocket으로 대체됐다. 브라우저 임시 키·Ephemeral 권한 경로는 현행에 추가하지 않는다.
- **관계를 제공하지 않는다는 설명·아이 선택/삭제:** 이전의 “보호자/아동 영구 연결을 제공하지 않는다”와 아동 조회/수정/삭제를 일반 작업으로 나열한 표현, 지정 고정 ID 선택으로 읽히던 흐름은 A1.4의 계정 1 : 아이 1 : 기기 1 데모 연결로 대체했다. 기존 관계를 1:1로 제한하고 아이 추가·삭제·선택과 공동 보호자는 현행 범위에서 제외한다. 아이 변경은 계정 삭제 뒤 새 가입이며 운영 페어링·물리 소유권 인증의 미결정 상태는 보존한다.
- **단일 Spring:** 보호자 인증·기기/활동을 한 서버에 배치했던 이유는 Core 인증/업무와 Realtime 기기/활동의 두 실행 앱 경계로 대체했다. 기존 도메인 소유권 원칙은 유지한다.

원본이 **2026-10-01 확인**으로 기록한 참고 링크도 이력으로 보존한다: [Spring OAuth2 Login](https://docs.spring.io/spring-security/reference/servlet/oauth2/login/core.html), [기본 인가/Callback 구성](https://docs.spring.io/spring-security/reference/servlet/oauth2/login/advanced.html), [Google 로그인 구성](https://codelabs.developers.google.com/codelabs/sign-in-with-google-button), [카카오 쿼터](https://developers.kakao.com/docs/ko/getting-started/quota), [네이버 FAQ](https://developers.naver.com/products/intro/faq/faq.md), [네이버 검수](https://developers.naver.com/docs/login/verify/verify.md). 이번 개정은 audit와 해당 조항을 구체화한 A1.4를 기준으로 하며 링크를 다시 조회하거나 제공자 정책의 현재성을 새로 검증한 것은 아니다.

**대체된 이전 선택 — A1.4:** 원음 미저장·고정 ID 사전 시드/등록 UI 없음·장기 기기 자격 브라우저 저장을 기본으로 두지 않던 데모는 A1.5 지정 원음 저장·코드 등록/자격·브라우저 저장 데모 예외로 대체됐다. 이전 코드 후보와 현행 코드 등록을 같은 발급 계약으로 재채택하지 않으며 형식·수명·구현은 TBD다.

---

# 17. Consequences

## 장점

- 보호자·기기·내부 서비스의 권한 범위와 검증 책임이 명확하다.
- Google 로그인과 내부 Guardian·Spring Session의 기존 구현 선택을 유지한다.
- 자체 비밀번호 관리 부담과 브라우저 공급자 비밀 노출을 줄인다.
- 코드 등록으로 계정의 아이와 기기를 1:1로 데모 연결하고 아이/기기 선택 없이 상태 카드와 시작 대상을 정한다. 자격 검증 후 대기 연결과 보호자 시작 명령을 사용한다.
- 현재 권한을 음성 입력/승인/송신·사후 처리 경계에 적용하고 실제 제공은 기기 관측과 분리한다.
- 실제 기기 인증·페어링의 후속 재설계 지점이 남는다.

## 단점

- Core Session·CSRF와 Realtime 기기/내부 경로를 각각 구현·검증해야 한다.
- 코드 등록과 계정의 아이–기기 데모 연결은 위장 방지·운영 페어링·실제 기기 소유권을 보장하지 않는다. 1:1 제약과 계정당 진행 세션 1개를 앱 간 요청/동시성 검사에 반영해야 한다.
- PostgreSQL Session Migration·만료와 Principal 재배포 호환성·Token 최소 보관을 운영해야 한다.
- Google 등록/Callback·정책 변경·장애와 계정 연결/탈퇴 정책에 영향을 받는다.
- 내부 인증·권한 전달·철회 경합·연결 교체와 늦은 메시지를 다루는 계약이 추가된다.
- Core 장애 중 보호자 제어와 현재 권한 확인의 의존이 남는다.

---

# 18. Risks

## Google·Session 선택과 실제 시험의 혼동

Google만·Spring Security OAuth2 Client·Spring Session JDBC 방향은 확정이지만 앱 등록·Callback·취소/실패·프로필 누락·Session 만료/무효화/재시작·Cookie·CSRF는 미검증이다. OAuth Principal·Authorized Client 저장 위치와 Token 최소 보관/폐기·state/OIDC·허용 Redirect를 확인한다.

## Authentication과 Consent/인가 혼동

로그인·내부 인증·ID 지식만으로 작업 권한을 허용하지 않는다. 현재 관계/동의를 확인하며 새 이용의 확인 불가는 차단한다. 철회 알림과 실제 송신/저장 경합은 G-03을 남긴다.

## 데모 등록의 신뢰 과장

계정의 아이–기기 데모 관계와 통제된 시연 허용 대상은 운영 인증과 구분한다. 관계나 공개 ID만 확인해 현재 권한·연결·점유·할당 검사를 생략하지 않는다. 중복 연결·미할당 보고·구 연결의 지연 메시지를 검증하고 실제품 등록/운영 페어링을 후속으로 남긴다.

## Cookie/CSRF와 내부 경로 설정 오류

Core Cookie·REST CSRF와 OAuth state/OIDC·WSS Origin 검증을 구분한다. 공개 프록시가 내부 경로를 노출하거나 내부 주체를 브라우저 Header에서 신뢰하지 않도록 시험한다. HTTPS Secure와 각 앱의 FilterChain/경로 Matcher 상세는 TBD다.

## 승인·ACK와 실제 중단/재생 혼동

출력 승인·전달 직전 권한 재확인, 취소 뒤 늦은 승인 무효화, 기기 실제 재생 보고를 별도로 검증한다. WSS 종료·서버 최종 상태·공급자 정리를 성공으로 한꺼번에 채우지 않는다. 보고가 없으면 미확인으로 보존한다.

---

# 19. Pending과 첫 볼트 인증 시험

아래 구현·시험은 모두 미검증이다. 합의가 필요한 기대값을 정하기 전 완료로 기록하지 않는다.

- [x] 최신 사용자 확정 Google만; 기존 Spring Security OAuth2 Client + Spring Session JDBC와 자체 ID/Password 제외 유지
- [x] 코드 등록·기기 자격·대기 WSS·상태 카드·보호자 시작 push 방향, 서버 키/주 연결의 Realtime 소유
- [x] 보호자 계정 1 : 아이 1 : 기기 1, 아이/기기 선택 없는 시작 대상 유도, 기기 없음 시작 차단과 계정당 진행 세션 1개 방향
- [ ] GuardianChild·아이–기기의 1:1 제약 구현, 계정에서 아이/기기를 유도하는 계약, 활동의 아이·기기 기록과 원자적인 동시 시작 검증
- [ ] Google 앱 등록·Redirect URI·Scope·OAuth/OIDC·고유 회원번호 매핑·공개 조건·시험 계정과 성공/취소/오류/동의 거부/프로필 누락·state/nonce 검증
- [ ] Guardian 필드·가입 완료/미완료 Context·추가 로그인 계정 연결/해제·서비스 계정 삭제/재가입의 상세; 아이 변경은 계정 삭제 뒤 새 가입 방향 유지, 이메일 자동 병합 금지 검증
- [ ] Core Spring Session의 Flyway/스키마 책임·만료·로그아웃·재시작·직렬화/Authorized Client·Token 최소 보관/폐기
- [ ] Cookie 이름/Path/Domain/TTL·동시 로그인·CSRF 취득/응답/Header와 로그인/로그아웃 후 Token 갱신, Token 누락/변조 거부
- [ ] 두 보호자의 Child/Session/결과/리포트 접근 격리, 가입 미완료·기기·내부 Context로 보호자 권한 취득 차단
- [ ] 계정의 아이와 코드 등록한 기기/유효 자격·허용 Origin·현재 WSS/heartbeat·점유·동시 원자 할당·중복 연결/교체·상태 카드와 시작 push; 기기 없음 표시/시작 차단·계정당 진행 세션 1개·타 계정 아이/기기 거부
- [ ] 미할당/타 활동/종료 활동·구 연결 음성/보고 거부, ACK/기기 준비/실제 재생 구분, playbackMark 중복/역전/늦은/보고 불가
- [ ] 운영용 기기 인증·권한 만료/폐기 감지·재연결·중단 보고 수신 순서·실제 입력/출력/공급자 종료 확인의 상세
- [ ] 내부 서비스 인증·비밀 주입/회전·공개 내부 경로 차단·주체/행위/대상/기한·현재 권한, HTTP 중복·Outbox/Inbox 영속 ACK 검증
- [ ] 버퍼 게이트 권한·동일 승인 음성·송신 직전 재검사·취소 뒤 늦은 승인 폐기, 원음 우회 저장/로그·브라우저 비밀 노출 방지
- [ ] 필수 미동의/철회 시 새 시작 차단, 유효한 종료·최소 상태 확인 허용, 철회 후 신규/지연 저장 차단과 G-03 원자성/효력·D-08 보존/열람/삭제
- [ ] 보호자 로그아웃/만료 시 진행 활동 처리(D-03), Core 장애 중 제어 의존 수용 또는 별도 경로(G-02, 미채택)
- [ ] Origin 목록·각 앱 SecurityFilterChain/Matcher·REST/WS 오류 형식·노출 최소화와 프런트 상태 재확인 계약
- [ ] 실제 운영 전 Device Registration/Pairing/Identity·회전/폐기·분실 대응

A1.5 §12의 결정은 §7·§14·§15와 아래 알림/전사 경계에 연결했다. 코드 등록/교체·브라우저 자격 무효화·원음 필수 동의/저장/삭제·전사 노출/재연결·푸시 재조회 인가를 실제 시험해야 하며 구현 완료로 표시하지 않는다.

## 알림·전사 권한

**Core→보호자 알림 / Core→Realtime→기기 WSS 명령 / 서버→보호자 실시간 전사**는 세 통로다. MVP 알림은 Core가 발송하며 Realtime 활동 상태 변화는 기존 Outbox/Inbox 영속 전달로 Core에 도달한다. 이 알림은 기기 START나 실시간 전사를 대신하지 않는다. 활동 알림 페이로드는 **sessionId·상태만**이며 대화 내용·전사·아이 정보는 담지 않는다. sessionId를 아는 것만으로 기록 조회 권한을 주지 않는다.

푸시는 최신 상태 재조회 힌트이며 상태 원본이 아니다. 앱 진입·복귀·재연결·알림 클릭 때 권한 확인 후 서버 최신 상태를 다시 조회한다. 알림 지연/유실에도 같은 조회 계약을 사용하고 조회 실패 시 마지막 확인 시각과 미확인을 표시한다(구체 필드/표현 TBD). MVP 지원은 **PC Chrome·Edge**이며 모바일·Safari는 후속이다. 제공 방식(Web Push 등)·구독 저장·발송 실패 처리/재시도는 TBD다. 계정·기록 삭제 완료도 알림 용도에 포함되지만 세션 없는 알림의 식별/페이로드와 로그아웃·삭제 뒤 구독 처리는 다음 연결 지점이며 임의 sessionId나 새 필드를 만들지 않는다.

보호자 실시간 전사는 **아이 입력 전사와 게이트를 통과하여 실제 기기에 전달된 AI 출력 전사만** 다룬다. 미검사·차단·승인 후 미송신 AI 전사는 노출하지 않는다. 생성됨 / 승인됨 / 기기에 전달됨 / 기기가 재생 보고함을 구분하며 전달만으로 실제 재생 완료를 표시하지 않는다. 실제 제공 범위는 기존 `playbackMark` 관측·부분 재생/확인 불가와 연결한다. GPT-Live 이벤트는 서버 Adapter가 해석하고 보호자 메시지와 구분하며 다른 Realtime API 이벤트명으로 중계하지 않는다. Realtime→Core→보호자 또는 별도 경로·전송 방식·진행 중 입력 전사 표시·재연결 중복/순서/누락·구독 인가·물리 필드/메시지명은 **TBD**다.

팀 판단 기록은 기존 [decision-log.md](decision-log.md)에 연결한다. 문서 정합과 위 구현·실연동 시험의 완료를 구분한다.

---

# 20. Source of Truth와 다음 연결 지점

| 문서 | 책임·후속 연결 |
| --- | --- |
| ADR-008 | 보호자 Google/Session 선택, 인증/인가/동의, Cookie·CSRF, 계정 1 : 아이 1 : 기기 1과 코드 등록·기기 자격/운영 인증 경계, 내부 인증·서버 Secret |
| ADR-004 + protocol | 기기 WSS 음성/제어, GPT-Live 주 연결·버퍼 게이트, 할당·ACK·재생 범위·취소 계약 |
| ADR-005 + session-state | 상태·사유·Timeout·즉시 아동 ENDING·앱별 복구·늦은 결과/보고 |
| architecture/auth | 로그인/Logout/CSRF·대기 WSS/내부 신뢰 시퀀스, FilterChain·Cookie·오류·시험 상세 |
| ADR-007 | Core/Realtime 모듈 배치·auth/consent 및 활동 소유 경계 반영 완료. 실제 패키지·원격 계약·인증 구현은 TBD |
| data-model + result-pipeline | 앱별 스키마/Flyway·쓰기 권한, GuardianChild·아이/Device 1:1 제약과 활동의 아이·기기 기록, Outbox/Inbox/run·전사/실제 재생 참조·철회/삭제 경합 |
| ADR-006 + first-bolt | 후속: 내부 경로 공개 차단·두 앱 키 주입/라우팅, Google·코드 등록·게이트·실제 재생·Core 장애/철회 실연동 시험 |

---

# 21. Decision Summary

```text
Guardian: Core Spring Security, Google만 가입/로그인
Libraries: Spring Security OAuth2 Client + Spring Session JDBC 기존 Accepted 유지
Password: 자체 ID/Password 가입·로그인 제공 안 함
Service Session: 내부 Guardian 인증 + PostgreSQL Session + HttpOnly Cookie
Provider Token: 보호자 API Credential로 재사용 안 함
Logout: 우리 Session 무효화 + Cookie 폐기; Google Logout/탈퇴와 별개
Relationship: MVP 보호자 계정 1 : 아이 1 : 기기 1; 계정당 진행 세션 1개
Child change: 아이 추가·삭제·선택/공동 보호자 없음; 아이 변경은 계정 삭제 뒤 새 가입
Resource Access: 계정의 아이·기기/활동/결과/리포트 관계 + 작업별 현재 권한
Consent: 현재 DB 상태, 확인 불가 새 이용 차단, 철회 G-03 상세 TBD
Device demo: 코드 발급 → 보호자 입력 → 아이–기기 연결 → 기기 자격 → 대기 WSS/상태 카드/시작 push
Device trust: 데모 연결 제공, 운영 페어링·인증·실제 기기 소유권 증명 아님
Assignment: 계정의 아이 → 그 아이의 기기; 기기 없음 시작 차단 + 허용/현재 연결/점유/원자 할당 검사
Playback: 기기 관측 playbackMark; 승인·ACK·송신과 실제 재생 구분, 필드 TBD
Voice provider: GPT-Live gpt-live-1 주 WebSocket, Realtime 서버 키·버퍼 게이트
Internal: 서비스 인증과 사용자 인가 분리, 즉시 HTTP + 영속 Outbox/Inbox, Broker 없음
Secrets/raw audio: 서버 비밀 비노출; 기기 자격 브라우저 저장은 데모 예외; 아이 입력만 지정 저장소·계정 삭제까지, AI 출력 버퍼 메모리 전용
CSRF: Cookie 변경 REST/Logout 보호, Google Callback state/OIDC, 상세 시험 TBD
Open gates: 코드/자격·교체/무효화·원음 소유/완료/삭제·푸시/전사 구현, 운영 인증·G-02/G-03와 기존 설정/실연동 TBD
```
