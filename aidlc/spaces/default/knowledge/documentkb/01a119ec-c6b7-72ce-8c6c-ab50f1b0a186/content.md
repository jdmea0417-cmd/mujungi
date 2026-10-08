# 무중 기술명세서 v0.2

2026-10-08 · 기준 문서: 화면 설계 최종안 v1.5 · 요구사항 명세서 v1.4 · 기능명세서 v1.3

> 이 문서는 요구사항 명세서 v1.4를 구현하기 위한 시스템 구조, 서비스 경계, 인터페이스, 데이터 모델, 처리 방식과 기술 기본값을 정의한다. 요구사항 ID(FR·NFR·D-CON·AC-CON), 기능 ID(기능명세서), 화면 ID(화면 설계)는 원 문서의 것을 그대로 쓴다.
>
> 요구사항 명세서와 기능명세서에서 'ADR'로 넘긴 결정과 값은 이 문서가 관리한다(값은 §13, 미결은 §14).
>
> 문서끼리 다르면 화면 설계 → 요구사항 명세서 → 기능명세서 → 이 문서 순으로 앞의 문서를 따른다.
>
> GPT-Live 지시문 원문, 승인 문구 원문, 판정 질문 문안, 분석 AI 프롬프트는 AI 동작 명세에서 관리한다(§1.3).

상태 표시

| 표시 | 뜻 |
|---|---|
| ✅ | 확정 (팀 결정 또는 요구사항 확정 구조) |
| 🟡 | 제안 — 팀 확인 필요 |
| 🧪 | 첫 볼트·PoC 결과로 정함 |
| ⬜ | 미정 |
| ⏱ | 임시값 — 바꿀 수 있음 |

## 1. 결정 사항과 기술 스택

### 1.1 아키텍처

백엔드는 Spring Boot 서비스 두 개로 나눈다. ✅ (요구사항 확정 구조, D-CON-01)

| 서비스 | 맡는 일 | 하지 않는 일 |
|---|---|---|
| 보호자 서버 core-api | 부모 앱 API, 구글 로그인·세션, 계정·아동·동의·종료 표현, 기기 등록·기기 토큰 발급·활동 배정, 활동 시작·종료 요청 전달, 기록 통합 저장(원음·전사·상태 이력·진행용 판정 기록), 부모 앱으로 실시간 전사 중계, 분석 AI 호출(장면 대본·결과·생활 팁·목표 후보·리포트), 대본 사전 점검(J7) 호출, 푸시 발송 | 진행 중 대화 상태를 판정하지 않는다. 아동 서비스 상태를 받지 못했다는 이유만으로 대화 중단·종료를 판정하지 않는다 |
| 아동 서비스 realtime-activity | 기기 연결 종단, GPT-Live 음성 중계, 진행 중 세션 상태 기계, 판정 턴(자동 응답 보류·제브 호출·다음 행동 지시), 종료 의도 인식, 음성 대화 장애 판정·기술적 일시 정지·복구·기술 종료, 종료 확인, 오디오 공백 측정, 보호자 서버로 상태·기록·전사 전달(실패 시 보관 후 재전달) | 기록의 최종 저장소가 아니다. 부모 앱과 직접 통신하지 않는다 |

이 구조로 얻는 것

- 아동 서비스가 기기 연결과 음성 모델 연결의 끝점이므로 음성 대화 장애를 직접 확인한다. 보호자 서버나 부모 앱이 멈춰도 대화와 종료 확인을 이어 간다.

- 오래 열린 음성 연결과 지연에 민감한 상태 기계를, 계정·기록·분석을 맡는 서버와 따로 배포하고 따로 장애를 다룬다.

- 두 서비스 모두 Java·Spring이라 팀이 익힌 기술 하나로 만든다.

감수하는 것

- 서비스 사이 계약(§5.3·§5.4), 재전달, 중복 제거가 필요하다.

- 이를 줄이기 위해 기록 저장은 보호자 서버 한 곳, 진행 중 상태 결정은 아동 서비스 한 곳에만 둔다.

### 1.2 기술 스택

| 영역 | 선택 | 상태 | 비고 |
|---|---|---|---|
| 저장소 | GitHub 조직, Monorepo | ✅ | §2.4 |
| 업무 관리 | Jira Kanban + GitHub 연동 | ✅ | 브랜치 feature/{JiraKey}-설명, 커밋 {JiraKey} type: 내용, PR 리뷰 1명 + CI 통과 후 병합 |
| 부모 앱·아동 기기 웹 | React + Next.js 한 애플리케이션. /parent(부모 앱)와 /device(아동 기기 웹)를 경로로 나눔 | ✅ | 마이크·WebSocket·SSE를 쓰는 컴포넌트는 Client Component |
| 프론트엔드 언어·도구 | TypeScript, ESLint, Prettier, Vitest | 🟡 |  |
| UI 라이브러리 | — | ⬜ | 화면 담당자가 후보 제안 |
| 프론트엔드 서버 상태 | TanStack Query | ✅ | useEffect+fetch+setInterval로 조회를 직접 만들지 않는다 |
| API 타입 | openapi-typescript (springdoc 명세에서 생성) | ✅ | 생성 파일은 직접 고치지 않는다. openapi-fetch 사용 여부 🟡 |
| 백엔드 언어·프레임워크 | Java 21, Spring Boot 4.1.x, Gradle 멀티 프로젝트 | 🟡 | 부캠 환경·라이브러리와 맞지 않으면 Spring Boot 3.5.x |
| 보호자 인증 | Spring Security OAuth2 Login(구글) + 서버 세션. JWT 없음 | ✅ | 세션 저장소 Spring Session JDBC 🟡 |
| API 문서 | springdoc 3.x | 🟡 | Spring Boot 4용 |
| 관계형 DB | PostgreSQL 16 이상, 스키마 core(보호자 서버)·activity(아동 서비스) | 🟡 | Flyway, JPA ddl-auto=validate 🟡 |
| 지식 저장소 | Neo4j 5 — 화용 지식 전용, 아동 개인 기록 없음 | 🟡 | KnowledgeAdapter로 PostgreSQL 구현과 교체 가능 |
| 실시간 음성 모델 | GPT-Live(gpt-live-1), 아동 서비스가 서버 간 WebSocket으로 연결 | ✅ | 판정 턴 보류 방식 🧪 (§3.3) |
| 진행 중 판정 모델 | 제브(Jev, TypeSafe AI), REST | ✅ | 접근권·한국어 정확도·확신 문턱 🧪 (§5.7) |
| 분석 AI | OpenAI 텍스트 LLM, Responses API, 구조화 출력 | ✅ 공급자 / 🧪 작업별 모델 | §7.4 |
| AI SDK | openai-java (OpenAIClient Bean 직접 등록, OpenAI Spring Boot starter 사용 안 함) | ✅ | GPT-Live 지원 범위 🧪 |
| 음성 인식·합성 | 별도 STT·TTS 없음 | ✅ | 아이가 듣는 음성은 GPT-Live 목소리 하나 |
| 실시간 전달 | 기기↔아동 서비스: 연결 하나로 음성·상태·명령 ✅, WebSocket 🟡 / 아동 서비스→보호자 서버: 내부 WebSocket + 재전달 🟡 / 보호자 서버→부모 앱: SSE 🟡 |  | §5 |
| 푸시 | Web Push(VAPID) + 서비스 워커 | 🟡 | 부모 앱은 PWA 매니페스트를 둔다. Java 라이브러리 🧪 |
| 긴 작업 | Spring 내부 워커 + DB 작업 테이블. Kafka·RabbitMQ·Redis 큐 없음 | ✅ 방향 | §7.6 |
| PDF | 서버에서 HTML 템플릿 → PDF 생성, 한글 글꼴 포함 | 🟡 | 후보: OpenHTMLtoPDF + Noto Sans KR |
| 시험 | JUnit 5, Testcontainers, ArchUnit, Vitest | 🟡 | §10 |
| CI / CD | GitHub Actions / 수동 deploy.sh | 🟡 |  |
| 실행 환경 | AWS EC2 1대(서울), Docker Compose, Nginx, DB는 컨테이너(RDS 없음) | ✅ | HTTPS 방식 ⬜ (§11) |
| Python | ai/ 폴더: PoC·평가 실험·데이터 가공. 운영 서버 아님 | 🟡 |  |

### 1.3 이 문서 밖에서 정하는 것

| 문서 | 내용 |
|---|---|
| AI 동작 명세 | GPT-Live 기본 지시문과 다음 행동 지시 문안, 승인 문구 원문(시작 안내·마무리·종료 의도 확인), 판정 지점별 질문 문안, 분석 AI 프롬프트·출력 스키마, 금지 표현 사전, 생활 팁 검사 기준, 기능명세서 §9.1의 빈칸(G-03·06·08·09·16) |
| 테스트 계획서 | 시나리오·장애 주입 절차와 환경 |
| WBS | 일정·담당 |

## 2. 시스템 구성

### 2.1 구성도

부모 앱 (/parent) ── HTTPS REST · SSE ──▶ Nginx ──▶ 보호자 서버 (core-api)

아동 기기 (/device) ── WSS 기기 연결 ─────▶ Nginx ──▶ 아동 서비스 (realtime-activity)

보호자 서버 ◀── 내부 채널 (명령 · 이벤트 · 전사) ──▶ 아동 서비스

아동 서비스 ── WSS 음성 모델 연결 ──▶ GPT-Live

아동 서비스 ── HTTPS ──▶ 제브 (J1~J6)

아동 서비스 ──▶ PostgreSQL activity 스키마 (진행 중 상태 · 전달 대기, 임시)

보호자 서버 ── HTTPS ──▶ 분석 AI (OpenAI) · 제브 (J7) · Web Push

보호자 서버 ──▶ PostgreSQL core 스키마 · 원음 파일 볼륨 · Neo4j (화용 지식)

※ 모두 EC2 한 대의 Docker Compose 안에서 돈다 (외부 API 제외)

- 부모 앱은 보호자 서버하고만 통신한다.

- 아동 기기는 등록 뒤 아동 서비스하고만 통신하며 GPT-Live에 직접 연결하지 않는다. 예외로 등록 전(D0)의 등록 코드 발급과 기기 토큰 수령은 보호자 서버의 기기용 API(/device-api)를 쓴다(DEV-REG-001).

- 두 서버는 서로의 DB 스키마를 읽거나 쓰지 않는다. 필요한 데이터는 내부 채널로 주고받는다.

- 내부 채널과 /internal/\*\*은 Nginx 밖으로 노출하지 않는다.

### 2.2 연결과 서버 간 전달

| 연결 | 구간 | 전송 | 장애 판정 | 장애 시 |
|---|---|---|---|---|
| 기기 연결 | 아동 기기 ↔ 아동 서비스 | WSS 하나 (음성은 바이너리 프레임, 제어는 JSON) | 아동 서비스 | 기술적 일시 정지 → 같은 기기 토큰·같은 세션의 재연결 대기 → 실패하면 기술 종료. 기기는 재생을 스스로 멈춘다 |
| 음성 모델 연결 | 아동 서비스 ↔ GPT-Live | WSS (/v1/live/sessions) | 아동 서비스 | 위와 같음. 다시 연결할 때의 맥락 복원 🧪 |
| 부모 앱 연결 | 부모 앱 ↔ 보호자 서버 | HTTPS REST + SSE | — | 활동은 계속된다. 앱에 "실시간 상태를 불러오지 못하고 있어요"와 마지막 확인 시각·상태 |
| 서버 간 전달 (연결 3종 아님) | 아동 서비스 → 보호자 서버 | 내부 WebSocket + 재전달 | 보호자 서버 | 내부 상태 '기기 상태 확인 불가'(04·05 '응답 없음', 12·21 '잠깐 끊김'). 활동은 계속되고 아동 서비스가 보관 후 재전달 |
| 외부 AI 호출 (연결 3종 아님) | 아동 서비스·보호자 서버 → 제브·분석 AI | HTTPS | — | 연결 장애로 보지 않는다. 제브 시간 초과는 판정 보류(§3.2) |

기기 연결과 음성 모델 연결을 합쳐 음성 대화 연결이라 하고, 둘 중 하나의 장애를 음성 대화 장애라 한다. '관리 연결만 장애'인 상태는 없다.

### 2.3 데이터 소유권

| 데이터 | 저장 위치 | 비고 |
|---|---|---|
| 계정·동의·아동·종료 표현·기기 등록·푸시 구독 | 보호자 서버 core |  |
| 세션·상태 이력·종료 3축·연결 관측·끊김 구간 | core | 아동 서비스가 전달 |
| 전사·재생 기록·제공한 도움·진행용 판정 기록 | core | 아동 서비스가 전달 |
| 아동 원음 | 보호자 서버 파일 볼륨 + core 메타데이터 | 원음 보관 동의(필수). 무중 출력 음성은 저장하지 않는다 |
| 목표 후보·담은 목표·장면 대본·장면 기록·기록용 재판정·결과·생활 팁·리포트 | core |  |
| 진행 중 세션 상태·전달 대기 데이터 | 아동 서비스 activity | 전달 확인 뒤 지운다. 보관 한계 §13 |
| 화용 지식(행동·핵심 기능·상황 유형·사회적 단서·지원 방법) | Neo4j | 아동 개인 기록을 두지 않는다 |
| 승인 문구·판정 지점 선택지 | 저장소 content/ (버전) | 세션마다 버전을 고정한다 |
| 시작 전 미참여 세션 | 저장하지 않음 | 종료 확인을 받아 부모 앱에 알린 뒤 세션과 하위 데이터를 지운다 🟡 (§4.7) |

