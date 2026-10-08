<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
<!-- example: 2026-05-29T10:14:32Z — chose REST over GraphQL; the consuming team only needs CRUD, revisit if subscriptions land -->
- 2026-10-08T07:45:34Z — ADR_2S를 기술명세서 자리(화면→요구사항→기능명세→ADR→기술명세→AI)에 두기로 한 결정(Q15)에 따라 승인된 단계는 유지하고 ADR 차이는 이후 단계로 넘김; 충돌 21건은 evidence.md와 대조 보고서에 기록했다.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->
- 2026-10-08T07:45:34Z — 병합 커밋·자동 배포·Java 포매터 없음은 org 기본값·기술명세서·ADR-006과 다른 팀 결정으로 기록; 사람이 질문에서 직접 고른 답이다.

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->
- 2026-10-08T07:45:34Z — 강한 규칙을 보안·개인정보 11개로 한정하고 기술 관례 12개는 일반 관례로 둠(Q14=B); 4주 MVP에서 기술 선택 변경 여지를 남겼다.

## Open questions
<!-- example: 2026-05-29T10:14:32Z — confirm the retention window with compliance before the next stage hardens the schema -->
