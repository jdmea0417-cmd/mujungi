**Collaborator:** aidlc-developer-agent

## Contribution

> 검토 범위: 코드 스타일과 관례 — 이름 규칙, 계층·모듈 경계, 오류 처리, 파일 구성, Java·TS 포매터/린터, 브랜치·커밋 이름 충돌, 5인 병렬 개발. 근거는 기술명세서 v0.2(`aidlc/spaces/default/knowledge/documents/04_기술명세서_v0.2.md`), `aidlc/spaces/default/memory/org.md`, 범위 정의 `intent-backlog.md`, 그리고 저장소 루트 실제 상태다. 저장소에는 소스·커밋이 없으므로 관찰한 코드 관례는 없다. 아래에서 **[추론]**이라고 적은 것은 문서에 없는 해석이며 팀 사실로 옮기면 안 된다. 값을 정하지 않은 항목은 인터뷰 질문으로만 낸다.

### 1. Code Style 절에 빠진 기술명세 내용 (추가 제안)

리드 초안 Code Style 절은 도구와 라이브러리 규칙은 잘 옮겼지만, 기술명세에 이미 적힌 **구조 규칙**과 **오류 형식**이 Code Style에 없다. 둘 다 코드 생성 단계가 가장 먼저 참조할 내용이므로 Code Style 절에도 두는 것을 제안한다.

1. **계층·모듈 경계(§2.5)** — 초안은 ArchUnit 규칙을 Testing Posture 절에만 적었다. 이 네 가지는 시험 도구 설정이기 전에 코드 구조 규칙이다.
   - 모듈 사이 순환 의존 금지
   - 다른 모듈의 Repository 직접 접근 금지
   - Controller → Repository 직접 접근 금지
   - 외부 SDK 타입(openai-java, 제브 클라이언트 등)은 Adapter 밖으로 나가지 않는다
   - 제안: Code Style에 "규칙", Testing Posture에 "ArchUnit으로 강제"로 나눠 적는다. 또한 이 네 가지를 `discovered-rules.md`의 Forbidden 후보로 올린다(현재 후보 목록에 없음).
2. **모듈 구성(§2.5)** — 서버 내부는 기능 단위 모듈로 나뉜다(core-api: auth, guardian, consent, child, device, session, record, goal, practice, result, report, push, content, knowledge, ai / realtime-activity: devicelink, voice, runtime, judge, relay, observe). 새 코드는 이 모듈 중 하나에 들어간다는 점을 Code Style 또는 Way of Working에 적는 것을 제안한다. 모듈 목록 자체를 바꿀 때 누가 결정하는지는 **[미정]**.
3. **오류 형식(§5.9)** — REST 오류는 Spring `ProblemDetail`에 서비스 오류 코드 필드 `code`(예: `DEVICE_NOT_READY`, `SESSION_STATE_CONFLICT`)를 더한 형식이다. 오류 코드 목록도 §5.9에 있다. 초안 어디에도 없으므로 Code Style에 추가를 제안한다.
4. **`@MockitoBean`·Jackson 3·`com.fasterxml.jackson.databind` 직접 사용 금지(§11.4)** — 초안 Code Style에는 있지만 `discovered-rules.md` 후보에는 `databind` 금지가 없다. Forbidden 후보로 올릴지 인터뷰에서 함께 확인하는 것을 제안한다.
5. **프런트엔드 조회 규칙(§1.2 ✅, §11.4)** — "`useEffect`+`fetch`+`setInterval`로 조회를 직접 만들지 않는다"는 ✅ 결정인데 `discovered-rules.md` 후보에 없다. Forbidden 후보 추가를 제안한다.
6. **Client Component 경계(§1.2 ✅)** — 마이크·WebSocket·SSE를 쓰는 컴포넌트는 Client Component로 둔다. 초안 Code Style에 없으므로 추가를 제안한다.

### 2. 이름 규칙

