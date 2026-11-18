---
name: legal-fact-checker
metadata:
  title: 법령팩트체커
  version: 2.0.0
description: 법령·규제·정책·자격 기준 등 법적 근거가 조금이라도 관련된 모든 산출물(보고서, 제안서, 공문, 의견서, 워드/한글 문서 등)을 작성하기 전 또는 작성 도중에 반드시 발동하여, 내부 지식에 의존하지 않고 연결된 한국 법령 조회 커넥터(Korean-law-mcp — search_law·get_law_text 등)를 선제적으로 호출하여 법령·판례·해석례를 검증한 뒤 그 결과를 산출물에 반영하도록 강제하는 스킬. 사용자가 명시적으로 법령을 묻는 경우뿐만 아니라, 정보화전략계획·정책 제안서·기술 자문 의견서·평가위원 의견서·입찰 검토서·공문 등 일반 문서 작성 요청에서도 자격·면허·등록·입찰·계약·평가·기술표준·고시·안전·환경·개인정보·보안·행정절차 중 어느 한 가지라도 다뤄질 가능성이 있으면 자동 발동해야 한다. 정보통신공사업법, 국가기술자격법, 국가계약법, 지방계약법, 개인정보보호법, 정보통신망법, 건설안전특별법, 중대재해처벌법, 건축법, 행정절차법 등이 언급되거나 그 주제가 함의될 때 반드시 사용한다. 절대 내부 지식만으로 법적 근거를 단정하지 말고, 의심되면 발동하라.
---

# Legal Fact Checker v2 (법령 사전 검증 및 인용 스킬)

문서 산출물에 들어갈 법적 진술을 **항상** 외부 권위 출처(법제처 데이터를 조회하는 한국 법령 조회 커넥터)로 검증한 뒤 작성한다. 본 스킬은 다른 문서 생성 스킬의 **상위 검증 레이어**로 동작한다.

> v2 변경 요지: 도구 체계를 Korean-law-mcp 10종 노출 도구 기준으로 다시 썼고(§2), 커넥터 이름 대신 기능으로 찾으며(§0), 커넥터 장애 대응 절차를 추가했다(§3.4).

---

## 0. 커넥터 찾기 (이름에 의존하지 않음)

커넥터의 화면 이름은 환경마다 다르다(예: '법령 정보 검색', 'korean-law', 'Korean Law MCP'). 이름이 아니라 **도구 구성**으로 찾는다.

- 도구 이름 끝이 `search_law`, `get_law_text`, `search_decisions`, `get_annexes`, `legal_research` 등인 도구 묶음
- 도구 설명이 `Korean-law-mcp —` 로 시작함
- 실제 도구 이름에는 접두어가 붙는다. 예) 플러그인 설치 시 `mcp__plugin_legal-fact-checker_korean-law__search_law`, claude.ai 커넥터로 연결 시 `mcp__<커넥터ID>__search_law`. **접두어는 무엇이든 상관없다.**
- 도구가 지연 로딩(deferred) 상태이면 먼저 도구 검색으로 위 이름들을 불러온 뒤 호출한다.
- 같은 도구 묶음이 두 개(플러그인 + 커넥터) 보이면 아무 쪽이나 하나를 쓰고, 오류가 나면 다른 쪽으로 바꿔 본다.
- 어떤 묶음도 보이지 않으면 §3.4(커넥터 장애 대응)로 간다.

이하 본문에서는 접두어를 생략하고 도구 이름만 적는다.

---

## 1. 발동 원칙 (Trigger Doctrine)

### 1.1 명시적 발동
사용자가 다음과 같이 요청한 경우 즉시 발동한다.
- "관련 법령 찾아줘", "법적 근거가 뭐야", "이거 합법이야"
- "○○법 제○조 알려줘", "최신 개정 내용 확인해줘"
- 계약서·약관·협정서 검토 요청

### 1.2 묵시적(자동) 발동 — 가장 중요
사용자가 **일반 문서**를 요청했더라도, 산출물에 다음 중 하나라도 포함될 가능성이 있으면 작성 **착수 전** 자동 발동한다.

