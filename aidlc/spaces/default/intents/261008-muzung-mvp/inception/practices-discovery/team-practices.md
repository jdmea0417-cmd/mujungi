# 팀 작업 방식 — 무중 MVP

> 팀 작업 방식 인터뷰(Q1~Q17)와 요약 확인("Looks correct")을 반영한 최종본이다. 괄호 안의 `Q<n>`은 `practices-discovery-questions.md`의 질문 번호이고, `§`는 기술명세서 v0.2의 절 번호다. 기본값(`org.md`)·기술명세서·ADR_2S와 다른 팀 결정은 그 자리에 "팀 결정"으로 적었다. 근거와 차이는 `evidence.md`에 있다.

## Way of Working

- 트렁크는 `main` 하나다. 첫 커밋 전에 기본 브랜치를 `master`에서 `main`으로 바꾼다. `dev` 같은 오래 유지되는 통합 브랜치는 두지 않는다. 시연 직전 고정이 필요하면 `main`에 태그(예: `demo-1105`)를 붙여 그 태그를 배포한다. (Q2, Q9 확인)
- 작업 브랜치 이름은 `feature/{JiraKey}-설명`, 커밋 메시지는 `{JiraKey} type: 내용` 형식을 쓴다. 업무는 Jira Kanban으로 관리하고 GitHub와 연동한다. (§1.2, ADR-001)
- PR은 리뷰어 1명 승인과 CI 통과 뒤에 `main`에 병합한다. (§1.2)
- PR은 **병합 커밋(merge commit)** 으로 `main`에 합치고, 브랜치 수명은 따로 정하지 않는다. 이는 기본값(스쿼시 병합, 1~2일 브랜치)과 다른 팀 결정이다. Construction의 Bolt 작업 공간도 같은 방식으로, `main`에서 갈라져 `main`으로 병합 커밋(no-fast-forward)으로 합치며 Bolt 브랜치의 개별 커밋을 이력에 남긴다. (Q1)
- 저장소는 새 **비공개** GitHub 저장소 `mujungi` 하나다. 코드와 AI-DLC 기록(설정 `.claude/`와 기록 `aidlc/`)을 모두 이 저장소에 둔다. 이 단계(팀 작업 방식)를 마친 뒤 새 폴더 `mujungi/`를 만들고 AI-DLC 설정과 기록 전체를 그대로 옮겨 이어서 진행한다. (Q3, Q12, Q16, Q17)
- 저장소는 Monorepo다. 코드 폴더는 기술명세서 §2.4 구조(`backend/`(core-api·realtime-activity·contracts), `frontend/`, `content/`, `ai/`, `infra/`, `docs/`, `AGENTS.md`, `CLAUDE.md`)를 따르고, 이중 폴더(`mujungi/mujungi/...`)는 만들지 않는다. (§2.4, Q3)
- 두 서버 사이 계약을 바꿀 때는 `contracts/`를 먼저 고치고 양쪽 서버를 같은 PR에서 맞춘다. (§2.4)
- 여러 사람이 함께 고치는 공유 파일(`contracts/` 계약, Flyway 마이그레이션 번호, `content/` 승인 문구·대본, `AGENTS.md`)의 담당·리뷰 방식은 Unit 나누기 단계에서 정한다. (Q4)
- 구현 단계의 Unit은 5명이 병렬로 개발할 수 있게 나눈다. (의도 파악 Q11, project.md 교정)
- 참고 문서가 서로 다를 때는 이 순서로 앞의 문서가 이긴다: 화면 설계 v1.6 → 요구사항 v1.4 → 기능명세서 v1.3 → ADR_2S → 기술명세서 v0.2 → AI 동작 명세 v0.1. 화면·요구사항과 다른 ADR_2S 내용은 따르지 않으며, 이미 승인된 단계는 메모만 남기고 유지한다. (Q15)

## Walking Skeleton

- 끝까지 이어지는 얇은 첫 조각을 먼저 만든다. (Q5)
- 첫 Unit은 아이 기기 → 아동 서비스(realtime-activity) → 보호자 서버(core-api) → 보호자 앱이 끝까지 이어지는 가장 작은 통합 조각이며, AI는 **가짜 모델**(가짜 GPT-Live·가짜 제브·가짜 LLM)을 쓴다. 실제 GPT-Live·제브는 PoC 결과를 보고 나중 Unit에서 붙인다. 이는 ADR_2S `architecture/first-bolt.md`(실제 GPT-Live·출력 게이트를 첫 볼트에 넣음)와 다른 팀 결정이다. (Q5)
- 첫 조각은 Code Generation까지 마친 뒤, 사람이 정한 구현 확인 명령으로 실제 동작을 보이고 사람이 승인해야 나머지 Unit으로 넘어간다. 확인 명령 자체는 Construction 진입 때 사람이 고른다. (org 기본값, 범위 `skeleton: on`)
- 일은 위험 먼저 순서로 한다. GPT-Live·제브 PoC와 아동 서비스 핵심(세션 상태·연결·판정 턴)을 앞에 두고, PoC에 기대지 않는 보호자 앱·기록·리포트는 병렬로 진행한다. (범위 정의 Q7)

