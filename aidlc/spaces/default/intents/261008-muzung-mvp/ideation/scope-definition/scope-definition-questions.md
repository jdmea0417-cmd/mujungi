# 범위 정의 질문 — 무중 MVP

이 파일은 범위 정의 단계에서 오간 질문과 답을 모두 기록합니다. 질문의 근거는 의도 정리서(`ideation/intent-capture/intent-statement.md`), 타당성 평가서·제약 목록(`ideation/feasibility/`), 주 입력 문서의 요구사항 목록(§7)입니다.

이미 정해진 것은 다시 묻지 않습니다.
- Must 55건은 줄이지 않는다(intent-capture Q9).
- 화면 설계 v1.6을 따르므로 v1.6에서 다시 들어온 항목(아이 삭제, 동의 철회·기록 삭제, 29 리포트 10개 항목 등)도 범위에 든다(intent-capture Q10).
- 운영 7단계를 진행 계획에 포함한다(타당성 단계 중 지시).

각 `[Answer]:` 뒤에 고른 선택지 글자를 적고, 직접 쓰려면 `X` 뒤에 내용을 적습니다.

## Q1. Should 12건을 다루는 원칙

Should는 "단순화·연기할 수 있다"고 정해져 있지만(intent-capture Q9), 지금 무엇을 연기할지는 정하지 않았습니다. 타당성 검토 결과 구현 기간이 PoC 결과(10/19~10/23 대체 경로 결정)에 따라 약 2주로 줄 수 있습니다. Should를 어떻게 다룰까요?

A. 지금은 모두 범위에 넣고, 10/23 PoC 결과를 본 뒤 팀 합의로 연기할 항목을 정한다
B. 지금 일부를 연기 후보로 표시해 두고(Q2에서 고름), 나머지는 범위에 넣는다
C. 지금은 모두 연기하고, Must가 끝난 뒤 여유가 있으면 넣는다
D. Not yet defined
X. Other (please specify)

[Answer]: A

## Q2. 연기 후보로 표시할 Should 묶음

Q1에서 B를 고르는 경우에만 답합니다(다른 답이면 `Not applicable`). 12건을 성격별로 묶었습니다. 연기 후보로 표시할 묶음을 모두 고르세요. (select all that apply)

A. 계정·데이터 관리 — 데이터 삭제·동의 철회(FR-A08), 아동 삭제(FR-A12), 계정 삭제(FR-A13)
B. 안내 문구 — 보호자 안내 문구(FR-A05), 자유대화 불완전 종료 안내(FR-B10), 연습 부분 종료 안내(FR-C08)
C. 편의 기능 — 종료 표현 설정(FR-A09), 배경 정보 확인·수정(FR-B03), 다시 듣기·속도 조절·설명 요청(FR-E04), 변형 장면 중복 방지(FR-F08)
D. 결과 보강 — 생활 팁(FR-C09), 결과 생성 재시도(FR-F17)
E. Not applicable
X. Other (please specify)

[Answer]: E

## Q3. Could 3건

Could는 "여유가 있을 때 한다"고 정해져 있습니다(intent-capture Q9). 대상은 참여 거부 안내(FR-B11), 다음 활동 선택(FR-D04), 기기 독립성(NFR-11)입니다. 범위 문서에 어떻게 적을까요?

