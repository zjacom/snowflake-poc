# demo PoC Context

## 고객 정보
- 회사명: demo
- 담당자:
- PoC 시작일: 2026-06-12
- PoC 목표: 테스트

## Snowflake 연결
- 자격증명: `.env` 파일 (Account / User / Password / Role / Warehouse / Database 모두 거기서 관리)
- 메모:

## 데이터 소스 (1단계)
- [ ] 파일 직접 수신 (CSV, JSON 등)
- [x] 클라우드 스토리지 연결 (S3 버킷, Iceberg 등)
- [ ] ETL/ELT (RDBMS → Snowflake)
- [ ] 고객 직접 적재

### S3 연결 정보
- S3 버킷: seunghoon-bitcoin 
- 경로 (prefix): bitcoin
- 파일 형식: csv
- AWS Region: ap-northeast-2
- Storage Integration 이름 (기존 존재 시):

## 목표 아웃풋 (3단계, 고객 사전 합의)
- [x] Semantic View
- [x] Cortex Analyst
- [ ] Cortex Search
- [x] Cortex Agent
- [ ] Snowflake Intelligence
- [x] Streamlit Dashboard
- [ ] 기타:

## 비해당 항목
- Cortex Search: 이번 PoC 범위 외
- Snowflake Intelligence: 이번 PoC 범위 외

## 소통 기록
| 날짜 | 내용 |
|------|------|
| 2026-06-12 | PoC 시작 |