| 영역 | 트리거 키워드 예시 | 주요 검증 대상 법령 |
|------|---|---|
| 자격·면허·등록 | 기술사사무소, 정보통신공사업, 감리업, 등록기준, 자본금, 기술인력 | 정보통신공사업법, 기술사법, 국가기술자격법, 엔지니어링산업진흥법 |
| 입찰·계약·평가 | 적격심사, 낙찰자 결정, 평가위원, 협상계약, 종합심사 | 국가계약법, 지방계약법 및 시행령·시행규칙, 조달사업법 |
| 기술 표준·고시 | 표준품셈, KS, 방송통신표준, 형식승인, 기술기준 | 정보통신공사업법, 전기통신사업법, 방송통신발전기본법, 관련 고시 |
| 안전·환경 | 산업안전, 중대재해, 위험성평가, 환경영향평가 | 산업안전보건법, 중대재해처벌법, 건설안전특별법, 환경영향평가법 |
| 개인정보·보안 | 개인정보, CCTV, 영상정보처리기기, ISMS, 망분리 | 개인정보보호법, 정보통신망법, 위치정보법, 정보통신기반보호법 |
| 행정 절차 | 인허가, 신고, 처분, 공람, 의견청취, 청문 | 행정절차법, 행정기본법, 민원처리법 |

> **판단 임계치**: "이 조항을 인용해도 되는가?"라는 의문이 0.1초라도 든다면 발동한다. 미발동의 비용보다 과발동의 비용이 훨씬 작다.

### 1.3 비발동
다음 경우는 발동하지 않는다.
- 단순 인사·잡담·일정 조회·번역·코드 작성 등 법령 무관 업무
- 사용자가 "법령 검토 생략하고 초안만"이라고 명시적으로 거부한 경우

---

## 2. 도구 선택 전략 (Hybrid Approach)

커넥터는 **노출 도구 10종**을 직접 호출하고, 그 밖의 전문 도구(90여 종)는 `discover_tools`로 찾아 `execute_tool`로 실행한다.

### 2.1 노출 도구 10종 (직접 호출)

**직접 조회 — 단일 법령·조문이 명확할 때**

| 도구 | 용도 | 주요 파라미터 |
|---|---|---|
| `search_law` | 법령·조례·행정규칙명 검색 → `lawId`, `mst` 획득. 0건이면 자치법규·행정규칙으로 자동 폴백, 약칭 자동 변환, 시행예정 개정 병기 | `query`(필수), `display` |
| `get_law_text` | 조문 전문. `jo="제8조"`처럼 자연어 조문 표기 권장. `jo="부칙"`이면 부칙 원문(공포번호·시행일·경과조치). `efYd`(YYYYMMDD)로 과거 시점 본문 | `mst` 또는 `lawId`, `jo`, `efYd` |
| `get_annexes` | 별표·서식 본문 추출(금액·기준·자격표 등). `query`로 별표명 검색 → 목록의 `bylSeq`로 재호출하면 본문. `date`로 과거 시점 별표 | `lawName`(필수), `query`, `bylSeq`, `annexNo`, `jo`, `knd`(1=별표, 2=서식, 3=부칙별표, 4=부칙서식, 5=전체), `date` |
| `search_decisions` | 23개 도메인 통합 검색(판례 `precedent`, 법령해석례 `interpretation`, 헌재 `constitutional`, 행정심판 `admin_appeal`, 개인정보위 `pipc`, 공정위 `ftc`, 고용노동부 `moel`, 중앙부처 1차 해석 `ministry` 등) | `domain`(필수), `query`, `display`, `page`, `sort`, `options` |
| `get_decision_text` | 위 검색 결과의 본문 | `domain`(필수), `id`(필수), `full`, `options` |

**복합 분석 — 범위가 넓거나 모호할 때**

| 도구 | 용도 | 주요 파라미터 |
|---|---|---|
| `legal_research` | 다단계 리서치 통합(구 chain 도구들의 기능). `task` 값으로 유형 선택 — 아래 표 참조 | `query`, `task`, `scenario`, `articles`, `parentLaw`, `text`, `fromDate`, `toDate`, `includeHistory`, `lawId`, `mst`, `domain`, `maxClauses` |
| `legal_analysis` | 정밀 분석 5종. `mode`: `verify_citations`(텍스트 속 조문·판례 인용 실존 교차검증, `text` 필수) / `cite_check`(판례 생사 확인, `caseNumber` 필수) / `applicable_law`(사건 시점 시행 법령·조문·경과조치, `lawName`+`date` 필수) / `impact_map`(한 조문을 인용한 판례·해석례·조례 역방향 그래프, `lawName`+`jo` 필수) / `amendment_table`(개정안 신구조문대비표, `lawName`+`amendments` 필수) | `mode`(필수) 외 mode별 |
| `ordinance_radar` | 조례가 인용한 상위법의 개정 시점과 조례 시행일을 대조해 정비 필요 여부 표시 | `ordinanceName` 또는 `ordinSeq`(별칭 `query`, `id`) |

