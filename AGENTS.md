# AGENTS.md

Agent가 Snowflake PoC를 실행할 때 가장 먼저 읽는 운영 매뉴얼이다.
`CLAUDE.md` 는 자동 로드되므로 이미 읽은 상태다. 이 파일을 읽은 후 실행 계획으로 이동한다.

---

## 1. 하네스 구조

```
CLAUDE.md          ← 자동 로드 (행동 원칙, 디렉토리 구조)
AGENTS.md          ← 지금 이 파일 (실행 절차, 단계별 지침)
pocs/
├── _template/     ← 새 PoC 시작 시 복사 원본
└── <회사명>/      ← 실제 작업 디렉토리
    ├── CONTEXT.md             ← 고객 정보, 목표 아웃풋 (작업 범위 기준)
    ├── exec-plans/active/     ← 현재 수행할 실행 계획
    ├── exec-plans/completed/  ← 완료된 계획
    └── generated/             ← 모든 결과물 저장
```

**작업 범위의 단일 기준은 `CONTEXT.md` 다.**
체크된 목표 아웃풋 외에는 만들지 않는다.

---

## 2. 새 PoC 시작 절차

```
1. pocs/_template/ 을 pocs/<회사명>/ 으로 복사한다
2. .env.example 을 .env 로 복사하고 자격증명을 채운다
3. CONTEXT.md 를 채운다 (고객 정보, 연결 메타데이터, 데이터 소스, 목표 아웃풋)
4. 목표 아웃풋에 없는 outputs/ 파일은 삭제한다
5. Snowflake 연결을 확인한다 (아래 섹션 참조)
6. exec-plans/active/01-data-ingestion.md 로 이동한다
```

---

## 3. 단계별 실행 지침

### 1단계 — 데이터 확보

데이터 소스 유형에 따라 아래 방식 중 하나를 따른다.

| 데이터 소스 | 접근 방식 |
|-----------|---------|
| CSV / JSON 파일 직접 수신 | Snowflake Stage 생성 → `COPY INTO` 또는 `INSERT` |
| S3 버킷 (파일) | External Stage (S3 Integration) → `COPY INTO` |
| S3 버킷 (Iceberg) | Iceberg Catalog Integration → External Iceberg Table |
| RDBMS → Snowflake (ETL/ELT) | 고객 또는 사용자가 제공하는 파이프라인 확인 후 적재 검증 |
| 고객 직접 적재 | 적재 완료 여부 확인 후 2단계로 이동 |

완료 조건: Snowflake에서 `SELECT` 가 실행되는 상태 + `generated/db-schema.md` 초안 작성.

---

### 2단계 — 데이터 파악 및 검증

아래 순서로 진행한다.

```
1. 전체 테이블 목록과 행 수 파악
2. 주요 테이블 컬럼 타입, Null 비율, 유니크 값 확인
3. 테이블 간 관계(FK 추정) 파악
4. 이해되지 않는 컬럼이나 도메인 → 사용자에게 질문 목록 작성 후 확인 요청
5. 확인된 내용 기반으로 generated/table-descriptions.md 작성
6. 데이터 품질 이슈 generated/data-quality-report.md 에 기록
```

**사용자 확인 없이 도메인 의미를 추측하지 않는다.**
불명확한 항목은 반드시 목록으로 정리해 사용자에게 한 번에 물어본다.

완료 조건: `table-descriptions.md`, `data-quality-report.md` 작성 완료 + 고객 확인 사항 해소.

---

### 3단계 — 데이터 활용

**`CONTEXT.md` 의 목표 아웃풋 체크리스트만 보고 작업 목록을 결정한다.**

| 아웃풋 | 저장 위치 | 핵심 고려사항 |
|--------|---------|------------|
| Semantic View | `outputs/semantic-view.sql` | 비즈니스 용어 반영, 2단계 table-descriptions 기반 |
| Cortex Analyst | `outputs/cortex-analyst.md` | Semantic View 가 선행되어야 함 |
| Cortex Search | `outputs/cortex-search.md` | 검색 대상 컬럼, chunking 전략 명시 |
| Cortex Agent | `outputs/cortex-agent.md` | 사용할 Tool 목록 (Analyst / Search / SQL) 명시 |
| Snowflake Intelligence | `outputs/snowflake-intelligence.md` | Agent 구성 기반 |
| Streamlit Dashboard | `outputs/streamlit-dashboard.py` | 고객이 요청한 시각화 항목 기반 |

