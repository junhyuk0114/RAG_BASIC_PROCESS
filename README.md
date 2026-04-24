# RAG_OLLAMA 프로젝트

본 프로젝트는 Ollama와 LangChain을 활용하여 PDF 문서 기반의 질의응답 시스템(RAG, Retrieval-Augmented Generation)을 구축하는 실습 프로젝트입니다. 특히 2026학년도 한국폴리텍대학 서울강서캠퍼스 하이테크과정 모집요강을 예시 데이터로 활용합니다.

## 0. 주제
- **로컬 및 클라우드 LLM을 활용한 PDF 기반 RAG 시스템 구축**
- 주요 타겟: 모집요강 등 특정 도메인 문서에 대한 정확한 정보 검색 및 답변 생성

## 1. 기본 원칙
- **정확성 우선**: 컨텍스트에 없는 내용은 추측하지 않고 "모름"으로 답변하여 할루시네이션(Hallucination) 방지
- **출처 명시**: 답변 생성 시 해당 정보가 포함된 문서의 페이지 번호를 함께 제공
- **유연한 모델 전환**: OpenAI(GPT)와 로컬 모델(Ollama, GGUF) 간의 쉬운 전환 지원
- **한국어 최적화**: 한국어 임베딩 및 프롬프트 템플릿을 통한 고품질 답변 생성

## 2. 사전 준비사항
로컬 모델(Ollama)을 사용하여 한국어 임베딩을 수행하기 위해 다음 명령어를 통해 임베딩 모델을 미리 다운로드해야 합니다.
```bash
ollama pull bge-m3
```

## 3. 핵심 아키텍처
- **Data Load**: `PyPDFLoader`를 통한 PDF 페이지별 텍스트 추출
- **Split**: `RecursiveCharacterTextSplitter`를 사용하여 문맥 유지를 고려한 청크(Chunk) 분할
- **Embedding**: `OpenAIEmbeddings` 또는 `OllamaEmbeddings(bge-m3)` 등을 활용한 벡터화
- **Vector DB**: `Chroma`를 활용한 벡터 데이터 저장 및 유사도 검색(Top-K)
- **Chain**: LangChain Expression Language(LCEL) 기반의 RAG 체인 구성 (Retrieval -> Prompt -> Generation)
- **Interface**: FastAPI를 통한 API 서버 기능 및 대화형 챗봇 모드 지원

## 4. 프로젝트 구조
RAG_OLLAMA
├── codeset/
│   ├── RAG_OLLAMA_수정본.ipynb       # 메인 실습 노트북
│   ├── requirements_fine_rag.txt    # 필요 라이브러리 목록
│   └── dataset/
│       └── 모집요강.pdf               # 분석 대상 PDF 문서
├── vector_db/                       # Chroma DB 영구 저장 폴더
└── models/                          # GGUF 등 로컬 모델 저장 폴더

## 5. 코딩 스타일
- **현대적 Python**: Python 3.12+ 환경 권장
- **LCEL(LangChain Expression Language)**: `|` 연산자를 활용한 직관적인 체인 구성
- **설정 분리**: 하이퍼파라미터(청크 크기, 오버랩, K값 등)를 상단에 배치하여 튜닝 용이성 확보
- **모듈화**: 로딩, 분할, 임베딩, 검색 단계를 명확히 구분하여 구현

## 6. 보안 및 에러 처리
- **환경 변수 관리**: `OPENAI_API_KEY` 등 민감 정보는 환경 변수를 통해 관리
- **예외 처리**: 모델 로딩 실패, API 호출 오류 등을 위한 `try-except` 블록 적용
- **데이터 검증**: FastAPI 구현 시 `Pydantic`을 활용한 입출력 데이터 유효성 검사
- **필터링**: 프롬프트를 통해 외부 지식 활용을 제한하고 주어진 문서(Context) 내에서만 답변하도록 강제