## Testing Posture

- **Methodology**: test-after
- **Ordering**: 각 Bolt에서 시험 가능한 계층 하나를 먼저 구현하고, 곧바로 그 계층의 시험을 작성해 실행해 통과시킨 뒤 다음 계층으로 넘어간다.
- 커버리지: 줄 커버리지 80%를 서비스(모듈)별로 잰다. 미달이면 PR을 막는다. 측정에서 빼는 범위는 생성 코드(openapi-typescript 생성물 등), `contracts/` DTO, `ai/`, 시험 도구·가짜 모델 코드뿐이다. 측정 도구 후보는 Java JaCoCo, TypeScript Vitest 커버리지다. 이 하한과 제외 범위는 시험을 통과시키려고 낮추거나 넓히지 않는다. (Q7)
- CI(GitHub Actions)는 PR마다 build·lint·ArchUnit과 함께 단위·계약·통합·시나리오·장애 주입 시험을 **모두** 가짜 AI로 실행한다. (Q8)
- 실제 마이크·스피커를 쓰는 음성 수동 시험, PoC(`ai/` 스크립트), 실제 AI 시험은 CI에서 돌리지 않고 사람이 따로 실행한다. CI에서는 실제 유료 AI를 부르지 않는다. (Q8, §10, ADR-002)
- 시험 수준과 도구는 기술명세서 §10을 따른다.
  - 단위: 세션 상태 전이표, 연습 상태 기계, 종료 표현 일치, 확신 문턱·시간 제한 대체 경로, 장면 결과 라벨 대응, 금지 표현 검사 — JUnit 5, Vitest
  - 계약: 보호자 서버 ↔ 아동 서비스 명령·이벤트, 기기 프로토콜, OpenAPI 명세와 생성 타입 — `contracts/` 스키마 검증
  - 통합: 시작 → 종료 확인 → 결과, 재전달·중복 제거, 동의 철회·아동·계정 삭제 연쇄 — Testcontainers(PostgreSQL, Neo4j)
  - 시나리오: 합성 페르소나 대본을 입력 전사로 주입하는 하네스, 가짜 GPT-Live·가짜 제브·가짜 LLM — JUnit + 대본 파일
  - 장애 주입: AC-CON-01~10 — 기기 연결·GPT-Live 연결·내부 채널·실시간 갱신 연결을 각각 끊는 시험 도구
  - 음성 수동: 종료 표현, 재생 중 끊기, 에코 재유입, 판정 턴 공백 — 수동 체크리스트
  - PoC: 제브 한국어 아동 발화 정확도·확신 문턱, GPT-Live 출력 게이트, 승인 원문 낭독 일치율 — `ai/` 스크립트
- 시나리오 하네스 필수 사례(§10): 중복 시작 / 상황 준비 중 취소·참여 전 취소·진행 중 종료 구분 / 기본·추가 종료 표현과 역할 대사·휴식 요청 구분 / 첫 반응 전 단서 차단 / 판정 보류에서 교정 없음 / 도움 뒤 반응이 첫 반응을 덮어쓰지 않음 / 시작 전 미참여 무기록 / 동의 철회 뒤 수신 자료 미저장 / 리포트 인용 일치·결측 표시 / GPT-Live 요청에 판정 정보·보호자 입력 원문 없음.
- 선택된 시험 전략은 Standard다. §10의 계약·시나리오·장애 주입 수준은 Standard에 **더해** 필수로 둔다.
- 구조 규칙은 ArchUnit으로 CI에서 막는다: 모듈 순환 의존, 다른 모듈 Repository 직접 접근, Controller → Repository 직접 접근, 외부 SDK 타입이 Adapter 밖으로 나가는 것. (§2.5)
- 시험 데이터는 합성 데이터와 성인 역할극만 쓴다. 합성 결과를 실제 아동 성능으로 보고하지 않는다. (§8, §10)
- 시나리오·장애 주입의 절차와 환경은 별도 테스트 계획서에서 정한다(§1.3). 가짜 모델의 담당·위치와 테스트 계획서 담당은 기능 설계·빌드와 시험 단계에서 정한다.

## Deployment

