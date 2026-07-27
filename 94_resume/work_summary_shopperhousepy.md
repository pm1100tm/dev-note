# ShopperHousePy 프로젝트 작업 내역

## 1. 프로젝트 초기 설정 및 인프라 구축 (2025.09 ~ 2025.10 초)

### 프로젝트 초기화

- 프로젝트 업로드 및 기본 설정
- .gitignore 재설정
- README.md 작성 및 업데이트
- Git Actions 추가

### Docker 환경 구축

- Dockerizing (local, dev 환경)
- Docker compose 파일 설정 (스키마 기본 실행 파일 추가)
- 개발환경 compose 파일 수정

### API 기본 설정

- FastAPI 기본 설정
- 헬스체크 API 추가
- Swagger 설정 (service-key 입력 추가)
- 공통 응답 형식(CommonResponse) 적용

## 2. Warehouse 이미지 검색 기능 개발 (2025.09 ~ 2025.10)

### 기본 기능 구현

- Warehouse 검색 API 추가
- Warehouse 이미지 검색 로직 구현 및 수정
- FastAPI multipart request를 위한 의존성 추가

### 검색 로직 개선

- Mock 데이터 기반 이미지 검색 실장
- Qdrant 검색 결과 객체 생성 로직 추가
- 웨어하우스 검색 후 DB 상품 조회 쿼리 수정
- 상품 유사도 가중치 관련 유틸 모듈 생성

### 스코어 가중치 처리

- 상품 유사도 검색 시 태그 값에 가중치를 더 많이 부여하는 옵션 추가
- 유사도 가중치 값을 동적으로 받도록 수정
- 웨어하우스 이미지 검색 로직 mock 데이터 기반 실장

## 3. 상품 검색 및 리뷰 분석 기능 (2025.09 ~ 2025.10)

### 리뷰 분석 API

- 리뷰 분석 API 실장
- 리뷰 API 응답 형식 수정 (startAt 문자열 형식 변경)

### 상품 검색 기능

- 상품 검색 API response 필드 문서에 맞춰 수정
- 상품 리뷰 검색 API response 필드명 수정
- 상품 검색 로직 변경 (partner_name → partner_names)
- 상품 검색 결과에 가격 정보 추가
- 상품 옵션을 리턴 타입에 추가

### 검색 API 분리

- 상품 텍스트 검색 분석용 API 추가 및 리턴 타입 분리
- 상품 이미지 검색 분석용 API 추가 및 리턴 타입 분리
- 일반 상품 검색과 분석용 상품 검색 API의 쿼리스트링 객체 분리

### 가격 필터 및 Tag 검색

- 가격 필터 조건 수정
- Tag 유사도 검색 포함하도록 수정

## 4. LLM(Google Gemini) 연동 및 질문 분석 (2025.10)

### Google Gemini 클래스 구현

- Google Gemini 처리 클래스화
- Gemini 클래스 로직 수정 및 개선
- GoogleGemini 클래스 테스트 코드 작성

### 질문 분석 기능

- 질문 분석(analyze_question) 구현
- LLM 질문 수정 및 프롬프트 개선
- 질의문 임베딩 데이터 변환 시 기본 값 리턴 처리
- 질의문 분석 where 파싱 에러 회피 코드 추가

### 카테고리 분석

- LLM 카테고리 분석 범위 점수 부여 및 narrow down
- LLM 카테고리 값 편집 임시 코드 추가
- LLM category 분석 시 category4 데이터 활용
- 카테고리 값 편집 처리 스키마 내부로 이동

### 유사 질문 생성

- 유사 질문 생성 및 저장 로직 구현
- Gemini 클래스에 유사 질문 프롬프트 실행 메서드 추가
- 유사 질문 생성 및 질문/유사 질문 저장 로직 수정
- 유사 질문 프롬프트 rag file 명 명확하게 변경

### 옵션 분석

- 옵션 분석 로직 수정 및 구현
- analyze_options 메서드 구현
- run_query_related_search 구현 추가
- 옵션 분석 값 analyze_info에 설정

## 5. 검색 이력 관리 및 UI 개발 (2025.10)

### 검색 이력 관리

- SearchInfo 테이블 생성 및 관리
- 질문 이력 관련 테이블 생성일 값 설정 수정
- 검색 이력 데이터 취득 메서드 명확하게 수정
- SearchInfoHistory 응답 객체 추가
- 검색 요청 값, 검색 결과 수 컬럼 추가

### 성능 측정 및 로깅

- time_to_db 데코레이터 구현 및 적용
- 로그 컨텍스트 데코레이터 방식으로 변경
- 산출물 생성 위한 curl.py, html_util.py 추가

