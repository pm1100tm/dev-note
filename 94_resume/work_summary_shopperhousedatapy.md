# ShopperHouseDataPy 프로젝트 작업 내역

## 1. 프로젝트 초기 설정 (2025.09)

### 프로젝트 초기화

- First commit
- README.md 작성 및 업데이트
- .gitignore 추가

### Git Actions

- git action 추가
- deploy.yml 수정

## 2. 데이터 처리 CLI 개발 (2025.09 ~ 2025.10)

### 데이터 전처리 스크립트

- run_data_qvc_review 분석 및 코드 정리
- run_data_processing 분석 및 코드 정리
- cli 코드 주석 추가

### 카테고리 및 속성 데이터

- run_category_attributes 분석 및 코드 정리
- 리뷰 속성 데이터 업데이트 CLI 추가
- 상품 옵션 테이블명 변경

### 리뷰 분석

- run_review_analyze 분석 및 코드 정리
- run_review_summary 분석 및 코드 정리

### 임베딩 생성

- run_embedding_create 분석 및 코드 정리
- run_embedding_qdrant 분석 및 코드 정리

### 제품 데이터 관리

- product 20251027 insert 코드 추가
- 불필요한 파일 제거 (products.jsonl)

## 3. Warehouse 이미지 관리 (2025.09)

### Warehouse 업로드 로직

- warehouse 전체 로직 temp 코드 작성
- warehouse 전체 로직 코드 작성

### Warehouse 어셋 관리

- warehouse 어셋 카운트 & 삭제 로직 디벨롭
- warehouse 코드 수정
- warehouse 코드 실행 방지 추가
- warehouse undeploy 코드 수정

### README 문서화

- warehouse 관련 README.md 수정

## 4. 이미지 임베딩 처리 (2025.11)

### 영상 프레임 이미지 변환

- 영상 프레임 이미지 변환 스케치
- 영상 프레임 단위 이미지 추출 코드
- 영상 프레임 단위 이미지 추출 및 테이블 저장
- opencv 관련 라이브러리 설치

### 이미지 임베딩 모델

- sentence-transformers 라이브러리 설치
- 이미지 데이터 임베딩 스케치
- 이미지 임베딩 처리 추가
- AdVideoFrame 모델 추가

### 임베딩 모델 교체 및 최적화

- 이미지 임베딩 모델 교체 샘플 코드 추가
- GoogleSigLIP 추가
- GoogleSigLIP 추가에 따른 의존성 추가
- 임베딩 모델 클래스화

### 이미지 처리 코드 정리

- image 임베딩 코드 정리
- 이미지 스플릿, 이미지 임베딩 코드 분리

## 5. Qdrant 벡터 DB 연동 (2025.11)

### Qdrant Repository 구현

- qdrant_repo.py 구현
- qdrant.py 수정
- 이미지/텍스트 Qdrant 검색 스케치
- 이미지/텍스트 벡터 데이터 Qdrant 저장하도록 수정

### Qdrant 검색

- 이미지 by 이미지 검색 테스트

## 6. GCS 및 VertexAI 연동 (2025.11)

### GCS (Google Cloud Storage) 연동

- GCS 관리 클래스 추가
- get_bytes_from_gcs_uri 메서드 추가

### VertexAI 멀티모달 임베딩

- Vertex AI 멀티 모달 임베딩 가져오기 구현
- 이미지 + 메타데이터 임베딩 스케치

### 이미지 메타데이터

- 이미지 메타 데이터 취득 처리 추가
- AdVideoFrameQueryResult 스키마 수정

### Gemini 프롬프트 처리

- Gemini 호출 관련 클래스 로직 추가
- 프롬프트 관련 파일 추가
- PromptRequest 스키마 추가
- \_run_multimodal_prompt 메서드 수정

## 7. 이미지 분석 및 저장 (2025.11)

### 이미지 분석 결과 저장

- 이미지 분석 결과 저장 로직 구현
- analysis_result 필드 추가
- 페이로드 추가

### 광고 영상 이미지 추출

- 광고 영상 이미지 추출 로직 수정
- AdFrame 테이블 관련 DB 로직 분리
- BaseCRUD 생성

### 텍스트 임베딩 처리

- 텍스트 임베딩 시 32단어 제약 대응
- 커스텀 에러 클래스 추가

## 8. Docker 환경 및 설정 (2025.09 ~ 2025.11)

### Docker 설정

- DataPy GCE용 docker 파일 수정
- 도커 이미지 경량화 작업
- 도커파일 수정
- 도커 네트워크 수정 (개발환경에 맞춰서)

### Settings 관리

- Settings lru cache로 추가
- settings.py 수정
- requirements.txt 업데이트

### .gitignore 관리

- .gitignore 업데이트
- .iml 파일 삭제

## 9. 테스트 및 문서화 (2025.11)

### 테스트 설정

- 임시 테스트 코드 실행 위한 설정 코드 추가

### README 문서화

- README.md 수정 및 업데이트
- 프로젝트 설명 및 실행 방법 문서화

---

# ShopperHouseDataPy 프로젝트 요약

## 담당 역할

데이터 처리 및 임베딩 파이프라인 개발

## 핵심 기능

1. **데이터 전처리 CLI**
   - 상품 데이터 처리 (QVC 리뷰 데이터)
   - 카테고리 및 속성 데이터 관리
   - 리뷰 분석 및 요약

2. **이미지 임베딩 파이프라인**
   - 영상 프레임 추출 및 변환
   - 이미지 임베딩 생성 (sentence-transformers, GoogleSigLIP)
   - 이미지 스플릿 및 처리

3. **벡터 DB 관리**
   - Qdrant 벡터 DB 연동
   - 임베딩 데이터 Qdrant 저장
   - 이미지/텍스트 검색 기능

4. **VertexAI 멀티모달 임베딩**
   - GCS 연동 및 이미지 메타데이터 처리
   - Gemini 프롬프트 처리
   - 이미지 분석 결과 저장

5. **Warehouse 이미지 관리**
   - Warehouse 업로드/다운로드 로직
   - 어셋 카운트 및 삭제 관리

## 주요 기술 스택

- **Backend**: Python
- **Database**: PostgreSQL
- **Vector DB**: Qdrant
- **AI/ML**: sentence-transformers, GoogleSigLIP, VertexAI, Google Gemini, OpenCV
- **Storage**: Google Cloud Storage
- **DevOps**: Docker

## 특이사항

- 데이터 처리 전용 프로젝트로 CLI 기반 작업
- 이미지 임베딩 모델 교체 및 최적화 수행
- VertexAI 멀티모달 임베딩을 활용한 고급 이미지 분석
- 도커 이미지 경량화 작업 수행
