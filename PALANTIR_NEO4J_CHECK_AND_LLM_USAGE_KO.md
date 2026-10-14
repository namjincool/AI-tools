# 현재 팩의 기반 기술 확인과 LLM RAG 사용법

확인일: 2026-10-14, Asia/Seoul. 조사 대상은 현재 작업 폴더의 `rag_ontology_2026-10-14`와 그 팩이 사용하는 설치된 LocalCrab 실행 코드다. PC의 모든 시스템이나 별도 원격 서버 전체를 조사한 결과가 아니다.

## 확인 결론

현재 팩은 **LocalCrab의 로컬 SQLite 그래프·Chroma 벡터 색인·JSON 문서 저장소를 사용하는 후보 팩**이다. Palantir의 설계 검증 지침을 참고한 기록은 있지만, 이 팩을 Palantir Foundry에 구현·배포했다는 근거는 확인되지 않았다. 조사한 실행 경로는 Neo4j DB를 생성하거나 질의하지 않으며, Neo4j 공식 `neo4j-graphrag` 구현을 사용한 근거도 확인되지 않았다.

| 확인 항목 | 결과 | 확인 근거 |
|---|---|---|
| Palantir 설계·검증 지침 참고 | 확인됨. 실제 업무 질문 기반 검증 원칙을 참고했다고 제작 보고서에 기록 | [기존 제작 결과](../rag_ontology_2026-10-14/REPORT_KO.md), [기존 Aside 대조 기록](../rag_ontology_2026-10-14/methods/aside_verification.md) |
| Palantir Foundry 플랫폼에 구현·배포 | 조사한 팩의 코드·설정에서 확인되지 않음 | [조사 범위와 결과](basis_inspection.json), [팩 서버 설정](../rag_ontology_2026-10-14/localcrab.mcp.json) |
| 현재 그래프 저장소 | `LocalGraphStore`, SQLite `graph.db` | [설치된 저장소 생성 코드 사본](runtime_factory.py), [설치된 그래프 저장소 코드 사본](runtime_local_graph_store.py) |
| 현재 벡터 저장소 | `ChromaStore`, `local_mode=True`, 로컬 `chroma/` | [설치된 저장소 생성 코드 사본](runtime_factory.py) |
| 현재 문서 저장소 | `LocalDocStore`, 로컬 `docs/` | [설치된 저장소 생성 코드 사본](runtime_factory.py) |
| Neo4j 드라이버 설치 | 설치 환경에 `neo4j` 6.2.0 존재. 설치 여부와 실제 사용 여부는 별도 | [패키지 메타데이터 조회](basis_inspection.json) |
| Neo4j 공식 GraphRAG 패키지 사용 | 팩 코드의 import·호출 근거 없음. 조사한 LocalCrab 환경에 `neo4j-graphrag` 패키지 메타데이터도 없음 | [검사한 파일 목록·검색 패턴·결과](basis_inspection.json) |
| 현재 검색 구현 | `query_localcrab.py`가 설치된 LocalCrab `ontology_query`를 호출. 네이티브 검색 코드는 벡터·BM25·그래프 확장을 포함 | [팩 검색 코드](../rag_ontology_2026-10-14/scripts/query_localcrab.py), [설치된 검색 엔진 사본](runtime_hybrid_query.py) |

코드의 `"neo4j": graph`라는 내부 키는 Neo4j DB 연결의 증거가 아니다. 실제 `graph` 값은 `make_graph_store`의 반환값이며, 그 함수는 현재 SQLite `LocalGraphStore`를 만든다. [현재 실행 컨텍스트 함수 사본](runtime_context.py), [실제 팩토리 사본](runtime_factory.py)