**메타**

| 도구 | 용도 | 주요 파라미터 |
|---|---|---|
| `discover_tools` | 숨은 전문 도구를 의도·카테고리로 검색해 이름과 설명을 돌려줌 | `intent`(필수) |
| `execute_tool` | 숨은 전문 도구를 이름과 인자로 실행 | `tool_name`(필수), `params`(필수, 객체) |

### 2.2 `legal_research` task 매핑 (v1 chain 도구 대응표)

| v1 안내 도구 | v2 호출 | 쓰임 |
|---|---|---|
| `chain_law_system` | `legal_research(task="law_system", query=..., articles=[...])` | 법률·시행령·시행규칙 3단 + 위임 + 별표. `scenario`: `delegation`·`impact` |
| `chain_action_basis` | `legal_research(task="action_basis", query=...)` | 처분·허가·자격·등록 근거 + 해석례 + 판례 + 행심. `scenario="penalty"`: 과태료·과징금·처분기준 |
| `chain_procedure_detail` | `legal_research(task="procedure_detail", query=...)` | 절차·수수료·별표서식. `scenario="manual"`: 훈령·예규·고시·조례 특칙 |
| `chain_amendment_track` | `legal_research(task="amendment_track", query=...)` | 개정 이력·신구대조·연혁. `scenario`: `timeline`·`time_travel`(`fromDate`·`toDate`) |
| `chain_ordinance_compare` | `legal_research(task="ordinance_compare", query=..., parentLaw=...)` | 조례 전국 비교 + 상위법 적합성. `scenario="compliance"` |
| `chain_dispute_prep` | `legal_research(task="dispute_prep", query=..., domain=...)` | 불복·소송 준비(판례 + 심판례 + 도메인 결정례) |
| `chain_document_review` | `legal_research(task="document_review", text=...)` | 계약서·약관 조항별 리스크(`text` 필수) |
| `chain_full_research` | `legal_research(task="full_research", query=...)` (기본값) | 도메인·법령명 불명확 시 폴백. `scenario`: `customs`·`action_plan` |

> 참고: 위 `chain_*` 이름의 도구 자체도 커넥터 내부에 숨은 도구로 남아 있어 `execute_tool(tool_name="chain_law_system", params={"query": ...})`처럼 실행할 수 있다(필수 인자 `query` 확인). 다만 파라미터가 문서화된 **`legal_research`를 우선 사용**한다.

### 2.3 자주 쓰는 숨은 도구 (`execute_tool` 경유)

`discover_tools`로 확인한 이름이다. 필수 인자가 확인된 것만 표기했고, 나머지 인자는 `discover_tools` 설명과 오류 메시지(`[INVALID_PARAMETER] ...`)를 보고 맞춘다.

| 범주 | 도구 이름 | 확인된 필수 인자 / 비고 |
|---|---|---|
| 행정규칙(고시·훈령·예규) | `search_admin_rule` | `query` 필수. 종류 필터(1=훈령, 2=예규, 3=고시) |
| | `get_admin_rule` | `id` 필수(검색 결과의 행정규칙 ID). 긴 규칙은 조·장·키워드·페이지로 부분 조회 |
| | `compare_admin_rule_old_new` | 행정규칙 신구 비교 |
| 자치법규 | `search_ordinance` | `query` 필수(지역명 포함 권장) |
| | `get_ordinance` | 조례 전문·특정 조문 |
| 법령 비교 | `get_three_tier` | `mst` 또는 `lawId` 필수. 법률-시행령-시행규칙 위임·인용 조문 |
| 조문 정밀 | `get_article_detail`, `get_batch_articles`, `get_article_with_precedents` | 조·항·호·목 단위, 여러 조문 일괄, 조문+판례 |
| 이력 | `get_article_history`, `get_law_history`, `search_historical_law`, `get_historical_law` | 조문별·법령별 개정 이력, 연혁법령 |
| 검색 보조 | `search_all`, `search_ai_law`, `suggest_law_names`, `advanced_search`, `search_law_bulk` | 통합검색, 자연어 의미검색, 법령명 자동완성, 조건 검색, 법령 일괄 조회 |
| 판례·해석례 | `search_precedents`, `get_precedent_text`, `find_similar_precedents`, `search_interpretations`, `get_interpretation_text` | 대개 `search_decisions`/`get_decision_text`로 충분 |
| 문서 분석 | `analyze_document` | 계약서·약관 리스크 분석(체인 없이) |

