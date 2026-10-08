<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->
- 2026-10-08T08:20:00Z — 요구사항 ID는 단계 안내의 `FR{n}` 대신 원문 ID(FR-A01, NFR-01 등)를 그대로 쓴다; 사용자가 요청 진행 방식 1에서 원문 ID를 추적 키로 쓰라고 지시했다.
- 2026-10-08T08:20:00Z — 이번 단계 질문은 동작을 바꾸는 팀 결정 대기(TD) 항목과 목표값 없는 비기능 항목으로 한정하고, §13의 값(⬜·🧪)은 해당 설계 단계로 넘긴다; 사용자가 "그 항목은 해당 단계에서 묻는다"고 했다.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->
- 2026-10-08T09:10:00Z — requirements.md는 카드 내용을 다시 옮겨 쓰지 않고 ID·우선순위·화면·한 줄 요약과 원본 카드 참조로 정리했다; 카드가 원본이고 완료 기준·하지 않는 것을 중복하면 두 문서가 어긋날 위험이 있다.
- 2026-10-08T09:10:00Z — 질문 수가 Standard 기준(5~8)보다 많은 12개(후속 1개 포함)였다; 팀 결정 대기 항목(TD)이 모두 요구사항 동작을 바꾸는 것이라 묶지 않았다.

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