- 초안의 "TS는 camelCase, Python은 snake_case 등"은 org 기본값 문장을 옮긴 것인데, 이 프로젝트의 주 언어인 **Java가 빠져 있다**. Java 관례(클래스 PascalCase, 메서드·필드 camelCase, 상수 UPPER_SNAKE_CASE, 패키지 소문자)를 명시하는 것을 제안한다. 이것은 언어 관례이지 팀 결정이 아니므로 확인만 받으면 된다.
- 기술명세가 이미 쓰는 이름에서 보이는 접미사: `LiveVoiceAdapter`, `KnowledgeAdapter`(Adapter), `LlmClient`, `JudgeClient`(Client), `ScriptSpeaker`. **[추론]** 외부 시스템 경계 타입에 `Adapter`/`Client` 접미사를 쓰는 것으로 보이나 둘의 구분 기준(예: 교체 가능한 구현을 숨기는 포트 = Adapter, 단순 호출 래퍼 = Client)은 문서에 없다. 5명이 각자 다른 접미사를 쓰지 않도록 인터뷰에서 기준을 정하는 것을 제안한다. **[미정]**
- Java 기본 패키지 이름(예: `<그룹>.muzung.core...`)은 문서에 없다. Gradle 멀티 프로젝트를 처음 만들 때 바로 필요하다. **[미정]**
- 오류 코드는 §5.9 예시가 모두 `UPPER_SNAKE_CASE`이다. 새 오류 코드도 같은 형식을 따른다고 적을 수 있다(기존 예시의 일관성에서 나온 것이므로 확인 필요).
- 계약 DTO(`contracts/`) 이름과 JSON 필드 표기(camelCase / snake_case)는 문서에 없다. 기기 프로토콜과 서버 간 이벤트 스키마를 Java와 TS가 함께 쓰므로 먼저 정해야 한다. **[미정]**

### 3. 파일·저장소 구성

- **저장소 루트 불일치**: 기술명세 §2.4는 루트를 `muzung/`으로 그리고 "이중 폴더(`muzung/muzung/...`)를 만들지 않는다"고 한다. 현재 저장소 루트는 `finalproject_v1/`이며 `.claude/`, `aidlc/`, `.mcp.json`만 있다. 따라서 `backend/`·`frontend/` 등은 **현재 루트 바로 아래**에 만들어야 한다는 점을 Way of Working에 적는 것을 제안한다(새 `muzung/` 하위 폴더를 만들면 사실상 한 단계 더 깊어진다). 이 해석이 맞는지 인터뷰에서 확인이 필요하다.
- **`aidlc-docs/` vs `aidlc/`**: 기술명세 §2.4는 AI-DLC 산출물 위치를 `aidlc-docs/`로 적었지만 실제 프레임워크는 `aidlc/spaces/default/...`에 쓴다. 또 project.md 교정에 따르면 `docs/` 경로도 아직 없고 문서 원본은 `aidlc/spaces/default/knowledge/documents/`에 있다. 두 폴더(`aidlc-docs/`, `docs/`)를 만들지, 기술명세 구조를 실제 경로로 고칠지 정해야 한다. **[미정]**
- **`AGENTS.md`/`CLAUDE.md`**: 기술명세는 루트 `AGENTS.md`를 원본으로 두고 루트 `CLAUDE.md`가 `@AGENTS.md`를 가져오게 한다. 현재 AI-DLC 프레임워크는 `.claude/CLAUDE.md`를 쓴다. 두 파일은 함께 존재할 수 있으나 **[추론]** 규칙이 두 곳에 나뉘면 5명이 서로 다른 지시를 받을 위험이 있다. 루트 `CLAUDE.md`를 만들지, AGENTS.md 내용과 AI-DLC `team.md`의 관계(어느 쪽이 원본인지)를 정하는 것을 제안한다. **[미정]**
- **생성 코드 위치**: openapi-typescript 생성 파일을 "직접 고치지 않는다"(✅)는 있지만, 생성물을 어느 경로에 두고 저장소에 커밋하는지(빌드 때마다 생성 / 커밋)는 문서에 없다. 커밋 여부에 따라 병합 충돌 빈도가 달라진다. **[미정]**
- **`.gitignore` 공백**: 현재 `.gitignore`(AI-DLC 블록)에 `.env`, Gradle `build/`·`.gradle/`, Next.js `.next/` 항목이 없다. 기술명세 §2.4의 ".env를 저장소에 넣지 않는다"를 지키려면 첫 Bolt에서 추가해야 한다. 값은 표준 항목이므로 결정이 아니라 작업 항목으로 남기는 것을 제안한다(보안 측면 판단은 devsecops 검토 영역).

