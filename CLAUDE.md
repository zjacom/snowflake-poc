# Snowflake PoC 자동화

## 프로젝트 목적
하네스 엔지니어링(Harness Engineering) 방식으로 Snowflake PoC 작업을 자동화한다.
Agent는 명확한 실행 계획을 기반으로 작업하고, 결과를 문서화하며, 정해진 범위를 벗어나지 않는다.

## Agent 시작 절차
1. `AGENTS.md` 를 읽는다 — 하네스 구조 및 실행 규칙
2. `pocs/<회사명>/CONTEXT.md` 에서 고객 정보와 목표 아웃풋을 확인한다
3. `pocs/<회사명>/exec-plans/active/` 에서 현재 실행 계획을 확인한다
4. 계획에 따라 작업을 수행한다
5. 결과물은 `pocs/<회사명>/generated/` 에 저장한다
6. 완료된 계획은 `pocs/<회사명>/exec-plans/completed/` 로 이동한다

## 디렉토리 구조
```
CLAUDE.md                        ← 이 파일 (자동 로드, 항상 최신 유지)
AGENTS.md                        ← Agent 실행 매뉴얼
ARCHITECTURE.md                  ← 시스템 아키텍처
docs/
├── references/                  ← Snowflake 공통 참조 문서
├── DESIGN.md                    ← 설계 결정 기록
├── PLANS.md                     ← 상위 계획
└── SECURITY.md                  ← 보안 고려사항

pocs/
├── _template/                   ← 새 PoC 시작 시 복사하는 템플릿
│   ├── CONTEXT.md               ← 고객 정보, 목표 아웃풋, 소통 기록
│   ├── exec-plans/
│   │   ├── active/              ← 진행 중인 실행 계획 (4단계 템플릿 포함)
│   │   ├── completed/           ← 완료된 실행 계획
│   │   └── tech-debt-tracker.md
│   └── generated/
│       ├── db-schema.md
│       ├── table-descriptions.md
│       ├── data-quality-report.md
│       └── outputs/             ← 목표 아웃풋 (CONTEXT.md 체크 항목만 작성)
│           ├── semantic-view.sql
│           ├── cortex-analyst.md
│           ├── cortex-search.md
│           ├── cortex-agent.md
│           └── streamlit-dashboard.py
│
└── <회사명>/                     ← 실제 PoC (_template 복사 후 진행)
```

## Cortex Code Skills
로컬에 설치된 Cortex Code가 제공하는 Skills를 PoC 작업에 활용한다.
SKILL.md는 단계별 워크플로우 지침이며, Claude Code도 동일하게 읽고 따를 수 있다.

```
CORTEX_CODE_SKILLS_PATH=C:\Users\USER\AppData\Local\Programs\Cortex Code\resources\app\resources\snowflake\skills\cortex-code-skills
```

**적용 규칙**: SKILL.md 내 `snowflake_sql_execute(...)` 는 `snow sql -q "..."` 로 대체한다.
**로드 방법**: Read 툴로 해당 경로의 `SKILL.md` 를 읽고 지침을 따른다.

## 행동 원칙
- 실행 계획 없이 작업을 시작하지 않는다
- `CONTEXT.md` 의 목표 아웃풋에 없는 항목은 만들지 않는다
- PoC 범위를 벗어나는 작업은 `tech-debt-tracker.md` 에 기록하고 멈춘다
- 외부 시스템에 영향을 주는 작업 (DDL, 데이터 삽입 등)은 계획에 명시된 경우에만 수행한다
- 불확실한 경우 작업을 중단하고 사용자에게 확인을 요청한다