> **추측 금지**: 위 표에 없는 도구 이름을 지어내지 않는다. 필요하면 먼저 `discover_tools(intent="...")`로 실제 이름을 확인한다.

### 2.4 의사결정 트리

```
Q0. 커넥터 도구가 보이는가?
  NO  → §3.4 커넥터 장애 대응
  YES → Q1

Q1. 법령명·조문이 명확히 특정되는가?
  YES → search_law → get_law_text(mst, jo="제N조") → 끝
        (해석 보강: search_decisions(domain="interpretation" | "precedent"))
        (공포번호·시행일: get_law_text(mst, jo="부칙"))
  NO  → Q2

Q2. 금액·기준·자격·서식 등 '별표·서식' 내용인가?
  YES → get_annexes(lawName, query="별표명 키워드") → 목록의 bylSeq로 재호출
  NO  → Q3

Q3. 고시·훈령·예규(행정규칙) 내용인가?
  YES → execute_tool("search_admin_rule", {query}) → execute_tool("get_admin_rule", {id})
        (search_law도 0건이면 행정규칙으로 자동 폴백함)
  NO  → Q4

Q4. 조례(자치법규) 쟁점인가?
  YES → 단건 정비 검토: ordinance_radar(ordinanceName)
        전국 비교·상위법 적합성: legal_research(task="ordinance_compare", scenario="compliance")
  NO  → Q5

Q5. 단일 법령의 체계 전체(3단·위임)를 봐야 하는가?
  YES → legal_research(task="law_system")
  NO  → Q6

Q6. 처분·허가·자격·등록 기준 검토인가?
  YES → legal_research(task="action_basis") (과태료·처분기준이면 scenario="penalty")
  NO  → Q7

Q7. 행정 절차·서식·수수료 확인인가?
  YES → legal_research(task="procedure_detail") (내부지침 포함 시 scenario="manual")
  NO  → Q8

Q8. 개정 이력·시행일·행위시법 쟁점인가?
  YES → 이력: legal_research(task="amendment_track")
        특정 시점 적용 법령: legal_analysis(mode="applicable_law", lawName, date)
  NO  → Q9

Q9. 도메인이 모호하거나 복합 쟁점인가?
  YES → legal_research(task="full_research") (폴백)
  NO  → 발동 보류 후 사용자에게 질문
```

### 2.5 호출 효율 원칙
- **병렬 호출**: 독립적으로 검증 가능한 쟁점이 여러 개면 한 번의 메시지에서 동시 호출
- **재호출 회피**: `legal_research`가 이미 3단 구조를 반환하면 같은 법령에 대해 직접 조회를 중복 호출하지 않음
- **인용 검증**: 초안에 인용할 조문은 작성 직전에 한 번 더 `get_law_text(jo=...)`로 원문 확인(특히 항·호 번호)
- **현행 MST 재확인**: 개정이 의심되면 `search_law`로 [현행] MST를 다시 받는다(MST는 개정마다 바뀜)
- **공용 한도 절약**: 키 없이 쓰는 공개 서버는 공용 한도가 있으므로 같은 조회를 반복하지 않는다

---

## 3. 인용 형식 표준

### 3.1 본문 인용 (간결형)
본문에서는 **법령명 + 조항**만 표기한다.

```
「정보통신공사업법」 제8조 제1항에 따라 ...
「국가를 당사자로 하는 계약에 관한 법률 시행령」 제42조 제1항 제3호에 의거 ...
```