A. 이번 범위 밖(Won't this time)으로 적고, Must·Should가 끝나면 다시 검토한다
B. 범위에 넣되 가장 낮은 순서로 둔다
C. 일부만 넣는다(X에 항목을 적어 주세요)
D. Not yet defined
X. Other (please specify)

[Answer]: B

## Q4. 성공 기준과 Must 완료 기준의 관계

성공 기준은 "11/5에 전체 흐름 1회 이상 끊김 없이 완주"이고(intent-capture Q4), Must는 줄이지 않습니다(Q9). 의도 파악 검토에서 이 둘의 관계가 정해지지 않았다는 의견(R-02)이 있었습니다. 11/5에 일부 Must의 '완료 기준'(수용 테스트)을 통과하지 못하면 어떻게 보나요?

A. 성공은 전체 흐름 완주로 판단하고, 통과하지 못한 Must 완료 기준은 시연 뒤 보완 목록으로 넘긴다
B. 전체 흐름 완주와 Must 완료 기준 전부 통과가 모두 있어야 범위를 다 했다고 본다
C. 전체 흐름에 들어가는 Must는 완료 기준을 모두 통과해야 하고, 흐름 밖 Must(예외·장애 처리 등)는 보완 목록으로 넘길 수 있다
D. Not yet defined — 요구사항 분석에서 다시 묻는다
X. Other (please specify)

[Answer]: A

## Q5. "끊김 없이"의 판정 기준

성공 기준의 "끊김 없이"가 무엇인지 정해지지 않았다는 의견(R-01)이 있었습니다. 시연에서 무엇을 끊김으로 볼까요? (select all that apply)

A. 오류 화면이나 기술 문제 종료(P2) 없이 흐름이 이어진다
B. 진행 중에 개발자가 서버·DB를 손으로 고치거나 다시 시작하지 않는다
C. 같은 단계를 다시 시도하지 않는다(재시도 0회)
D. Not yet defined — 요구사항 분석에서 다시 묻는다
X. Other (please specify)

[Answer]: A, B

## Q6. 운영 단계의 범위

운영 7단계를 진행 계획에 넣었습니다. 강의장 시연 한 번이 목적이라(intent-capture Q5, 타당성 Q10) 운영 범위를 어디까지 잡을지에 따라 할 일이 크게 달라집니다. 운영 단계에서 다룰 환경은 어디까지인가요?

A. 시연 서버(EC2 1대) 하나 — 배포·관측·장애 대응·성능 검증을 시연 서버 기준으로 한다
B. 로컬 개발 환경 + 시연 서버
C. 별도 시험(스테이징) 서버 + 시연 서버
D. Not yet defined
X. Other (please specify)

[Answer]: A

## Q7. 진행 순서 원칙

Unit 나누기·납품 계획 단계에서 Bolt 순서를 정할 기준이 필요합니다. 타당성 검토에서 PoC 의존 작업과 의존하지 않는 작업을 5명이 병렬로 나눠야 한다는 결론이 나왔습니다. 어떤 원칙을 우선할까요?

A. 위험 먼저 — GPT-Live·제브 PoC와 아동 서비스 핵심을 먼저, 나머지는 병렬
B. 얇은 전체 흐름 먼저 — 보호자 준비부터 리포트 PDF까지 가장 단순한 경로를 먼저 연결하고 기능을 채움
C. 가치 먼저 — 보호자 준비·자유대화·기록을 먼저, 연습·리포트를 뒤에
D. Not yet defined — 납품 계획 단계에서 정한다
X. Other (please specify)

[Answer]: A

## Q8. 기능별 마감

11/5 전체 마감 외에, 특정 기능에 걸린 날짜가 있나요? 타당성 검토에서 정해진 날짜는 PoC 10/13, 대체 경로 결정 10/19~10/23입니다.

A. 없다 — 11/5 전체 마감과 PoC 일정뿐이다
B. 시연 리허설 날짜가 있다(X에 날짜를 적어 주세요)
C. 특정 기능의 완료 날짜가 있다(X에 기능과 날짜를 적어 주세요)
D. Not yet defined
X. Other (please specify)

[Answer]: A

## Consolidated Summary Confirmation

답변 요약:

- Q1 Should 12건: 지금은 모두 범위에 넣고, 10/23 PoC 결과를 본 뒤 팀 합의로 연기할 항목을 정한다
- Q2 연기 후보 묶음: Not applicable(Q1이 A)
- Q3 Could 3건(FR-B11, FR-D04, NFR-11): 범위에 넣되 가장 낮은 순서로 둔다
- Q4 성공 기준과 Must: 성공은 전체 흐름 완주로 판단하고, 11/5에 통과하지 못한 Must 완료 기준은 시연 뒤 보완 목록으로 넘긴다
- Q5 끊김 없음의 조건: 오류 화면·기술 문제 종료(P2)가 없고, 진행 중 개발자의 수동 개입(서버·DB 수정·재시작)이 없다
- Q6 운영 범위: 시연 서버(EC2 1대) 하나를 기준으로 배포·관측·장애 대응·성능 검증을 한다
- Q7 진행 순서: 위험 먼저 — GPT-Live·제브 PoC와 아동 서비스 핵심을 먼저, 나머지는 병렬
- Q8 기능별 마감: 없음 — 11/5 전체 마감과 PoC 일정(10/13, 10/19~10/23)뿐

Does this all look correct before I generate the artifact?

- Looks correct
- Request changes

[Answer]: Looks correct