### 2.4 저장소 구조 🟡

muzung/

├── backend/ \# Gradle 멀티 프로젝트

│ ├── core-api/ \# 보호자 서버

│ ├── realtime-activity/ \# 아동 서비스

│ └── contracts/ \# 서버 간 명령·이벤트 DTO, 기기 프로토콜 스키마

├── frontend/ \# Next.js: /parent, /device

├── content/ \# 승인 문구, 판정 지점·선택지 (버전 관리)

├── ai/ \# Python: PoC·평가·데이터 가공 (운영 서버 아님)

├── infra/ \# compose.yml, compose.dev.yml, nginx, deploy.sh

├── docs/ \# 화면 설계·요구사항·기능명세·기술명세 (Markdown)

├── aidlc-docs/ \# AI-DLC 산출물

├── AGENTS.md \# AI 개발 규칙 원본

└── CLAUDE.md \# @AGENTS.md

- 이중 폴더(muzung/muzung/...)를 만들지 않는다.

- contracts/가 두 서버 사이 계약의 기준이다. 계약을 바꿀 때는 이 모듈을 먼저 고치고 양쪽을 같은 PR에서 맞춘다.

- 실제 아동 자료, 실제 대화·전사, API 키, .env, 인증서 개인 키를 저장소와 AI-DLC 입력에 넣지 않는다.

### 2.5 서버 내부 모듈 🟡

| 서버 | 모듈 |
|---|---|
| core-api | auth, guardian, consent, child, device, session, record, goal, practice(장면 대본), result, report, push, content, knowledge, ai(LlmClient·JudgeClient) |
| realtime-activity | devicelink, voice(LiveVoiceAdapter), runtime(세션 상태 기계), judge(JudgeClient), relay, observe(연결 관측·공백 측정) |

- ArchUnit으로 막는다: 모듈 순환 의존, 다른 모듈 Repository 직접 접근, Controller → Repository 직접 접근, 외부 SDK 타입이 Adapter 밖으로 나가는 것.

## 3. 음성 대화 파이프라인

### 3.1 방식

- 아이 음성은 아동 기기 → 아동 서비스 → GPT-Live로 가고, GPT-Live의 음성은 반대 방향으로 돌아온다. ✅

- 별도 STT·TTS를 두지 않는다. 아이 발화 전사는 GPT-Live 입력 전사, 무중 발화 전사는 GPT-Live 출력 전사를 쓴다. ✅

- 승인 문구와 세션 고정 대본(기회 문장)도 GPT-Live가 원문을 낭독하고, 아동 서비스가 출력 전사를 원문과 대조한다. ✅ 방향 / 문구별 일치율 🧪

- 오디오 형식은 전 구간 PCM16 mono 24kHz로 맞춘다(GPT-Live 기본값, 중간 리샘플링 없음). 🟡

### 3.2 일반 턴과 판정 턴

일반 턴

아이 음성 → 아동 서비스 → GPT-Live → 응답 음성 → 아동 서비스(출력 게이트 통과) → 기기 재생

판정 턴 — 판정 지점 J1~J6에 해당하는 아이 차례(참여 질문 뒤, 기회 문장 직후, 되묻기 뒤, 도움 동의 질문 뒤, 집중 연습 뒤 다시 해 보기, 키워드가 아닌 종료 의도 후보)

1. 아동 서비스가 출력 게이트를 닫는다. GPT-Live 자동 응답을 기기로 보내지 않는다(§3.3). 기기 화면은 '생각 중'.

2. 아이 발화의 입력 전사를 받고 발화 끝을 판정해 전사를 확정한다(§3.7). 전사 제한 시간을 넘기면 판독 불가.

3. 기본·등록 종료 표현과 일치하면 판정 없이 종료 처리한다(§3.5).

4. 제브에 판정을 요청한다(§7.3). 제한 시간을 넘기면 판정 보류.

5. 확신 문턱을 확인하고 상태 기계가 다음 행동을 정한다(§4.3).

6. GPT-Live에 다음 행동 지시를 넣는다(session.instructions.append). 지시 뒤의 응답만 게이트를 통과한다.

7. 아이 말이 끝난 뒤 10초 ⏱ 안에 응답이 준비되지 않으면 판정 보류로 보고 무난한 되묻기나 맞장구를 지시한다.

8. 판정마다 진행용 판정 기록 한 건을 남긴다(§7.3).

- 판정 턴 중 종료 표현이 나오면 판정을 멈추고 종료 처리한다.

- 판정 턴 중 음성 대화 장애가 나면 그 판정을 버리고 판정 기록에 '장애 취소'로 남긴다.

- 제한 시간이 지난 뒤 도착한 전사·판정은 진행에 쓰지 않고 판정 기록에만 남긴다.

- 활동이 끝나면 분석 AI가 전체 전사로 다시 판정해 기록을 확정한다(기록용 재판정, §7.4).

### 3.3 출력 게이트 — 판정 턴의 자동 응답 보류 🧪

GPT-Live 공개 문서(2026-10 기준)에서는 자동 응답을 끄는 설정을 찾지 못했고, 사용자가 듣는 음성을 막으려면 중계하는 쪽이 출력 음성을 버려야 한다고 안내한다. 아동 서비스가 그 중계 지점이므로 다음과 같이 보류한다.

- 아동 서비스는 기기로 보내는 출력마다 outputId와 세션의 epoch를 붙인다.

- 게이트가 닫힌 동안 받은 session.output_audio.delta는 기기로 보내지 않고 버린다. 그 출력 전사는 '보류된 출력'으로 진행용 판정 기록에만 남기고, 대화 기록과 부모 앱 중계에는 넣지 않는다(아이가 듣지 않은 말이다).

- 다음 행동 지시는 "하던 말을 멈추고 ~하라" 형식으로 넣는다. 문안은 AI 동작 명세.

첫 볼트에서 확인한다.

- 버린 응답이 GPT-Live 대화 맥락에 남아 다음 응답을 흐리는 빈도

- 지시 주입부터 첫 음성까지의 지연과 10초 상한 안에 드는지

- 기준에 못 미칠 때의 대안 🟡: OpenAI Realtime API(gpt-realtime, 자동 응답 생성을 끄는 턴 감지 설정이 있음). 요구사항의 확정 구조(GPT Live)를 바꾸는 일이므로 요구사항 변경이 먼저 필요하다. 어느 쪽이든 LiveVoiceAdapter 뒤에 두어 상태 기계는 바뀌지 않게 한다.

### 3.4 출력 종류

| 종류 | 내용 | 출처 | 출력 방식 |
|---|---|---|---|
| 승인 문구 | 시작 안내(자유대화·연습), 마무리, 종료 의도 확인 | content/ 승인 원문, 세션마다 버전 고정 | GPT-Live 원문 낭독 + 출력 전사 대조 |
| 세션 고정 대본 | 장면 설명, 기회 문장, 상황 다시 설명 | 분석 AI가 만들고 자동 확인·J7을 통과한 대본 | 같음 |
| 지시 기반 생성 | 되묻기, 집중 연습(코치 역할, 대본의 요점·표현 예시를 받아 생성), 다시 말해 달라는 확인, 무난한 되묻기·맞장구, 역할 전환 알림 | 아동 서비스의 다음 행동 지시 | GPT-Live 실시간 생성 |
| 자유 생성 | 자유대화 질문·후속 질문, 장면 속 상대 역할의 일반 대사 | GPT-Live 기본 지시문 + 주제 힌트·장면 대본 | GPT-Live 자동 응답 |
| 고정 안내 | 음성 대화 장애 때 아이에게 들려줄 안내 | 결정 대기(D-CON-09, TD-01) | 결정 전에는 제공하지 않는다. 제공하지 않으면 부모 앱이 확인된 상황과 아이에게 알릴 필요를 안내한다(VOICE-NOTICE-001) |

- 원문 대조가 어긋날 때의 처리(다시 낭독, 해당 문구만 사전 녹음 클립으로 전환)는 첫 볼트 결과와 AI 동작 명세로 정한다.

- 사전 녹음 클립은 TTS가 아니라 GPT-Live 목소리로 만든다. 고정 안내를 제공하기로 정하거나 원문 대조 일치율이 낮은 문구가 생길 때만 만든다.

- 아이에게 들리는 승인 문구·고정 대본은 ScriptSpeaker 경계를 거친다(구현: GPT-Live 낭독 / 클립 재생). 문구별 출력 방식을 바꿔도 상태 기계는 바뀌지 않는다.

### 3.5 종료 의도 인식 (FR-E05)

1. 아이 발화의 입력 전사를 정규화(공백·문장부호 제거)해 기본 표현 '무중아 그만할래'와 그 아동의 추가 종료 표현(세션 시작 때 고정)과 비교한다. 일치하면 확인 없이 종료한다. 제브를 부르지 않는다.

2. 일치하지 않아도 종료 의도 후보이면 J6으로 판정한다(종료 명확 / 불명확 / 종료 아님 / 휴식 요청). 불명확하면 승인된 확인 문구로 짧게 묻는다. J6을 돌릴 발화 범위와 일반 턴 자동 응답과의 순서는 G-16(AI 동작 명세).

3. 휴식 요청은 종료가 아니다. 세션 이벤트 REST_REQUESTED로 기록하고 대화를 이어 간다(공백 종료 사유 판정에 씀).

4. 무중 출력이 재생되는 동안 들어온 입력 전사가 직전 출력 전사와 비슷하면 아이 발화로 쓰지 않는다(재생 음성을 종료 요청으로 오인하지 않음). 🟡

5. 종료가 확정되면 §4.4 순서로 마무리한다.

### 3.6 출력 중단과 재생 기록 (FR-F14, NFR-06)

- 세션마다 정수 epoch를 둔다. 기술적 일시 정지, 종료 처리 시작, 판정 턴의 출력 폐기 때 epoch를 1 올리고 기기에 output.stop{epoch}을 보낸다.

- 기기는 현재 epoch보다 낮은 음성을 즉시 버리고 재생 중인 음성도 멈춘다.

- 기기 연결이 끊기면 기기가 스스로 재생을 멈추고 대기열을 비운다. 다시 연결돼도 끊기기 전 출력을 이어 재생하지 않는다.

- 기기는 출력마다 playback{outputId, started\|ended\|stopped, 시각}을 보고한다. 아동 서비스는 기회 문장의 재생 구간을 기록하고, 질문 핵심이 재생되지 않았으면 그 기회를 무효로 한다(아동 실패로 기록하지 않음).

- 무응답 대기는 실제 재생이 끝난 시각부터 잰다.

### 3.7 발화 끝 판정과 오디오 공백 측정 🧪

- 발화 끝: GPT-Live 공개 문서에서는 입력 전사 완료 이벤트를 찾지 못했다. 아동 서비스가 기기에서 받은 음성에 음성 구간 검출(VAD)을 적용하고, 무음이 일정 시간 이어지고 입력 전사 delta가 멈추면 전사를 확정한다. 무음 길이는 §13.

- 오디오 공백(FR-F18): 같은 VAD로 아동 서비스가 기기에서 받은 아이 음성의 마지막 시각을 갱신해 연속 공백을 잰다.

  - 기기 연결·음성 모델 연결이 정상이고 활동이 진행 중(ACTIVE)일 때만 잰다.

  - 연속 4분 → 내부 공백 상태(화면 표시 없음), 추가 2분 → 자동 종료. 공백 전 휴식 요청이 없으면 '대화 거부', 있으면 '비정상 오디오 공백 지속'.

  - 기술적 일시 정지·상태 미확인 구간은 넣지 않는다. 연속성을 증명할 수 없는 구간을 이어 붙이지 않는다.

  - 부모 앱 닫힘·조회 실패는 측정에 영향을 주지 않는다.

  - 상태 전환 때 타이머를 초기화·재시작하는 방식은 TD-12(D-CON-10).

### 3.8 에코 제거 🧪

- 기기 브라우저는 getUserMedia({ audio: { echoCancellation: true, noiseSuppression: true, autoGainControl: true } })로 마이크를 연다.

- 아동 서비스는 §3.5의 4번 규칙으로 재생 음성 재유입을 거른다.

- PoC로 노트북·태블릿 스피커의 재유입률을 확인한다. 부족하면 재생 중 입력을 GPT-Live로 보내지 않는 방식(session.input_audio.mute)을 검토한다. 이 경우 아이가 무중의 말을 끊을 수 없다.

### 3.9 연결 감시

| 대상 | 감시 방법 | 판정 주체 |
|---|---|---|
| 기기 연결 | WebSocket ping/pong + 앱 수준 heartbeat, 소켓 종료·오류 | 아동 서비스 |
| 음성 모델 연결 | 소켓 종료·오류, error 이벤트, 일정 시간 이벤트 없음 | 아동 서비스 |
| 서버 간 전달 | 내부 채널 heartbeat, 마지막 수신 시각 | 보호자 서버 |
| 부모 앱 연결 | SSE 끊김, 요청 실패 | 부모 앱 |

- 판정 시간·상태 유효시간·복구 대기는 §13(D-CON-03~05).

- 아동 기기의 「끄기」는 기기 연결 끊김으로 처리한다(FR-E16).

- 불명확 발화·침묵·마이크 문제·AI 지연·제브 지연을 연결 장애로 판정하지 않는다.

## 4. 세션 상태 모델

### 4.1 상태 축

한 값에 모든 의미를 담지 않는다. 한 축으로 다른 축을 추정하지 않는다(예: 아동 중단 + 부분 결과 + 관련 경험 미확인).

