# 공공조달 온톨로지 RAG 구축·검증 결과

2026-10-14 · 보관본 근거 검색 후보 팩. 현재 법적 적용이나 특정 사업 사실을 승인한 결과가 아닙니다.

## 측정된 결과

- 원본 파일: **140개**, 기존 SHA-256와 모두 일치. 파일 변경·삭제 없음.
- 활성 문서: **23개**, 보류 문서 **5개**.
- 원문 청크: **2,259개**(내용이 있는 단위 2,248, 빈 PDF 레이아웃 단위 11).
- 그래프 노드: **3,341개** — Document 23, TextUnit 2,259, Concept 476, 원문 그대로의 Claim 583.
- 관계: **4,120개** — contains 2,259, mentions 1,278, supports 583.
- 끊어진 관계 **0**, 고아 노드 **0**, SHACL 적합 **통과**. 잘못된 해시 누락·깨진 문서 참조·역전 위치의 부정 사례 **3개 모두 거부**.
- 추출문 문자 범위 전체 재조립 **23/23 통과**. 원본 HTML 텍스트 대조 **1,601단위**, PDF 네이티브 텍스트 대조 **630단위** 통과. HWP 활성 추출문 **28단위**의 원본 페이지/시각 구조 연결은 미확인.
- 실제 로컬크랩 재조회: TextUnit 전체 원문 **2,259/2,259**, 그래프 원문 **2,259/2,259**, 관계 근거 참조 **4,120/4,120** 일치.
- 네이티브 벡터 레코드 **9,376개**. 토큰 제한을 넘는 원문 1173개에 추가 창 7128개를 적용했고 모든 긴 원문의 문자 범위를 덮었습니다. 원문 청크 자체는 자르거나 변경하지 않았습니다.

## 요청한 단계의 처리

| 단계 | 수행한 내용 | 상태/남은 제약 |
| --- | --- | --- |
| 4-1 실제 질문 | 사용자 QA-01~QA-10을 그대로 저장하고 개념·관계·필요 자료를 대응 | 실제 사업 RFP/요구사항 ID 없음. 작성 예시를 사업 원문으로 취급하지 않음 |
| 4-2 원문 연결 | 문서 ID·원본/추출문 해시·버전 표현·URL·조문/절·PDF 페이지·문자/줄 위치 보존 | HWP는 본문·표 순서 추출, 원본 페이지 연결 미확인 |
| 4-3 제약 추출 | 로컬크랩 manifest 타입·관계만 사용. 원문 인용과 명시 용어 추출. 확장 후보를 별도 보관 | 583 인용 Claim은 정형 업무 규칙/법적 공리가 아님. 적용·위계·교차 요구사항 관계를 추정하지 않음 |
| 4-4 구조/지지 분리 | JSONL 참조·문자 재조립·해시·인용·용어 위치·SHACL 실행. 부정 사례 검사 | 의미상 지지·표 셀 해석·도메인 공리 정확도는 독립 검토 필요 |
| 4-5 검색 비교 | 동일 질문으로 오프라인 3개 모드와 네이티브 LocalCrab BM25/하이브리드 실행 | 공식 Global Search/OG-RAG와의 입력·알고리즘 차이를 기록. 실제 구현/성능 재현으로 주장하지 않음 |
| 4-6 평가 | QA별 보류/참고/정책 결과·근거·시간·컨텍스트 양, 네이티브 검색 결과 기록 | 독립 정답·실제 생성 모델 측정은 미완료. null 지표를 임의 수치로 채우지 않음 |

## QA 결과의 해석

QA-01~QA-07·QA-09는 확정 사업 RFP가 없어 보류합니다. QA-08은 D1 보관본 제7조·제8조의 일반 규정 원문만 참고로 제공합니다. QA-10은 근거 없는 기술/솔루션을 필수 도입 사실로 확정하지 않는 사용자 원칙을 적용합니다.