### 4. 포매터·린터

- TS: ESLint, Prettier(§1.2 🟡, 타당성 Q5에서 팀 확정). 설정 세부(예: 공유 설정 프리셋, Next.js 기본 ESLint 설정 사용 여부)는 문서에 없다. **[미정]** — 첫 Bolt에서 정해도 되는지 인터뷰에서 확인.
- **Java: 문서에 없다.** org 기본값은 "언어 기본"이라고만 한다. 5명이 각자 IDE 기본 포매터를 쓰면 공백·import 순서 차이로 diff가 커진다. 인터뷰 선택지 예시(결정은 팀이 한다):
  - A. Gradle Spotless + google-java-format
  - B. Gradle Spotless + palantir-java-format
  - C. 공유 `.editorconfig`/IDE 설정만 두고 자동 포매터 없음
  - D. 포매터 + Checkstyle(또는 다른 정적 분석)까지 CI에 넣음
  - X. Other (please specify)
  - 어느 것을 고르든 "CI에서 검사만 하고 실패 시 PR을 막는다" / "로컬에서 적용만 한다" 중 무엇인지도 함께 정해야 한다. 기술명세 §11.3의 "PR마다 lint"가 Java에도 적용되는지 문서로는 알 수 없다. **[미정]**
- Python(`ai/`): 초안과 같이 적용 여부만 묻는 것에 동의한다. 운영 서버가 아니므로 CI 대상에서 뺄 수도 있다.
- org.md Code Style은 "에이전트는 린터 설정을 먼저 읽고, 린터가 다루지 않을 때만 제안한다"고 한다. Java 린터가 정해지지 않으면 코드 생성 단계에서 에이전트 제안이 기준이 되므로 Construction 전에 정해 두는 것이 좋다.

### 5. 오류 처리

- REST 경계: `ProblemDetail` + `code`(§5.9). 공통 예외 처리기(`@RestControllerAdvice` 등) 하나로 모으는지, 모듈별로 두는지는 문서에 없다. **[미정]**
- 외부 연동 경계(GPT-Live, 제브, OpenAI LLM): 기술명세는 실패 시 대체 경로를 기능 수준에서 정했다(예: `CONTEXT_RESTORE_FAILED` 기술 종료, 시간 제한 대체 경로). **[추론]** Adapter가 SDK 예외를 그대로 던지면 "SDK 타입이 Adapter 밖으로 나가지 않는다" 규칙을 어기게 되므로, Adapter가 SDK 예외를 우리 예외 타입으로 바꿔 던진다는 규칙이 자연스럽게 따라온다. 이를 Code Style 규칙으로 명시할지 확인이 필요하다.
- 로그와 예외 메시지: §8·§11.4는 아동 발화 원문·전사, AI 요청·응답 본문을 로그에 남기지 않게 한다. **[추론]** 예외 메시지나 `ProblemDetail.detail`에 전사·AI 응답을 넣으면 예외 로그를 통해 이 규칙을 어길 수 있다. "예외 메시지에 아동 발화·AI 본문을 넣지 않는다"를 별도 규칙으로 둘지 인터뷰 질문으로 제안한다.
- 구현 단계 가드레일(`phases/construction.md`)은 "복구 가능 오류(재시도·대체)와 치명 오류(즉시 실패)를 구분"하라고 한다. 재전달·중복 제거(§4.5)가 이미 설계되어 있으므로 재시도 정책을 어디(relay 모듈 등)에 둘지는 설계 단계 몫이며 practices에서 정할 필요는 없다고 본다.

### 6. 브랜치·커밋 이름 충돌 (org Bolt vs 기술명세 Jira)