| 축 | 값 (코드) | 소유 |
|---|---|---|
| 활동 상태 status | SCENE_PREPARING 상황 준비 중(연습만) / REQUESTED 시작 요청됨 / DEVICE_CONFIRMING 기기 실행 확인 중 / PREPARING 안내·참여 확인 / ACTIVE 진행 / TECH_PAUSED 기술적 일시 정지 / END_REQUESTED 종료 요청됨 / ENDING 종료 처리 중 / ENDED 종료 확인됨 / CANCELLED 시작 전 취소 | 진행 중에는 아동 서비스가 정하고 보호자 서버가 저장 |
| 요청 종류 request_kind | END 종료 / CANCEL 취소(참여 전) — END_REQUESTED·ENDING에서 씀 | 보호자 서버 |
| 취소 종류 cancel_kind | SCENE_CANCELLED 상황 준비 중 취소(21) / SCENE_FAILED 장면 대본 두 번 실패 / START_TIMEOUT 처음 기기 응답 없음 / PREP_CANCELLED 참여 전 준비 취소 | 보호자 서버 |
| 종료 사유 end_reason | COMPLETED 정상 마무리 / NOT_PARTICIPATED 시작 전 아동 미참여(저장 안 함) / CHILD_STOPPED 아동 중단 / GUARDIAN_STOPPED 보호자 중단 / TECHNICAL 기술 문제 / CONVERSATION_REFUSED 대화 거부 / ABNORMAL_SILENCE 비정상 오디오 공백 지속 / CONSENT_WITHDRAWN 동의 철회 | 아동 서비스가 확정, 보호자 서버가 저장 |
| 기술 문제 원인 tech_cause | DEVICE_LINK / VOICE_MODEL_LINK / CONTEXT_RESTORE_FAILED / UNKNOWN | 아동 서비스 |
| 결과 상태 result_status | PENDING 준비 중 / FULL 전체 결과 / PARTIAL 부분 결과 / INSUFFICIENT 자료 부족·결과 없음 / FAILED 생성 실패 | 보호자 서버 |
| 부족·미실시 사유 shortfalls\[\] | EXPERIENCE_UNCONFIRMED 관련 경험 미확인 / UNREADABLE 판독 불가 / JUDGMENT_HELD 판정 보류 / SCENE_NOT_DONE(사유: TIME_UP 등) | 보호자 서버 |
| 연결 관측 | 기기 연결 UP/DOWN, 음성 모델 연결 UP/DOWN, 서버 간 전달 RECEIVING/NOT_RECEIVING(기기 상태 확인 불가), 대상별 마지막 확인 시각 | 기기·음성 모델: 아동 서비스 / 서버 간: 보호자 서버 / 앱 조회: 부모 앱 |
| 연습 진행 | 현재 목표(세션 계획 순서), 장면 번호·종류(기본/변형), 대화 역할(친구/코치), 턴 모드(일반/판정/보류), 단계(§4.3) | 아동 서비스 |
| 종료 확인 | 종료 요청 시각, 아동 서비스 수신 시각, 종료 확인 시각, 확보 기록 범위 | 보호자 서버 |

화면 표기 대응: END_REQUESTED·ENDING → '끝내는 중', 확인 대기 시간을 넘긴 END_REQUESTED·ENDING → '종료 확인 안 됨', TECH_PAUSED와 서버 간 미수신 구간 → 12·21 '잠깐 끊김', 서버 간 미수신 → 04·05 '응답 없음', end_reason이 COMPLETED가 아닌 기록 → 26 '중간에 끊김'.

### 4.2 활동 상태 전이

| 전이 | 주체 | 조건 | 남기는 것 |
|---|---|---|---|
| (없음) → SCENE_PREPARING | 보호자 서버 | 연습 시작 요청, 시작 조건 충족(§4.6) | 목표 순서 고정 |
| SCENE_PREPARING → REQUESTED | 보호자 서버 | 모든 목표의 장면 대본이 자동 확인·J7 통과 | 장면 대본 |
| SCENE_PREPARING → CANCELLED | 보호자 서버 | 21 「취소」(SCENE_CANCELLED) 또는 대본 두 번 실패(SCENE_FAILED, 18에 이유) | 기기에 아무것도 보내지 않음. 기록 아님 |
| (없음) → REQUESTED | 보호자 서버 | 자유대화 시작 요청 수락 | 배경 정보 |
| REQUESTED → DEVICE_CONFIRMING | 아동 서비스 | 시작 명령 수신, 기기에 배정 전송 |  |
| DEVICE_CONFIRMING → PREPARING | 아동 서비스 | 기기가 시작 안내 재생 시작을 보고 | 기기 실행 확인 시각 |
| REQUESTED·DEVICE_CONFIRMING → END_REQUESTED(CANCEL) | 보호자 서버 | 처음 기기 응답 없음 판정 시간 초과. 아동 서비스에 취소 명령 | 앱은 08·18에 '시작하지 못함' |
| PREPARING → ACTIVE | 아동 서비스 | J1 '참여' |  |
| PREPARING → ENDING (NOT_PARTICIPATED) | 아동 서비스 | 거부 → 대안 1회 → 다시 거부, 또는 휴식 요청 없는 무응답 | 마무리 문구 1회 |
| PREPARING → END_REQUESTED(CANCEL) | 보호자 서버 | 참여 전 P1 끝내기, 동의 철회, 계정 삭제 🟡. 참여와 겹치면 보호자 서버에 먼저 도착한 쪽 |  |
| PREPARING·ACTIVE → TECH_PAUSED | 아동 서비스 | 음성 대화 장애 확인 | 원인, 출력 중지 확인 여부, 멈추기 전 상태 |
| TECH_PAUSED → 멈추기 전 상태 | 아동 서비스 | 복구 성공 + 재개 조건(§13 D-CON-07) | 끊김 구간 |
| ACTIVE·TECH_PAUSED → END_REQUESTED(END) | 보호자 서버 | P1 끝내기, 동의 철회, 계정 삭제 | 요청 시각 |
| END_REQUESTED → ENDING | 아동 서비스 | 종료·취소 명령 수신 | 수신 시각 |
| ACTIVE → ENDING | 아동 서비스 | 정상 마무리 / 아동 종료 의도 / 공백 자동 종료 |  |
| TECH_PAUSED → ENDING | 아동 서비스 | 복구 실패·복원 불가 | TECHNICAL + 원인 |
| ENDING → ENDED | 아동 서비스 | 출력 중지 + GPT-Live 세션 종료 확인 = 종료 확인 | 종료 사유, 확인 시각, 확보 기록 범위 |
| ENDING(CANCEL) → CANCELLED | 아동 서비스 | 안내 중지 + GPT-Live 세션 종료 확인 | START_TIMEOUT 또는 PREP_CANCELLED, 시각. 그동안의 전사·원음·판정 기록은 지운다 |
| ENDED → 결과 정리 | 보호자 서버 | 종료 확인 수신(동의 철회·시작 전 미참여 제외) | result_status=PENDING |

- 시작 안내 중(PREPARING) 기술 문제로 끝난 경우의 기록 여부와, 시작 안내 중 아이가 종료 표현을 말한 경우의 처리는 요구사항에 없다(§14.1).

- 아동 서비스에 닿지 않아 취소 확인을 받지 못하면 END_REQUESTED로 남고 '종료 확인 안 됨'으로 표시하며 새 시작을 막는다.

규칙

- ENDED·CANCELLED는 되돌아가지 않는다. 늦은·중복 이벤트, 복구 이벤트, 늦게 온 판정은 기록만 하고 상태를 바꾸지 않는다.

- 종료 요청이 남아 있거나 종료가 확인된 세션은 복구만으로 재개하지 않는다.

- 진행 중이거나 종료가 확인되지 않은 세션은 보호자 계정(기기 하나)마다 하나만 허용한다. PostgreSQL 부분 유니크 인덱스로 강제한다(status NOT IN ('ENDED','CANCELLED')).

- 대상 아동, 종료 표현, 승인 문구 버전, 판정 지점·선택지 버전, 목표 순서는 시작 때 고정한다. 아동 정보 수정·아동 선택은 진행 중 세션에 영향이 없다.

- 결과 정리는 종료 확인 뒤에만 시작한다.

- 동의 철회로 끝난 활동은 결과를 만들지 않고 result_status=INSUFFICIENT(결과 없음)로 둔다.

- 시작 전 미참여(NOT_PARTICIPATED)는 종료 확인을 받은 뒤 부모 앱에 알리고 세션과 하위 데이터를 지운다(§4.7).

### 4.3 연습 진행 상태 기계 (아동 서비스, 연습 2안 「상황 반응 조절형」)

목표마다 아래 흐름을 돌고, 목표가 여럿이면 같은 활동(놀이 주제) 안에서 다음 목표로 이어 간 뒤 마지막에 한 번 마무리한다. 고정 단계 구조가 아니며 도움이나 변형 장면을 반드시 거치지 않는다.

| 단계 | 하는 일 | 판정 | 판정 결과 → 다음 행동 |
|---|---|---|---|
| SCENE_INTRO | 상대·활동·필요한 것을 짧게 설명하고 장면 시작. 정답 표현 없음 | — | → OPPORTUNITY |
| OPPORTUNITY | 기회 문장(세션 고정 대본) 낭독, 재생 완료 확인 | J2 | 충분히 전달 → 교정 없이 이어 가기 → SCENE_WRAP / 일부 전달 → REASK / 도움 요청 → FOCUS / 상황 이해 못함·무관 발화 → 상황 쉽게 다시 설명 후 OPPORTUNITY / 판독 불가·보류 → "다시 말해 줄래?" 확인 후 다시 J2 |
| REASK | 상대 역할 그대로 자연스럽게 1회 되묻기 | J3 | 전달됨 → 이어 가기 → SCENE_WRAP / 여전히 부족 → HELP_OFFER / 도움 요청 → FOCUS / 판독 불가 → 확인 |
| HELP_OFFER | 아이가 요청하지 않은 도움을 제안 | J4 | 수락 → FOCUS / 거절 → 원래 장면 이어 가기 또는 SCENE_WRAP / 불명확·판정 보류 → 집중 연습을 시작하지 않고 교정 없이 원래 장면 이어 가기 |
| FOCUS | 코치 역할로 전환을 알리고("잠깐, 무중이 도와줄게") 목표 관련 부분 하나만 짧게 집중 연습. 표현 예시 가능 | — | → RETRY |
| RETRY | 다시 해 보기 | J5 | J2 선택지 + 코치 표현 그대로 따라 함 → 친구 역할 복귀 알림("다시 친구 역할로 돌아갈게") → 원래 장면 또는 SCENE_WRAP |
| SCENE_WRAP | 장면 정리 | — | 시간·참여 의사가 남고, 그 목표가 10분 ⏱ 전이고, 집중 연습이 길어지지 않았으면 VARIANT 제안 / 아니면 NEXT_GOAL 또는 CLOSING |
| VARIANT | 한 요소를 바꾼 변형 장면(같은 놀이 주제의 다음 장면 가능) | — | → OPPORTUNITY와 같은 흐름 |
| NEXT_GOAL | 다음 목표의 장면으로 전환 지시 | — | → SCENE_INTRO |
| CLOSING | 승인 마무리 문구 1회 | — | → 종료 처리(§4.4) |

- 첫 반응 전에는 목표 단서·정답·시범을 주지 않는다. 같은 말 반복·속도 조절만 허용한다.

- 판정 보류(문턱 미달·시간 초과)에는 교정이 따르는 다음 행동을 주지 않는다.

- 무응답: 기회 직후·되묻기 뒤에 발화가 없으면 1회 재안내 → 다시 없으면 그 장면 미실시. 기술적 일시 정지 중에는 재안내하지 않는다.

- 목표당 시간 10분 ⏱은 ACTIVE 시간으로 잰다(기술적 일시 정지 제외 🟡). 10분에 이르면 변형 장면 없이 그 목표를 정리한다.

- 상황 다시 설명·집중 연습의 반복 횟수와 집중 연습 길이는 §13에 값으로 둔다(문안은 AI 동작 명세).

- 아동별 장면 노출 이력(scene_exposure)은 장면을 실제로 시작할 때(SCENE_INTRO 재생) 남긴다. 생성만 하고 쓰지 않은 장면은 노출로 치지 않는다.

- 상태 기계가 정한 다음 행동만 GPT-Live에 지시한다. 판정 라벨·확률·판정 기준은 넣지 않는다(FR-F15).

- 현재 역할(친구/코치)과 제공한 도움(종류·내용·표현 예시 여부·동의 응답·시각)을 이벤트로 남긴다.

시작 안내 (자유대화·연습 공통, J1)

| J1 결과 | 다음 행동 |
|---|---|
| 참여 | ACTIVE. 자유대화 진행 또는 연습 SCENE_INTRO |
| 휴식 요청 | 휴식 요청으로 기록하고 시작 안내를 이어 간다(거부 아님) |
| 거부 | 다른 선택지를 1회 제안 → 다시 거부하면 승인 마무리 문구 뒤 시작 전 미참여로 종료. 특정 주제만 거부하면 다른 주제 제안(종료 아님) |
| 무관·불명확·판독 불가·판정 보류 | 참여 의사를 다시 확인 |
| 휴식 요청 없는 무응답 | 대기 시간(§13) 뒤 시작 전 미참여로 종료 |

자유대화는 연습 상태 기계 없이 GPT-Live 자동 응답으로 진행한다. 시작 안내 응답(J1)과 종료 의도(J6)만 판정 턴이다.

### 4.4 종료 처리 순서

1. 종료 계기: 보호자 종료(P1) / 아동 종료 의도 / 정상 마무리 / 공백 자동 종료 / 기술 종료 / 동의 철회 / 시작 전 미참여. 보호자 종료·동의 철회·계정 삭제는 보호자 서버가 END_REQUESTED로 받아 아동 서비스에 전달한다.