오프라인 3개 모드 × 10개 질문의 범위·출처 검사는 **30/30** 통과했습니다. 이 값은 LLM 답변 정확도가 아닙니다. 개념/관계/공리 재현율, 수치/요구사항 정답률, 의미상 인용 지지율은 독립 정답이 없어 **미측정**입니다. 검색 시간은 각 JSON의 실제 측정값을 사용하며 품질 우열을 단정하지 않습니다.

사용자 지정 서버 `http://192.168.113.143:8080/`는 Open WebUI이며 `/api/models`가 인증 없이 HTTP 401를 반환했습니다. 모델명/인증이 확인되지 않아 생성 답변 정확도·의미상 인용 지지·모델 시간/토큰은 미측정입니다.

## 원문 보존과 복구 사항

원본 140개는 `source_snapshot/`에 복사했고, 기존 보류/중복/오류/수집 보조 자료의 분류도 유지했습니다. 후보만 검색에 사용하므로 ‘원본 전체를 의미 분석/색인했다’고 주장하지 않습니다. OCR 필요, 추출 오류, 과거판/출처 미확인 5개 문서는 `quality/held_documents.json`에서 확인합니다.

RFP HWP 원본은 제목·안내문에 작성 예시임을 명시하며 사업명·예산·기간 빈칸과 예시 표를 포함합니다. 기존 추출문의 문서 구조 반복을 그대로 보존하고, 활성 검색에는 HWP MCP가 만든 본문·표 흐름 추출문을 사용했습니다. 해당 표/페이지 매핑의 시각 검수 완료를 주장하지 않습니다.

로컬크랩의 `LocalDocStore.upsert_source`가 원문을 4,096자로 제한하는 것을 코드에서 확인했습니다. 긴 원문 **7개**를 팩 내부에서 전체 내용으로 복구하고 모든 Evidence를 재조회했습니다. 최초 관계 응답 검사에서 정상 `audited` 상태를 오류로 분류했던 검사를 수정했고, 실제 관계/근거 전체를 재확인했습니다. 최초 기록은 감사 파일에 남겼습니다.

Windows HNSW의 한글 경로에서 바이너리 인덱스 파일이 생성되지 않고 다른 프로세스 재열기가 실패하는 것을 작은 비문서 시험 데이터로 재현했습니다. ASCII 경로 시험은 통과했습니다. 활성 벡터 인덱스는 허용된 ASCII 작업 경로에 구축했고 `localcrab_export/`로 내보냈습니다. ZIP 이동 후 `scripts/activate_localcrab.py`로 ASCII 경로에 활성화합니다. 기존 전역 로컬크랩 데이터베이스는 변경하지 않았습니다.

기본 임베딩 모델 all-MiniLM-L6-v2의 한국어 도메인 정확도는 검증하지 않았습니다. 모델 점수는 사실 지지/법적 적용 승인 값이 아닙니다. 긴 원문의 끝부분이 검색에서 빠지지 않도록 토큰 제한 안의 추가 벡터 창과 원문 오프셋을 별도 기록했습니다.

## 공식 근거와 구현 차이