리드 초안이 짚은 충돌에 동의하며, 인터뷰에서 고를 수 있게 선택지를 구체화한다.

- 사실 관계
  - 기술명세 §1.2(✅): 브랜치 `feature/{JiraKey}-설명`, 커밋 `{JiraKey} type: 내용`, PR 리뷰 1명 + CI 통과 후 병합.
  - org 기본값: 트렁크 `main`, Bolt 브랜치를 `main`으로 스쿼시 병합, Bolt 하나 = `main` 커밋 하나(Bolt slug로 이름). 작업 브랜치 이름은 AI-DLC 도구가 정한다(리드 근거 표의 `bolt-<id8>_<slug>`).
  - 현재 브랜치는 `master`, 커밋 없음.
- 선택지 예시(결정은 팀이 한다)
  - A. Bolt 하나 = Jira 이슈 하나 = PR 하나. PR 브랜치는 `feature/{JiraKey}-설명`, PR 제목을 `{JiraKey} type: 내용`으로 쓰고 스쿼시 병합해 `main` 커밋 메시지가 Jira 형식을 따르게 한다(GitHub 저장소 설정에서 스쿼시 커밋 메시지 기본값을 PR 제목으로 둘 수 있다). AI-DLC가 만드는 Bolt 작업 브랜치와 PR 브랜치의 관계(같은 브랜치로 맞출지, Bolt 브랜치에서 PR 브랜치로 옮길지)는 도구 동작을 확인해야 한다.
  - B. Bolt 하나에 Jira 이슈 여러 개 — PR 하나에 Jira 키 여러 개. 커밋 형식 `{JiraKey}`가 하나만 들어가므로 대표 키 규칙이 필요하다.
  - C. 기술명세 형식만 쓰고 org 스쿼시 원칙은 따르지 않음(병합 커밋/리베이스). org 기본값과 어긋나므로 이유를 남겨야 한다.
  - X. Other (please specify)
- `{JiraKey} type: 내용`의 `type` 값 목록(예: feat, fix, refactor, test, docs, chore)은 문서에 없다. **[미정]**
- Jira 프로젝트 키(예: `MZ-123`의 접두사)도 문서에 없다. 브랜치 이름 검사를 CI나 훅에 넣으려면 필요하다. **[미정]**
- 트렁크 이름 `main` vs 현재 `master`: org 기본값과 Construction worktree 기준 브랜치가 `main`이므로, 첫 커밋 전에 `main`으로 맞추는 쪽이 비용이 가장 적다. 확인만 받으면 된다.

### 7. 5인 병렬 개발 관점에서 미리 정할 것

project.md 교정(Unit은 5인 병렬 개발이 가능하도록 나눈다)과 `intent-backlog.md`의 병렬 메모(P4·P6은 첫 주부터, P1·P5는 가짜 모델로 병렬 시작)를 기준으로, 동시에 여러 사람이 손대는 **공유 지점**에서 충돌이 날 수 있다. 각 항목은 규칙 값이 아니라 인터뷰 질문으로 제안한다.

1. **`contracts/`** — 기술명세는 "계약을 먼저 고치고 양쪽 서버를 같은 PR에서 맞춘다"고 한다. 두 서버를 다른 사람이 맡으면 한 PR이 두 사람 작업을 묶게 된다. **[추론]** 계약 변경 PR을 누가 열고 누가 리뷰하는지(예: 계약 담당 한 명, 또는 양쪽 서버 담당이 함께 리뷰)를 정해 두는 것이 좋다. **[미정]**
2. **Flyway 마이그레이션 버전 번호** — core·activity 스키마마다 Flyway를 쓴다(§11.3). 두 사람이 동시에 같은 순번(`V3__...`)을 만들면 병합 뒤 충돌한다. 순번 방식 / 날짜·시각 기반 버전(Flyway가 허용하는 형식) / 마이그레이션 담당자 지정 중 무엇을 쓸지 정해야 한다. **[미정]**
3. **OpenAPI 생성 타입** — 백엔드 담당이 API를 바꾸면 프런트엔드 생성물이 바뀐다. 생성물 커밋 여부(3절)와 §11.3의 🧪 "명세·생성 타입 불일치 CI 검사"가 이 문제와 직접 연결된다.
4. **`content/`(승인 문구·판정 지점)와 `AGENTS.md`** — 버전 관리되는 공유 파일이다. 수정 책임자가 문서에 없다. **[미정]**
5. **ArchUnit 규칙을 첫 Bolt에서 켜는지** — 병렬 개발 시작 전에 모듈 경계 검사가 CI에 있어야 사람마다 다른 방식으로 모듈을 넘나드는 일을 초기에 막을 수 있다. **[추론]** Walking Skeleton 질문(첫 조각의 범위)과 함께 묻는 것을 제안한다.