2. 아동 서비스: ENDING으로 바꾸고 epoch를 올린다. 대기 출력, 진행 중인 판정 턴, 연습 진행을 멈춘다.

3. 기술 문제 종료가 아니면 승인 마무리 문구를 1회 낭독한다(시작 전 미참여 포함). 음성 대화 장애 중이면 낭독하지 않는다. 참여 전 준비 취소는 안내를 멈추고 종료 확인만 보낸다(ACT-START-002).

4. 마무리 재생이 끝나면 GPT-Live에 session.close를 보내고 session.closed를 기다린다(최대 15초 🟡).

5. 출력 중지와 GPT-Live 세션 종료를 확인하면 종료 확인 이벤트(종료 사유·원인·확인 시각·확보 기록 범위)를 보호자 서버에 보낸다. 기기 연결이 끊긴 상태에서도 확정한다.

6. 보호자 서버: ENDED를 저장하고 결과 작업을 등록한다(동의 철회·시작 전 미참여 제외). 부모 앱 이동은 §4.7.

- 보호자 서버는 종료 명령 후 확인 대기 시간(§13)이 지나도 확인이 없으면 '종료 확인 안 됨'과 마지막 확인 시각을 보여 주고 새 활동 시작을 막는다.

- 통신 장애 중 원격 즉시 종료를 보장하지 않는다.

### 4.5 순서·중복·멱등

- 부모 앱의 시작·종료 요청은 Idempotency-Key 헤더를 반드시 붙인다. 같은 키의 요청은 같은 결과를 돌려준다.

- 보호자 서버 → 아동 서비스 명령은 commandId로 중복을 거른다. 명령에는 sessionId를 넣어 다시 연결한 기기에 이전 세션의 명령이 적용되지 않게 한다.

- 아동 서비스 → 보호자 서버 이벤트는 세션마다 1씩 늘어나는 seq와 eventId를 가진다. 보호자 서버는 (session_id, seq) 유니크로 중복을 막고 순서대로 반영한다.

- 이벤트마다 발생 시각(occurredAt, 아동 서비스 시계)과 수신 시각(receivedAt, 보호자 서버)을 함께 저장한다. 늦게 받은 이벤트를 현재 상태처럼 보이지 않게 한다.

### 4.6 활동 시작 조건 (CR-START)

보호자 서버는 홈 조회와 시작 요청 때 다음을 확인한다.

| 조건 | 충족하지 않으면 |
|---|---|
| 기기 연결 준비: 이 계정의 등록 기기가 기기 토큰으로 아동 서비스에 연결돼 있고, 그 사실을 상태 유효시간 안에 받았다(04 기기 상태 줄이 '연결됨') | 자유대화·연습 탭 잠금, 누르면 이유 → 05 |
| 동의 4개가 모두 있다 | 탭 잠금, 누르면 이유 → 02 |
| 이전 활동의 종료가 확인됐다 | 탭 잠금, 누르면 이유 |
| 진행 중인 활동이 없다 | 탭을 잠그지 않고 두 탭 모두 그 세션의 진행 화면(12·21)으로 보낸다 |
| (연습만) 담은 목표가 하나 이상 있다 | 탭은 열어 두고 18에 '아직 담은 목표가 없어요'와 「자유대화 하러 가기」. 시작 요청은 NO_SAVED_GOALS로 거절 |

시작 요청(08 「자유대화 시작」, 18 「연습 시작」) 때 보호자 서버가 다시 확인한다.

### 4.7 종료 뒤 화면 연결

보호자 서버는 종료 확인을 받으면 SSE end 이벤트로 아래 정보를 보낸다. 부모 앱은 이에 따라 이동한다.

| 종료 | 부모 앱 | 기록 | 결과 정리 |
|---|---|---|---|
| 정상 마무리·아동 중단·보호자 중단·공백 자동 종료 | 팝업 없이 17·23 | 남김 | 함 |
| 기술 문제 | P2 → 17·23 | 남김 | 함 |
| 동의 철회 | 02 관리 모드에서 04 | 철회 전 대화 기록만 남김 | 하지 않음(결과 없음) |
| 시작 전 미참여 | P3(무응답: "아이가 대답하지 않아 시작하지 못했어요" / 거절: "아이가 지금은 하지 않기로 했어요") → 04 | 남기지 않음 | 하지 않음 |
| 참여 전 준비 취소 | 04 | 취소 사실·시각만 | 하지 않음 |

- end 이벤트에는 종료 종류, 기술 문제 여부, P3 구분(무응답/거절), 결과 정리 상태를 담는다.

- 시작 전 미참여 세션은 end 이벤트를 보낸 직후 지운다. 부모 앱이 이후 그 세션을 조회해 404를 받으면 04로 간다.

### 4.8 푸시 발송 조건 (FR-A10, CR-PUSH)

| 사건 | 근거 | 문구 |
|---|---|---|
| 음성 대화 장애 확인 | 아동 서비스의 TECH_PAUSED 이벤트 | "대화 중 음성 연결 문제가 발생했어요." |
| 기기 상태 미확인 | 보호자 서버의 서버 간 전달 NOT_RECEIVING 판정(활동 중) | "기기 상태를 확인할 수 없어요." |
| 기술 문제로 종료 확인 | END_CONFIRMED + TECHNICAL | "연결 문제로 활동이 종료됐어요." |

- 감시와 발송 요청은 보호자 서버가 하며 부모 앱 실행 여부와 무관하다. 앱 닫힘·부모 앱 연결만 장애는 알림 사유가 아니다.

- 종료 알림은 종료 확인 뒤에만 보낸다. 공백 자동 종료 등 다른 종료의 알림 여부·문구, 발생·회복 기준은 D-CON-06(§13).

- 같은 사건은 한 번만 보낸다(dedup_key). 대화 내용·발화 원문을 넣지 않는다. 발송 요청과 실제 전달을 구분해 기록한다.

- 알림을 누르면 알림 문구가 아니라 최신 세션 상태를 다시 조회해 화면을 정한다.

## 5. 인터페이스

### 5.1 부모 앱 → 보호자 서버 (REST)

| 메서드·경로 | 용도 | 관련 FR · 화면 |
|---|---|---|
| GET /oauth2/authorization/google | 구글 로그인 시작 | FR-A01 · 01 |
| GET /login/oauth2/code/google | 구글 인증 완료. 처음이면 계정 생성, 서버 세션 발급 | FR-A01 |
| GET /api/me | 로그인 상태, 첫 동의 여부, 아동 수(다음 화면 결정) | FR-A01 · 01→02·03·04 |
| POST /api/logout | 로그아웃. 활동 종료가 아니다 | FR-A01 · SET-00 |
| DELETE /api/me | 계정 삭제. 진행 중 활동이 있으면 종료 확인 뒤 삭제(202) | FR-A13 · SET-00 |
| GET /api/consents | 동의 4개 상태 | FR-A03 · 02 |
| PUT /api/consents | 첫 동의(4개 모두), 철회, 다시 동의 | FR-A03·A08 · 02 |
| GET /api/consents/items/{item} | 항목 세부(수집하는 것·목적·보관·동의하지 않으면) | FR-A03 · 02-1 |
| GET /api/record-deletions/preview | 삭제 대상별 영향 | FR-A08 · 02 |
| POST /api/record-deletions | 기록 삭제(원음·전사·리포트) | FR-A08 · 02 |
| GET·POST /api/children | 아동 목록, 등록(첫 등록·추가) | FR-A02 · 03 |
| PATCH /api/children/{childId} | 아동 정보 수정 | FR-A02 · 03 |
| DELETE /api/children/{childId} | 아동 삭제. 진행 중·종료 미확인 활동이 있으면 409 | FR-A12 · P4 |
| GET·PUT /api/children/{childId}/stop-expressions | 추가 종료 표현 조회·저장(기본 표현은 고정) | FR-A09 · SET-01 |
| GET /api/home?childId= | 선택 아동, 기기 상태 줄, 진행 중 배너, 시작 가능 여부와 잠금 이유 | FR-A06·A07 · 04 |
| GET /api/devices | 기기 등록·연결 대상별 상태·마지막 확인 시각 | FR-A06 · 05 |
| POST /api/devices | 등록 코드로 기기 등록 | FR-A11 · 05a |
| POST /api/sessions | 시작 요청. 자유대화(배경 정보) / 연습(담은 목표 ID 목록). Idempotency-Key 필수 | FR-B05·C05 · 08·18 |
| POST /api/sessions/{id}/cancel | 상황 준비 중 취소(기기에 아무것도 보내지 않음) | FR-C05 · 21 |
| POST /api/sessions/{id}/end | 끝내기. 참여 전이면 준비 취소, 이후면 종료 요청. Idempotency-Key 필수 | FR-B07 · P1 |
| GET /api/sessions/{id} | 활동 상태, 연결 관측, 종료 확인 여부, 시작 시각 | FR-B06·C06·F01 · 12·21 |
| GET /api/sessions/{id}/transcript?afterSeq= | 확정 전사와 끊김 구간(재조회·이어 붙이기) | FR-B06·F04 · 12·21 |
| GET /api/sessions/{id}/stream | 실시간 이벤트(SSE, §5.2) | FR-B06·C06 · 12·21 |
| GET /api/children/{childId}/goals | 담은 목표(담은 곳·연습 횟수·즐겨찾기) | FR-C01 · 18 |
| PATCH /api/goals/{goalId} | 즐겨찾기 지정·해제 | FR-C01 · 18 |
| DELETE /api/goals/{goalId} | 담은 목표 삭제 | FR-C01 · 18 |
| POST /api/goal-candidates/{candidateId}/save | 목표 후보 담기 | FR-F06 · 27 |
| GET /api/children/{childId}/calendar?month= | 활동이 있는 날 | FR-D01 · 26 |
| GET /api/children/{childId}/records?date= | 그날 활동(종류·길이·중간에 끊김) | FR-D01 · 26 |
| GET /api/records/{sessionId} | 기록 상세: 대화 기록·끊김 구간·장면 결과·생활 팁·목표 후보·종료 사유·확보 범위·결과 상태 | FR-D02·C09 · 27 |
| POST /api/records/{sessionId}/result/retry | 결과 다시 요청 | FR-F17 · 27 |
| POST /api/children/{childId}/reports | 범위(기간·활동)로 리포트 생성 요청 | FR-D05 · 28 |
| GET /api/reports/{reportId} | 리포트 상태와 10개 항목 | FR-D05 · 29 |
| GET /api/reports/{reportId}/pdf-summary | 저장 확인 시트 내용(기간·활동 수·항목 수·아이 정보 범위) | FR-D06 · 29 |
| POST /api/reports/{reportId}/pdf | PDF 생성·내려받기, 저장 이력 | FR-D06 · 29 |
| POST·DELETE /api/push-subscriptions | 웹 푸시 구독 등록·해제 | FR-A10 |

아동 기기용 (로그인 없음, 보호자 서버)

| 메서드·경로 | 용도 | 관련 FR · 화면 |
|---|---|---|
| POST /device-api/registrations | 등록 코드 발급. 기기는 코드와 조회 토큰을 브라우저에 보관해 다시 켜도 같은 코드를 보여 준다 | FR-A11 · D0 |
| GET /device-api/registrations/{id} | 등록 대기. 보호자가 05a에서 등록하면 기기 토큰을 한 번 돌려준다 | FR-A11 · D0→T19 |

- 동의 철회 상태에서는 시작 요청, 리포트 생성, PDF 저장, 결과 다시 요청을 거절하고(CONSENT_REQUIRED), 기록·기존 리포트 조회와 삭제는 허용한다.

- 모든 요청에서 리소스가 로그인한 보호자 소유인지 확인한다(아동·세션·기록·목표·리포트).

### 5.2 보호자 서버 → 부모 앱 실시간 (SSE) 🟡

GET /api/sessions/{id}/stream (text/event-stream, 세션 쿠키로 인증)

| 이벤트 | 내용 |
|---|---|
| status | 활동 상태, 상태 한 줄에 쓸 값(상황 준비 중·시작 요청됨·실행 확인 중·참여 기다리는 중·대화 중·연습 중·다음 목표로 넘어감·끝내는 중·종료 확인 안 됨), 시작 시각. '전송하지 못함'은 부모 앱이 스스로 표시한다 |
| transcript.delta | utteranceId, 화자(무중/아이), 지금까지의 글자. 앱은 '말하는 중'으로 표시 |
| transcript.final | utteranceId, 화자, 확정 글자, 시작·끝 시각, seq |
| gap | 끊김 구간 시작·끝(화면은 '잠깐 끊김' 하나, 내부 구분은 저장만) |
| observation | 연결 대상별 상태, 마지막 확인 시각 |
| end | 종료 확인, 종료 종류(§4.7), 기술 문제 종료 여부(P2), P3 구분(무응답/거절), 결과 정리 상태 |

- Last-Event-ID로 이어 받는다. SSE가 끊기면 앱은 "실시간 상태를 불러오지 못하고 있어요"와 마지막 확인 시각을 보여 주고, TanStack Query로 GET /api/sessions/{id}와 .../transcript?afterSeq=를 다시 조회해 실제 받은 전사만 이어 붙인다.

- 보내지 않는 것: 판정 결과·확률, 다음 행동 지시, 보류된 출력, 원음.

- 홈(04)·우리 집 기기(05)의 상태는 TanStack Query로 주기 조회한다(주기 §13).

### 5.3 보호자 서버 → 아동 서비스 (내부 명령) 🟡