규칙:
- 법령명은 **「 」(낫표)** 로 묶는다.
- 조·항·호는 **제○조 제○항 제○호** 형식.
- 호 안의 목은 **가목·나목** 형식.
- 약칭(예: '정통공사업법')은 **사용하지 않는다**. 첫 등장 시 정식 명칭 사용.

### 3.2 참고 법령 섹션 (상세형)
문서 말미 또는 부록에 **「참고 법령」** 또는 **「근거 법령」** 섹션을 두고 일괄 표기한다.

```markdown
## 참고 법령

| 구분 | 법령명 | 조항 | 시행일·법령번호 | 비고 |
|------|--------|------|----------------|------|
| 1 | 정보통신공사업법 | 제8조 제1항·제2항 | 2026.10.22. 시행 (법률 제21555호, 2026.4.21. 공포) | 감리 발주·감리원 배치 |
| 2 | 정보통신공사업법 시행령 | 제10조 제1항, 별표 2 | (조회 시점의 시행일·대통령령 번호) | 감리원 자격 |
```

규칙:
- **시행일과 법령번호**(법률·대통령령·부령 번호)를 반드시 명시한다. 법령번호는 `get_law_text(jo="부칙")` 응답의 `부칙 <제NNNNN호, 공포일>`에서 확인한다.
- 시행일이 미래인 경우 "2027.1.1. 시행 예정"으로 표기.
- 판례 인용 시 별도 「참고 판례」 표를 둔다(법원, 사건번호, 선고일, 요지).

### 3.3 인용 검증 실패 표시 (Mark-and-Proceed)
법령 조회가 실패하거나 결과가 모호하면 **작성을 중단하지 않고**, 해당 부분에 다음과 같이 표시 후 진행한다.

```
⚠️ [법령 검증 필요] 「○○법」 제○조의 정확한 조항 번호 및 시행일은 사용자 확인 필요.
근거: 법령 조회 커넥터 조회 시 [구체 사유: 결과 다중/검색 불일치/타임아웃/커넥터 미연결/요청 제한 등]
```

문서 말미에 **「검증 미완 항목」** 섹션을 두어 한눈에 보이도록 정리한다.

### 3.4 커넥터 장애 대응 (v2 신설)

다음 중 하나이면 장애로 본다.
- (a) 커넥터 도구 묶음이 보이지 않음(미연결·비활성)
- (b) 호출이 오류로 끝남(예: `Tool call failed: 500`, 시간 초과)
- (c) 응답에 요청 제한을 뜻하는 표현이 있음(예: "요청 제한", "rate limit", "429", "한도 초과")

처리 순서:
1. **1회 재시도** — (b)·(c)는 같은 호출을 한 번 더 한다. (a)는 도구 검색으로 한 번 더 찾아본다. 옛 주소(`korean-law-mcp.fly.dev`)가 "새 주소 안내"만 돌려주는 경우도 (a)로 본다.
2. **그래도 실패하면 Mark-and-Proceed** — §3.3 형식으로 해당 명제를 표시하고 나머지 작성은 계속한다. 내부 지식으로 빈칸을 메우지 않는다.
3. **사용자 안내** — 산출물과 별도로 다음을 짧게 알린다.
   - 커넥터 상태: 미연결 / 서버 오류 / 요청 제한 중 어느 것인지
   - 연결 주소: `https://mcp.gomdori.app/law` (Korean-law-mcp 공개 서버. 옛 주소 `korean-law-mcp.fly.dev/mcp`는 2026.11.17.부터 조회 중단)
   - 요청 제한이면: 키 없이 쓰면 공용 한도를 거치므로, 법제처 Open API(open.law.go.kr)에서 인증키(OC)를 발급받아 주소 끝에 `?oc=본인키`를 붙여 다시 연결하면 본인 한도로 조회된다는 점
   - 인증키 값을 대화에 적어 달라고 요청하지 않는다. 사용자가 직접 커넥터 설정에 넣도록 안내만 한다.

---

## 4. 작업 절차 (Workflow)

### 4.1 사전 검증 절차 (문서 작성 전)
1. **쟁점 식별**: 사용자 요청에서 법적 검토가 필요한 명제(claim)를 추출하여 리스트화.
   - 예) "감리원은 정보통신기술사여야 한다", "적격심사는 90점 이상이어야 한다", "CCTV 영상은 30일 이상 보관해야 한다"