### 8. 인터뷰에 추가할 질문 (리드 질문 1~15에 더함)

리드 질문과 겹치지 않는 것만 적는다. ✅와 Q5에서 확정된 🟡 항목은 묻지 않는다.

- (Code Style) Java 포매터·린터와 CI 강제 여부 — 리드 질문 13을 4절 선택지로 구체화.
- (Code Style) Java 기본 패키지 이름.
- (Code Style) `Adapter`/`Client` 접미사 구분 기준.
- (Code Style) 계약 DTO의 JSON 필드 표기(camelCase / snake_case).
- (Code Style) Adapter가 SDK 예외를 우리 예외로 바꾸는 규칙, 예외 메시지에 아동 발화·AI 본문 금지 규칙을 강한 규칙으로 둘지.
- (Code Style) openapi-typescript 생성물 위치와 커밋 여부.
- (Way of Working) 저장소 루트 해석(현재 루트 바로 아래에 `backend/` 등), `aidlc-docs/`·`docs/` 처리, 루트 `AGENTS.md`/`CLAUDE.md`와 AI-DLC `team.md`의 원본 관계.
- (Way of Working) 커밋 `type` 목록, Jira 프로젝트 키.
- (Way of Working) `contracts/` 변경 PR 책임자, Flyway 버전 번호 방식, `content/`·`AGENTS.md` 수정 책임자.

## Positions

- AGREE: 기술명세 §1.2·§11.4의 ✅ 항목과 타당성 Q5에서 확정된 🟡 항목(ESLint·Prettier, Java 21·Spring Boot 4.1.x, 저장소 구조·모듈 포함)을 팀 사실로 보고 다시 묻지 않는 리드 판단에 동의한다.
- AGREE: Java 포매터·린터를 기술명세에 없는 [미정] 항목으로 남기고 인터뷰에서 정하게 한 것에 동의한다.
- AGREE: 기술명세 브랜치·커밋 형식과 org Bolt 스쿼시·브랜치 형식을 충돌로 표시하고 추론으로 메우지 않은 것에 동의한다.
- OBJECT: Code Style 절의 이름 규칙이 org 기본값 문장(TS·Python)을 그대로 옮겨 주 언어인 Java를 빠뜨렸다. Java 관례를 명시해야 한다.
- OBJECT: ArchUnit 네 가지 구조 규칙(§2.5)과 오류 형식 `ProblemDetail` + `code`(§5.9)가 Code Style 절에 없다. 계층·모듈 경계와 오류 처리는 코드 스타일의 핵심이므로 Code Style에도 적고, 구조 규칙은 `discovered-rules.md` Forbidden 후보에도 올려야 한다.
- OBJECT: `discovered-rules.md` 후보 목록에 기술명세 ✅/§11.4 규칙 중 `com.fasterxml.jackson.databind` 직접 사용 금지와 `useEffect`+`fetch`+`setInterval` 조회 금지가 빠져 있다. 후보로 올린 뒤 인터뷰에서 강·약을 정해야 한다.
- OBJECT: 기술명세 §2.4 저장소 구조의 `aidlc-docs/`·`docs/`·루트 `muzung/`가 실제 저장소(`finalproject_v1/` 루트, `aidlc/` 작업 공간)와 다르다는 점이 초안에 없다. Way of Working에 확인 항목으로 넣어야 한다.