| 메서드·경로 | 용도 |
|---|---|
| POST /internal/activity/sessions | 시작 명령(commandId, 세션 계획) |
| POST /internal/activity/sessions/{id}/end | 종료 명령(사유: 보호자 종료·동의 철회·계정 삭제) |
| POST /internal/activity/sessions/{id}/cancel | 참여 전 준비 취소 |
| GET /internal/activity/devices/{deviceId} | 기기 연결 상태 조회 |
| POST /internal/activity/devices/{deviceId}/revoke | 기기 토큰 무효(계정 삭제·재등록) |

세션 계획 — GPT-Live에 무엇이 가는지 함께 표시한다.

| 항목 | 내용 | GPT-Live에 전달 |
|---|---|---|
| 세션 기본 | sessionId, 종류(자유대화/연습), deviceId, 기기 토큰 해시(재연결 검증용) | 아니오 |
| 아동 정보 | 이름 또는 별명, 나이, 관심사 | 이름·관심사만 |
| 종료 표현 | 기본 표현 + 그 아동의 추가 표현 | 아니오 |
| 승인 문구 | 버전과 원문(시작 안내·마무리·종료 의도 확인) | 낭독 지시로만 |
| 판정 지점 | 버전, J1~J6 질문·선택지 | 아니오 |
| 자유대화 | 주제 힌트(배경 정보를 보호자 서버가 바꾼 것. 형태는 G-03), 활용 가능한 이전 기록 요약, 승인된 일상 질문 은행(FR-F03) | 예. 보호자 입력 원문, 보호자가 입력했다는 사실, 이전 판정 결과는 전달하지 않음 |
| 연습 목표 | 목표 순서대로: goalId, 목표 행동 표현, 핵심 기능 정의, 판정 기준, 장면 대본(기본·변형), 기회 문장 | 장면 대본·기회 문장만. 핵심 기능 정의·판정 기준은 아니오 |
| 제한값 | 목표당 시간, 응답 공백 허용 시간, 전사·제브 제한 시간, 무응답 대기 등(§13) | 아니오 |

### 5.4 아동 서비스 → 보호자 서버 (내부 채널) 🟡

- 아동 서비스가 보호자 서버의 내부 WebSocket(/internal/ws/relay)에 연결한다.

- 이벤트는 먼저 activity.outbox에 쓰고 보낸다. 보호자 서버가 ack{sessionId, seq}를 보내면 지운다. 다시 연결하면 확인받지 못한 것부터 순서대로 다시 보낸다.

- transcript.delta는 outbox에 쓰지 않고 바로 보낸다(확인 없음, 저장 없음, 부모 앱 중계만). 확정 전사는 이벤트로 보낸다.

- 원음은 발화 단위로 POST /internal/sessions/{id}/audio(multipart)로 올린다. outbox에 파일 경로로 등록하고 성공하면 임시 파일을 지운다.

- 보호자 서버는 동의 철회·삭제 뒤에 받은 내용 자료(확정 전사, 원음, 판정 기록, 제공한 도움, 실시간 전사)를 저장하거나 중계하지 않는다. 상태·종료 이벤트(STATUS_CHANGED, END_STARTED, END_CONFIRMED, OBSERVATION)는 그대로 반영해 '동의 철회' 종료가 확정되게 한다. 버린 자료도 재전송이 멈추도록 확인(ack)은 보낸다.

- 기기 토큰 검증은 POST /internal/devices/verify로 한다. 아동 서비스는 진행 중 세션의 기기 토큰 해시를 세션 계획으로 받아 두어, 보호자 서버에 닿지 않을 때도 같은 세션의 재연결을 검증한다.

| 이벤트 | 내용 |
|---|---|
| SESSION_ACCEPTED / DEVICE_START_CONFIRMED | 명령 수신, 기기의 시작 안내 재생 시작 |
| STATUS_CHANGED | 활동 상태 전이 |
| PARTICIPATION | J1 결과(참여 / 미참여 사유) |
| REST_REQUESTED | 휴식 요청 |
| UTTERANCE_FINAL | 화자, 확정 전사, 시작·끝 시각, 불확실 구간, 역할(친구/코치), 출력 종류, outputId |
| PLAYBACK | 출력 재생 시작·끝·멈춤 |
| TECH_PAUSED / RESUMED | 원인, 출력 중지 확인 여부 |
| OBSERVATION | 기기 연결·음성 모델 연결 상태, 확인 시각 |
| PRACTICE_PROGRESS | 목표·장면·단계(저장만, 화면에 표시하지 않음) |
| HELP_GIVEN | 도움 종류(되묻기·집중 연습·상황 재설명·판정 보류 확인), 내용, 표현 예시 여부, 동의 응답 |
| JUDGMENT | 진행용 판정 기록(§7.3) |
| GAP_DETECTED | 4분 공백 내부 상태 |
| END_STARTED / END_CONFIRMED | 종료 사유·원인·확인 시각·확보 기록 범위 |
| DEVICE_CONNECTED / DEVICE_DISCONNECTED | 기기 연결 변화(활동이 없을 때도 보냄. 04·05 표시용) |
| USAGE | GPT-Live 사용량(session.closed), 제브 호출 수 |

### 5.5 아동 기기 ↔ 아동 서비스 (기기 연결) 🟡

- 주소: wss://{host}/ws/device. 첫 메시지 hello{deviceToken, clientVersion}으로 인증한다. 토큰을 URL에 넣지 않는다.

- 음성은 바이너리 프레임(PCM16 mono 24kHz, 20ms 단위), 제어는 JSON 텍스트 프레임.

| 방향 | 메시지 |
|---|---|
| 기기 → 아동 서비스 | hello, 음성 프레임, playback{outputId, epoch, event, at}(event: started / ended / stopped), power{off}(끄기), ping |
| 아동 서비스 → 기기 | welcome{deviceId}, assign{sessionId}, display{state}(IDLE / LISTENING / THINKING / SPEAKING / CLOSING), output.begin{outputId, epoch}, 음성 프레임(헤더에 epoch·outputId), output.end{outputId}, output.stop{epoch}, session.end{sessionId}, error{code}, pong |

- 기기 화면(T19)의 '연결 끊김·다시 연결 중'은 기기가 스스로 판단한다. 나머지 표시 상태는 display를 따른다.

- 메시지 스키마는 contracts/에 둔다. 인형 형태의 기기도 같은 프로토콜로 동작한다(NFR-11).

### 5.6 아동 서비스 ↔ GPT-Live (음성 모델 연결)

| 용도 | 이벤트·설정 |
|---|---|
| 연결 | wss://api.openai.com/v1/live/sessions, Authorization: Bearer {OPENAI_API_KEY}(서버 환경변수) |
| 시작 | session.start — model: "gpt-live-1", instructions(기본 지시문), audio.format: {type: "audio/pcm", rate: 24000}, audio.output.voice → session.started를 받은 뒤 음성·명령 전송 |
| 입력 음성 | session.input_audio.append (base64 PCM16) |
| 출력 음성 | session.output_audio.delta (start_ms·end_ms 포함) |
| 전사 | session.input_transcript.delta(아이), session.output_transcript.delta(무중) |
| 지시 | session.instructions.append — delegation_id: null, content 500토큰 이하 → session.instructions.appended |
| 입력 차단 | session.input_audio.mute / unmute |
| 설정 변경 | session.update (모델·오디오 형식은 시작 때 고정) |
| 종료 | session.close → session.closed(최종 사용량) |
| 오류 | error |

- 한 활동에 GPT-Live 세션 하나를 쓴다. 친구·코치 역할 전환과 다음 목표 전환은 지시로 한다.

- delegation은 쓰지 않는 것을 기본으로 한다. 필수라면 client 방식으로 한다. 🧪

- openai-java가 GPT-Live WebSocket을 지원하지 않으면 LiveVoiceAdapter 안에 JDK WebSocket 클라이언트로 직접 구현한다. 🧪

- 세션 최대 길이, 재연결 때 맥락 복원, 데이터 보관 설정은 첫 볼트에서 확인한다. 🧪

- 기능명세서에 예시로 적힌 input_audio_transcription.\*·output_audio_transcript.\*는 Realtime API의 이벤트 이름이다. GPT-Live에서는 위 표의 이름을 쓴다.

### 5.7 제브 (Jev, TypeSafe AI)

- 호출: POST https://api.typesafe.ai/v1/systemone. 모델 버전을 고정한다(예: jev-1.13.0). jev-latest는 쓰지 않는다. 🟡

- 보내는 것: 판정 질문, 선택지(판정 지점별 고정, 최대 255개), 문맥(목표 판정 기준·장면·최근 몇 턴·전사).

- 받는 것: 고른 선택지, 선택지별 확률, 전체 신뢰도. 정확한 필드 이름은 계정 승인 뒤 API 문서로 확정한다. 🧪

- Java SDK가 없으므로 JudgeClient 인터페이스 뒤에 REST 구현(Spring RestClient, 연결 재사용)을 둔다.

- 호출 위치: J1~J6은 아동 서비스, J7은 보호자 서버.

- 공개 자료 기준 응답 시간 70~500ms. 제한 시간 후보 800ms.

- 접근은 대기 명단 승인이 필요하다. 🧪

- 승인 전 개발·시험에는 JudgeClient의 가짜 구현을 쓴다. 한국어 아동 발화 정확도가 PoC 기준에 못 미칠 때의 대안(LLM 구조화 출력 + 확률 등)은 🟡이며, 요구사항의 확정 구조(제브)를 바꾸는 일이므로 요구사항 변경이 먼저 필요하다.

### 5.8 분석 AI (OpenAI 텍스트 LLM)

- openai-java OpenAIClient를 Bean으로 등록해 Responses API를 호출한다. LlmClient 인터페이스 뒤에 둔다.

- 작업마다 구조화 출력(JSON Schema)으로 받고, 서버가 스키마와 인용을 검증한다(§7.4).

- 스키마 불일치는 1회 다시 요청하고, 그래도 실패하면 작업 실패로 처리한다.

- SDK 모델 객체를 Controller 응답으로 그대로 내보내지 않는다. 우리 DTO로 바꾼다.

### 5.9 오류 형식

Spring ProblemDetail에 서비스 오류 코드를 더한다.

{

"title": "Device not ready",

"status": 422,

"detail": "기기 연결을 확인해 주세요.",

"code": "DEVICE_NOT_READY"

}

| 코드 | 상태 | 뜻 |
|---|---|---|
| CONSENT_REQUIRED | 422 | 동의 4개 중 없는 것이 있음 |
| DEVICE_NOT_REGISTERED / DEVICE_NOT_READY | 422 | 등록 기기 없음 / 기기 연결 준비 미확인 |
| SESSION_IN_PROGRESS / END_NOT_CONFIRMED | 409 | 진행 중 세션 있음 / 이전 활동 종료 미확인 |
| NO_SAVED_GOALS | 422 | 담은 목표 없음 |
| SCENE_GENERATION_FAILED | 422 | 장면 대본 두 번 실패 |
| CHILD_HAS_ACTIVE_SESSION | 409 | 진행 중·종료 미확인 활동이 있는 아동 삭제 |
| REGISTRATION_CODE_INVALID / DEVICE_OWNED_BY_OTHER | 422 | 없는 코드 / 다른 계정에 묶인 기기 |
| SESSION_STATE_CONFLICT | 409 | 허용되지 않은 상태 전이 |

## 6. 데이터 모델

### 6.1 보호자 서버 core 🟡

| 테이블 | 주요 열 |
|---|---|
| guardian | id, google_sub(유니크), email, consent_revoked_at, created_at |
| consent | guardian_id, item(VOICE_TRANSCRIPT·GUARDIAN_VIEW·REPORT·RAW_AUDIO), granted, terms_version, changed_at |
| consent_history | guardian_id, item, granted, changed_at |
| child | id, guardian_id, display_name, age, interests(jsonb), created_at |
| stop_expression | id, child_id, phrase, created_at (기본 표현은 저장하지 않고 코드에 고정) |
| device | id, guardian_id, token_hash, registered_at, last_seen_at, revoked_at |
| device_registration | id, code(유니크), poll_token_hash, device_id, created_at, registered_at |
| session | id, guardian_id, child_id, device_id, kind(FREE·PRACTICE), status, end_reason, tech_cause, result_status, shortfalls(jsonb), background(jsonb), goal_order(jsonb), stop_expressions(jsonb), content_version, judge_spec_version, requested_at, device_confirmed_at, started_at, end_requested_at, end_received_at, end_confirmed_at, last_seq, version |
| session_event | session_id, seq, event_id, type, occurred_at, received_at, payload(jsonb) — 상태 이력 |
| connection_observation | device_id, session_id, target(DEVICE_LINK·VOICE_MODEL_LINK·RELAY), state, confirmed_at, received_at |
| gap_segment | session_id, kind(TECH_PAUSED·UNOBSERVED), started_at, ended_at — 화면의 '잠깐 끊김'·'끊김' |
| utterance | id, session_id, seq, speaker(CHILD·MUZUNG), role(FRIEND·COACH), output_kind(APPROVED·SCRIPT·DIRECTED·FREE), text, uncertain_spans(jsonb), started_at, ended_at |
| audio_clip | id, utterance_id, session_id, file_path, format, duration_ms, bytes |
| playback | session_id, output_id, utterance_id, started_at, ended_at, stopped |
| session_reference | session_id, source_type(LIFE_RECORDING 예약), file_path — MVP에서는 비어 있음 |
| experience | session_id, items(jsonb: 인물·상황·상대의 말·아동 반응·설명, 출처 태그 GUARDIAN_INPUT·CHILD_RECALL, 근거 utterance 범위) |
| goal_candidate | id, child_id, source_session_id, name, description, behavior_label, core_function, opportunity_condition, scene_condition, judgment_criteria, reason, evidence(jsonb: utterance id·시각), created_at |
| saved_goal | id, child_id, candidate_id, favorite, saved_at, practice_count, last_practice(jsonb) |
| scene | id, session_id, goal_id, seq, type(BASE·VARIANT), script(jsonb), opportunity_line, fingerprint, check_result(jsonb: 자동 확인·J7) |
| scene_exposure | child_id, fingerprint, session_id, exposed_at |
| scene_record | scene_id, first_response_utterance_id, after_help_utterance_id, result_label, result_sentence, not_done_reason |
| help_given | id, scene_id, seq, type(REASK·FOCUS·REEXPLAIN·CLARIFY), content, example_given, consent_answer, given_at |
| judgment_live | id, session_id, goal_id, scene_id, point(J1~J7), input_utterance_id, options_version, top1, p1, top2, p2, held, hold_reason(THRESHOLD·TIMEOUT·NO_TRANSCRIPT·LINK_CANCELLED), latency_ms, next_action, withheld_output_text |
| judgment_record | scene_id, value(functional·partial·none·uncertain), rationale, model, differs_from_live |
| result_job | id, session_id(유니크), status, attempts, next_run_at, last_error |
| life_tip | id, session_id, seq(최대 2), text, evidence(jsonb: 장면·발화), check_passed |
| report | id, child_id, range(jsonb), session_ids(jsonb), sections(jsonb: 10개 항목), status, created_at |
| report_pdf_log | report_id, saved_at |
| push_subscription | id, guardian_id, endpoint, p256dh, auth, created_at |
| push_dispatch | id, guardian_id, session_id, event_type, dedup_key(유니크), message, requested_at, result |
| usage_log | session_id, provider(GPT_LIVE·JEV·LLM), task, units(jsonb), logged_at |

