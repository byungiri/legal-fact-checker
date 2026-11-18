# 변경 이력

## v2.0.0 — 2026-11-18

Korean-law-mcp 서버 주소 이전(`korean-law-mcp.fly.dev/mcp` → `mcp.gomdori.app/law`, 옛 주소는 2026.11.17.부터 조회 중단)과 도구 체계 개편에 맞춰 다시 썼습니다.

### 바뀐 점

- **도구 체계 교체(§2)**: v1이 안내하던 `chain_law_system`·`chain_full_research`·`chain_action_basis`·`chain_procedure_detail`·`chain_amendment_track`·`chain_ordinance_compare`·`chain_dispute_prep`·`chain_document_review`는 노출 도구에 없습니다. 노출 도구 10종(`search_law`, `get_law_text`, `search_decisions`, `get_decision_text`, `get_annexes`, `legal_research`, `legal_analysis`, `ordinance_radar`, `discover_tools`, `execute_tool`) 기준으로 도구 표·의사결정 트리·호출 예시를 다시 썼습니다.
  - 체인 기능 → `legal_research(task=...)`로 대응표 제공(§2.2)
  - 별표·서식 → `get_annexes`(목록 → `bylSeq` 2단계)
  - 조례 → `ordinance_radar`, 전국 비교는 `legal_research(task="ordinance_compare")`
  - 복합 분석 → `legal_research`·`legal_analysis`
  - 고시·훈령·예규 → `execute_tool`로 `search_admin_rule`·`get_admin_rule`
  - 숨은 전문 도구는 `discover_tools`로 실제 확인한 이름만 표기(§2.3)
- **커넥터 이름 의존 제거(§0)**: "'법정보조회시스템:' 접두어" 대신 도구 구성(`search_law` 등, 설명 `Korean-law-mcp —`)으로 커넥터를 찾습니다. 플러그인·claude.ai 커넥터 어느 쪽으로 연결해도 동작합니다.
- **커넥터 장애 대응 신설(§3.4)**: 미연결·서버 오류·요청 제한 시 1회 재시도 → 실패하면 Mark-and-Proceed(§3.3) 표시 → 사용자에게 커넥터 상태와 인증키(OC) 안내.
- **최종 인용 교차검증 추가(§4.3)**: 초안 완성 후 `legal_analysis(mode="verify_citations")`, 판례는 `mode="cite_check"`.
- **법령번호 확인 경로 명시(§3.2)**: `get_law_text(jo="부칙")` 응답의 `부칙 <제NNNNN호, 공포일>`.
- **산출 예시 정정(§9)**: v1 예시의 감리원 배치·자격 근거("제8조의2", "별표 4의2")를 2026.11.18. 조회로 확인한 근거(법 제8조, 시행령 제10조 제1항·별표 2)로 바꿨습니다.
- **안티패턴 2개 추가(§7)**: 없는 도구 이름 지어내기, 장애 시 내부 지식으로 대체하기.
- **협업 표(§8)**: 공개된 `pe-hwpx`로 한글 양식 스킬 항목을 바꾸고, "설치된 경우"로 표기.
- **플러그인 구성**: `.mcp.json` 추가(서버 `korean-law`, type `http`, `https://mcp.gomdori.app/law`, 인증키 미포함). `plugin.json` version 2.0.0, author·license·repository·keywords 추가, description의 커넥터 명칭 정리(v1에서 500자에서 끊겼던 문장 복구).
- **시험 기록**: TC-6(v2 절차 검증) 추가. TC-1~5는 공개용으로 회사명·현장명·발주기관명을 일반화.

### 그대로 둔 것

발동 원칙(§1), 인용 형식(§3.1~3.3), 작업 절차의 골격(§4), 모순·충돌 처리 원칙(§6), 안티패턴 1~6(§7), 협업 방식(§8).

## v1.0.0 — 2026-04-28

- 최초 작성. '법정보조회시스템' 커넥터와 체인 도구(`chain_*`) 기준의 도구 선택 전략, 인용 형식, Mark-and-Proceed 규칙.
- 시험 기록 TC-1~5.
