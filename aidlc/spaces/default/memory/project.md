# Project-Level Rules

> Project-specific specialisation and corrections. Loaded after `org.md` and
> `team.md` as strict-additive guidance; contradictions with broader policy
> are rejected. Populated by practices-discovery and the self-learning loop.
>
> Use sparingly: most teams don't need a project layer. Reach for it
> only when this specific project needs stable, durable guidance beyond the
> team practice (for example, package-specific release checks or an additional
> regression suite for a legacy component).

## Way of Working

<!-- Project-specific specialisation. Example: -->
<!-- This monorepo requires package-scoped branch names and a package owner -->
<!-- review in addition to the team's normal merge policy. -->

## Walking Skeleton

<!-- Project-specific specialisation. Example: -->
<!-- The walking skeleton must exercise the legacy service adapter as well -->
<!-- as the new service boundary. -->

## Testing Posture

<!-- Project-specific specialisation. -->

## Guard Policy

<!-- Project-specific. Mode: strict, relaxed, or off. Strict here holds for every intent and cannot be changed from chat. A section under the retired Change Control heading, written by an earlier release, is still read. -->

## Deployment

<!-- Project-specific specialisation. -->

## Code Style

<!-- Project-specific specialisation. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

## Forbidden

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: NEVER [behavior] (affirmed [date]) -->
<!-- Example: NEVER throw exceptions across service layer boundaries (affirmed 2026-05-17) -->

- NEVER API 키, 내부 공유 비밀, `.env`, 인증서 개인 키를 저장소나 AI-DLC 입력에 넣지 않는다(2번, 기술명세 §2.4·§11.3). (affirmed 2026-10-08)

- NEVER 실제 아동 자료, 실제 대화·전사를 저장소나 AI-DLC 입력에 넣지 않는다(3번, 기술명세 §2.4·§8). (affirmed 2026-10-08)

- NEVER 비밀 키(OPENAI_API_KEY, JEV_API_KEY, GOOGLE_CLIENT_SECRET, VAPID_PRIVATE_KEY, INTERNAL_SHARED_SECRET 등)를 브라우저에 보내거나 프론트엔드 번들에 넣지 않는다. 단, 공개용 값인 VAPID 공개 키와 구글 클라이언트 ID는 예외다(4번, 기술명세 §8). (affirmed 2026-10-08)

- NEVER 아동 발화 원문·전사·원음과 AI 요청·응답 본문을 애플리케이션 로그에 남기지 않는다. 로그에는 sessionId 같은 식별자만 남긴다(5번, 기술명세 §8·§11.4). (affirmed 2026-10-08)

- NEVER GPT-Live 요청에 판정 기준·판정 라벨·확률·이전 판정 결과·보호자 입력 원문을 넣지 않는다(7번, 기술명세 §11.4, 타당성 C-T05). (affirmed 2026-10-08)

- NEVER CI에서 실제 유료 AI API를 부르지 않는다. CI는 가짜 모델만 쓴다(8번, 기술명세 §10, ADR-002). (affirmed 2026-10-08)

- NEVER 합성 데이터로 얻은 결과를 실제 아동 성능으로 보고하지 않는다(9번, 기술명세 §10). (affirmed 2026-10-08)

- NEVER CSRF 보호를 끄지 않는다(10번, 기술명세 §8). (affirmed 2026-10-08)

- NEVER `/internal/` 경로와 내부 서버 간 채널을 Nginx 밖으로 노출하지 않는다(10번, 기술명세 §8·§11.1). (affirmed 2026-10-08)

- NEVER AI(무중) 출력 음성을 저장하지 않고, 아동 원음 바이트를 DB·로그·임시 파일에 두지 않는다. 아동 원음은 원음 보관 동의에 따라 정한 원음 저장소에만 둔다(11번, 기술명세 §2.3, ADR_2S DL-012-01/02). (affirmed 2026-10-08)

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

- ALWAYS OpenAI·제브·구글·VAPID 키와 내부 공유 비밀, DB 접속 정보는 서버 환경 변수로만 넣는다(1번, 기술명세 §8·§11.3). (affirmed 2026-10-08)

