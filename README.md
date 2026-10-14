# 공공조달 원문 근거 온톨로지 RAG 후보 팩

2026-10-14 · 현재 폴더의 자료로 생성했습니다. **로컬 검색과 온톨로지 구조·출처 검증은 완료했으며, 운영 승인·현행 법령 판단·특정 사업 답변 정확도는 미확정입니다.**

원본 140개 파일은 변경 없이 `source_snapshot/`에 보존했습니다. 색인 후보 23개 문서를 처리하고 기존 보류 5개 문서는 사유와 함께 분리했습니다. RFP는 ‘공공SW사업 제안요청서 작성 예시’이며 확정된 사업 문서가 아닙니다.

## 바로 실행

이 폴더에서 다음 명령을 실행합니다. 검색은 Python 표준 라이브러리만 사용하며 네트워크가 필요하지 않습니다.

```powershell
python .\rag.py qa --id QA-01
python .\rag.py qa --id QA-08
python .\rag.py search --query "제안요청서" --source D1 --article "제5조"
python .\rag.py context --query "협상적격자" --source D1 --article "제8조" --budget 18000
python .\rag.py evaluate
```

`context`의 JSON을 로컬 LLM에 전달할 수 있습니다. 원문·출처·문서 버전 해시·조문/절·PDF 페이지·추출문 문자 위치가 함께 반환됩니다. 예산을 초과하는 청크는 잘라 넣지 않고 제외 사유를 기록합니다. 법령 조문 번호가 부칙에서 반복되므로 `source_id`, 버전 해시와 `section_context`를 함께 확인합니다.

## 산출물

| 파일/폴더 | 용도 |
| --- | --- |
| `graph/nodes.jsonl`, `graph/edges.jsonl` | 로컬크랩 manifest에 맞춘 타입·관계와 근거 참조 |
| `ontology/graph.ttl`, `ontology/shapes.ttl` | RDF 그래프와 실제 실행한 SHACL 제약 |
| `evidence/chunks.jsonl`, `evidence/index.jsonl` | 원문 전체 및 출처·위치 추적 |
| `sources.json`, `source_snapshot/` | 23개 활성 문서의 버전 기록과 원본 140개 파일 |
| `raw/`, `parsed/`, `legacy_parsed/` | 원본·활성 추출문·기존 RFP 추출문 보존 |
| `rag.sqlite3`, `rag.py` | 오프라인 BM25·한국어 부분어·명시 용어 그래프 검색 |
| `localcrab_export/` | 검증된 로컬크랩 SQLite 그래프·문서·Chroma 인덱스의 이식용 내보내기 |
| `localcrab.mcp.json` | 이 팩을 별도 로컬크랩 서버로 여는 설정; 경로 이동 시 재생성 필요 |
| `golden_questions.json`, `ontology/cq_mapping.json` | 사용자 QA-01~QA-10과 구현/자료 부족의 대응표 |
| `quality/report.json`, `quality/shacl_report.txt` | 구조와 출처/지지 검사를 구분한 결과 |
| `quality/localcrab_import.json`, `quality/vector_coverage.json` | 실제 적재·전체 원문 재조회·임베딩 창 범위 검증 |
| `quality/qa_evaluation.json`, `quality/localcrab_query_benchmark.json` | 동일 10개 질문의 검색·보류·출처·시간 기록 |
| `REPORT_KO.md`, `methods/aside_verification.md` | 단계별 결과, 공식 근거와 방법 차이 |
| `typesafe_request.json`, `typesafe_response.json`, `qa_state.json` | 승인된 TypeSafe 분류 원기록과 이후 로컬 QA 상태 |

## 그래프의 사실 범위

`Document → contains → TextUnit`, `TextUnit → mentions → Concept`, `TextUnit → supports → Claim`만 사용했습니다. 용어는 원문에 명시된 법령 제목/정의 표현에서 가져왔습니다. Claim 583개는 **조문 그대로의 인용**이며 새롭게 해석한 업무 규칙이나 승인된 법적 공리가 아닙니다. 동일 개체 판단은 NFC와 바깥 공백 정리만 적용했고, 이름이 비슷하다는 이유로 병합하지 않았습니다.

작성 예시의 사업금액·날짜·장비 수량·요구사항 ID는 사업 사실로 승인하지 않았습니다. 법령 간 위계, 업무별 적용, 특정 요구사항 간 보안·성능·검수 관계도 근거 없이 추가하지 않았습니다. 확장 후보는 `ontology/expansion_candidates.json`에 근거 위치와 함께 별도로 기록했습니다.

## 검증 범위와 미확인 항목

최종 통계와 검증 결과는 `REPORT_KO.md`와 `quality/`에서 확인합니다. 추출문은 원문 전체를 재조립해 비교했고, 그래프·청크·인용·용어 위치를 검사했습니다. HTML/PDF는 원본의 네이티브 텍스트와 공백만 정규화해 대조했습니다. RFP HWP는 본문·표 순서를 네이티브 변환했지만 정확한 원본 페이지 연결과 시각적 표 의미 검수는 완료되지 않았습니다.

