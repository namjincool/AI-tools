# OpenCrab Ontology Pack·RAG를 TruLens로 검증하는 기술 가이드

조사 기준일: 2026-10-17  
작업 분류(TypeSafe): `research_and_write_guide` — TypeSafe/Jev 분류 결과이며, 기술 사실의 근거로 사용하지 않았습니다.

## 근거 표기

- **[확인됨]** 공식 문서, 공개 저장소, 제공된 로컬 TruLens 파일 또는 현재 연결된 OpenCrab MCP 응답에서 직접 확인했습니다. 근거 범위는 문맥에 표시합니다.
- **[추론]** 확인된 사실에서 기술적으로 도출한 판단입니다.
- **[미확인]** 공개 자료나 이 세션에서 읽을 수 있는 데이터로 확인하지 못했습니다.
- **[예시]** 절차 설명용 가상 입력·코드입니다. 실제 OpenCrab Pack 데이터나 실행 결과가 아닙니다.

---

## 1. OpenCrab 및 TruLens 사전 조사 결과

### OpenCrab의 정체

- **[확인됨]** 여기서 말하는 OpenCrab은 [opencrab.sh](https://opencrab.sh/)와 연결된 LocalCrab / MetaOntology OS 및 온톨로지 팩 생태계입니다. 공식 사이트가 연결하는 공개 저장소는 [AlexAI-MCP/OpenCrab](https://github.com/AlexAI-MCP/OpenCrab)입니다.
- **[확인됨]** 저장소 README는 이 저장소를 LocalCrab 온톨로지 팩토리와 호스팅 OpenCrab 생태계의 공개 통합 저장소라고 설명합니다. 로컬 수집·증거 인덱싱·온톨로지 추출·Neo4j 검증·팩 내보내기 코드는 공개 저장소에 있지만, 호스팅 opencrab.sh SaaS 구현은 저장소에 포함되지 않는다고 명시합니다. 저장소의 로컬 구현을 호스팅 SaaS 내부와 동일시할 수 없습니다. ([README](https://github.com/AlexAI-MCP/OpenCrab))
- **[확인됨]** 공개 저장소가 설명하는 로컬 흐름은 자료 수집, OCR/CLIP을 포함한 증거 처리, MetaOntology 문법 기반 추출, Neo4j/Cypher 검증, OpenCrab Pack v1 ZIP 내보내기입니다. ([README](https://github.com/AlexAI-MCP/OpenCrab), [Pack v1 규격](https://github.com/AlexAI-MCP/OpenCrab/blob/main/docs/opencrab-pack-v1.md))
- **[확인됨]** OpenCrab 사용 가이드는 원격 MCP 연결과 pack 검색/질의 사용을 안내합니다. 다만 호스팅 SaaS의 내부 검색·생성 구현은 공개하지 않습니다. ([OpenCrab 가이드](https://opencrab.sh/guide))
- **[확인됨·세션 관찰]** 이 세션의 OpenCrab MCP 상태 응답은 `formal_ontology` 런타임 메타데이터와 9개 공간(subject, resource, evidence, concept, claim, community, outcome, lever, policy), RDF/OWL/SHACL/PROV-O/SKOS/DCAT/JSON-LD 표준 이름을 보고했습니다. 이는 현재 연결된 도구의 응답 내용이며, SaaS 공개 소스코드로 독립 검증한 사실은 아닙니다.
- **[확인됨·세션 관찰]** OpenCrab MCP 조회가 이 대화에서 성공했지만, 특정 팩 쿼리에서 실제 검색 결과가 무관한 문맥을 반환한 경우도 있었습니다. 연결 성공을 검색 품질 보증이나 TruLens 연동 완료로 해석하면 안 됩니다. 이 세션에서는 조회만 했고 팩을 변경하지 않았습니다.

### TruLens 설치본 및 API 범위

- **[확인됨·로컬 메타데이터]** 사용자가 지정한 `04_trulens` 설치본의 TruLens 버전은 2.14.0이며 Python 요구사항 메타데이터는 `>=3.10,<4.0`입니다. [TruLens METADATA](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens-2.14.0.dist-info/METADATA>)
- **[확인됨·로컬 환경 메타데이터]** 조사 당시 실행 환경에는 `trulens==2.14.0`, `trulens-providers-openai==2.14.0`가 설치되어 있었고, MCP Python SDK는 설치되어 있지 않았습니다. 이는 패키지 메타데이터 확인이며 import·애플리케이션 실행 검증은 아닙니다.
- **[확인됨·TruLens 2.14 소스]** 2.14.0 소스는 `Metric`을 새 평가 API로 내보내고, `Feedback`은 레거시 호환 API로 표시합니다. 다만 이 버전의 `TruApp`는 평가 객체를 생성자 인자 `feedbacks=`로 받습니다. 따라서 아래 2.14.0 예제는 `Metric` 객체를 `feedbacks=`에 전달합니다. 최신 온라인 문서의 `metrics=` 예제를 2.14.0에 그대로 복사하지 않도록 버전 차이를 반영했습니다. [core/__init__.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/__init__.py>), [apps/app.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/apps/app.py>), [core/app.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/app.py>)
- **[확인됨·실행 환경의 2.14.0 provider 소스]** Context Relevance, Answer Relevance, Groundedness API 이름과 OpenAI provider 생성자의 model_engine 인자는 활성 Python 3.13 site-packages의 2.14.0 소스에서 확인했습니다. [llm_provider.py](<C:/Users/n9911/AppData/Local/Programs/Python/Python313/Lib/site-packages/trulens/feedback/llm_provider.py>), [OpenAI provider.py](<C:/Users/n9911/AppData/Local/Programs/Python/Python313/Lib/site-packages/trulens/providers/openai/provider.py>)
- **[확인됨]** TruLens 공식 Quickstart는 RAG 추적과 Context Relevance, Answer Relevance, Groundedness를 구성하는 방법을 설명합니다. TruLens MCP 가이드는 MCP 도구 호출을 OpenTelemetry span으로 기록하는 방법을 설명합니다. 이것은 TruLens가 OpenCrab과 전용으로 통합되어 있다는 뜻은 아닙니다. ([Quickstart](https://www.trulens.org/getting_started/quickstarts/quickstart/), [MCP 계측 가이드](https://www.trulens.org/component_guides/instrumentation/mcp/))

### 연동 가능 여부

- **[추론]** OpenCrab이 이 세션에서 MCP 도구를 제공하므로, 개발자 애플리케이션이 같은 MCP 서버에 연결해 도구 호출과 반환된 근거를 받는다면 해당 호출을 TruLens의 MCP span 및 RAG span으로 기록할 수 있습니다.
- **[미확인]** 호스팅 OpenCrab의 내부 Retriever/Generator, 원격 MCP 도구의 입력 스키마, 결과 JSON 구조, transport, 계정별 인증 방식은 공개 소스만으로 확정할 수 없습니다. 연결 계정의 도구 목록과 실제 스키마를 실행 시 확인해야 합니다.
- **판정:** **[추론]** 표준 MCP 클라이언트를 거치는 애플리케이션 계층에서 연동할 수 있지만, “기존 opencrab.sh SaaS의 내부 호출을 TruLens가 자동 계측한다”고 말할 근거는 없습니다.

---

## 2. Ontology Pack과 RAG 구조 분석

### Pack 구조

- **[확인됨]** 공개 Pack v1 규격은 다음 파일을 정의합니다. ([규격 원문](https://github.com/AlexAI-MCP/OpenCrab/blob/main/docs/opencrab-pack-v1.md))

| 파일 | 공개 규격상 역할 |
|---|---|
| `manifest.json` | 팩 ID·버전·문법 버전·출처·수량·품질 요약·검색 힌트·파일 경로·해시 |
| `graph/nodes.jsonl` | 노드 ID, 라벨, space, node type, 속성, evidence 참조, 품질 |
| `graph/edges.jsonl` | 출발·도착 노드, 두 space, 관계명, confidence, evidence 참조, 속성 |
| `evidence/index.jsonl` | 원천 자료, 해시, 수집/파싱 상태와 위치, 연결된 문서·청크·노드·엣지 |
| `quality/report.json` | grammar/schema/evidence 참조/orphan node/broken edge/Neo4j import 등 검사 결과 |
| `neo4j/import.cypher`, `neo4j/opencrab_ingest.jsonl`, `neo4j/export_status.json` | 로컬 Neo4j 재현·검사·정규화 내보내기 상태 |
| `README.md`, `sample_queries.json`, `community_reports.json` | 팩 설명, 예시 질의, 커뮤니티 요약 자료 |

- **[확인됨]** 공개 예시에서 노드는 `id`, `label`, `space`, `node_type`, `properties`, `evidence_refs`, `quality`를 가집니다. 엣지는 `from_id`, `to_id`, `from_space`, `to_space`, `relation`, `confidence`, `evidence_refs` 등을 가집니다. 규격은 endpoint 존재 여부, grammar상 관계 유효성, 승격된 항목의 근거 참조 등을 검사하도록 요구합니다. 규격의 샘플 값은 예시이지 실제 팩 측정값이 아닙니다.
- **[확인됨]** Pack 품질 파일은 grammar/schema 검사, 누락된 근거, 고아 노드, 깨진 엣지, Neo4j import 등을 다룹니다. 규격의 “Recommended defaults” 숫자는 저장소의 권고값이며 TruLens 점수 임계값이나 이 사용자의 팩에서 측정된 결과가 아닙니다.
- **[미확인]** 특정 Ontology Pack ZIP, manifest, nodes, edges, evidence index는 이번 조사에서 열지 않았습니다. 실제 팩의 클래스·관계·제약조건·데이터 오류는 판정하지 않았습니다.

### 공개 로컬 엔진 검색 흐름과 SaaS 경계

- **[확인됨]** 공개 `opencrab/ontology/query.py`의 로컬 Hybrid Query 코드는 ChromaDB 벡터 검색, 결과 node ID로 Neo4j 이웃 탐색, 결과 병합·중복 제거·재순위화 흐름을 설명합니다. 해당 코드에는 BM25 검색과 rerank 옵션도 있습니다. ([query.py](https://github.com/AlexAI-MCP/OpenCrab/blob/main/opencrab/ontology/query.py))
- **[추론]** 로컬 엔진을 시험할 때 질문, 검색된 청크/노드/엣지 ID, 점수, evidence 참조, 최종 답변을 각각 기록하면 검색 단계와 생성 단계를 분리하기 쉽습니다.
- **[미확인]** 이 로컬 코드를 호스팅 SaaS의 실제 처리 순서, 랭킹 가중치, 임베딩 모델, 생성 모델이라고 단정할 수 없습니다.
- **[확인됨·세션 관찰]** 현재 연결된 MCP 쿼리 응답에는 질문별 lexical/vector/graph 후보 및 evidence/answer 형태의 정보가 있었습니다. 쿼리 범위에 따라 후보 종류와 수가 달랐으므로 모든 SaaS 질의가 항상 같은 경로를 사용한다고 일반화하지 않습니다.

### 일반 RAG 평가 파이프라인

- **[추론·일반 구조]** 질문 → Retriever가 문서 청크/온톨로지 근거를 가져옴 → 컨텍스트 구성 → Generator가 답변 생성 → 답변과 근거를 반환하는 경계로 계측할 수 있습니다. OpenCrab에서 어느 단계가 원격 서비스 내부이고 어느 단계가 호출 애플리케이션에 있는지는 배포 방식에 따라 다릅니다.
- **[확인됨]** TruLens는 계측 가능한 app method의 입력/출력 및 retrieval context를 기록하고 평가 지표를 trace span에 연결하도록 지원합니다. ([Selector 문서](https://www.trulens.org/component_guides/evaluation/feedback_selectors/selecting_components/), [MCP 문서](https://www.trulens.org/component_guides/instrumentation/mcp/))

---

## 3. TruLens 검증 가능 범위 및 제약사항

| 검증 영역 | 목적 | TruLens 지원 범위 | 입력과 실행 방법 | 결과 해석과 한계 |
|---|---|---|---|---|
| Ontology Pack | 노드·엣지·문법·근거 참조의 구조 품질 확인 | **[확인됨] 기본 기능 아님.** TruLens는 추적·평가 프레임워크이지 OpenCrab grammar/SHACL 검증기가 아닙니다. | 실제 Pack 파일, grammar/schema, quality report; Pack validator/그래프 DB/온톨로지 검증기로 결정론적 검사 | 실패가 데이터 결함인지 grammar 오류인지 원천 확인이 필요합니다. LLM 점수로 유효한 스키마를 증명할 수 없습니다. |
| Ontology Retrieval | 질의에 적절한 노드·관계·evidence를 가져왔는지 확인 | **[확인됨]** 검색 문맥 relevance judge는 가능. 정확한 ID 일치에는 ground truth/custom 계산 필요 | 질의, 검색 결과 텍스트·ID, 정답 evidence/node/edge ID; Context Relevance와 ID 기반 Precision/Recall | 관련 문장과 정답 노드 검색은 같지 않습니다. LLM relevance만으로 완전성·정확성을 증명하지 못합니다. |
| RAG Retrieval | 질문 관련 문맥의 순위·포함 여부 평가 | **[확인됨]** Context Relevance 및 ground-truth retrieval의 Hit Rate/NDCG/Recall@k가 문서화됨 | 질의, 검색 문맥·점수·ID, k, query별 정답 문맥 집합 | 근거 집합을 직접 만들어야 합니다. 검색 점수만으로 답변 품질을 단정하지 않습니다. |
| RAG Generation | 답변이 질문에 맞고 근거가 지지하는지 평가 | **[확인됨]** Answer Relevance와 Groundedness 기본 provider 함수 제공 | 원 질문, 최종 답변, retrieval contexts; 선택적으로 예상 답변 | LLM judge 편향 가능. 잘못된 원문을 충실히 요약해도 Groundedness는 높을 수 있습니다. |
| Hallucination | 근거 밖 주장/근거 부족 답변 탐지 신호 수집 | **[추론]** Groundedness 저하, 문맥 부족 시 거부 여부 등으로 의심 사례를 찾음. 보편적인 참/거짓 판정기로 취급하지 않음 | 질문, 답변 주장, 검색 근거, 답변 가능/불가능 레이블, 출처 검증 | **[확인됨]** RAG Triad는 위험 신호를 평가합니다. 한 점수가 “환각 없음”을 보장하지 않습니다. |
| Ontology–RAG 정합성 | 답변의 엔티티·관계 주장이 특정 팩 사실과 맞는지 확인 | **[확인됨] 기본 Ontology consistency 검사기는 확인되지 않음.** 사용자 함수를 Metric으로 기록할 수는 있음 | 버전 고정 팩, 정규화된 답변 triple/ID, 유효 관계·evidence/contradiction label | 추출·정규화 오류, 팩 자체 오류가 점수에 반영됩니다. 원천 진실 판정은 별도 확인해야 합니다. |
| 비교 평가 | Ontology 기반 검색과 일반 검색 비교 | **[추론]** TruLens 추적·메트릭/버전 비교 활용 가능 | 같은 질문 세트, 생성 모델·프롬프트·토큰 예산, gold retrieval IDs/answer, 두 검색 결과 | 설정을 통제하지 않으면 차이를 Ontology 효과라고 해석할 수 없습니다. |

---

## 4. Ontology Pack 검증 절차

다음 결과는 TruLens 점수와 별도로 보관합니다.

1. **팩 고정 — [예시 절차].** Pack ID, version, format/grammar 버전, archive SHA-256, 평가 날짜를 기록합니다. 비교 실험 중 팩을 바꾸지 않습니다.
2. **manifest 및 파일 확인 — [확인됨: Pack v1 규격].** 필수 파일과 포맷 버전을 검사합니다. manifest의 수량·해시와 실제 JSONL 수량·해시가 일치하는지 확인합니다.
3. **스키마·문법 검사 — [확인됨: Pack v1 규격].** 노드 space/node type, 엣지의 (from_space, to_space, relation)이 해당 팩 버전 grammar에 유효한지 검사합니다. 실제 grammar/schema가 없으면 결과는 **미확인**입니다.
4. **그래프 무결성 — [확인됨: Pack v1 규격].** 모든 edge endpoint 존재 여부, 중복 ID·고아 노드·잘못된 references를 검사합니다.
5. **근거 추적성 — [확인됨: Pack v1 규격].** promoted node/edge의 evidence_refs가 evidence index 항목을 가리키는지 확인합니다. 원천 파일·해시·위치와 주장 내용이 맞는지 표본 또는 전수 확인합니다.
6. **모순은 구조 오류와 분리 — [추론].** 상충 사실이 그래프에 있다는 이유만으로 즉시 깨진 그래프라고 판정하지 않습니다. 명시된 contradiction relation인지, 출처·시점·적용 범위가 다른지 검토합니다. 의미적 진실성은 원천 자료 검토가 필요합니다.
7. **RAG 검색 테스트 — [예시 절차].** 골든 질의마다 정답 evidence IDs, 허용 관계 경로, 기대 abstain 여부를 작성합니다. 실제 검색 ID·text·rank와 비교하고 별도 TruLens trace에 query/context/answer를 기록합니다.
8. **결과 분리.** Pack 구조 검사 보고서, retrieval IR 지표, TruLens LLM judge 점수, 사람의 source audit을 다른 결과로 유지합니다. 구조 검사 성공과 답변 정확성을 하나의 pass/fail로 합치지 않습니다.

**[미확인]** 공개 저장소에는 grammar validator가 있지만, 대상 팩의 실제 문법 파일·ZIP을 확인하지 않았으므로 이 문서에는 validator 실행 결과가 없습니다.

---

## 5. RAG 평가 지표와 검증 절차

### 지표 정의 및 지원 여부

| 지표 | 정의 | TruLens API / 확인 버전 | 입력·구성 | 해석 및 한계 |
|---|---|---|---|---|
| Context Relevance | 검색된 각 컨텍스트가 질문과 얼마나 관련되는지 평가 | **[확인됨]** OpenAI Provider의 `context_relevance_with_cot_reasons`; `Metric`, `Selector.select_context(collect_list=False)`, 평균 집계. 설치본 2.14.0 및 공식 API 문서 확인 | 질문 + 각 retrieved context. gold context 필수 아님 | 점수가 높아도 사실 정확성·정답 문맥 누락·정답 ID를 보장하지 않습니다. |
| Answer Relevance | 최종 답변이 원 질문에 응답하는 정도 | **[확인됨]** `relevance_with_cot_reasons`; query/answer Selector | query + final answer. 정답 답변 없이 가능 | 질문 주제에 맞는 정도이지 사실성·근거성·완전성이 아닙니다. |
| Groundedness | 답변 주장이 retrieval context에 의해 지지되는 정도 | **[확인됨]** `groundedness_measure_with_cot_reasons` 또는 `groundedness_measure_with_cot_reasons_consider_answerability`; 전체 contexts를 묶어 입력 | 질문, 답변, retrieved contexts | 근거 자체가 틀려도 답변이 그 근거를 따르면 높게 나올 수 있습니다. abstention 취급은 선택한 함수의 의미를 확인합니다. |
| Faithfulness | 팀이 정의하는 “답변 주장과 허용 근거의 일치” | **[확인됨]** 조사한 Provider API에서 범용 함수명 `faithfulness`는 확인하지 못함. Groundedness와 같은 지표라고 표기하지 않음 | 팀 정의, 답변 claims, evidence, 필요 시 citations | **[추론]** 위 정의라면 Groundedness가 관련 측정이지만 동일하다고 단정하지 않습니다. 별도 의미라면 custom metric 필요 |
| Retrieval Precision@k | 상위 k 중 관련 정답 항목 비율: (|R_k ∩ G| / |R_k|) | **[확인됨]** 공식 retrieval 예제에서 기본 `precision_at_k` 함수는 확인하지 못함. ID 기반 custom 계산 권장 | query, 실제 retrieved IDs/rank, 정답 relevant IDs, k | 검색 항목 정밀도. 빈 검색·빈 gold set에 대한 정책을 먼저 정해야 함 |
| Retrieval Recall@k | 정답 relevant 항목 중 상위 k에 검색된 비율: (|R_k ∩ G| / |G|) | **[확인됨]** 공식 ground truth retrieval quickstart에 Recall@k 설명; `GroundTruthAgreement` 기반 평가 흐름 | query별 gold relevant chunks/evidence, 실제 검색 목록/score, k | gold annotation의 완전성에 좌우됨 |
| Hallucination 관련 평가 | 검색 근거가 없는 답변 주장을 찾는 위험 평가 | **[확인됨]** 기본 RAG Triad에 Groundedness/Context Relevance/Answer Relevance. 보편적 binary hallucination 함수로 부르지 않음 | question, contexts, answer, abstain label, 출처 확인 | judge 점수는 의심 사례 우선순위 신호. 저점수로 환각 확정, 고점수로 무환각 확정 금지 |
| Ontology Consistency | 답변의 정규화된 subject-relation-object 주장이 고정된 Pack snapshot의 허용 사실/제약과 맞는 정도 | **[확인됨]** ontology 추론·grammar 검증용 기본 TruLens 함수는 확인되지 않음. `Metric(implementation=사용자 함수,...)`로 custom 점수 기록 가능 | 동일 pack ID/version, 추출 triple/IDs, edges/constraints/evidence, 모순 정책 | 공식 TruLens 정의가 아님. 추출 오류·원천 팩 오류를 따로 추적해야 함 |

- **[확인됨]** 공식 retrieval ground-truth 예제는 query와 expected chunks를 포함한 데이터셋으로 IR 지표와 expected response 비교를 보여줍니다. Recall@k, NDCG@k, Hit Rate는 문서화되어 있습니다. [Retrieval Ground Truth 가이드](https://www.trulens.org/getting_started/quickstarts/groundtruth_evals_for_retrieval_systems/)
- **[확인됨]** 공식 Selector 문서는 retrieval context를 각각 평가할 때 `collect_list=False`, 답변 전체를 근거와 비교할 때 `collect_list=True`를 설명합니다. [Selector 문서](https://www.trulens.org/component_guides/evaluation/feedback_selectors/selecting_components/)
- **[추론·예시 정의]** Ontology consistency를 단순화한다면 답변에서 추출한 triple 집합 P와 검증된 허용 triple 집합 G에 대해 (|P∩G|/|P|)를 볼 수 있습니다. 모순·미확인·출처 불일치를 별도 상태로 두고, 팩에 없다는 이유만으로 현실에서 거짓이라고 판정하지 않습니다. 이는 사용자 정의 지표입니다.

### Custom Metric 구성 원칙

1. **[확인됨]** 2.14.0의 `Metric`은 평가 callable, name, selector, 선택적 aggregation을 사용합니다. callable은 float 또는 점수와 metadata를 반환할 수 있습니다. [Metric 소스](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/metric/metric.py>), [Custom Metric 문서](https://www.trulens.org/component_guides/evaluation/feedback_implementations/custom_feedback_functions/)
2. **[예시]** Precision@k는 정답 ID와 결과 ID로 Python에서 결정론적으로 계산할 수 있습니다. LLM judge로 대체할 필요는 없습니다.
3. **[추론]** Ontology consistency는 정규화 triple/evidence를 입력으로 받아 valid / contradicted / unknown / unparseable 상태별 비율과 근거 ID를 기록하는 것이 하나의 수치보다 감사하기 쉽습니다.
4. **[미확인]** OpenCrab 결과의 실제 JSON 경로·node ID·evidence ID 필드는 확인되지 않아 전용 evaluator는 작성하지 않았습니다.

---

## 6. TruLens 설치 및 실행 가이드

### Step 1. 실행 환경 구성

- **[확인됨·로컬]** TruLens 2.14.0은 Python 3.10 이상, 4 미만 메타데이터입니다. 아래 MCP SDK 2.2.0 예제는 Python 3.11 이상을 권장합니다. 조사된 활성 환경은 Python 3.13입니다.
- **[확인됨]** 공식 MCP SDK 릴리스 기록은 2.2.0을 표시하고 v2 API를 제공합니다. v2는 v1과 API 변경이 있으므로 아래 코드는 v2 API로 고정합니다. ([릴리스](https://github.com/modelcontextprotocol/python-sdk/releases), [공식 저장소](https://github.com/modelcontextprotocol/python-sdk))
- **[예시] Windows PowerShell 설치.** 새 가상환경에서 버전을 고정합니다. 이 명령들은 가이드용이며 이번 조사 중 실행하지 않았습니다.

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install "trulens==2.14.0" "trulens-providers-openai==2.14.0" "mcp==2.2.0" openai
.\.venv\Scripts\python.exe -m pip check
.\.venv\Scripts\python.exe -c "import importlib.metadata as m; print({p:m.version(p) for p in ['trulens','trulens-providers-openai','mcp','openai']})"
```

- **[예상 출력]** 설치 명령이 성공하면 pip가 오류 없이 끝나고, 버전 확인은 설치된 네 패키지의 실제 버전을 출력합니다. 이 환경에서 새 가상환경 설치를 실행한 것은 아닙니다.

- **[확인됨·로컬 README]** 현재 작업 공간 README는 한글 경로에서 만든 .venv의 `ensurepip` 실패 사례를 기록하며 ASCII 경로 가상환경을 우회책으로 설명합니다. venv 생성이 실패하면 영문 경로에 새 프로젝트/venv를 두십시오. [작업 공간 README](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/05_ragsystem/README.md>)
- **[확인됨]** TruLens OpenAI 평가 provider는 별도 패키지 `trulens-providers-openai`로 배포됩니다. ([optional packages](https://www.trulens.org/contributing/optional/), [TruLens 2.14.0 PyPI](https://pypi.org/project/trulens/2.14.0/))
- **API 키 — [확인됨·공식 provider 문서].** Generator와 judge가 OpenAI를 호출할 경우 `OPENAI_API_KEY`가 필요합니다. 사용 가능한 실제 모델명은 사용자가 `OPENAI_MODEL`과 `OPENAI_JUDGE_MODEL`로 지정합니다. 이 문서는 특정 모델이 계정에서 사용 가능하다고 가정하지 않습니다.
- **버전 확인.** `pip check`와 metadata 명령에서 TruLens core/provider가 2.14.0인지, MCP SDK가 2.2.0인지 확인합니다. 패키지 메타데이터는 계정 인증·OpenCrab schema·실제 provider 호출 성공을 검증하지 않습니다.

### Step 2. 기본 실행, 기록, 평가, 결과 확인

- **[확인됨]** 로컬 2.14.0 소스에서 `TruSession`, `Metric`, `Selector`, `TruApp`, OTel instrumentation, Dashboard API를 확인했습니다.
- **[확인됨·2.14.0 로컬 소스]** OTel tracing은 기본 활성화로 판단하며 `TRULENS_OTEL_TRACING=0` 또는 `false` 환경 설정으로 비활성화됩니다. 실행 전 이 변수가 tracing을 끄고 있지 않은지 확인하십시오. [otel/utils.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/otel/utils.py>)
- **[예시]** 아래 Section 7 코드는 질문→검색 context→답변을 생성하고 Context Relevance, Answer Relevance, Groundedness를 기록합니다.
- **저장 — [확인됨·로컬 소스].** 기본 로컬 DB는 현재 작업 디렉터리의 `default.sqlite`이며 TruSession의 database URL로 다른 SQLAlchemy 호환 저장소를 지정할 수 있습니다. [session.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/session.py>)
- **결과 확인 — [확인됨].** `recording.retrieve_feedback_results()`와 `session.get_leaderboard()`로 평가 결과와 요약을 확인할 수 있습니다. [recording.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/otel/recording.py>), [Quickstart](https://www.trulens.org/getting_started/quickstarts/quickstart/)
- **Dashboard — [확인됨].** Python에서 `from trulens.dashboard import run_dashboard; run_dashboard(session)`를 사용합니다. 설치된 2.14.0 소스에도 같은 함수가 있습니다. [Dashboard 문서](https://www.trulens.org/getting_started/dashboard/), [run.py](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/dashboard/run.py>)

### Step 3. OpenCrab 연동 시 계측 지점

- **[추론]** 개발자 애플리케이션이 제어할 수 있는 경계에서 아래 값을 분리해 기록합니다.
  - 입력 질문과 Pack ID/version
  - MCP tool name, input arguments, 성공/실패, 응답 원문
  - 실제 Generator에 전달한 evidence text/ID/rank/score/source locator
  - 생성된 최종 answer
- **[확인됨]** TruLens MCP 계측은 `SpanAttributes.SpanType.MCP`와 tool name/input/output/error 속성을 사용합니다. RAG 평가에는 retrieval span의 `RETRIEVED_CONTEXTS`와 root span의 질문/답변을 기록할 수 있습니다. ([MCP 계측](https://www.trulens.org/component_guides/instrumentation/mcp/), [Context selector](https://www.trulens.org/component_guides/evaluation/feedback_selectors/selecting_components/))
- **[미확인]** SaaS가 내부 Retriever/Generator span을 외부로 내보내는지는 확인하지 못했습니다. 외부 MCP 호출 계측으로 SaaS 내부 trace가 소급 생성되지는 않습니다.

### Step 4. Ontology Pack 검증과 RAG 검증 분리

1. Pack validator/grammar/schema/evidence 검사 결과를 따로 보관합니다.
2. 골든 질의에 정답 node/edge/evidence IDs와 기대 답변 또는 abstain을 지정합니다.
3. MCP 결과에서 실제 evidence만 context로 뽑도록 JSON path/text field를 확인합니다.
4. TruLens에 query/context/answer를 기록하고 RAG Triad를 평가합니다.
5. Retrieval IDs로 별도 Precision/Recall@k를 계산합니다.
6. 답변 relation/node 주장을 고정된 Pack snapshot과 대조하는 custom/external consistency checker를 추가합니다.
7. 구조 검사, retrieval 점수, judge 점수, human source audit을 별도 열로 비교합니다.

---

## 7. OpenCrab 연동 코드 및 실행 방법

### 전제와 버전

- **버전:** TruLens 2.14.0과 OpenAI provider 2.14.0은 사용자 제공/활성 환경의 메타데이터에서 확인했습니다. 아래 TruLens 코드는 해당 버전의 `TruApp(..., feedbacks=metrics)` API를 사용합니다.
- **MCP:** MCP 호출 방식은 공식 Python SDK 2.2.0의 `Client(url)`, `list_tools()`, `call_tool()`에 맞췄습니다. 사용자 환경에는 MCP SDK가 없으므로 설치가 필요합니다.
- **필요 패키지:** `trulens==2.14.0`, `trulens-providers-openai==2.14.0`, `mcp==2.2.0`, `openai` 및 MCP SDK HTTP client 의존성.
- **실행 전제:** OpenCrab MCP endpoint와 인증 방식, 지원 transport, 실제 tool name/input schema, 결과의 evidence JSON 경로, OpenAI 계정에서 사용 가능한 모델명이 있어야 합니다.
- **[미확인 인증 주의]** OpenCrab 공개 가이드는 key가 포함된 MCP URL 형태를 안내하지만 연결 계정 UI의 인증 안내는 Authorization header를 사용한다고 표시되어 있었습니다. 두 안내가 달라 코드에서 인증을 고정하지 않습니다. 계정에서 실제 transport/auth 조합을 확인하십시오. 비밀이 담긴 URL/token을 코드·로그·버전 관리에 남기지 마십시오.
- **[미확인 transport]** 코드의 기본 연결 경로는 MCP SDK 2.2.0의 Streamable HTTP입니다. OpenCrab endpoint가 이 transport를 지원하는지 공개 가이드에서 확정하지 못했으므로, 계정의 연결 정보에서 확인해야 합니다. SSE/다른 transport라면 클라이언트 transport 구성을 바꿔야 합니다.

### 전체 예제 — generic MCP adapter + TruLens 2.14.0

**[예시]** 이 코드는 OpenCrab 전용 내부 API나 결과 필드를 가정하지 않습니다. 도구 목록에서 지정한 MCP tool을 찾고 live schema의 질문 필드에 질의를 전달합니다. 검색 context는 도구 응답을 확인한 뒤 JSON Pointer와 text field로 지정해야 합니다. SaaS가 evidence text를 반환하지 않으면 Groundedness 평가를 수행할 수 없습니다.

```python
# opencrab_trulens_bridge.py
from __future__ import annotations

import argparse
import asyncio
import json
import os
import statistics
from typing import Any

from mcp import Client
from openai import OpenAI as GeneratorClient
from trulens.apps.app import TruApp
from trulens.core import Metric, Selector, TruSession
from trulens.core.otel.instrument import instrument
from trulens.otel.semconv.trace import SpanAttributes
from trulens.providers.openai import OpenAI as JudgeProvider


def required_env(name: str) -> str:
    value = os.getenv(name)
    if not value:
        raise RuntimeError(f"Required environment variable is missing: {name}")
    return value


def get_pointer(obj: Any, pointer: str) -> Any:
    # JSON Pointer path must come from an inspected MCP response.
    if not pointer.startswith("/"):
        raise ValueError("OPENCRAB_CONTEXT_JSON_POINTER must start with '/'")
    current = obj
    for raw_token in pointer[1:].split("/"):
        token = raw_token.replace("~1", "/").replace("~0", "~")
        if isinstance(current, list):
            current = current[int(token)]
        elif isinstance(current, dict):
            current = current[token]
        else:
            raise TypeError(f"Cannot continue JSON Pointer at token: {token}")
    return current


async def call_opencrab_tool_async(
    endpoint: str, tool_name: str, arguments: dict[str, Any]
) -> str:
    async def call_with_client(client: Client) -> str:
        async with client as connected:
            listed = await connected.list_tools()
            available = {tool.name: tool for tool in listed.tools}
            if tool_name not in available:
                raise RuntimeError(
                    "Configured tool not found. Available tools: "
                    + ", ".join(sorted(available))
                )

            tool = available[tool_name]
            schema = tool.input_schema or {}
            properties = schema.get("properties", {})
            query_field = required_env("OPENCRAB_QUERY_FIELD")
            if query_field not in properties:
                raise RuntimeError(
                    f"Query field '{query_field}' is absent from the live tool schema."
                )

            args = json.loads(os.getenv("OPENCRAB_QUERY_ARGS_JSON", "{}"))
            if not isinstance(args, dict):
                raise TypeError("OPENCRAB_QUERY_ARGS_JSON must be a JSON object.")
            args[query_field] = arguments["query"]

            pack_id = os.getenv("OPENCRAB_PACK_ID")
            pack_field = os.getenv("OPENCRAB_PACK_FIELD")
            if pack_id:
                if not pack_field:
                    raise RuntimeError(
                        "Set OPENCRAB_PACK_FIELD to the actual schema field "
                        "when OPENCRAB_PACK_ID is set."
                    )
                if pack_field not in properties:
                    raise RuntimeError(
                        f"Pack field '{pack_field}' is absent from the live tool schema."
                    )
                args[pack_field] = pack_id

            missing = [
                key for key in schema.get("required", []) if key not in args
            ]
            if missing:
                raise RuntimeError(
                    "Missing required tool arguments; set them in "
                    f"OPENCRAB_QUERY_ARGS_JSON: {missing}"
                )

            result = await connected.call_tool(tool_name, args)
            if result.is_error:
                raise RuntimeError("OpenCrab MCP tool returned is_error=true.")

            payload = result.structured_content
            if payload is None:
                payload = {
                    "content": [
                        {
                            "type": getattr(block, "type", ""),
                            "text": getattr(block, "text", ""),
                        }
                        for block in result.content
                        if getattr(block, "text", None) is not None
                    ]
                }
            return json.dumps(payload, ensure_ascii=False, default=str)

    bearer_token = os.getenv("OPENCRAB_BEARER_TOKEN")
    if bearer_token:
        # SDK v2 transport accepts a caller-configured HTTP client.
        import httpx2
        from mcp.client.streamable_http import streamable_http_client

        async with httpx2.AsyncClient(
            headers={"Authorization": f"Bearer {bearer_token}"}
        ) as http_client:
            transport = streamable_http_client(
                endpoint, http_client=http_client
            )
            return await call_with_client(Client(transport))

    # Use this branch only if the account endpoint carries the required auth.
    return await call_with_client(Client(endpoint))


class OpenCrabRAG:
    def __init__(self) -> None:
        self.endpoint = required_env("OPENCRAB_MCP_URL")
        self.tool_name = required_env("OPENCRAB_QUERY_TOOL")
        self.generator_model = required_env("OPENAI_MODEL")
        self.context_pointer = required_env("OPENCRAB_CONTEXT_JSON_POINTER")
        self.context_text_field = os.getenv("OPENCRAB_CONTEXT_TEXT_FIELD")
        self.generator = GeneratorClient()

    @instrument(
        span_type=SpanAttributes.SpanType.MCP,
        attributes={
            SpanAttributes.MCP.TOOL_NAME: "tool_name",
            SpanAttributes.MCP.INPUT_ARGUMENTS: "arguments",
            SpanAttributes.MCP.OUTPUT_CONTENT: "return",
        },
    )
    def call_tool(self, tool_name: str, arguments: dict[str, Any]) -> str:
        return asyncio.run(
            call_opencrab_tool_async(self.endpoint, tool_name, arguments)
        )

    @instrument(
        span_type=SpanAttributes.SpanType.RETRIEVAL,
        attributes={
            SpanAttributes.RETRIEVAL.QUERY_TEXT: "query",
            SpanAttributes.RETRIEVAL.RETRIEVED_CONTEXTS: "return",
        },
    )
    def retrieve(self, query: str) -> list[str]:
        raw = self.call_tool(
            self.tool_name,
            {"query": query},
        )
        payload = json.loads(raw)
        selected = get_pointer(payload, self.context_pointer)
        items = selected if isinstance(selected, list) else [selected]

        contexts: list[str] = []
        for item in items:
            if isinstance(item, str):
                text = item
            elif isinstance(item, dict) and self.context_text_field:
                value = item.get(self.context_text_field)
                if not isinstance(value, str):
                    raise TypeError(
                        "Configured context text field is not a string."
                    )
                text = value
            else:
                raise TypeError(
                    "Selected evidence must be text or an object with the "
                    "configured OPENCRAB_CONTEXT_TEXT_FIELD."
                )
            if text.strip():
                contexts.append(text)

        if not contexts:
            raise RuntimeError(
                "No evidence text was extracted; inspect the live MCP result "
                "and configure OPENCRAB_CONTEXT_JSON_POINTER."
            )
        return contexts

    @instrument(span_type=SpanAttributes.SpanType.GENERATION)
    def generate(self, query: str, contexts: list[str]) -> str:
        completion = self.generator.chat.completions.create(
            model=self.generator_model,
            temperature=0,
            messages=[
                {
                    "role": "system",
                    "content": (
                        "Answer only from the supplied evidence. If evidence "
                        "does not answer the question, say that the evidence "
                        "is insufficient. Do not invent identifiers or facts."
                    ),
                },
                {
                    "role": "user",
                    "content": (
                        "Evidence:\n"
                        + "\n---\n".join(contexts)
                        + "\n\nQuestion:\n"
                        + query
                    ),
                },
            ],
        )
        answer = completion.choices[0].message.content
        return answer or "The evidence is insufficient to answer."

    @instrument(
        span_type=SpanAttributes.SpanType.RECORD_ROOT,
        attributes={
            SpanAttributes.RECORD_ROOT.INPUT: "query",
            SpanAttributes.RECORD_ROOT.OUTPUT: "return",
        },
    )
    def query(self, query: str) -> str:
        contexts = self.retrieve(query)
        return self.generate(query, contexts)


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("question")
    parser.add_argument(
        "--dashboard",
        action="store_true",
        help="Start the local TruLens dashboard after evaluation.",
    )
    args = parser.parse_args()

    judge_model = required_env("OPENAI_JUDGE_MODEL")
    provider = JudgeProvider(model_engine=judge_model)

    metrics = [
        Metric(
            implementation=provider.context_relevance_with_cot_reasons,
            name="Context Relevance",
            selectors={
                "question": Selector.select_record_input(),
                "context": Selector.select_context(collect_list=False),
            },
            agg=statistics.mean,
        ),
        Metric(
            implementation=provider.groundedness_measure_with_cot_reasons_consider_answerability,
            name="Groundedness",
            selectors={
                "source": Selector.select_context(collect_list=True),
                "statement": Selector.select_record_output(),
                "question": Selector.select_record_input(),
            },
        ),
        Metric(
            implementation=provider.relevance_with_cot_reasons,
            name="Answer Relevance",
            selectors={
                "prompt": Selector.select_record_input(),
                "response": Selector.select_record_output(),
            },
        ),
    ]

    session = TruSession()
    app = OpenCrabRAG()
    # TruLens 2.14.0 App constructor accepts Metric objects via feedbacks=.
    recorder = TruApp(
        app,
        app_name="OpenCrab-RAG",
        app_version="ontology-pack-eval-1",
        feedbacks=metrics,
    )

    with recorder as recording:
        answer = app.query(args.question)

    print("Answer:")
    print(answer)
    print("\nTruLens metric results:")
    print(recording.retrieve_feedback_results(timeout=180).to_string(index=False))
    print("\nLeaderboard:")
    print(session.get_leaderboard())

    if args.dashboard:
        from trulens.dashboard import run_dashboard
        run_dashboard(session)


if __name__ == "__main__":
    main()
```

### 설정과 실행

1. 별도 MCP Inspector/클라이언트 또는 첫 실행에서 `tools/list` 결과를 확인하고 실제 검색 tool 이름과 input schema를 읽습니다. **[확인됨]** 공식 SDK v2는 `Client(url)`, `list_tools()`, `call_tool()` 패턴을 문서화합니다. ([MCP Client 예제](https://github.com/modelcontextprotocol/python-sdk))
2. 실제 tool 응답을 확인하고 evidence text 배열/객체의 JSON Pointer 경로와 객체일 때 text 필드를 지정합니다. **[미확인]** 이 문서에 OpenCrab 결과 경로를 고정해 넣지 않았습니다.
3. OpenAI key와 OpenCrab endpoint/auth를 비밀 저장소 또는 현재 PowerShell 세션의 환경변수로 설정합니다. 아래는 **변수 이름 예시**이며 필드명·값은 live schema로 확인해야 합니다.

```powershell
$env:OPENCRAB_MCP_URL = "<계정에서 받은 MCP endpoint>"
$env:OPENCRAB_QUERY_TOOL = "<tools/list에서 확인한 tool name>"
$env:OPENCRAB_QUERY_FIELD = "<input schema의 질문 필드>"
$env:OPENCRAB_QUERY_ARGS_JSON = "{}"
$env:OPENCRAB_CONTEXT_JSON_POINTER = "<실제 결과의 evidence 배열 경로>"
$env:OPENCRAB_CONTEXT_TEXT_FIELD = "<evidence 객체라면 실제 text 필드>"
$env:OPENAI_API_KEY = "<안전하게 주입할 비밀값>"
$env:OPENAI_MODEL = "<사용 가능한 생성 모델명>"
$env:OPENAI_JUDGE_MODEL = "<사용 가능한 평가 모델명>"
# 계정이 Authorization: Bearer 방식을 요구할 때만 설정
$env:OPENCRAB_BEARER_TOKEN = "<비밀 토큰>"
# 팩 ID 제한이 필요할 때 실제 input schema에 맞춰 두 값을 설정
$env:OPENCRAB_PACK_FIELD = "<input schema의 pack ID 필드>"
$env:OPENCRAB_PACK_ID = "<실제 pack ID>"
```

- 이 설정 블록은 값 자리를 채우는 예시입니다. 실행 출력은 없으며, 이번 조사에서는 endpoint·키를 입력하거나 이 블록을 실행하지 않았습니다. 토큰이 포함된 URL과 문서 내용을 로컬 TruLens DB에 저장해도 되는지 조직의 데이터 취급 정책을 먼저 적용해야 합니다.

4. 사용자 질문으로 실행합니다.

```powershell
.\.venv\Scripts\python.exe .\opencrab_trulens_bridge.py "팩에 있는 실제 관계 질의"
.\.venv\Scripts\python.exe .\opencrab_trulens_bridge.py "팩에 있는 실제 관계 질의" --dashboard
```

### 예상 출력과 실행 검증 상태

- **[예상 출력 형식, 값은 미측정]** 콘솔에는 생성 답변, TruLens metric 결과 표, leaderboard가 표시되고 로컬 DB에 record가 남습니다. Context Relevance는 검색 context별 평가 결과를 집계합니다. 실제 점수/답변은 OpenCrab 응답과 계정 모델 및 judge 변동에 따라 달라지므로 예상 수치를 적지 않습니다.
- **[실행 검증 상태: 실행하지 않음]** 전체 예제는 실행하지 않았습니다. 외부 MCP 인증, 결과 schema mapping, 모델 접근, TruLens trace/database/dashboard 생성을 검증하지 않았습니다. 확인한 것은 패키지 메타데이터, 소스 API, 공식 문서 및 현재 대화의 OpenCrab MCP read-only 조회입니다.
- **한계:** MCP 결과에 최종 답변만 있고 근거 evidence가 없다면 Context Relevance/Groundedness 입력을 만들 수 없습니다. 응답에 answer와 evidence가 함께 있으면 evidence만 선택해야 합니다. 예제는 OpenCrab SaaS 내부 생성 결과를 직접 계측하지 않습니다.

---

## 8. 통합 테스트 시나리오

아래 X/Y는 **[예시]** 자리표시자입니다. 실제 Pack 내용·테스트 결과가 아닙니다. 각 케이스의 기대 evidence IDs, answer/abstain, 관계 경로, 실제 retrieved IDs를 골든 데이터에 채우십시오.

| 시나리오 | 질문/입력 | 기대 동작 | 평가 방식 | 판정 |
|---|---|---|---|---|
| 팩에 있는 사실 | **[예시]** “X와 Y 사이에 어떤 관계가 기록되어 있나?”; 실제 팩에 X/Y와 edge/evidence 필요 | 해당 edge와 근거를 검색하고 그 관계만 설명 | 정답 edge/evidence ID Recall@k, Context Relevance, Groundedness, source audit | gold edge/evidence 검색 및 근거와 답변 일치가 필요. 수치 기준은 업무 기준으로 사전 합의 |
| 팩에 없는 정보 | **[예시]** 팩에 없는 Z 속성 질의; gold에는 “이 Pack snapshot에서 미확인”이라고 기록 | 사실을 만들지 않고 근거 부족을 답하거나 범위를 제한 | expected abstain, 미지원 claim 수, groundedness, 사람 검토 | 팩 미수록과 현실에서 거짓을 구분. 미지원 구체 사실이 없어야 함 |
| 상충 관계 | **[예시]** X→Y의 상반된 source/time/claim이 있는 데이터 | 모순과 출처/범위를 분리해 응답 | gold contradiction status, evidence ID recall, answer claim audit, custom consistency | 한쪽을 임의 확정하지 않고 상충을 드러내야 함. explicit contradiction edge와 출처 충돌은 별개 |
| 문맥 부족 | 일부 경로만 반환되도록 검색 범위/조건을 제한 | 빠진 경로를 추측하지 않고 불충분을 밝힘 | retrieved gold IDs, Groundedness, expected abstain | 부족한 문맥으로 단정하지 않는지 확인 |
| 잘못된 근거 검색 | **[예시]** 질문과 유사하지만 다른 엔티티/반대 사실 근거를 포함 | 엔티티 ID와 evidence를 구분하고 오검색 근거로 답하지 않음 | gold ID Precision@k, entity mismatch, Groundedness, human audit | LLM relevance만으로 통과 처리하지 않고 exact IDs/출처 대조 |
| 일반 RAG vs Ontology RAG | 동일 질문 세트·동일 생성 조건으로 두 retriever 실행 | 차이와 사례별 원인을 추적 | Precision/Recall@k, Context Relevance, Groundedness, Answer Relevance, gold answer agreement, latency/cost | 같은 pack snapshot, 모델/프롬프트/검색 budget을 통제한 paired 비교 |

- **[예시]** 골든 데이터에 query ID, 질문, pack ID/version, 기대 답변 또는 abstain 여부, relevant evidence/node/edge IDs, 허용 주장, 모순 레이블을 두는 방식이 가능합니다. 이것은 제안 스키마이며 OpenCrab의 실제 데이터 모델이 아닙니다.
- **[미확인]** 실제 OpenCrab Pack이 위 시나리오에 해당하는 facts/contradictions를 포함하는지 조사하지 않았습니다. 실제 검증 결과는 없습니다.

---

## 9. 평가 결과 해석과 개선 방안

### 검색 문제와 생성 문제 구분

- **[추론]** 정답 evidence ID가 검색되지 않거나 낮은 rank에 있고 답변도 사실을 놓치면 우선 Retrieval을 조사합니다.
- **[추론]** 검색 근거가 충분하고 맞는데 답변이 누락·왜곡·과잉 주장하면 generation/prompt를 조사합니다.
- Context Relevance가 높아도 정답 근거가 검색됐다는 보장은 없습니다. exact evidence IDs 및 Recall을 같이 봅니다.
- Groundedness가 낮으면 context 부족, 상충 문맥, unsupported claim, claim 분할/judge 오류 가능성을 조사합니다. 하나의 점수로 원인을 확정하지 않습니다.

### Ontology 오류와 RAG 오류 구분

- Pack 검사 실패와 원천 불일치가 함께 보이면 Pack 생성/정규화/출처 연결 오류 후보입니다.
- Pack 검사는 통과하지만 gold edge/evidence가 검색되지 않으면 indexing/filtering/query/ranking 오류 후보입니다.
- 정답 근거가 검색됐는데 답변이 어긋나면 generation/prompt/answer composition 오류 후보입니다.
- 팩 자체가 원천과 다르면 TruLens만으로 해결할 수 없습니다. 원천을 검토하고 Pack을 수정한 뒤 재평가해야 합니다.
- 위 분류는 **[추론]**이며 trace·Pack 파일·원천 자료를 함께 봐야 합니다.

### 환각 의심 답변 추가 확인

1. 답변을 검증 가능한 개별 주장으로 나눕니다.
2. 각 주장이 근거 evidence에 직접 지지되는지 확인합니다.
3. 근거가 Pack의 원천 source 및 locator/hash와 맞는지 확인합니다.
4. 반대 evidence, 시점/범위 차이, 명시된 contradiction edge를 확인합니다.
5. judge와 결정론적 ID/관계 검사를 비교하고 원문 표본을 사람이 검토합니다.

### 개선 전후 비교

- **Pack:** 누락/깨진 evidence refs, 잘못된 node identity, 관계 endpoint/grammar, 원천 위치, 중복/모순 상태를 수정합니다.
- **Retrieval:** 질의 유형별 lexical/vector/graph 경로, scope/filter, ranker, top-k, multi-hop depth를 바꾸고 변경 변수를 기록합니다. 이 지점은 공개 로컬 엔진의 설정이고 SaaS에서 조정할 수 있는지는 **[미확인]**입니다.
- **Generation:** retrieved context만 사용, 출처 인용, 부족/모순 시 abstain, 근거 밖 관계 추론 제한을 실험합니다.
- **전후 측정:** 고정 Pack snapshot 및 같은 골든 질의로 baseline/변경판을 실행하고 generator/judge/prompt/temperature/context budget을 기록합니다. IR, judge, source audit, latency/token cost와 per-query regressions를 함께 봅니다.
- **임계값:** TruLens의 universal pass threshold는 확인하지 못했습니다. Pack 규격의 recommended defaults는 Pack QA 권고값이며 TruLens 점수 기준이 아닙니다.

---

## 10. 미확인 사항과 추가로 필요한 자료

### 미확인 항목

1. **호스팅 SaaS 내부 구현:** 공개 저장소에서 제외되어 내부 Retriever/Generator 코드, 저장소 구조, 모델/reranker 설정을 확인할 수 없습니다.
2. **실제 Pack:** 대상 pack ID 및 v1 ZIP/manifest/nodes/edges/evidence/quality/grammar를 조사하지 않았습니다.
3. **원격 MCP 설정:** endpoint의 transport와 인증 안내가 공개 guide와 계정 UI에서 일치하지 않았습니다.
4. **tool schema/output:** Codex 연결 plugin schema는 해당 connector의 관찰값입니다. 이를 외부 endpoint wire schema와 같다고 확인할 수 없습니다.
5. **실제 평가 환경:** 평가 key/모델 접근, 질문 세트, 정답 ID, latency/token 측정, TruLens trace/database/dashboard는 실행하지 않았습니다.
6. **온톨로지 정책:** 특정 Pack의 contradiction 처리, closed-world 가정, 유효 기간, 출처 신뢰 순위를 알 수 없습니다.

### 실제 프로젝트 검증에 필요한 자료

- 대상 Pack의 immutable ID/version, ZIP 또는 필수 파일과 적용 grammar/schema
- 민감값 제거한 MCP tool list, input JSON Schema, 결과 샘플
- 계정에서 확인한 endpoint transport/auth 방식(비밀 토큰 값은 제공할 필요 없음)
- 골든 질문, 정답 evidence/node/edge IDs, expected answer 또는 abstain
- 실제 generation 단계가 호출되는 코드/서비스 경계 및 사용할 모델명
- 비교에 필요한 수용 기준과 비용/지연 기준

자료가 제공되기 전까지 위 코드는 **설정 가능한 일반 MCP adapter 예제**입니다. 완성된 OpenCrab 제품 플러그인이라고 부르지 않습니다.

---

## 11. 근거가 되는 공식 문서 및 소스코드 링크

### OpenCrab

- [공식 제품 사이트](https://opencrab.sh/)
- [사용 가이드와 MCP 연결 설명](https://opencrab.sh/guide)
- [공개 GitHub 저장소와 SaaS 경계](https://github.com/AlexAI-MCP/OpenCrab)
- [OpenCrab Pack v1 ZIP 규격](https://github.com/AlexAI-MCP/OpenCrab/blob/main/docs/opencrab-pack-v1.md)
- [LocalCrab/OpenCrab 관계 문서](https://github.com/AlexAI-MCP/OpenCrab/blob/main/docs/localcrab-opencrab-relationship.md)
- [공개 Local Hybrid Query 코드](https://github.com/AlexAI-MCP/OpenCrab/blob/main/opencrab/ontology/query.py)

### TruLens

- [Quickstart](https://www.trulens.org/getting_started/quickstarts/quickstart/)
- [RAG Triad](https://www.trulens.org/getting_started/core_concepts/rag_triad/)
- [Metric/custom metric](https://www.trulens.org/component_guides/evaluation/feedback_implementations/custom_feedback_functions/)
- [OpenAI provider API](https://www.trulens.org/reference/trulens/providers/openai/provider/)
- [Retrieval span Selector](https://www.trulens.org/component_guides/evaluation/feedback_selectors/selecting_components/)
- [MCP span instrumentation](https://www.trulens.org/component_guides/instrumentation/mcp/)
- [Retrieval Ground Truth](https://www.trulens.org/getting_started/quickstarts/groundtruth_evals_for_retrieval_systems/)
- [Dashboard](https://www.trulens.org/getting_started/dashboard/)
- [TruLens 2.14.0 PyPI](https://pypi.org/project/trulens/2.14.0/)
- [Optional provider packages](https://www.trulens.org/contributing/optional/)

### MCP 및 TypeSafe

- [MCP Python SDK 공식 저장소](https://github.com/modelcontextprotocol/python-sdk)
- [MCP SDK 릴리스](https://github.com/modelcontextprotocol/python-sdk/releases)
- [MCP SDK HTTP transport/auth 설정](https://py.sdk.modelcontextprotocol.io/client/transports/)
- [TypeSafe Python SDK](https://docs.typesafe.ai/sdk/python)
- [TypeSafe Choice primitive](https://docs.typesafe.ai/primitives/choice)

### 확인한 로컬 파일

- [TruLens 2.14.0 METADATA](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens-2.14.0.dist-info/METADATA>)
- [TruLens Core API export](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/__init__.py>)
- [TruLens App wrapper](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/apps/app.py>)
- [TruLens Metric source](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/metric/metric.py>)
- [TruLens recording source](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/04_trulens/trulens/core/otel/recording.py>)
- [현재 작업 공간 README](<C:/Users/n9911/OneDrive/바탕 화면/남진 apps/05_ragsystem/README.md>)

---

## 확인 상태 요약

- TypeSafe 분류: **확인됨** — `research_and_write_guide`.
- OpenCrab: 공식 제품/가이드/공개 저장소 조사 및 연결된 MCP read-only 조회 **확인됨**.
- TruLens: 지정 라이브러리 버전 메타데이터와 2.14.0 소스 확인 **확인됨**.
- MCP–TruLens 외부 Python 연동/평가: **실행하지 않음**.
- TruLens 기록, 실측 점수, 대시보드 결과: **생성하지 않음**.
- OpenCrab 실제 Pack 파일 및 사용자 골든 데이터 검증: **미확인**.

