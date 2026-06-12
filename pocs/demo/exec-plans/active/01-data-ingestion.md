# 1단계: 데이터 확보

## 목표
S3 버킷의 Bitcoin CSV 데이터를 Snowflake에 적재하여 쿼리 가능한 상태로 만든다.

## 데이터 소스
- S3 버킷: `seunghoon-bitcoin`
- prefix: `bitcoin`
- 파일 형식: CSV
- AWS Region: `ap-northeast-2`
- Snowflake 계정 리전: 확인 필요 (연결 정보 기준)

## 작업 목록

### 1-1. S3 파일 구조 파악
- [ ] `snow sql`로 S3 외부 스테이지 생성 전 파일 목록 미리 확인 (LIST 명령)

### 1-2. Storage Integration 생성 (ACCOUNTADMIN 필요)
- [ ] `CREATE STORAGE INTEGRATION` — S3 ap-northeast-2 대상
- [ ] IAM Role ARN 확인 및 Trust Policy 설정 (사용자 협조 필요)

### 1-3. External Stage 생성
- [ ] `CREATE STAGE` — Storage Integration + S3 경로 + CSV 파일 포맷 지정

### 1-4. 테이블 생성 및 데이터 적재
- [ ] CSV 헤더/컬럼 파악 후 `CREATE TABLE`
- [ ] `COPY INTO` 실행

### 1-5. 적재 검증
- [ ] `SELECT COUNT(*)` 및 샘플 확인
- [ ] `generated/db-schema.md` 초안 작성

## 완료 조건
- Snowflake에서 데이터 SELECT 가능
- `generated/db-schema.md` 초안 작성 완료

## 주의사항
- Storage Integration 생성은 ACCOUNTADMIN 권한 필요 → 현재 역할 확인됨 (ACCOUNTADMIN)
- IAM Trust Policy 설정은 AWS 콘솔 접근 필요 → 사용자 직접 수행
- DDL 실행 전 사용자 확인 후 진행
