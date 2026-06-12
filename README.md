# Snowflake PoC 자동화 하네스

Claude Code Agent를 활용해 Snowflake PoC를 자동화하는 프로젝트.
고객마다 격리된 디렉토리에서 작업하며, Agent는 실행 계획 기반으로 동작한다.

---

## 새 PoC 시작하기

### 1단계 — 디렉토리 준비

`pocs/_template/` 을 새 회사명으로 복사한다.

```powershell
Copy-Item -Recurse pocs/_template pocs/<회사명>
```

### 2단계 — 자격증명 설정

```powershell
Copy-Item pocs/<회사명>/.env.example pocs/<회사명>/.env
```

`.env` 파일을 열고 Snowflake 계정 정보를 채운다. (`.env` 는 git에 올라가지 않는다)

```
SNOWFLAKE_ACCOUNT=abc12345.ap-northeast-1.aws
SNOWFLAKE_USER=...
SNOWFLAKE_PASSWORD=...
SNOWFLAKE_ROLE=...
SNOWFLAKE_WAREHOUSE=...
SNOWFLAKE_DATABASE=...
SNOWFLAKE_SCHEMA=...
```

### 3단계 — 고객 정보 입력

`pocs/<회사명>/CONTEXT.md` 를 열고 아래 항목을 채운다.

- 고객 정보 (회사명, 담당자, PoC 목표)
- Snowflake 연결 메타데이터 (account, 기본 role, warehouse)
- 데이터 소스 유형 체크
- **목표 아웃풋 체크** — 고객과 사전 합의한 항목만 체크한다

### 4단계 — 불필요한 아웃풋 파일 제거

`pocs/<회사명>/generated/outputs/` 에서 `CONTEXT.md` 에 체크하지 않은 파일을 삭제한다.

```
outputs/
├── semantic-view.sql       ← 미사용이면 삭제
├── cortex-analyst.md       ← 미사용이면 삭제
├── cortex-search.md        ← 미사용이면 삭제
├── cortex-agent.md         ← 미사용이면 삭제
└── streamlit-dashboard.py  ← 미사용이면 삭제
```

### 5단계 — Agent 실행

Claude Code를 열고 아래와 같이 지시한다.

```
pocs/<회사명>/ PoC를 시작해줘.
```

Agent는 `CLAUDE.md` → `AGENTS.md` → `CONTEXT.md` → `exec-plans/active/` 순서로 읽고 작업을 시작한다.

---

## 디렉토리 구조

```
pocs/
├── _template/          ← 새 PoC 시작 시 복사 원본
└── <회사명>/
    ├── .env            ← 자격증명 (git 제외)
    ├── .env.example    ← 자격증명 템플릿
    ├── CONTEXT.md      ← 고객 정보, 목표 아웃풋
    ├── exec-plans/
    │   ├── active/     ← 진행 중인 실행 계획
    │   ├── completed/  ← 완료된 실행 계획
    │   └── tech-debt-tracker.md
    └── generated/
        ├── db-schema.md
        ├── table-descriptions.md
        ├── data-quality-report.md
        └── outputs/    ← 최종 결과물
```

## PoC 사이클

| 단계 | 내용 | 실행 계획 파일 |
|------|------|-------------|
| 1. 데이터 확보 | 파일/S3/ETL/고객 직접 적재 | `01-data-ingestion.md` |
| 2. 데이터 파악 | 스키마 탐색, 품질 검증, 설명 작성 | `02-data-discovery.md` |
| 3. 데이터 활용 | Semantic View, Cortex, Streamlit 등 제작 | `03-data-utilization.md` |
| 4. 피드백 반영 | 고객 피드백 수집 및 수정 | `04-feedback-iteration.md` |

## Agent 문서

- [`AGENTS.md`](AGENTS.md) — Agent 실행 매뉴얼 (단계별 상세 지침)
- [`CLAUDE.md`](CLAUDE.md) — 하네스 행동 원칙 (자동 로드)