### 프론트엔드 개발

- index.html 기본 UI 구현
- 검색 결과 화면 출력 구현
- 상품 검색 결과 화면 디벨롭
- 검색 분석 결과 화면 표시 수정
- textarea에 질의문 입력 후 엔터 동작 코드 추가
- 버튼 중복 클릭 방지 코드 추가
- 에러 메시지 표시 처리
- 검색 결과에서 options 표시 부분 스크롤 추가
- 가중치 기본 값 설정 및 UI 적용

### 리팩토링

- 코드 리팩토링 (2025.10.23)
- 공통 코드 정리 및 모듈화
- 주석 추가 및 코드 클린
- output 폴더 .gitignore에 추가

## 6. 광고 영상 분석 기능 (SHP-2차 POC) (2025.11 ~ 2025.12)

### 영상 프레임 추출 및 저장

- AdVideoFrame 모델 추가
- 영상 프레임 분할 처리 구현
- 영상 프레임 분할 테스트 코드 추가
- opencv 라이브러리 추가
- 영상 프레임 리사이즈 처리

### 영상 스트리밍 처리

- 광고 영상 스트리밍 처리 임시 API 추가
- .m3u8 영상 샘플링 엔드포인트 정의
- video url로 영상 데이터 취득 시 order by 추가

### 이미지 분석 (VertexAI/Gemini 연동)

- VertexAI 임베딩 처리 클래스 추가 및 테스트 코드 실장
- 이미지 분석 프롬프트 처리 추가 및 테스트 코드
- Gemini 호출 관련 클래스 로직 추가
- 이미지 분석 결과 임베딩 처리 수정
- 사용자 질의 분석 프롬프트 변경

### 이미지 임베딩 및 Qdrant 연동

- 이미지 임베딩 값 테이블 저장 처리
- 텍스트로 유사 광고 이미지 추출
- 이미지 임베딩 시 storage 주소 값 사용
- AdVideoFrame 객체 생성 및 Qdrant Collection 설정
- Qdrant 관련 헬퍼 코드 추가

### 영상 프레임 분석 흐름

- 영상 프레임 분석 흐름 샘플 코드 추가
- AdVideoComponent 클래스 추가
- AdVideoFrameCRUD 수정
- 이미지 스플릿 및 암복호화 및 저장 경로 수정
- 파일 암호화 처리

## 7. 이미지 클러스터링 및 샘플링 (2025.11)

### 이미지 클러스터링 구현

- 이미지 클러스터링 모듈 스케치
- ImageClusteringCLIP 테스트 코드 실장
- huggingface 모델 다운로드 경로 지정 및 환경변수 설정
- 이미지 클러스터 처리 주석 및 의존성 정보 추가
- pytest.ini 수정

### 영상 샘플링 처리

- 영상 샘플링 목록 취득 API 구현
- 영상 샘플링 목록 취득 리턴 타입 정의
- 영상 샘플링 목록 취득 쿼리 정의
- 영상 샘플링 클러스터 목록 추출 API 스케치
- 클러스터링된 이미지 목록 추출하도록 쿼리 수정

### 샘플링 이미지 스토리지 처리

- 샘플링 이미지 스토리지 적재 로직 수정
- GCS (Google Cloud Storage) 관리 클래스 추가
- storage_upload_path 값 설정

### 클러스터 처리 로직 수정

- 영상 샘플링 이미지 클러스터링/분석 로직 수정
- 클러스터 처리 호출 및 적용
- 임베딩 대상 취득 시 클러스터된 데이터만 취득
- 이미지 스플릿 처리 수정
- ad_video_component_v1.py 추가

## 8. 영상 기반 검색 기능 (2025.11 ~ 2025.12)

### 영상 기반 검색 로직

- 영상 기반 검색 스케치 및 구현
- 텍스트/영상 기반 분기 처리 로직 추가
- 텍스트/영상 검색 판단 값 출력
- search_from_ad_video 값 사용하도록 변경

### 검색 결과 처리

- 영상 기반 검색 0건 처리
- 영상 기반 검색 시 상품 데이터 중복 제거 처리
- 질문과 추출 이미지 유사성 비교
- 실제 추출된 이미지로 검색하도록 수정

### Signed URL 처리

- GCS generate_signed_url 메서드 추가
- signed_url 화면 적용
- 영상 기반 검색 리턴 시 signed_url 적용
- signed url 유효시간 값 10분으로 지정

### 검색 결과 리턴 필드 추가