- 시작 전 미참여로 끝난 세션은 종료 확인을 받아 부모 앱에 알린 뒤 세션과 하위 행을 지운다(§4.7).

- CANCELLED 세션은 session 행에 상태·취소 종류·시각만 남기고, 그동안 받은 전사·원음·판정 기록·장면 대본은 지운다. 26에는 보이지 않는다.

- 26에는 ENDED이면서 end_reason이 있는 세션만 나온다.

### 6.2 아동 서비스 activity 🟡

| 테이블 | 주요 열 |
|---|---|
| runtime_session | session_id, device_id, status, plan(jsonb), snapshot(jsonb: 목표·장면·역할·턴 모드·epoch·타이머), updated_at |
| outbox | id, session_id, seq, type, payload(jsonb), file_path, created_at, sent_at, attempts |

- activity 스키마는 별도 DB 사용자로 접근해 core를 읽거나 쓰지 못하게 한다.

- 전달 확인된 행은 지운다. 종료된 세션의 남은 행은 보관 한계(§13)가 지나면 지운다.

### 6.3 원음 파일 🟡

- 경로: /data/audio/{guardianId}/{childId}/{sessionId}/{utteranceId}.wav (EC2 EBS 볼륨을 보호자 서버 컨테이너에 마운트)

- 형식: WAV(PCM16 mono 24kHz). 아이 발화 구간만 저장한다.

- 행과 파일을 함께 지운다. 파일 삭제 실패는 재시도 작업으로 남긴다.

### 6.4 지식 그래프 (Neo4j) 🟡

- 화용 지식 전용이다: 화용 행동, 핵심 기능, 상황 유형, 사회적 단서, 관계, 지원 방법과 그 관계.

- 아동 개인 기록(세션·발화·판정)을 두지 않는다. 운영 기록은 PostgreSQL에만 둔다.

- 분석 AI가 목표 후보·4요소, 장면 대본을 만들 때 참고 문맥으로 조회한다.

- 지식 데이터는 ai/에서 가공해 시드로 넣는다. 온톨로지 상세는 별도 문서에서 정한다.

- KnowledgeAdapter 뒤에 두어 규모·일정상 필요하면 PostgreSQL 구현으로 바꾼다.

### 6.5 동의·보존·삭제 (CR-DATA)

| 상황 | 처리 |
|---|---|
| 동의 철회(4개 중 하나라도) | 진행 중 활동 종료 요청(CONSENT_WITHDRAWN) → 이후 수집·실시간 중계 중단. 철회 뒤 받은 내용 자료는 발생 시각과 관계없이 저장하지 않는다(수신 시각 기준, §5.4). 철회 전 기록은 유지하고 열람·삭제만 허용한다. 새 활동, 새 리포트, PDF 저장, 결과 다시 요청을 막는다. 철회 때 끝나지 않은 결과 작업은 멈추고 '자료 부족·결과 없음'으로 둔다 🟡. 로그인 세션은 유지한다 |
| 기록 삭제 | 대상(원음·전사·리포트) 선택 → 영향 안내 → 확인. 그 기록에서 만든 목표 후보·요약·생활 팁·리포트·진행용 판정 기록을 함께 지운다 |
| 아동 삭제 | 기본안: 그 아동의 기록(전사·원음·활동 기록·판정 기록·결과·생활 팁·리포트), 목표(후보·담은 목표), 종료 표현을 함께 지운다(TD-20). 진행 중·종료 미확인 활동이 있으면 거절 |
| 계정 삭제 | 진행 중 활동이 있으면 종료 확인 뒤 삭제. 계정과 모든 아동·기록·원음 파일·푸시 구독 삭제, 기기 토큰 무효(아동 서비스에 revoke), 서버 세션 무효 |
| 삭제 방식 | 실제 삭제(hard delete) 🟡. 삭제된 자료가 재시도·재전달로 되살아나지 않게 수신 단계에서 거른다 |

## 7. AI 처리

### 7.1 모델 세 가지의 역할

| 모델 | 맡는 일 | 호출 주체 | 받는 정보 | 받지 않는 정보 |
|---|---|---|---|---|
| GPT-Live | 실시간 음성 대화(친구·코치 역할), 진행 중 실시간 도움, 입력·출력 전사 | 아동 서비스 | 기본 지시문, 아이 이름·관심사, 주제 힌트 또는 장면 대본·기회 문장, 다음 행동 지시, 낭독할 승인 원문 | 판정 기준, 판정 라벨·확률, 이전 판정 결과, 보호자 입력 원문 |
| 제브 | 진행 중 판정(J1~J6), 생성 대본 사전 점검(J7) | 아동 서비스(J1~J6), 보호자 서버(J7) | 판정 질문, 선택지, 목표 판정 기준, 장면, 최근 몇 턴, 전사 / 점검할 대본 | 아동 이름·나이, 보호자 입력, 원음 |
| 분석 AI | 장면·기회 대본, 경험 구조화, 기록용 재판정, 목표 후보·4요소, 결과 문장, 생활 팁, 리포트 | 보호자 서버 | 작업별 최소 정보(§7.4) | 원음, 작업에 필요 없는 아동 정보 |

공급자 요청마다 보낸 필드 이름 목록을 감사 로그로 남겨 목적 밖 필드가 없는지 검사한다(NFR-04). 요청·응답 본문 전체는 로그에 남기지 않는다.

### 7.2 GPT-Live 사용 규칙

- 기본 지시문: 무중 캐릭터, 아이 이름·관심사, 한 번에 질문 하나, 과도한 칭찬·추궁 금지, 표시·생성 금지 규칙(CR-SAFE), 첫 반응 전 정답 금지. 문안은 AI 동작 명세.

- 자유대화: 보호자 서버가 배경 정보를 주제 힌트로 바꿔 넣는다. 보호자가 입력했다는 사실은 넣지 않는다(FR-E03, G-03).

- 연습: 장면 대본과 상대 역할, 다음 행동 지시만 넣는다.

- 지시 한 번의 길이는 500토큰 이하로 한다(GPT-Live 제한).

### 7.3 제브 판정

| 지점 | 때 | 선택지 |
|---|---|---|
| J1 | 시작 안내 응답 | 참여 / 휴식 요청 / 거부 / 무관·불명확 / 판독 불가 |
| J2 | 기회 문장 직후 반응 | 충분히 전달 / 일부 전달 / 도움 요청 / 상황 이해 못함 / 무관 발화 / 판독 불가 |
| J3 | 되묻기 뒤 반응 | 전달됨 / 여전히 부족 / 도움 요청 / 판독 불가 |
| J4 | 도움 동의 | 수락 / 거절 / 불명확 |
| J5 | 집중 연습 뒤 다시 해 보기 | J2 선택지 + 코치 표현 그대로 따라 함 |
| J6 | 종료 의도(등록 표현과 일치하지 않을 때) | 종료 명확 / 불명확 / 종료 아님 / 휴식 요청 |
| J7 | 생성 대본 사전 점검 | 정답 문장 포함 / 첫 반응 전 단서 / 평가·압박 표현 / 통과 |

- 확신 문턱: 1순위 확률이 하한보다 낮거나 1·2순위 차이가 작으면 판정 보류. 값은 PoC로 정한다. 🧪

- 보류를 '일부 전달'·'여전히 부족'처럼 교정이 따르는 선택지로 바꾸지 않는다. J7 보류는 통과가 아니다.

- 판정 지점·질문·선택지는 content/judge-points/에서 버전으로 관리하고 세션 동안 고정한다.

- 진행용 판정 기록(judgment_live): 판정 지점, 세션·목표·장면, 입력 전사 참조, 선택지 버전, 1·2순위와 확률, 보류 여부·사유, 걸린 시간, 다음 행동 지시, 보류된 출력 전사. 문턱·제한 시간 조정과 PoC 검증에 쓰고, 보호자 화면과 리포트에는 쓰지 않는다.

제브 PoC 계획 🟡 (검증 과제, 팀 결정 항목 아님)

| 항목 | 내용 |
|---|---|
| 자료 | 판정 지점(J1~J7)별 한국어 발화 세트. 성인 역할극·합성 발화로 만들고 실제 아동 발화는 쓰지 않는다. 지점마다 선택지별 사례와 정답 라벨을 붙인다. 짧은 대답, 얼버무림, 전사 오류가 섞인 문장을 포함한다 |
| 측정 | 보류를 뺀 정확도, 보류율, 확률 보정 정도(예측 확률과 실제 정답률의 차이), 응답 시간 p50·p95 |
| 문턱 정하기 | 1순위 확률 하한과 1·2순위 차이를 바꿔 가며 정확도·보류율을 표로 만들고, 교정이 따르는 선택지(일부 전달·여전히 부족)의 오판이 가장 적은 지점을 고른다 |
| 통과 기준 | 정확도·보류율 기준값 ⬜ (PoC 첫 결과를 보고 정한다) |
| 결과 기록 | ai/poc/jev/에 자료·스크립트·결과표를 두고 이 문서 §13에 문턱값을 적는다 |

### 7.4 분석 AI 작업

| 작업 | 시점 | 입력 | 출력 | 검사 |
|---|---|---|---|---|
| 주제 힌트 | 자유대화 시작 요청 때 | 배경 정보(상황 유형·출처·궁금한 내용) | 대화 주제 힌트(원문 인용 없음, 형태는 G-03) | 보호자 입력 원문·보호자가 입력했다는 사실이 남지 않았는지 |
| 장면·기회 대본 | 연습 시작 요청 때 목표마다(결정 전 기준: 시작 전에 모두, TD-16) | 목표 4요소, 아동 나이·관심사, 지난 장면 기록·노출 이력, 지식 그래프 문맥 | 장면 설명, 상대 역할, 기회 문장, 상황 다시 설명, 집중 연습 요점·표현 예시, 변형 장면 대본 | 목표·기회 조건 포함, 금지 표현, J7. 통과 못 하면 1회 다시 생성, 두 번 실패하면 시작하지 않음. 노출된 장면과 같은 지문(fingerprint)이면 다시 생성 |
| 경험 구조화 | 자유대화 종료 확인 뒤 | 전체 전사, 배경 정보 | 인물·상황·상대의 말·아동 반응·설명 + 출처 태그 + 근거 발화 범위, 다음 자유대화용 이전 기록 요약 | 원문에서 찾을 수 없는 항목 버림. 의도·감정 필드 없음. 요약에 판정 결과를 넣지 않음 |
| 기록용 재판정 | 연습 종료 확인 뒤 | 전체 전사, 진행용 판정 기록, 목표 핵심 기능·판정 기준, 장면 대본 | 장면별 functional / partial / none / uncertain + 근거 | 진행 중 판정과 다르면 차이 표시 |
| 목표 후보 | 종료 확인 뒤 | 전사, 경험 구조화·장면 기록, 지식 그래프 문맥 | 후보, 추천 이유, 근거 발화·시각, 목표 행동 표현, 4요소 | 4요소가 모두 있어야 후보. 근거 부족이면 '후보 없음'. 연령 미사용 |
| 결과 문장 | 종료 확인 뒤 | 구조화 데이터, 인용, 재판정 값 | 27의 결과 문장 | 수치·원인·효과 생성 금지. 불일치면 고정 템플릿 |
| 생활 팁 | 종료 확인 뒤(자유대화·연습 모두) | 그 활동의 장면 | 팁 최대 2개 + 근거 장면 | 금지 표현 자동 검사 통과분만. 없으면 '추가 관찰 필요'(검사 기준 미결) |
| 리포트 | 28 요청 때 | 범위 안 기록의 구조화 데이터·인용·재판정 값 | 10개 항목 | 결측은 결측으로 표시. '진단 결과 아님' 고정 문구 |