- ALWAYS 데모·시험·PoC에는 합성 데이터와 성인 역할극만 쓴다(3번, 기술명세 §8, 타당성 C-R01). (affirmed 2026-10-08)

- ALWAYS 모든 요청에서 로그인한 보호자가 해당 아동·세션·기록·목표·리포트의 소유자인지 서버에서 확인한다(6번, 기술명세 §8). (affirmed 2026-10-08)

- ALWAYS 기기 토큰은 해시로만 저장한다(10번, 기술명세 §8). (affirmed 2026-10-08)

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->
- 구현 단계 전 Unit은 5인 병렬 개발이 가능하도록 나눈다(의도 파악 Q11 추가 지시). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:intent-capture:d1bdf53e909d43c1889c26af009837d737d9b422759e24fe90cdd6c72f0d3c4b -->
- 주 입력 문서는 aidlc/spaces/default/knowledge/documents/02_요구사항_입출력명세_v1.4.md를 쓴다(docs/ 경로는 없음, 의도 파악 Q1). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:intent-capture:c9c787fb39163bd7e3cfdf6b5f79e1e2c6638978944b8cfc7c487cff6b8aa8b6 -->
- PoC 미달 시 대체 경로 결정 기간 '19~23일'은 10/19~10/23이다(타당성 Q3·Q9). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:feasibility:fa524bab232aa70638577000672a0973521490fdac974d9136297a04401d871b -->
- 이 MVP 진행 계획에는 운영 7단계(4.1~4.7)를 포함한다(총 29단계, 타당성 단계 중 사용자 지시). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:feasibility:746a6339da7ee95f3171f25e169127c89d0d3e9f4e9b53cb4403b0f5530e8246 -->
- 참고 문서에 확정(✅)으로 적힌 내용은 다시 묻지 않고, 제안(🟡)·PoC(🧪)·미정(⬜) 항목만 질문한다. (learned 2026-10-08) <!-- cid:261008-muzung-mvp:feasibility:124a99fdb26471f4896af3476765cd768373d4540df43fce36412a9cee6109a5 -->
- 화면은 새로 그리지 않고 화면 설계 v1.6을 유일한 원본으로 두어 화면 번호로 참조한다(대략 화면 Q1). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:rough-mockups:26429501014ed5a1c50b806ce292c5ffdbbb8aa205b4315e9a50d91c16e6fbd8 -->
- 문서가 서로 다르면 화면 설계 → 요구사항 → 기능명세서 → ADR_2S → 기술명세서 → AI 동작 명세 순으로 앞의 문서를 따른다. 화면·요구사항과 다른 ADR 내용은 따르지 않고 이후 단계에서 확인한다(팀 작업 방식 Q15). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:practices-discovery:f3757ebf1974758929bc0f52e84a5daa9ad5c5403ffa5eece04e5740eba248b4 -->
- 강한 규칙(Mandated/Forbidden)은 보안·개인정보 규칙만 두고, 기술 관례는 팀 작업 방식의 일반 관례로 둔다(팀 작업 방식 Q14). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:practices-discovery:aa68325bf351d58dcaa532f8cea542193d0eb3b203244bd86dd9475e5a516cb3 -->
- 요구사항 ID는 단계 안내의 `FR{n}` 대신 원문 ID(FR-A01, NFR-01 등)를 그대로 추적 키로 쓴다(요구사항 분석). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:requirements-analysis:b813a6897be74b473cd0c93f4d67d44af90fd7e0b5baef4de93b2105a29846a7 -->
- 질문은 요구사항 동작을 바꾸는 팀 결정 대기(TD) 항목과 목표값 없는 비기능 항목으로 한정하고, 기술명세서 §13의 값(⬜·🧪)은 해당 설계 단계에서 묻는다(요구사항 분석). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:requirements-analysis:a853482e036ac18e4a97e4d3cdded03a90a12a5e357a3e2389045ec39791060b -->
- 산출물은 요구사항 카드를 다시 옮겨 쓰지 않고 ID·우선순위·화면·한 줄 요약과 원본 카드 참조로 정리한다(요구사항 분석). (learned 2026-10-08) <!-- cid:261008-muzung-mvp:requirements-analysis:064c5e18e16cf9649722e6bd7b629de9622cf3f23b68c51f50c8b969e93f9ae0 -->