- 추출된 이미지 리턴 값에 포함되도록 수정
- frame_objects 리턴 필드 추가
- 영상 기반 검색 추출 이미지 표시되도록 수정
- ProductSearchAnalysisResponse 필드 추가
- QdrantAdVideoFramePayload 필드 추가

### 영상 재생 시간 처리

- 영상 재생 시간 값 기반 검색 기준 계산 수정
- 영상 플레이 시간 계산 로직 별도 분리
- ad_video_current_time_sec 기본 값 지정

## 9. Celery 비동기 처리 및 Redis 캐시 (2025.11 ~ 2025.12)

### Celery 설정

- 비동기 큐 관련 패키지 설치
- celery 모니터링 패키지 설치 (flower)
- celery 초기화 파일 위치 이동
- celery 컨테이너 환경 변수 주입
- Celery Task 실행 기록 저장 관련 설정 추가

### Redis 캐시 구현

- Redis 설정 코드 스케치
- FastAPI 실행 시 Redis 구동 설정
- 프레임 목록 취득 시 캐시 로직 추가
- Redis 관련 환경 변수 추가 및 코드 수정
- REDIS 관련 README.md 업데이트

### Docker 환경 개선

- 로컬 Dockerfile 이미지 변경
- 로컬 compose 파일 변경
- 개발환경 Dockerfile 수정
- 개발환경 Qdrant Dockerfile 추가
- 개발환경 compose 파일 수정
- flower 컨테이너 네트워크 설정

## 10. 코드 정리 및 최적화 (2025.11 ~ 2025.12)

### 성능 개선

- 추출 프레임 중복 제거 코드 수정
- 프레임 멀티모달 임베딩 값 오설정 수정
- 기존 분석 영상 있는 경우 처리
- Qdrant 유사도 검색 결과 0건 케이스 수정
- cosine 메서드 수정

### 코드 정리

- core 패키지로 만듦
- settings.py 코드 정리 및 환경변수 정리
- 미사용 코드 제거 및 import 제거
- 커스텀 에러 클래스 정리
- product endpoint 삭제
- Trivial 코드 정리

### 버그 수정 및 핫픽스

- JSONB 컬러 null 설정 값 수정
- 벡터 데이터 미출력 되도록 수정
- 광고 영상 원본 파일 삭제
- filtered_categories 미존재 경우 처리
- compose.yml 수정

### Streamlit 테스트

- streamlit 코드 추가 및 수정
- 스트림릿 테스트 1, 2, 3
- requirements.txt 수정

## 11. 문서화 및 CI/CD

### README 업데이트

- README.md 지속적 업데이트
- 프로젝트 설명 및 실행 방법 문서화

### 주석 및 NOTE 추가

- TODO 제거 및 NOTE 추가
- refactor 지점 지정
- 잠재적 오류에 대한 주석 남김

---

# ShopperHousePy 프로젝트 요약

## 담당 역할

백엔드 개발 전반 담당

## 핵심 기능

1. **상품 검색 시스템**
   - 텍스트/이미지 기반 상품 검색 (Qdrant 벡터 검색)
   - 가중치 기반 유사도 검색
   - Tag 기반 검색 및 가격 필터링

2. **LLM 기반 질의 분석**
   - Google Gemini를 활용한 자연어 질의 분석
   - 카테고리/옵션 분석 및 narrow down
   - 유사 질문 생성

3. **광고 영상 분석 (2차 POC)**
   - 영상 프레임 추출 및 이미지 변환
   - VertexAI 멀티모달 임베딩을 활용한 이미지 분석
   - 이미지 클러스터링 (CLIP 모델)
   - 영상 기반 상품 검색

4. **비동기 처리 및 캐싱**
   - Celery를 활용한 백그라운드 작업 처리
   - Redis 캐시 적용으로 성능 최적화

5. **인프라 및 DevOps**
   - Docker 환경 구축 (local, dev)
   - GCS (Google Cloud Storage) 연동
   - Signed URL 생성 및 관리

## 주요 기술 스택

- **Backend**: Python, FastAPI
- **Database**: PostgreSQL
- **Vector DB**: Qdrant
- **Cache**: Redis
- **Task Queue**: Celery, Flower
- **AI/ML**: Google Gemini, VertexAI, CLIP, OpenCV
- **Storage**: Google Cloud Storage
- **DevOps**: Docker, Docker Compose, Git Actions

## 특이사항

- 텍스트 기반 검색에서 영상 기반 검색으로 확장하는 2차 POC 진행
- 이미지 클러스터링을 통한 대표 프레임 추출로 효율성 향상
- Signed URL을 통한 보안 강화