2. **도구 매핑**: §2.4 의사결정 트리에 따라 각 명제에 도구 1개 배정.
3. **병렬 조회**: 독립적 쟁점은 한 번에 동시 호출. 실패 시 §3.4.
4. **결과 정리**: 명제별로 (법령명, 조항, 시행일, 법령번호, 원문 요지, 출처)를 표 형태로 정리.
5. **모순 검토**: 조회 결과가 사용자가 알고 있던 사실과 다르면 즉시 사용자에게 보고하고 진행 여부 확인.

### 4.2 작성 중 검증 절차 (문서 작성 도중)
- 초안을 작성하다가 새 법령 인용이 필요해지면 **그 자리에서 즉시** 조회 후 인용한다.
- 절대로 "나중에 확인" 또는 "내부 지식으로 가정"하지 않는다.

### 4.3 최종 인용 교차검증 (v2 권장)
- 초안 완성 후 법령·판례 인용이 들어간 본문을 `legal_analysis(mode="verify_citations", text=<초안 본문>)`에 넣어 실존 여부를 교차검증한다.
- 판례를 인용했다면 `legal_analysis(mode="cite_check", caseNumber=...)`로 변경·폐기 여부를 확인한다.
- 검증에서 걸린 인용은 고치거나 §3.3으로 표시한다.

### 4.4 산출물 통합
- 본문: §3.1 간결형
- 말미: §3.2 상세형 표
- 검증 미완 사항: §3.3 표시 + 별도 섹션

---

## 5. 도메인별 자주 쓰는 호출 예시

### 5.1 정보통신공사업·감리·기술사 관련
```
search_law(query="정보통신공사업법", display=5)
→ get_law_text(mst=<획득값>, jo="제8조")       # 감리 발주·감리원 배치·감리관리시스템
→ get_law_text(mst=<획득값>, jo="제14조")      # 공사업 등록
→ get_law_text(mst=<획득값>, jo="부칙")        # 공포번호·시행일·경과조치
get_annexes(lawName="정보통신공사업법 시행령", query="감리원", knd="1")
→ get_annexes(lawName="정보통신공사업법 시행령", bylSeq=<목록의 번호>)   # 감리원 자격표
legal_research(task="law_system", query="기술사법", articles=["제5조","제6조"])
```

### 5.2 입찰·평가위원 관련
```
legal_research(task="action_basis", query="적격심사 낙찰자 결정 기준")
search_decisions(domain="interpretation", query="평가위원 제척", display=10)
execute_tool(tool_name="search_admin_rule", params={"query": "협상에 의한 계약체결기준"})
```

### 5.3 개인정보·CCTV 관련
```
search_law(query="개인정보 보호법", display=3)
→ get_law_text(mst=<값>, jo="제25조")          # 고정형 영상정보처리기기 설치·운영 제한
execute_tool(tool_name="search_admin_rule", params={"query": "표준 개인정보 보호지침"})
→ execute_tool(tool_name="get_admin_rule", params={"id": <검색 결과 ID>})
search_decisions(domain="pipc", query="영상정보처리기기 보관기간")
```

### 5.4 자치법규(조례)
```
ordinance_radar(ordinanceName="○○시 ○○ 조례")         # 상위법 개정 후 정비 필요 여부
legal_research(task="ordinance_compare",
               query="EV 충전시설 의무 설치",
               parentLaw="환경친화적 자동차의 개발 및 보급 촉진에 관한 법률",
               scenario="compliance")
```

### 5.5 개정 이력·행위시법
```
legal_research(task="amendment_track", query="정보통신공사업법 2026년 개정")
legal_analysis(mode="applicable_law", lawName="정보통신공사업법", date="2025-06-01", jo="제8조")
get_law_text(mst=<값>, jo="제8조", efYd="20250601")     # 그 날짜에 시행 중이던 본문
```

---

## 6. 모순·충돌 처리

조회 결과에 다음 모순이 발견되면 **사용자에게 즉시 보고**한다.

- **법률 vs 시행령 모순**: 위임 범위 일탈 가능성 → `legal_research(task="law_system", scenario="impact")` 또는 `execute_tool("get_three_tier", {mst})` 추가 호출
- **법령 vs 조례 모순**: 상위법 우선 원칙 → `legal_research(task="ordinance_compare", scenario="compliance")` 또는 `ordinance_radar`
- **현행법 vs 사용자 인식 모순**: 작성 중단하고 확인 요청
- **개정 전·후 적용 시점 모호**: `legal_research(task="amendment_track", scenario="timeline")` 또는 `legal_analysis(mode="applicable_law")`