아웃풋 간 의존 관계:
```
Semantic View → Cortex Analyst → Cortex Agent → Snowflake Intelligence
Cortex Search ──────────────────↗
```

완료 조건: 체크된 아웃풋 전부 `outputs/` 에 저장, 사용자 검토 가능한 상태.

---

### 4단계 — 고객 피드백 및 반복

```
1. 결과물을 사용자에게 공유하고 고객 전달을 요청한다
2. 피드백 내용을 CONTEXT.md 소통 기록에 추가한다
3. 수정 사항이 있으면 해당 exec-plan 을 active/ 로 되돌려 재작업한다
4. 수정 사항이 없으면 사용자에게 다음을 안내한다:
   - 예상 비용 산출 필요 여부
   - 고객 자체 구축 지원 계획
```

---

## 4. Snowflake 연결

### 자격증명 로딩
모든 `snow` 명령 실행 전, 작업 중인 회사의 `.env` 를 환경변수로 로드한다.

```powershell
# pocs/<회사명>/.env 로드
Get-Content "pocs/<회사명>/.env" | ForEach-Object {
    if ($_ -match '^\s*([^#][^=]*?)\s*=\s*(.*)\s*$') {
        [System.Environment]::SetEnvironmentVariable($matches[1].Trim(), $matches[2].Trim())
    }
}
```

로드 후 연결 확인:
```powershell
snow connection test
```

### 주요 snow CLI 명령

| 작업 | 명령 |
|------|------|
| SQL 인라인 실행 | `snow sql -q "SHOW DATABASES;"` |
| SQL 파일 실행 | `snow sql -f query.sql` |
| Stage에 파일 업로드 | `snow stage upload <로컬파일> @<stage명>` |
| Cortex Analyst YAML 배포 | `snow cortex analyst create -f outputs/cortex-analyst.yaml` |
| Streamlit 앱 배포 | `snow streamlit deploy` |

### 컨텍스트 파악용 기본 쿼리

```sql
-- 사용 가능한 DB / Role / Warehouse 파악
SHOW DATABASES;
SHOW ROLES;
SHOW WAREHOUSES;

-- 현재 세션 확인
SELECT CURRENT_ACCOUNT(), CURRENT_USER(), CURRENT_ROLE(), CURRENT_WAREHOUSE();

-- 스키마 및 테이블 목록
SHOW SCHEMAS IN DATABASE <db>;
SHOW TABLES IN SCHEMA <db>.<schema>;
```

### 주의사항
- `.env` 는 절대 git에 커밋하지 않는다 (`.gitignore` 에 등록되어 있음)
- 여러 회사 작업 시 `.env` 를 반드시 해당 회사 것으로 재로드한다
- `CONTEXT.md` 의 기본 Role / Warehouse 를 우선 사용하고, 권한 오류 시 사용자에게 확인한다

---

## 5. 공통 규칙

- **계획 우선**: `exec-plans/active/` 에 계획이 없으면 작업을 시작하지 않는다
- **범위 준수**: `CONTEXT.md` 에 없는 아웃풋은 만들지 않는다. 필요하다고 판단되면 사용자에게 먼저 확인한다
- **범위 초과 항목 기록**: 지금 당장 하지 않을 작업은 `exec-plans/tech-debt-tracker.md` 에 기록한다
- **파괴적 작업 사전 확인**: DDL(CREATE, DROP, ALTER), 데이터 삽입/삭제는 계획에 명시된 경우에만, 실행 전 사용자에게 확인한다
- **불확실한 경우 중단**: 다음 단계가 불명확하거나 도메인 지식이 부족한 경우 추측하지 않고 사용자에게 질문한다
- **결과물 경로**: 모든 생성 파일은 반드시 `pocs/<회사명>/generated/` 하위에 저장한다