- [Palantir 온톨로지 설계 검증](https://www.palantir.com/docs/foundry/ontology/ontology-design-validation): 실제 업무 질문 기반 검증을 참고했습니다. 질문을 사용자에게 받아 사용했고, 설계자가 만든 후보를 실제 업무 질문으로 표시하지 않았습니다.
- [GraphRAG 기본 데이터 흐름](https://microsoft.github.io/graphrag/index/default_dataflow/): TextUnit/문서 연결을 참고했지만 조문·페이지·버전 연결은 이 팩의 별도 전처리/검사로 보존합니다.
- [Wikontic](https://aclanthology.org/2026.eacl-long.388/): 타입·관계 제약과 개체 정규화 원칙을 참고했습니다. Wikidata 정렬 파이프라인이나 논문 성능을 재현했다고 주장하지 않습니다.
- [W3C SHACL](https://www.w3.org/TR/shacl/): 구조 제약 검사에 사용했습니다. 형식 적합만으로 원문 의미/법적 적용을 검증했다고 판단하지 않습니다.
- [GraphRAG Local Search](https://microsoft.github.io/graphrag/query/local_search/) / [Global Search](https://microsoft.github.io/graphrag/query/global_search/): Local은 개체/연결/원문, Global은 커뮤니티 보고서 기반입니다. 이 팩의 명시 용어 그래프와 문서 목록은 각각 완전한 공식 구현과 다릅니다.
- [OG-RAG](https://aclanthology.org/2025.emnlp-main.1674/): 온톨로지에 근거한 사실 묶음의 하이퍼그래프 검색 방법입니다. 이 팩의 정확한 인용 묶음은 원 알고리즘의 구현이 아닙니다.
- [CQ4OE, 2026-09-22 arXiv v1](https://arxiv.org/html/2609.26029): 용어와 계층·속성·공리 평가를 구분한 연구를 참고했습니다. 해당 논문 존재/날짜를 확인했으며 이 팩의 추출 재현율을 그 논문 수치로 대신하지 않았습니다.
- [TypeSafe Choice](https://docs.typesafe.ai/primitives/choice): 승인된 작업 설명·개수·상태를 실제 API로 분류했습니다. 결과는 구축 작업의 분류이며 사실 검증/운영 승인이 아닙니다. 사용량 입력 971, 출력 184 토큰; 금액은 미측정입니다. QA 수신 뒤 추가 상태 재전송은 자동 검토가 거절하여 하지 않았고 후속 상태는 로컬에만 기록했습니다.
- [Open WebUI API](https://docs.openwebui.com/reference/api-endpoints/): `/api/models`, `/api/chat/completions`의 인증 필요 API를 확인했습니다. 지정 서버의 모델은 인증 전에는 확인할 수 없었습니다.

Aside MCP의 공식 문서 대조 결과도 `methods/aside_verification.md`에 보존했습니다. 연구 근거는 제작 방법 기록이며 공공조달 사실 청크에 혼합하지 않았습니다.

## 활성 문서별 청크

| 문서 ID | 원문 역할 | 청크 수 |
| --- | --- | ---: |
| D1 | normative_body | 54 |
| D2 | normative_body | 94 |
| D3 | normative_body | 129 |
| D4 | normative_body | 53 |
| D5 | normative_body | 486 |
| D6 | normative_body | 29 |
| D7 | reference_guide | 133 |
| D9 | normative_body | 127 |
| D10 | normative_body | 88 |
| D11 | normative_body | 530 |
| D12 | normative_body | 497 |
| ATT-national_negotiation_160777029 | normative_attachment | 1 |
| ATT-pps_evaluation_161222773 | normative_attachment | 1 |
| ATT-pps_evaluation_161222851 | normative_attachment | 1 |
| ATT-pps_evaluation_161222859 | normative_attachment | 1 |
| ATT-pps_evaluation_161222899 | normative_attachment | 1 |
| ATT-pps_evaluation_161223081 | normative_attachment | 1 |
| ATT-pps_evaluation_161223087 | normative_attachment | 1 |
| ATT-sw_evaluation_151563131 | normative_attachment | 1 |
| ATT-sw_evaluation_151563861 | normative_attachment | 1 |
| ATT-sw_management_128124887 | normative_attachment | 1 |
| ATT-sw_management_128124933 | normative_attachment | 1 |
| TEMPLATE-RFP | rfp_example | 28 |

## 제공 상태

JSONL·RDF/SHACL·SQLite·로컬크랩 내보내기·질문/평가 기록·재현 스크립트를 제공합니다. 원문 보존/구조/적재 검증을 통과했으나 **promotion과 운영 승인은 수행하지 않았습니다**. 확정 RFP, 기관/적용 시점, 원문 표·페이지 검수, 독립 정답과 모델 평가가 완료되기 전에는 이 후보 팩을 현행 법률 또는 특정 사업의 확정 답변으로 사용하지 않습니다.