- 배포 환경은 시연 서버 하나(AWS EC2 1대, 서울)뿐이며 스테이징은 두지 않는다. Docker Compose로 nginx·web·core-api·realtime-activity·postgres·neo4j 컨테이너를 띄우고 DB는 컨테이너로 둔다. (범위 정의 Q6, §11.1)
- `main`에 병합하면 GitHub Actions가 시연 서버에 **자동 배포**한다. 이는 기술명세서 §11.3·ADR-006의 "수동 `deploy.sh`, 자동 CD 미채택"과, 기본값의 "스테이징 자동 배포 + 운영은 별도 수동 승인"과 다른 팀 결정이다. 시연 서버가 유일한 환경이므로 배포 전 별도 승인 단계는 두지 않고, PR 리뷰 1명 + CI 통과가 배포 전 관문이다. (Q9)
- 배포 뒤 헬스 체크를 돌린다. 실패하면 파이프라인은 알림만 보내고 자동으로 되돌리지 않는다. 되돌리기는 사람이 GitHub Actions에서 이전 git SHA를 골라 다시 배포한다. 헬스 체크 대상·알림 경로·이미지 태그 방식의 세부는 배포 파이프라인 단계에서 정한다. (Q10)
- 시연 직전에는 `main`의 태그를 기준으로 배포 버전을 고정할 수 있다. (Q9 확인)
- DB 스키마 변경은 서비스마다 Flyway 마이그레이션(core, activity)으로 하고, JPA가 운영 스키마를 바꾸지 않게 한다. Flyway 버전 번호 규칙의 세부는 기능 설계 단계에서 정한다. (§11.3)
- EC2는 Stop으로 멈추고 Terminate를 일상 중지 방법으로 쓰지 않는다. (§11.1)
- HTTPS 방식과 인스턴스 크기는 인프라 설계 단계에서 정한다. (§11.1 ⬜·🧪)

## Code Style

- 프론트엔드(TypeScript): ESLint와 Prettier를 쓰고 CI에서 PR마다 검사한다. 실패하면 PR을 막는다. (§1.2, §11.3)
- 백엔드(Java): 별도 포매터·린터 도구를 두지 않고 IDE 기본 포매터와 코드 리뷰로 모양을 맞춘다. 이는 기본값("린터를 CI에서 돌려 실패 시 PR 차단")과 다른 팀 결정이다. Java 코드의 자동 검사는 build·시험·ArchUnit으로 한다. (Q11)
- 이름 규칙은 언어 관례를 따른다. Java는 클래스 PascalCase, 메서드·필드 camelCase, 상수 UPPER_SNAKE_CASE, 패키지 소문자. TypeScript는 camelCase(타입·컴포넌트 PascalCase). Python(`ai/`)은 snake_case. 오류 코드는 기존 예시처럼 UPPER_SNAKE_CASE다. (§5.9)
- 서버 내부 코드는 기술명세서 §2.5의 기능 모듈에 둔다. 모듈 순환 의존, 다른 모듈 Repository 직접 접근, Controller → Repository 직접 접근, 외부 SDK 타입이 Adapter 밖으로 나가는 것을 하지 않으며 ArchUnit으로 막는다. (§2.5)
- REST 오류 응답은 Spring `ProblemDetail`에 서비스 오류 코드 필드 `code`를 더한 형식이다(예: `DEVICE_NOT_READY`, `SESSION_STATE_CONFLICT`). (§5.9)
- AI 개발 도구 공통 규칙은 저장소 루트 `AGENTS.md`에 원본을 두고 `CLAUDE.md`는 `@AGENTS.md`로 가져온다. (§2.4, §11.4)
- 아래 기술 관례(인터뷰 Q14의 12~23번)는 일반 관례로 지킨다. 상황에 따라 조정이 필요하면 리뷰에서 근거를 남기고 정한다.
  - 보호자 인증은 서버 세션으로 하며 JWT 라이브러리를 넣지 않는다. (§8, §11.4)
  - TTS·STT 라이브러리를 넣지 않는다. 아이에게 들리는 승인 문구·고정 대본은 ScriptSpeaker를 거친다. (§1.2, §11.4)
  - OpenAI 호출은 openai-java를 직접 쓰고 OpenAI Spring Boot starter를 쓰지 않는다. SDK 모델 객체를 Controller 응답으로 그대로 반환하지 않는다. (§11.4)
  - 우리 코드에서 `com.fasterxml.jackson.databind`를 직접 쓰지 않는다(SDK 내부 Jackson 2는 허용). Jackson 3 기준, `@MockBean` 대신 `@MockitoBean`. (§11.4)
  - openapi-typescript 생성 파일을 직접 고치지 않고, API 타입은 생성물만 쓴다. (§1.2, §11.4)
  - 프론트엔드 서버 조회는 TanStack Query로 하고 `useEffect`+`fetch`+`setInterval`로 조회를 직접 만들지 않는다. (§1.2, §11.4)
  - DB 스키마는 Flyway로만 바꾸고 JPA가 운영 스키마를 바꾸게 하지 않는다. (§11.3)
  - 다른 앱(서버)의 Repository를 직접 수정하지 않는다. (ADR_2S)
  - 계약은 `contracts/`를 먼저 고치고 양쪽 서버를 같은 PR에서 맞춘다. (§2.4)
  - PR은 리뷰어 1명 승인과 CI 통과 뒤에 병합한다. (§1.2)
  - EC2를 Terminate로 일상 중지하지 않는다. (§11.1)
- 비밀 정보 유출 방지: 저장소 루트 `.gitignore`에 `.env` 등 비밀 파일 패턴을 넣고, 변수 이름만 적은 `.env.example`을 둔다. GitHub 비밀 스캔과 푸시 보호를 켠다. 비공개 저장소에서는 요금제에 따라 이 기능을 켜지 못할 수 있으며, 그 경우의 대안은 CI 파이프라인 단계에서 다시 정한다. (Q13)