---

## 7. 안티 패턴 (절대 금지)

다음 행위는 본 스킬의 존재 이유에 정면으로 반한다.

1. ❌ "정보통신공사업법 제8조에 따르면..." 이라고 **조회 없이** 단언하기
2. ❌ "최근 개정으로..." 라고 **개정 이력 미확인** 상태로 진술하기
3. ❌ 시행일·법령번호 누락한 채 **「참고 법령」 섹션 작성**하기
4. ❌ 판례를 인용하면서 **사건번호 미기재**
5. ❌ 검증 실패를 사용자에게 알리지 않고 **그럴듯한 추정**으로 메우기
6. ❌ 사용자가 "법령 모르면 그냥 패스"라고 하지 않았는데 **검증 단계 생략**하기
7. ❌ 커넥터에 **없는 도구 이름**(예: v1의 `chain_*`를 직접 호출)이나 확인 안 된 파라미터를 지어내 호출하기
8. ❌ 커넥터 장애 시 재시도·표시·안내 없이 **내부 지식으로 대체**하기

---

## 8. 다른 스킬과의 협업

본 스킬은 다음 스킬들의 **상위 레이어**로 작동한다(설치되어 있는 경우).

| 협업 대상 스킬 | 협업 방식 |
|----------------|-----------|
| `ict-technical-review` | 기술 자문 의견서의 법적 근거 부분 사전 검증 |
| 평가·자문위원 신청서 작성 스킬 | 신청서·자기소개서의 자격 요건 검증 |
| `pe-hwpx` (한글 양식 작성) | 공문 본문에 인용된 법령 조항 검증 |
| `report-figure-creator` | 보고서 그림 캡션·각주의 법령 표기 검증 |

협업 트리거: 위 스킬이 발동되었고 산출물에 법적 진술이 포함될 때, **본 스킬을 먼저 실행**한 뒤 결과를 협업 스킬에 전달한다.

---

## 9. 산출 예시 (Reference Output)

아래 예시의 조문·별표 정보는 2026.11.18. 커넥터 조회로 확인한 것이다(test-results/TC-6). 실제 작성 때는 반드시 다시 조회한다.

### 9.1 명제 리스트 (4.1단계 산출물 예)
```
[검증 대상 명제]
1. 감리 발주·감리원 배치 의무 — 정보통신공사업법 제8조 제1항·제2항
2. 감리원 자격 기준 — 정보통신공사업법 시행령 제10조 제1항, 별표 2
3. 평가위원 제척 사유 — 국가계약법 시행령 (조문 특정 필요 → search_law/get_law_text)
4. 영상정보처리기기 보관기간 — 개인정보 보호법 제25조 + 표준 개인정보 보호지침(행정규칙)
```

### 9.2 본문 인용 예
```markdown
발주자는 「정보통신공사업법」 제8조 제1항에 따라 용역업자에게 공사의 감리를
발주하여야 하며, 감리원의 자격은 같은 법 시행령 제10조 제1항 및 별표 2에서 정한다.
2026년 4월 21일 개정으로 같은 조 제9항이 신설되어 과학기술정보통신부장관은
감리관리시스템을 구축·운영할 수 있다(2026.10.22. 시행).
```

### 9.3 참고 법령 섹션 예
```markdown
## 참고 법령

| # | 법령명 | 조항 | 시행일·법령번호 | 핵심 내용 |
|---|--------|------|----------------|-----------|
| 1 | 정보통신공사업법 | 제8조 제1항·제2항·제9항 | 2026.10.22. 시행 (법률 제21555호, 2026.4.21. 공포) | 감리 발주·배치, 감리관리시스템 |
| 2 | 정보통신공사업법 시행령 | 제10조 제1항, 별표 2 | 별표 2 개정 2024.4.30. | 감리원 등급별 자격 |

## 검증 미완 항목

⚠️ [법령 검증 필요] 시행령 대통령령 번호 — 이번 조회에서 시행령 부칙을 조회하지 않음.
get_law_text(mst=<시행령 MST>, jo="부칙")로 확인 후 보완.
```