- 모든 인용은 저장된 전사와 글자 단위로 일치해야 한다. 일치하지 않는 인용은 빼고 자료 부족으로 표시한다.

- 동의 철회·삭제된 자료는 입력으로 쓰지 않는다.

### 7.5 장면 결과 라벨 (27)

| 기록용 재판정 | 27 장면 결과 칩 |
|---|---|
| functional | 목표 행동이 처음 나온 시점(바로 / 되묻기 뒤 / 도움 뒤) + 목표 행동 표현(기본 '전달'). 예: 바로 전달, 되묻기 뒤 전달, 도움 뒤 전달 |
| partial·none으로 정리 | 다음 기회로 |
| uncertain | 보류 |
| 판정할 반응 없음 | 미실시 + 사유(예: 시간 종료) |

다시 말해 달라는 확인(판정 보류 확인)은 목표 단서가 아니므로 시점을 바꾸지 않는다.

### 7.6 결과 생성 파이프라인

1. 종료 확인 수신 → result_job 등록(세션당 하나, 유니크) → result_status=PENDING.

2. 워커가 §7.4의 종료 뒤 작업을 차례로 실행한다.

3. 실패하면 자동 재시도(추가 최대 3회, 5·15·30초 ⏱).

4. 최종 실패 → FAILED. 보호자가 27에서 다시 요청하면 같은 작업을 다시 실행한다. 원본 기록은 바꾸지 않고 결과는 한 번만 만든다.

5. 17·23에는 정리 상태(정리 중·정리 끝·정리 실패)만 보낸다.

- 결과 생성은 요청 스레드에서 기다리지 않는다. 메시지 큐 없이 PostgreSQL 작업 테이블을 쓴다.

### 7.7 금지 표현 검사

- 대상: 장면·기회 대본, 결과 문장, 생활 팁, 리포트, 승인 문구(배포 전).

- 1차 금지 표현 사전(정규식), 2차 LLM 분류 🟡. 대본은 J7을 함께 쓴다.

- 걸리면: 대본은 다시 생성, 결과 문장·리포트는 고정 템플릿으로 바꿈, 생활 팁은 보여 주지 않음, 승인 문구는 배포하지 않음.

- GPT-Live 출력 전사도 사후 검사해 검출 건수를 남긴다(시험·조정용). 🟡

### 7.8 승인 콘텐츠

- content/approved/: 시작 안내(자유대화·연습), 마무리, 종료 의도 확인 문구, 자유대화용 일상 질문 은행(FR-F03). 항목마다 ID·원문·버전.

- content/judge-points/: J1~J7 질문·선택지·버전.

- 배포 때 금지 표현 검사를 통과해야 한다. 세션 시작 때 버전을 세션 계획에 넣어 고정한다.

- 위험 발화 대응 문구는 이번 범위에서 제외한다.

## 8. 인증·보안·개인정보

| 항목 | 방식 | 상태 |
|---|---|---|
| 보호자 인증 | Spring Security oauth2Login(구글). 처음 오면 구글 식별자로 계정을 만든다. 서버 세션 쿠키 HttpOnly·SameSite=Lax·HTTPS에서 Secure. 로그인 성공 때 세션 ID 새로 발급. JWT 라이브러리를 넣지 않는다 | ✅ |
| 세션 저장소 | Spring Session JDBC(PostgreSQL). 테이블은 Flyway로 만든다 | 🟡 |
| 로그아웃 | 서버 세션 무효. 활동 종료가 아니다 | ✅ |
| CSRF | 끄지 않는다. SPA용 쿠키 기반 CSRF 토큰 | 🟡 |
| 리소스 접근 | 모든 요청에서 로그인 보호자 → 아동·세션·기록·목표·리포트 소유 확인 | ✅ |
| 기기 인증 | 등록 코드로 등록 → 기기 토큰(무작위 256비트, 해시 저장) 발급. 기기 연결은 hello로 토큰 제시. 등록되지 않았거나 무효인 토큰은 거부 | ✅ 방향 / 🟡 방식 |
| 서버 간 인증 | Docker 내부 네트워크 + 공유 비밀 헤더. /internal/\*\*과 내부 WebSocket은 Nginx에서 막는다 | 🟡 |
| API 키 | OpenAI·제브·구글·VAPID 키는 서버 환경변수로만 넣는다. 브라우저에 보내지 않는다 | ✅ |
| 동의 적용 | 시작 때 4개 확인. 철회 처리는 §6.5 | ✅ |
| 로그 | 주요 서버 로그에 sessionId. 아동 발화 원문·전사·원음, AI 요청·응답 본문을 애플리케이션 로그에 남기지 않는다 | ✅ |
| 데모 데이터 | 합성 데이터와 성인 역할극만 쓴다. 실제 아동 자료를 저장소·AI-DLC 입력에 넣지 않는다 | ✅ |
| 공개 저장소 여부 | 공개하면 실제 아동 자료·전사·키·.env·개인 키·재배포 조건 미확인 데이터셋을 넣지 않는다 | 🟡 |

## 9. 비기능 요구사항 구현

| 요구사항 | 구현 |
|---|---|
| NFR-01 표현 제한 | §7.7 검사, 금지 표현 시험 세트, 보호자 화면에 판정값·확률 없음 |
| NFR-04 최소 정보 전송 | §7.1 모델별 입력 제한, 공급자 요청 필드 감사 로그 |
| NFR-06 중단 안정성 | §3.6 epoch·출력 폐기, §4.2 종료 상태 불가역, §4.5 멱등·순서, 기기 자체 재생 중지 |
| NFR-09 참여 압박 금지 | 승인 문구·화면 문구 검토 목록, 연속 이용 보상·완료율 표시 없음 |
| NFR-11 기기 독립성 | §5.5 기기 프로토콜 고정. 부모 앱은 기기 형태에 의존하지 않음 |
| 공급자 교체 (요구사항에서 이관) | LiveVoiceAdapter(GPT-Live / Realtime API), JudgeClient(제브 / 대체), LlmClient(OpenAI), KnowledgeAdapter(Neo4j / PostgreSQL), ScriptSpeaker(GPT-Live 낭독 / 클립). 외부 SDK 타입은 Adapter 밖으로 나가지 않는다 |
| 기기 인터페이스 명세 (요구사항에서 이관) | contracts/의 기기 프로토콜 스키마 |
| 비용 추적 (요구사항에서 이관) | usage_log: 세션별 GPT-Live 사용량(session.closed), 제브 호출 수·토큰, 분석 AI 토큰 |
| 응답 지연 측정 (요구사항에서 이관) | 아이 발화 끝 → 첫 출력 음성 전송까지, 판정 턴은 전사 대기·제브 응답·지시 뒤 첫 음성으로 나눠 기록. 목표값 없이 측정부터 한다 |

## 10. 시험 전략

| 수준 | 대상 | 도구 |
|---|---|---|
| 단위 | 세션 상태 전이표, 연습 상태 기계, 종료 표현 일치, 확신 문턱·시간 제한 대체 경로, 장면 결과 라벨 대응, 금지 표현 검사 | JUnit 5, Vitest |
| 계약 | 보호자 서버 ↔ 아동 서비스 명령·이벤트, 기기 프로토콜, OpenAPI 명세와 생성 타입 | contracts/ 스키마 검증 |
| 통합 | 시작 → 종료 확인 → 결과, 재전달·중복 제거, 동의 철회·아동·계정 삭제 연쇄 | Testcontainers(PostgreSQL, Neo4j) |
| 시나리오 | 합성 페르소나 대본을 입력 전사로 주입하는 하네스. 가짜 GPT-Live·가짜 제브·가짜 LLM 사용 | JUnit + 대본 파일 |
| 장애 주입 | AC-CON-01~10: 앱 종료, 앱만 오프라인, 음성 모델 연결 장애, 기기 연결 장애, 복구 성공·실패, 종료 확인 미수신, 공백·장애 경계, 지연 알림, 서버 간 미수신 | 기기 연결·GPT-Live 연결·내부 채널·SSE를 각각 끊는 시험 도구 |
| 음성 수동 | 실제 마이크·스피커로 종료 표현, 재생 중 끊기, 에코 재유입, 판정 턴 공백 | 수동 체크리스트 |
| PoC | 제브 한국어 아동 발화 정확도·확신 문턱, GPT-Live 출력 게이트, 승인 원문 낭독 일치율 | ai/ 스크립트 |

- CI에서는 실제 유료 API를 부르지 않는다. 실제 AI 시험은 따로 실행한다.

- 시나리오 하네스 필수 사례: 중복 시작, 상황 준비 중 취소와 참여 전 취소와 진행 중 종료 구분, 기본·추가 종료 표현과 역할 대사·휴식 요청 구분, 첫 반응 전 단서 차단, 판정 보류에서 교정 없음, 도움 뒤 반응이 첫 반응을 덮어쓰지 않음, 시작 전 미참여 무기록, 동의 철회 뒤 수신 자료 미저장, 리포트 인용 일치·결측 표시, GPT-Live 요청에 판정 정보·보호자 입력 원문 없음.

- 합성 결과를 실제 아동 성능으로 보고하지 않는다.

## 11. 배포와 개발 환경

### 11.1 시연 서버

| 컨테이너 | 내용 |
|---|---|
| nginx | HTTPS 종단, 경로 분배 |
| web | Next.js (/parent, /device) |
| core-api | 보호자 서버 |
| realtime-activity | 아동 서비스 |
| postgres | 스키마 core·activity, 서비스별 DB 사용자 |
| neo4j | 화용 지식 |

- 볼륨: PostgreSQL 데이터, Neo4j 데이터, 원음 파일(EBS).

- EC2는 Stop으로 멈추고 Terminate를 일상 중지 방법으로 쓰지 않는다.

- 한 대에서 Spring 2개 + Next.js + PostgreSQL + Neo4j의 메모리를 실측해 인스턴스 크기를 정한다. 🧪

Nginx 경로

| 경로 | 보내는 곳 | 비고 |
|---|---|---|
| / | web |  |
| /api/, /oauth2/, /login/oauth2/, /device-api/ | core-api |  |
| /api/sessions/\*/stream | core-api | SSE: proxy_buffering off, 긴 읽기 시간 제한 |
| /ws/device | realtime-activity | WebSocket Upgrade |
| /internal/ | 차단 | 두 서버는 Docker 내부 네트워크로 직접 통신 |

HTTPS

- 원격 시연에서 브라우저 마이크, 서비스 워커, 웹 푸시는 모두 HTTPS가 필요하다.

- 방식 ⬜. 후보: 도메인 + Let's Encrypt / 무료·임시 서브도메인 / IP 인증서.

### 11.2 로컬 개발

- Next.js :3000, core-api :8080, realtime-activity :8081. Nginx 없이 실행한다.

- compose.dev.yml: PostgreSQL, Neo4j.

- localhost는 브라우저 보안 컨텍스트 예외를 쓴다.

- CORS는 개발 환경에서만 허용 Origin을 명시하고 allowCredentials(true).

### 11.3 설정·마이그레이션·CI/CD

- 환경 변수: OPENAI_API_KEY, JEV_API_KEY, GOOGLE_CLIENT_ID·GOOGLE_CLIENT_SECRET, VAPID_PUBLIC_KEY·VAPID_PRIVATE_KEY, INTERNAL_SHARED_SECRET, DB 접속 정보. 저장소에 넣지 않는다.

- DB 마이그레이션: 서비스마다 Flyway(core, activity). JPA는 운영 스키마를 바꾸지 않는다.

- CI(GitHub Actions): PR마다 build, test, lint, ArchUnit. OpenAPI 명세와 생성 타입 불일치 검사 추가 여부 🧪.

- CD: 수동 deploy.sh. 자동 CD는 필수 범위가 아니다.

### 11.4 AI 개발 도구 공통 규칙 (AGENTS.md에 적을 것)

- Java 21, Spring Boot 4.1.x 기준. Jackson 3 기준. @MockBean 대신 @MockitoBean. Spring Boot 3.x 예제는 4.1 호환을 확인하고 쓴다.

- 보호자 인증은 서버 세션. JWT 라이브러리를 넣지 않는다.

- OpenAI 호출은 openai-java를 직접 쓴다. OpenAI Spring Boot starter를 쓰지 않는다. SDK 모델 객체를 Controller 응답으로 그대로 반환하지 않는다. 우리 코드에서 com.fasterxml.jackson.databind를 직접 쓰지 않는다(SDK 내부 Jackson 2는 허용).

- springdoc은 Spring Boot 4용 3.x.

- 프론트엔드 서버 조회는 TanStack Query, API 타입은 openapi-typescript 생성물만 쓴다.

- 아이에게 들리는 승인 문구·고정 대본은 ScriptSpeaker를 거친다. TTS·STT 라이브러리를 넣지 않는다.

- GPT-Live 요청에 판정 기준·판정 라벨·확률·이전 판정 결과·보호자 입력 원문을 넣지 않는다.

- 아동 발화 원문을 애플리케이션 로그에 남기지 않는다.

## 12. 요구사항 추적