QA-01~QA-07·QA-09는 확정 사업 RFP가 없어 보류합니다. QA-08은 적용을 확정하지 않은 D1 보관본 일반 규정 원문만 제공합니다. QA-10은 사용자 지침에 따른 근거 없는 필수 사항 확정 금지를 적용합니다. 검사 통과율은 이 범위·출처 검사 결과이며 LLM 답변 정확도나 추출 재현율로 해석할 수 없습니다.

로컬크랩 기본 임베딩 모델의 한국어 도메인 정확도는 측정하지 않았습니다. 긴 원문은 잘리는 임베딩 토큰 범위를 추가 창으로 보완하고 각 창을 원래 청크에 연결했습니다. 전체 원문은 그래프·Evidence·SQLite에 보존합니다. 검색 점수·TypeSafe confidence는 사실의 진실 확률이나 운영 승인 수치가 아닙니다.

## 로컬크랩 실행과 재현

기존 전역 로컬크랩 데이터베이스는 변경하지 않았습니다. Windows의 한글 경로에서 HNSW 인덱스 파일 생성/재열기 오류를 재현하여, 활성 네이티브 인덱스는 ASCII 작업 경로에 두었습니다. `localcrab.mcp.json`은 이 별도 서버를 지정합니다. ZIP을 이동하거나 새 환경에서 사용할 때 `python .\scripts\activate_localcrab.py`를 먼저 실행합니다. 이 명령은 `localcrab_export`를 전용 ASCII 경로에 복사하며 전역 데이터베이스를 변경하지 않습니다. 설치된 로컬크랩 Python 환경에서 다음 명령으로 조회할 수 있습니다.

```powershell
& "$env:USERPROFILE\plugins\localcrab\.venv\Scripts\python.exe" .\scripts\query_localcrab.py --query "제안요청서" --method hybrid --limit 5
```

구축 재현에는 `requirements-build.txt`의 라이브러리와 설치된 로컬크랩이 필요합니다. 새 빈 폴더에서 실행하고, 문서/질문 검토 기록을 보존하세요. `import_localcrab.py`는 네이티브 도구로 노드를 먼저 생성하고 관계를 추가하며, 네이티브 어댑터로 모든 Evidence를 적재합니다. 로컬 저장 함수의 4,096자 제한은 팩 안에서 전체 원문으로 보완하고 다시 확인했습니다. 도구의 `audited` 응답은 문서 저장소의 정상 관계 감사 상태입니다.

SHACL 재검증은 `python .\scripts\validate_pack.py`, 잘못된 출처 거부 검증은 `python .\scripts\negative_controls.py`로 실행합니다. 제공된 ZIP에는 의존성 코드 `_deps`가 포함되지 않으므로 필요 시 라이브러리를 설치합니다.

## 로컬 LLM 답변 평가

사용자가 지정한 `http://192.168.113.143:8080/`는 Open WebUI이며 모델 목록이 인증 없이 HTTP 401을 반환했습니다. 인증된 모델명과 세션/API 인증이 준비되기 전에는 생성 답변 정확도는 미측정입니다. 다음 스크립트는 루프백 주소와 지정된 사내 주소만 허용합니다. 인증 키가 필요한 경우 `OPENWEBUI_API_KEY` 또는 `LOCAL_LLM_API_KEY` 환경변수로 설정하며 키는 로그/결과에 저장하지 않습니다.

```powershell
python .\scripts\evaluate_local_llm.py --model "사용할-모델명" --endpoint "http://localhost:1234/v1/chat/completions"
# Ollama의 OpenAI 호환 API 예:
python .\scripts\evaluate_local_llm.py --model "사용할-모델명" --endpoint "http://localhost:11434/v1/chat/completions"
# 사용자 지정 Open WebUI (인증 필요):
python .\scripts\evaluate_local_llm.py --model "사용할-모델명" --endpoint "http://192.168.113.143:8080/api/chat/completions"
```

원문 인용의 정확한 위치와 답변 범위는 자동 점검하지만 의미상 지지 여부와 업무 정확도는 독립 검토가 필요합니다. 검토되지 않은 답변은 후보로 저장합니다. 확정 RFP·기관·시점·요구사항 ID가 제공되어야 실제 사업 수치/조건 및 관계의 정답 평가를 진행할 수 있습니다.

Global Search의 커뮤니티 요약이나 OG-RAG 하이퍼그래프를 구현한 결과로 주장하지 않습니다. 세 실행 모드는 `baseline`, 사용자 정의 `local`, 인용 묶음 `fact_bundle`이며 공식 방법과의 차이는 보고서에 기록했습니다. 팩은 검토 후보 상태로 유지했고 promotion을 수행하지 않았습니다.
