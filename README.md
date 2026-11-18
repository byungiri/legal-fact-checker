# legal-fact-checker

**법령·판례를 먼저 조회해 확인한 뒤 문서에 인용하게 하는 Claude 스킬** — 기술사 실무에서 검증

보고서·공문·의견서·제안서처럼 법적 근거가 들어가는 문서를 쓸 때, Claude가 기억에 의존해 조문을 단정하지 않고
한국 법령 조회 커넥터(Korean-law-mcp)로 법제처 원문을 먼저 확인하게 합니다. 확인한 조문은 「법령명」 제○조 제○항
형식으로 본문에 인용하고, 문서 끝에 시행일·법령번호를 담은 「참고 법령」 표를 붙이며, 확인하지 못한 부분은
⚠️ 표시로 남겨 사람이 다시 보게 합니다. 기술법인 진성이 정보통신 분야 기술사 업무(감리·자문 의견서, 평가위원 업무, 공문)에 쓰면서 다듬었습니다.

> **English summary** — A Claude skill (packaged as a Claude Code plugin) that makes Claude verify Korean statutes,
> annexes, precedents and interpretations through the public Korean-law-mcp server before citing them in reports,
> official letters or opinions. It maps questions to the server's tools, formats citations in the Korean legal style
> with effective dates and law numbers, marks anything it could not verify, and handles connector outages
> (one retry, then mark-and-proceed, then tell the user). This repository does not contain the server code. MIT License.

## 누가 쓰면 좋은가

| 사용자 | 주로 쓰는 문서 | 도움이 되는 기능 |
| --- | --- | --- |
| 기술사·엔지니어링사 | 감리·자문 의견서, 기술검토서, 평가위원 의견서 | 자격·등록·감리 기준의 조문·별표 원문 확인, 참고 법령 표 |
| 공무원·공공기관 실무자 | 공문, 검토 보고서, 조례 정비 검토 | 시행일·법령번호 확인, 조례의 상위법 개정 추적 |
| 입찰·계약 담당자 | 입찰 검토서, 평가 안내 공문 | 계약법령·해석례 확인, 고시·예규 조회 경로 |
| 일반 문서 작성자 | 개인정보·CCTV·안전 관련 문서 | 법적 진술이 들어가면 자동 발동, 확인 못 한 내용 표시 |

## 동작 방식

1. **발동**: 법령을 직접 묻지 않아도 자격·입찰·계약·기술기준·개인정보·안전·행정절차 등이 문서에 들어갈 것 같으면 작성 전에 발동합니다.
2. **명제 정리**: "감리원은 ○○ 자격이 있어야 한다"처럼 확인할 문장을 먼저 뽑습니다.
3. **조회**: 질문 유형에 맞는 도구를 고릅니다(조문 `get_law_text`, 별표 `get_annexes`, 조례 `ordinance_radar`, 복합 리서치 `legal_research`, 인용 검증 `legal_analysis` 등).
4. **인용**: 본문에는 「법령명」 제○조 제○항, 끝에는 시행일·법령번호가 있는 「참고 법령」 표를 붙입니다.
5. **표시**: 확인하지 못한 내용은 ⚠️ [법령 검증 필요]로 표시하고 「검증 미완 항목」에 모읍니다.
6. **장애 대응**: 커넥터가 없거나 오류·요청 제한이면 1회 재시도 → 그래도 안 되면 표시 후 진행 → 사용자에게 커넥터 상태와 인증키(OC) 안내.

자세한 규칙은 [skills/legal-fact-checker/SKILL.md](skills/legal-fact-checker/SKILL.md)에 있습니다.

## 현재 상태: v2.0.0

| 항목 | 내용 | 상태 |
| --- | --- | --- |
| 도구 체계 | Korean-law-mcp 노출 도구 10종 기준 의사결정 트리·호출 예시 | 완료 |
| 커넥터 찾기 | 커넥터 이름 대신 도구 구성(`search_law` 등)으로 인식 | 완료 |
| 장애 대응 | 재시도 → Mark-and-Proceed → 사용자 안내 | 완료 |
| 커넥터 자동 등록 | 플러그인 설치 시 `.mcp.json`으로 `korean-law` 서버 등록 | 완료 |
| 시험 | TC-1~5(v1), TC-6(v2 절차 검증) — [test-results](skills/legal-fact-checker/test-results/) | 완료 |

변경 내용은 [CHANGELOG.md](CHANGELOG.md)를 보십시오.