| 요구사항 | 보호자 서버 | 아동 서비스 | 부모 앱 | 아동 기기 | 이 문서 |
|---|---|---|---|---|---|
| FR-A01~A05 (로그인·아동·동의·고지) | ● |  | 01·02·02-1·03 |  | §5.1, §8 |
| FR-A06·A07·A11 (기기·홈·등록) | 등록·토큰·시작 조건 | 기기 연결·관측 | 04·05·05a | D0·T19 | §4.6, §5.5, §8 |
| FR-A08·A12·A13 (철회·삭제) | ● 연쇄 삭제 | 종료·토큰 무효 | 02·P4·SET-00 |  | §6.5 |
| FR-A09 (종료 표현) | 저장 | 인식 | SET-01 |  | §3.5 |
| FR-A10 (푸시) | ● 감지·발송 | 장애·종료 이벤트 | 알림 선택 → 최신 상태 |  | §4.8, §13 |
| FR-B01~B11 (자유대화) | 시작·종료 요청, 결과 | 대화 실행 | 08·12·P1~P3·17 | 음성 | §3, §4 |
| FR-F03 (자유대화 주도) | 주제 힌트·이전 기록 요약·질문 은행 | GPT-Live 지시 |  |  | §5.3, §7.4, §7.8 |
| FR-C01~C09 (연습·목표·생활 팁) | 목표·대본·결과 | 연습 상태 기계 | 18·21·23·27 | 음성 | §4.3, §7.4 |
| FR-D01~D06 (기록·리포트) | ● |  | 26~29 |  | §5.1, §7.4~7.6 |
| FR-E01~E11 (아동 기기 대화) |  | ● |  | 음성 입출력 | §3, §4.3, §4.4 |
| FR-E16 (아동 기기 화면) |  | 표시 상태 |  | ● | §5.5 |
| FR-F01·F11·F14·F18 (상태·종료·재생·관측) | 저장·표시 | ● | 12·21·27 | 재생 중지 | §3.6, §3.7, §4 |
| FR-F02·F15·F20 (승인 문구·역할 분리·판정) | J7 | ● |  |  | §3.2~~3.5, §7.1~~7.3 |
| FR-F04 (전사·원음) | 저장·중계 | 수신·전달 | 12·21·27 |  | §5.2, §5.4, §6.3 |
| FR-F05~F10·F12·F17·F19 (구조화·목표·대본·재판정·결과) | ● 분석 AI |  | 27 |  | §7.4~7.6 |
| NFR | §9 | §9 |  |  | §9 |
| AC-CON-01~10 |  |  |  |  | §10 장애 주입 |

## 13. 값 목록

요구사항·기능명세서에서 'ADR 값'으로 넘긴 값이다. 잠정값(⏱ 중 '잠정')을 초기값으로 그대로 쓸지는 TD-09.

| 항목 | 근거 | 값 | 상태 |
|---|---|---|---|
| 공백 가드레일 | FR-F18 | 연속 4분 → 내부 공백 상태, 추가 2분 → 자동 종료 | ✅ 요구사항 확정값 |
| 목표당 연습 시간 | 공통규칙 '연습 분기' | 10분 | ⏱ |
| 연습 회차 상한(한 활동 전체 시간) | 공통규칙 '연습 회차 시간' | — | ⬜ |
| 응답 공백 허용 시간 | FR-F20 | 10초 | ⏱ |
| 전사 도착 제한 시간 | FR-F20 | 1.5초 (후보, 10초 안) | 🧪 |
| 제브 응답 제한 시간 | FR-F20 | 800ms (후보, 10초 안) | 🧪 |
| 제브 확신 문턱 | FR-F20 | 1순위 확률 하한, 1·2순위 차이 | 🧪 PoC |
| 제브 입력의 '최근 몇 턴' | FR-F20 | — | ⬜ |
| 판정 턴 맞장구 문구 | FR-F20 | 결정 전 기준: 2초 공백에 맞장구 없음 | ⬜ |
| 발화 끝 판정 무음 길이 | §3.7 | — | 🧪 |
| 판정 입력 전사 | FR-F04·F20 | GPT-Live 입력 전사만. 별도 STT 없음 | ✅ |
| 오디오 형식 | §3.1 | PCM16 mono 24kHz | 🟡 |
| 연습 무응답 대기 | 공통규칙 '무응답' | 재생 완료 후 10초 → 1회 재안내 → 10초 (잠정) | ⏱ |
| 시작 안내 무응답 대기 | FR-E02 | 4분+2분 / 1회 재안내 후 10초 (잠정) 중 선택 | ⬜ |
| 장애 판정 시간(연결별) | D-CON-03 | — | ⬜ |
| 상태 정보 유효시간('응답 없음' 판정 포함) | D-CON-04 | — | ⬜ |
| 복구 대기시간·재시도 횟수 | D-CON-05 | 기기 재연결 30초 (잠정) | ⏱ |
| 알림 발생·중복 억제 기준 | D-CON-06 | 같은 세션·같은 사건은 한 번(dedup_key). 발생 조건 세부 — | 🟡 / ⬜ |
| 복구 후 재개 조건 | D-CON-07 | 종료 전, 미해결 종료 요청 없음, 같은 기기 토큰·같은 세션, 활동 상태·대화 맥락 복원 확인 | 🟡 / 맥락 복원 방법 🧪 |
| 연결 준비 판정 시간 | 공통규칙 '활동 시작 조건' | — | ⬜ |
| 처음 기기 응답 없음 판정 시간(시작하지 못함 → 08·18) | FR-B05·C05 | — | ⬜ |
| 종료 확인 대기·재확인 간격 | FR-B07 | 명령 후 5초에 첫 확인 (잠정) | ⏱ |
| GPT-Live 종료 대기 | §4.4 | session.closed까지 최대 15초 | 🟡 |
| 결과 자동 재시도 | FR-F17 | 추가 최대 3회, 5·15·30초 간격 (잠정) | ⏱ |
| 아동 서비스 미전달 데이터 보관 한계 | FR-F01 | — | ⬜ |
| 등록 코드 형식 | FR-A11 | — | ⬜ |
| 기기 토큰 만료·재발급·등록 해제 | FR-A11 | — | ⬜ |
| 여러 목표의 순서 가중치(즐겨찾기·담은 순서) | FR-C02 | — | ⬜ |
| 상황 다시 설명·집중 연습 반복 횟수, 집중 연습 길이 | FR-E08, G-16 | — | ⬜ |
| 로그인 세션 만료 | FR-A01 | Spring Session JDBC, 만료 시간 — | 🟡 / ⬜ |
| 실시간 전사 중계 방식·지연 목표 | FR-B06·C06 | 내부 WebSocket → SSE (§5.2·§5.4), 지연 목표 — | 🟡 / ⬜ |
| 홈·기기 상태 조회 주기 | FR-A06·A07 | — | ⬜ |
| 아이 기기 에코 제거 | FR-E16·F20 | 브라우저 에코 제거 + 재생 음성 재유입 거르기(§3.8) | 🧪 |

## 14. 미결·검증 과제

### 14.1 팀 결정 대기

| 항목 | 내용 | 결정 전 기준 |
|---|---|---|
| TD-01 (D-CON-09) | 음성 대화 장애 중 아이에게 들려줄 고정 안내의 제공 여부·문구·횟수 | 제공하지 않음 |
| TD-04 후속 | 4분 공백 동안 기기가 다시 말을 걸지 | 정하지 않음 |
| TD-09 | 잠정값(10초·30초·5초 등)을 초기값으로 쓸지 | §13의 후보로만 |
| TD-12 (D-CON-10) | 상태 전환 때 공백 타이머 초기화·재시작 | 미확인 시간을 소급 누적하지 않음 |
| TD-16 | 목표가 여럿일 때 장면 생성 시점과 실패 때 세션 처리 | 시작 전에 모두 생성 |
| TD-20 | 아동 삭제 때 기록을 함께 지울지 | 함께 삭제(되돌릴 수 없음) |
| 생활 팁 검사 기준 | 생활 팁의 표시·생성 금지 자동 검사 기준 | CR-SAFE 목록으로 검사, 통과분만 표시 |
| 시작 안내 중 기술 문제 (요구사항 빈칸) | 참여 전에 음성 대화 장애의 복구가 실패하면 26에 기록으로 남길지 | 기술 문제(TECHNICAL)로 종료하고 P2를 띄운다. 26 표시 여부 미정 |
| 시작 안내 중 종료 표현 (요구사항 빈칸) | 참여 질문에 아이가 기본·등록 종료 표현을 말한 경우 | 시작 전 미참여(거절)로 처리, 마무리 문구 1회 🟡 |

### 14.2 AI 동작 명세에서 정할 것

G-03 배경 정보의 대화 활용 형태, G-06 부분 결과와 자료 부족 기준, G-08 종료 의도 확인 문구·횟수·무응답 처리, G-09 목표 전환 안내의 승인 문구 여부와 목표가 여럿일 때 장면 생성 시점(TD-16과 함께), G-16 J6 대상 발화 범위와 일반 턴 자동 응답과의 순서(상황 다시 설명·집중 연습 반복 횟수의 값은 §13).

### 14.3 첫 볼트·PoC로 확인할 것

| 항목 | 확인할 것 | 기준에 못 미치면 |
|---|---|---|
| GPT-Live 출력 게이트 | 판정 턴에서 자동 응답을 버리고 지시 뒤 응답만 내보낼 수 있는지, 버린 응답이 맥락을 흐리는지, 지연 | 요구사항 변경 후 OpenAI Realtime API로 교체 검토 🟡 (LiveVoiceAdapter) |
| 발화 끝·전사 확정 | 입력 전사 delta와 VAD로 발화 끝을 안정적으로 잡는지 | 무음 길이 조정, Realtime API 검토 |
| 승인 원문 낭독 | 한국어 승인 문구 반복 낭독 일치율, 지시부터 첫 음성까지 지연 | 해당 문구만 사전 녹음 클립 |
| openai-java의 GPT-Live 지원 | Live WebSocket 지원 범위 | JDK WebSocket 직접 구현 |
| 음성 모델 연결 재연결 | 새 GPT-Live 세션에 맥락을 복원할 수 있는지 | 복원 불가면 기술 종료(CONTEXT_RESTORE_FAILED) |
| 제브 | 대기 명단 승인, API 필드, 한국어 아동 발화 정확도, 확신 문턱(§7.3 PoC 계획) | 요구사항 변경 후 JudgeClient 대체 구현 검토 🟡 |
| 에코 | 노트북·태블릿 스피커 재유입률 | 재생 중 입력 차단(반이중) |
| Spring 조합 | Spring Boot 4.1 + Security OAuth2 + Session JDBC + springdoc 3 + Flyway + Testcontainers + openai-java(Jackson 2) 공존 | Spring Boot 3.5.x |
| EC2 메모리 | 컨테이너 6개 동시 실행 | 인스턴스 크기 조정, Neo4j 대신 PostgreSQL |
| 웹 푸시 | 안드로이드·iOS 수신. iOS는 홈 화면에 추가한 웹 앱에서만 받는다 | 앱 안 알림으로 보완 |
| 시연 장소 네트워크 | 실시간 음성 품질 | 사전 점검 |

### 14.4 팀 확인이 필요한 제안 (🟡)

Java 21·Spring Boot 4.1.x, PostgreSQL·Flyway, Spring Session JDBC, Neo4j 사용 범위, 서버 간 내부 WebSocket + outbox, 부모 앱 SSE, 기기 프로토콜, 원음 WAV 파일 볼륨, Web Push, 서버 PDF 생성, 저장소 구조·모듈, 실제 삭제, 공개 저장소 여부.

## 15. 외부 API 참고

| 대상 | 문서 |
|---|---|
| GPT-Live WebSocket | [<u>https://developers.openai.com/api/docs/guides/voice-websockets?api=live</u>](https://developers.openai.com/api/docs/guides/voice-websockets?api=live) |
| GPT-Live 서버 측 제어 | [<u>https://developers.openai.com/api/docs/guides/voice-server-controls?api=live</u>](https://developers.openai.com/api/docs/guides/voice-server-controls?api=live) |
| GPT-Live 위임·도구 | [<u>https://developers.openai.com/api/docs/guides/live-delegation</u>](https://developers.openai.com/api/docs/guides/live-delegation) |
| OpenAI Realtime API (대체 경로) | [<u>https://developers.openai.com/api/docs/guides/realtime-websockets</u>](https://developers.openai.com/api/docs/guides/realtime-websockets) |
| openai-java | [<u>https://github.com/openai/openai-java</u>](https://github.com/openai/openai-java) |
| 제브 (Jev) | 기본 주소 [<u>https://api.typesafe.ai</u>](https://api.typesafe.ai), 콘솔 [<u>https://console.typesafe.ai</u>](https://console.typesafe.ai) (공식 API 문서는 계정 승인 뒤 확인) |

## 16. 변경 이력

| 버전 | 날짜 | 내용 |
|---|---|---|
| v0.2 | 2026-10-08 | 화면 설계 최종안 v1.5, 요구사항 명세서 v1.4, 기능명세서 v1.3, ADR-002 v12를 기준으로 다시 작성. 백엔드를 보호자 서버·아동 서비스(둘 다 Spring Boot)로 구성(이전: core-api + FastAPI ai-service). 음성은 GPT-Live 서버 중계, 별도 STT·TTS 없음(이전: STT→LLM→TTS 연쇄). 진행 중 판정은 제브, 활동 전·후 처리는 분석 AI. 원음 보관(필수 동의), 기기 등록 코드·기기 토큰, 구글 로그인·서버 세션(이전: JWT), 실시간 전사 중계, 연습 2안 상태 기계 반영. 쉬기·건너뛰기·재개 명령과 키워드 호출어 방식 삭제. ADR-002 v12의 기술 기본값과 ADR-004·005·006에 맡기던 내용을 이 문서로 옮김. 역할 분담·일정은 WBS로 옮김. AI 동작 명세 v0.1과 맞춤(상황 다시 설명·집중 연습 요점·표현 예시를 장면 대본에 포함, 'AI 상세 명세'를 'AI 동작 명세'로 부름) |
| v0.1 | 2026-09-29 | 요구사항 명세서 v0.4.2 기준 첫 작성 |
