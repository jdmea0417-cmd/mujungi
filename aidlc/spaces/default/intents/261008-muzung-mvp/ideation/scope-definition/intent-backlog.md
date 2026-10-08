# 의도 백로그 (예비 작업 묶음) — 무중 MVP

이 백로그는 범위 문서(`scope-document.md`)의 범위를 동선별 예비 작업 묶음으로 나눈 것이다. 입력은 의도 정리서(`ideation/intent-capture/intent-statement.md`), 타당성 평가서(`ideation/feasibility/feasibility-assessment.md`), 제약 목록(`ideation/feasibility/constraint-register.md`)이다. 실제 Unit 경계와 의존 관계는 Unit 나누기 단계(2.7)에서 정하고, Bolt 순서는 납품 계획 단계(2.9)에서 정한다. 여기서는 우선순위와 순서 원칙(위험 먼저, scope Q7)만 정한다.

## 우선순위 방식

- MoSCoW는 요구사항 정의서의 우선순위를 그대로 쓴다. Must는 줄이지 않고(intent Q9), Should는 모두 포함한 뒤 10/23에 재검토하며(scope Q1), Could는 맨 뒤에 둔다(scope Q3).
- 순서는 위험 먼저다. 외부 AI 의존과 팀 경험 부족이 겹친 묶음을 앞에 둔다(scope Q7, RAID R-01~R-03).

## 예비 작업 묶음

| 순서 | 묶음 | 포함 요구사항 | Must / Should / Could | 위험 | 비고 |
|---|---|---|---|---|---|
| P0 | AI 연동 PoC | FR-F20(제브 판정), FR-F14(재생·중단), GPT-Live 출력 게이트·발화 끝·에코·재연결(기술명세서 §14.3) | 판정·재생 Must | 높음 | 10/13까지 GPT-Live 먼저, 제브는 승인 즉시(타당성 Q9) |
| P1 | 아동 서비스 핵심 — 세션·연결 | FR-F01, F18, E11, E01, E02, E05, E16, NFR-06 | Must 8 | 높음 | 가짜 모델로 PoC를 기다리지 않고 시작 가능 |
| P2 | 활동 진행 — 자유대화 | FR-B01, B02, B05, B06, B07, B08, E03, F02, F03, F04, F15, NFR-04, NFR-09 / FR-B03, B10, E04 | Must 13 / Should 3 | 높음 | GPT-Live 의존 |
| P3 | 연습 | FR-C05, C06, C07, E06, E07, E08, E09, E10, F07, F09 / FR-C08, F08 | Must 10 / Should 2 | 높음 | 제브 의존 |
| P4 | 보호자 준비 | FR-A01, A02, A03, A04, A06, A07, A10, A11 / FR-A05, A08, A09, A12, A13 | Must 8 / Should 5 | 중간 | PoC와 무관 — 병렬 시작 |
| P5 | 기록·목표 | FR-D01, D02, F05, F06, F10, F11, F19, C01, C02, C03, NFR-01 / FR-C09, F17 | Must 11 / Should 2 | 중간 | 분석 AI 의존(텍스트 LLM), 가짜 모델로 먼저 개발 가능 |
| P6 | 리포트 PDF | FR-D05, D06, F12 | Must 3 | 낮음 | PoC와 무관 — 병렬 시작 |
| P7 | Could | FR-B11, D04, NFR-11 | Could 3 | 낮음 | 맨 뒤(scope Q3) |

건수 확인: Must 8+13+10+8+11+3 = 53에 P0의 FR-F20·F14를 더해 55, Should 3+2+5+2 = 12, Could 3.

## 병렬 개발 메모 (5인, intent Q11)

- PoC에 의존하지 않는 P4·P6과 가짜 모델로 시작할 수 있는 P1·P5는 첫 주부터 병렬로 진행할 수 있다.
- P2·P3은 P0 결과(10/13, 대체 경로 결정 10/19~10/23)에 따라 내용이 바뀔 수 있다.
- 실제 담당 배치와 Unit 경계는 Unit 나누기·납품 계획 단계에서 정한다.

## 10/23 재검토 항목

- Should 12건 중 연기할 항목(scope Q1)
- PoC 미달 시 대체 경로와 그에 따른 요구사항 변경(타당성 Q3)