## 설치

### Claude Code (플러그인 — 커넥터 함께 등록)

```
/plugin marketplace add byungiri/legal-fact-checker
/plugin install legal-fact-checker@byungiri
```

설치하면 스킬과 함께 MCP 서버 `korean-law`(`https://mcp.gomdori.app/law`)가 등록됩니다. 새 버전은
`/plugin marketplace update`로 받습니다. 내려받은 폴더로 시험만 해 보려면 `claude --plugin-dir <폴더>`로 실행합니다.

이미 같은 서버를 커넥터로 연결해 두었다면 도구가 두 벌 보일 수 있습니다. 스킬은 어느 쪽이든 사용하며, 하나만 남기셔도 됩니다.

### claude.ai (웹·데스크톱 앱)

1. 설정 → 커넥터에서 맞춤 커넥터로 `https://mcp.gomdori.app/law` 를 추가합니다.
2. `skills/legal-fact-checker` 폴더를 zip으로 묶어 스킬 화면의 「Skill 업로드」로 올립니다.
3. 법적 근거가 들어가는 문서를 요청하면 스킬이 자동으로 쓰입니다.

### 선택: 본인 인증키(OC)

키 없이도 동작하지만 공개 서버의 공용 한도(분당·하루)를 함께 씁니다. 요청 제한이 자주 걸리면
[법제처 Open API](https://open.law.go.kr)에서 인증키(OC)를 발급받아 주소 끝에 `?oc=본인키`를 붙여 연결하십시오
(예: `https://mcp.gomdori.app/law?oc=본인키`). 플러그인 사용자는 `.mcp.json`을 고치는 대신 본인 PC의 사용자 설정에
같은 서버를 키 붙은 주소로 추가하는 편이 안전합니다. **인증키를 저장소에 커밋하지 마십시오.**

## 커넥터 출처

| 항목 | 내용 |
| --- | --- |
| 서버 | Korean-law-mcp (Korean Law MCP Server) |
| 소스·문서 | https://github.com/chrisryugj/korean-law-mcp (MIT License, 운영자 공개 저장소) |
| 공개 서버 주소 | `https://mcp.gomdori.app/law` (운영자가 공개한 주소. 옛 주소 `korean-law-mcp.fly.dev/mcp`는 2026.11.17.부터 조회 중단) |
| 데이터 | 법제처 국가법령정보 Open API |

본 저장소는 **서버 코드를 포함하지 않습니다.** 서버는 제3자가 운영하는 공개 서버이며, 가용성·한도·이용 조건은
운영자의 안내와 이용약관을 따릅니다. 본 저장소는 그 서버를 쓰는 방법(스킬 지침)과 연결 설정만 담고 있습니다.

## 구성

```
.claude-plugin/plugin.json        플러그인 정보 (v2.0.0)
.claude-plugin/marketplace.json   /plugin marketplace add 용 목록
.mcp.json                         커넥터 등록 (korean-law, http)
skills/legal-fact-checker/
  SKILL.md                        스킬 본문
  test-results/                   시험 기록 (TC-1~6)
```

## 개인정보

스킬과 설정에는 인증키·계정·개인정보가 들어 있지 않습니다. 시험 기록은 공개를 위해 회사명·현장명·발주기관명을
일반화했습니다. 조회 내용은 커넥터 서버로 전송되므로, 민감한 사건 내용은 질의에 넣지 않는 것을 권합니다.

## 오류 신고·개선 요청

Issues 에 남겨 주십시오. 요청 문장, 기대한 조문, 실제로 나온 결과(가능하면 도구 호출 기록)를 함께 적어 주시면
재현이 빠릅니다. 커넥터 서버 자체의 오류는 [Korean-law-mcp 저장소](https://github.com/chrisryugj/korean-law-mcp)에 알려 주십시오.

## 면책 고지

본 스킬은 법령 확인을 돕는 문서 작성 보조 도구이며 법률 자문이 아닙니다. 조회 결과는 법제처 데이터와 제3자 서버의
상태에 따라 달라질 수 있고, 생성 문서의 법령 인용·해석·제출 적정성에 대한 최종 확인과 책임은 작성자에게 있습니다.
중요한 판단은 국가법령정보센터(law.go.kr) 원문과 전문가 검토로 확인하십시오.

## 라이선스

MIT License. Copyright (c) 2026 기술법인 진성 (Jinseong Technical Corporation)