TypeSafe `jev-1.13.0`의 이번 분류는 `verify_basis_and_document_usage`, confidence 1.0이다. 이는 **이번 요청의 작업 유형**을 분류한 결과이며, Palantir/Neo4j 기반 여부나 답변 정확성을 검증한 값이 아니다. 승인한 작업 설명 한 문장과 분류 선택지만 전송했다. [실제 요청](typesafe_request.json), [실제 응답](typesafe_response.json), [TypeSafe API](https://docs.typesafe.ai/api)

## 공식 자료와 현재 구현의 차이

Aside MCP와 Aside 브라우저로 Palantir 공식 검증 지침 및 Neo4j 공식 GraphRAG 문서를 직접 열고 snapshot으로 읽었다. 페이지를 읽었다는 사실은 현재 팩이 해당 플랫폼에 구축되었다는 증거와 구분한다. [이번 공식 자료 확인 기록](official_sources.json)

Palantir는 구조가 맞는 것과 업무에 실제로 사용할 수 있는 것을 구분한다. 실제 업무 질문, 답변 경로, 시간, 사람과 에이전트의 수행을 검증하도록 안내한다. 현재 팩은 사용자 QA-01~QA-10을 저장하고 출처·범위 검사를 실행한 기록이 있지만, 독립 업무 정답과 사람·모델 수행 평가를 완료했다고 기록하지 않는다. [Palantir 공식 검증 지침](https://www.palantir.com/docs/foundry/ontology/ontology-design-validation), [팩 평가 범위](../rag_ontology_2026-10-14/REPORT_KO.md)

Palantir Ontology는 플랫폼의 데이터와 현실 개체를 연결하며, 객체·속성·링크뿐 아니라 액션·함수·동적 보안도 설명한다. 현재 팩의 문서·청크·용어·인용 구조가 그 플랫폼의 배포와 동작을 재현한다고 판단할 근거는 없다. [Palantir 공식 개요](https://www.palantir.com/docs/foundry/ontology/overview), [현재 팩의 사용 타입·관계 제약](../rag_ontology_2026-10-14/ontology/schema.json)

Neo4j 공식 Python GraphRAG 가이드는 Neo4j 드라이버, 검색기, LLM을 구성하고 검색 컨텍스트를 모델에 전달하는 예를 제공한다. 벡터 검색 뒤 Cypher로 주변 그래프를 조회하는 검색기도 설명한다. 현재 팩은 자체 LocalCrab 엔진을 사용한다. 두 구현 모두 그래프나 벡터 검색을 포함할 수 있다는 유사성만으로 Neo4j 공식 패키지를 기반으로 만들었다고 판단할 수 없다. [Neo4j 공식 패키지](https://neo4j.com/docs/neo4j-graphrag-python/current/index.html), [Neo4j 공식 RAG 가이드](https://neo4j.com/docs/neo4j-graphrag-python/current/user_guide_rag.html)

Microsoft 공식 GraphRAG의 커뮤니티 보고서·Global Search 및 OG-RAG 재현도 현재 팩의 제작 완료 범위에 포함되지 않는다. 이는 기존 manifest와 제작 보고서에 명시돼 있다. [팩 정의](../rag_ontology_2026-10-14/manifest.json), [제작 결과](../rag_ontology_2026-10-14/REPORT_KO.md)

## 실제 팩에 있는 그래프와 색인

이번 파일 직접 조회와 읽기 전용 SQLite 조회 결과:

| 항목 | 확인 수 |
|---|---:|
| Document | 23 |
| TextUnit | 2,259 |
| Concept | 476 |
| Claim | 583 |
| 전체 노드 | 3,341 |
| contains 관계 | 2,259 |
| mentions 관계 | 1,278 |
| supports 관계 | 583 |
| 전체 관계 | 4,120 |
| 존재하지 않는 끝점을 참조한 관계 | 0 |
| 내보내기 Chroma의 embeddings 레코드 | 9,376 |

`graph/nodes.jsonl`, `graph/edges.jsonl`, `evidence/chunks.jsonl`의 현재 해시는 manifest에 기록된 해당 해시와 일치했다. 내보내기 `graph.db`의 `graph_nodes`, `graph_edges` 수도 각각 3,341과 4,120이었다. [직접 조회 결과](basis_inspection.json)

실제 관계는 `Document → contains → TextUnit`, `TextUnit → mentions → Concept`, `TextUnit → supports → Claim`이다. Claim은 조문 그대로의 인용이며 업무 의무·법령 적용·개념 위계·요구사항 간 인과관계를 해석해 확정한 공리가 아니다. 추가 관계를 추정하지 않는다. [현재 스키마](../rag_ontology_2026-10-14/ontology/schema.json), [현재 그래프](../rag_ontology_2026-10-14/graph/edges.jsonl)

활성 검색 자료는 23개이며, RFP는 작성 예시다. 현행 법령 판단, 개별 사업 적용, 확정 사업 예산·일정·요구사항을 승인한 상태가 아니다. [자료 목록](../rag_ontology_2026-10-14/sources.json), [후보 상태](../rag_ontology_2026-10-14/manifest.json)

## LLM에서 RAG로 사용하는 방법

현재 제공 코드의 흐름은 `질문 → LocalCrab 검색 → 정식 원문 청크와 출처 확인 → LLM 입력 준비`다. `prepare_context.py`는 검색 결과의 `node_id`를 정식 청크에 연결하고, 원본·추출문 SHA-256과 추출문 문자 범위를 확인한다. 모델에 전달하는 필드는 질문, 자료 목록, 원문 evidence, 검색 방법, 제한 사항이다. **전체 온톨로지 그래프나 모든 관계를 자동으로 LLM에 전달하는 코드가 아니다.** [실제 준비 코드](../llm_usage_2026-10-14/prepare_context.py)

### 1. 검색 근거를 준비한 뒤 Qwen3:8b 또는 ChatGPT에 전달

현재 작업 폴더의 PowerShell에서 다음 명령을 사용한다. `질문을 여기에 입력하세요`를 실제 검색 질문으로 바꾼다.

```powershell
python .\llm_usage_2026-10-14\prepare_context.py --query "질문을 여기에 입력하세요" --limit 3
```

생성되는 파일은 `llm_usage_2026-10-14/paste_into_llm.txt`, `llm_context.json`, `native_retrieval_example.json`, `context_verification.json`이다. 이 명령은 해당 예시 파일을 새 질문의 결과로 갱신한다. 질문마다 다시 검색해서 해당 질문의 원문을 전달한다.

이번에는 기존 예시 파일을 갱신하지 않고, 같은 코드의 바이트 단위 동일 사본을 이 확인 폴더에서 실행했다.

```powershell
python .\implementation_basis_check_2026-10-14\prepare_context.py --query "제안요청서 작성 예시에서 요구사항 작성 관련 내용을 찾아 주세요." --limit 3
```

실제 결과는 원문 청크 3개, 문서 2개의 해시·문자 범위 검사 통과, 텍스트 2,625자였다. 반환 자료는 D7 가이드와 D5 기준이며, `TEMPLATE-RFP`가 이번 상위 3개 결과에 포함된 것은 아니다. 따라서 이 실행은 **검색 호출과 출처 연결의 성공 증거**이며, 질문에 대한 검색 정확도나 작성 예시의 정답을 검증한 결과가 아니다. 세 결과의 `graph_context`도 비어 있어 이번 결과만으로 그래프 관계가 LLM 답변에 사용됐다고 주장하지 않는다. [실제 검색 응답](native_retrieval_example.json), [출처 검사](context_verification.json), [코드 사본 일치 확인](basis_inspection.json)

바로 사용할 수 있는 이번 입력: [paste_into_llm.txt](paste_into_llm.txt), [llm_context.json](llm_context.json).

Qwen3:8b를 사용하는 대화에서는 실제 선택한 모델이 맞는지 확인하고 `paste_into_llm.txt` 전체를 붙여 넣는다. ChatGPT에서는 같은 텍스트를 붙여 넣거나 파일로 첨부할 수 있다. 파일 첨부 경로는 OpenAI 공식 문서에 설명돼 있다. 첨부만으로 LocalCrab 검색기가 ChatGPT에 등록되는 것은 아니므로, 새 질문에는 새로 검색한 입력을 제공한다. [공식 OpenAI 파일 사용 안내](https://learn.chatgpt.com/docs/use-chatgpt)

입력에 이미 들어 있는 지침을 적용한다: 원문·자료 목록에 근거해 한국어로 답하고, 사실 문장에 chunk_id와 정확한 인용·원문 위치를 붙이며, 작성 예시를 확정 사업 조건으로 바꾸지 않고, 현행성·적용을 추정하지 않는다. 근거가 부족하면 제공 자료로 확인 불가라고 답한다. 이 지침은 의미상 정확성을 자동 보증하지 않는다. [실제 system 메시지](llm_context.json)

### 2. Qwen3:8b API로 검색과 생성 연결

기존 `query_qwen3_8b.py`는 질문마다 LocalCrab 검색과 원문 검사를 실행하고, 모델 목록에서 정확한 ID `qwen3:8b`를 확인한 뒤 messages를 채팅 API로 전달하도록 작성돼 있다. 현재 서버에서 그 모델의 설치·API 응답 성공을 이번 확인 작업에서 검증하지 않았다. [실제 연결 코드](../llm_usage_2026-10-14/query_qwen3_8b.py)

사용자가 지정한 Open WebUI `http://192.168.113.143:8080/`에 대해 인증과 모델 ID가 준비된 경우:

```powershell
python .\llm_usage_2026-10-14\query_qwen3_8b.py --provider openwebui --query "질문을 여기에 입력하세요"
```

이 코드에는 로컬 환경변수 `OPENWEBUI_API_KEY`가 필요하다. `/api/models`에서 모델 ID를 확인하고 `/api/chat/completions`를 호출한다. 키 값은 채팅에 보내지 않는다. 별칭 모델 ID만 반환되는 서버에서는 현재 코드가 중단하므로 실제 ID를 확인해야 한다. [공식 Open WebUI API](https://docs.openwebui.com/reference/api-endpoints/)

동일 PC의 Ollama가 `localhost:11434`에서 실행되고 `qwen3:8b`를 제공하는 경우:

```powershell
python .\llm_usage_2026-10-14\query_qwen3_8b.py --provider ollama --query "질문을 여기에 입력하세요"
```

이 경로는 `/api/tags`와 `/api/chat`을 사용한다. 해당 Ollama 서버를 이번에 검사한 것은 아니다. [공식 모델 목록 API](https://docs.ollama.com/api/tags), [공식 채팅 API](https://docs.ollama.com/api/chat)

기존 코드가 향후 생성 요청에 성공하면 `qwen_response_candidate.json`을 만든다. 코드상 의미 지지 검증은 false, 답변 정확도는 null로 남긴다. 응답을 받는 것과 원문이 답변의 주장을 지지하는 것은 별도로 확인한다. 이번에는 모델 HTTP 호출이나 답변 생성은 수행하지 않았다.

### 3. 대화 도구로 자동 검색하는 연결 상태

현재 팩의 `localcrab.mcp.json`에는 `opencrab.exe serve`와 이 팩의 독립 `LOCAL_DATA_DIR`를 지정한 설정이 있다. 사용자 ChatGPT나 Qwen3:8b 대화에 이 서버가 실제 등록되고 호출됐다는 확인 결과는 이번에 만들지 않았다. 자동 도구 연결 절차는 기존 안내에 있고, 연결 성공을 주장하려면 실제 도구 호출과 반환 원문을 확인해야 한다. [현재 팩 MCP 설정](../rag_ontology_2026-10-14/localcrab.mcp.json), [기존 연결 안내](../llm_usage_2026-10-14/LLM_USAGE_KO.md), [기존 두 모델 사용법](../llm_usage_2026-10-14/QWEN3_8B_AND_CHATGPT_KO.md)

현재 설정이 지정하는 로컬 데이터 경로는 `C:\Users\Public\Documents\ESTsoft\CreatorTemp\procurement_rag_20261014_e89567c5a3`다. `localcrab_export/`는 이식용 데이터 사본이다. 현재 팩 대신 기본 전역 저장소를 읽는 것으로 임의 변경하지 않는다. [실제 실행 위치 기록](../rag_ontology_2026-10-14/localcrab_runtime.json)

## 이번에 확인한 것과 미확인인 것

확인: 공식 문서의 Aside 브라우저 열람, TypeSafe 실제 분류, 설치된 저장소·검색 코드, 팩 코드·설정 검사, 그래프/벡터 레코드 수, 주요 그래프·청크 해시 일치, 현재 LocalCrab 검색, 반환된 청크의 원본·추출문 해시와 문자 범위, LLM 전달 입력 생성.

미확인: Palantir Foundry 배포, Neo4j 기반으로의 구현, 실제 Qwen3:8b 모델 응답, ChatGPT MCP 신규 연결, 이번 질문에 대한 검색 정확도, LLM 답변 정확도·의미상 인용 지지, 법령 현행성·사업 적용·확정 사업 RFP. 확인되지 않은 항목을 구현 완료 상태로 표시하지 않는다.
