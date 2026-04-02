# Data Platform Flink — Repository Explainer

## What This Repository Does

This is MasterControl's **real-time data platform**, built on **Confluent Cloud for Apache Flink**. It replaces batch ETL (formerly dbt on Databricks) with streaming SQL pipelines that transform CDC (Change Data Capture) events from multiple product databases into a Kimball-style dimensional model consumed by BI dashboards (Sisense).

**In one sentence:** Raw Debezium CDC events flow in from Kafka topics, get deduplicated, joined, and shaped through layered Flink SQL statements, and land in dimension/fact Kafka topics that sink into a data lakehouse (Snowflake) for reporting.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PRODUCT DATABASES                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │   AQEM   │  │    QX    │  │    MX    │  │   AMX    │  │   MOF    │    │
│  │ Postgres │  │ SQL Svr  │  │ Outbox   │  │ Postgres │  │ Outbox   │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
│       │              │              │              │              │          │
│       ▼              ▼              ▼              ▼              ▼          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    DEBEZIUM CDC CONNECTORS                          │    │
│  │  clone.{env}.ap-*   clone.{env}.mc.*   clone.{env}.manufacturing.* │    │
│  └────────────────────────────────┬────────────────────────────────────┘    │
└───────────────────────────────────┼─────────────────────────────────────────┘
                                    │ Kafka Topics (Avro + Schema Registry)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│              CONFLUENT CLOUD FOR FLINK (this repository)                    │
│                                                                             │
│   ┌─────────┐    ┌──────────────┐    ┌────────────┐    ┌───────┐          │
│   │ Sources │───►│Intermediates │───►│ Dimensions │───►│ Views │          │
│   │ (dedup) │    │ (transform)  │    │   & Facts  │    │ (MVs) │          │
│   └─────────┘    └──────────────┘    └────────────┘    └───┬───┘          │
│                                                             │              │
│   ┌─────────┐         ┌──────────┐                         │              │
│   │  Seeds  │         │   UDFs   │                         │              │
│   │ (static)│         │  (Java)  │                         │              │
│   └─────────┘         └──────────┘                         │              │
└────────────────────────────────────────────────────────────┼──────────────┘
                                                              │
                              Kafka Topics (sink)             │
                                                              ▼
                    ┌──────────────────────────────────────────────┐
                    │          SNOWFLAKE / DATABRICKS               │
                    │   (Kafka Connect → Data Lakehouse)           │
                    │                                              │
                    │   ┌──────────────────────────────────┐      │
                    │   │         SISENSE (BI)              │      │
                    │   │  Dashboards / Pivot / Reporting   │      │
                    │   └──────────────────────────────────┘      │
                    └──────────────────────────────────────────────┘
```

---

## Repository Structure

```
data-platform-flink/
├── pipelines/                    # Core: all Flink SQL statements
│   ├── sources/                  # Layer 1: CDC deduplication
│   │   ├── aqem/                 #   Per-product folders
│   │   ├── qx/
│   │   ├── mx/
│   │   ├── amx/
│   │   ├── common/
│   │   └── adapt/               #   Adapter sources (permissions, global-nav, mof)
│   ├── intermediates/            # Layer 2: Business logic transforms
│   │   ├── aqem/
│   │   ├── qx/
│   │   ├── mx/
│   │   ├── amx/
│   │   └── common/
│   ├── dimensions/               # Layer 3a: Descriptive attributes (Kimball dims)
│   │   ├── aqem/
│   │   ├── qx/
│   │   ├── mx/
│   │   └── amx/
│   ├── facts/                    # Layer 3b: Measurable events (Kimball facts)
│   │   ├── aqem/
│   │   ├── qx/
│   │   ├── mx/
│   │   └── amx/
│   ├── views/                    # Layer 4: Materialized views (joins for BI)
│   │   ├── aqem/                 #   ~25 MVs for AQEM
│   │   └── qx/
│   ├── seeds/                    # Static reference data (INSERT VALUES)
│   ├── udf/                      # Custom Java UDFs (e.g., combine_operations)
│   ├── stage/                    # Legacy staging tables
│   ├── deadletter/               # Dead letter queue handling
│   ├── performance/              # Performance-related statements
│   ├── common.mk                 # Shared Makefile targets (Confluent CLI wrappers)
│   ├── inventory.json            # Auto-generated pipeline registry (~3200 lines)
│   └── pipeline_definition.py    # Generates aggregated pipeline inventory
│
├── config/                       # Environment configs
│   ├── shift-left-config.yml     # Template with env vars for CI/CD
│   ├── config.yaml.dev-flink     # Dev environment (Confluent Cloud)
│   ├── config.yaml.stage-flink   # Staging environment
│   └── config.yaml.stage-flink2b # Secondary staging
│
├── shift_left_extensions/        # Custom Python extensions for shift-left-utils
│   ├── sql_content_replace.py    # Env-aware SQL transforms (topic names, tenant filters)
│   └── naming_conventions.py     # Statement/pool naming rules
│
├── validation/                   # Data quality & parity testing
│   ├── continuous/               # Streaming validation (Flink → validation topic)
│   └── parity/                   # Batch vs streaming comparison (Databricks ↔ Flink)
│
├── tools/                        # Developer utilities
│   ├── pipeline_tree.py          # Dependency tree visualizer
│   ├── pipeline_deploy.py        # Deployment orchestrator
│   ├── flink_pool_report.py      # CFU utilization report
│   ├── check_standards.py        # DDL/DML naming/format validator
│   ├── copy-tables-for-testing/  # Table cloner for isolated testing
│   └── per-topic-validation/     # Topic comparison SQL generator
│
├── .github/                      # CI/CD
│   ├── workflows/                # GitHub Actions (deploy, test, lint, cleanup)
│   ├── scripts/                  # Deployment/check/test Python scripts
│   └── actions/                  # Reusable actions (setup-shift-left)
│
├── docs/                         # Architecture & process documentation
│   ├── architecture-decision.md  # Env isolation strategy
│   ├── dev-process.md            # Developer workflow guide
│   ├── implementation_decisions.md  # SQL patterns, SID design, CDC handling
│   ├── ci-cd.md                  # Deployment pipeline docs
│   ├── schema_evolution.md       # Schema compatibility strategies
│   └── ...
│
├── cursor_instructions/          # AI-assisted coding instructions
│   ├── aqem/                     # CDC timestamp implementation guide
│   ├── amx/                      # AMX development instructions
│   └── qx/                       # QX development instructions
│
├── archive/                      # Deprecated pipeline code
├── staging/                      # Work-in-progress SQL (pre-review)
├── databricks/                   # Databricks prototype views
└── todo/                         # Task planning docs
```

---

## Pipeline Anatomy — How One Pipeline Works

Every pipeline (source, intermediate, dimension, fact, or view) follows the same structure:

```
pipelines/{layer}/{product}/{pipeline_name}/
├── Makefile                          # Deployment targets (include common.mk)
├── sql-scripts/
│   ├── ddl.{table_name}.sql          # CREATE TABLE (Kafka-backed Flink table)
│   └── dml.{table_name}.sql          # INSERT INTO ... SELECT (the transform)
├── tests/
│   ├── test_definitions.yaml         # Test harness config
│   ├── ddl_*.sql                     # Test table definitions (_ut suffix)
│   ├── insert_*.sql                  # Test input data
│   ├── validate_*.sql                # Expected output assertions
│   └── README.md                     # Test documentation
├── pipeline_definition.json          # Auto-generated lineage metadata
└── tracking.md                       # Migration status notes
```

### DDL Pattern (All Tables)

Every table is a Kafka-backed Flink table using Avro with Schema Registry:

```sql
CREATE TABLE IF NOT EXISTS {table_name} (
    id          STRING NOT NULL,
    name        STRING,
    tenant_id   STRING NOT NULL,
    ts_ms       BIGINT,                          -- CDC timestamp
    PRIMARY KEY(sid) NOT ENFORCED                 -- Surrogate or natural key
) DISTRIBUTED BY HASH(sid) INTO {N} BUCKETS WITH (
    'changelog.mode' = 'upsert',                 -- Upsert semantics
    'key.format' = 'avro-registry',
    'value.format' = 'avro-registry',
    'key.avro-registry.schema-context' = '.flink-dev',  -- Env-swapped at deploy
    'value.avro-registry.schema-context' = '.flink-dev',
    'kafka.retention.time' = '0',                -- Infinite retention (compacted)
    'kafka.producer.compression.type' = 'snappy',
    'scan.startup.mode' = 'earliest-offset',
    'scan.bounded.mode' = 'unbounded'
)
```

### DML Pattern by Layer

Each layer has a distinct SQL pattern:

#### Sources — CDC Deduplication

```sql
INSERT INTO src_{product}_{table}
WITH valid_tenants AS (
    SELECT tenant_id FROM stage_tenant_dimension
    UNION ALL
    SELECT tenant_id FROM seed_{product}_config_tenants
),
source AS (
    -- Non-delete records with tenant filter
    SELECT COALESCE(IF(e.op = 'd', e.before.id, e.after.id), 'NULL') AS id,
           COALESCE(e.connector_name, 'NULL') AS connector_name,
           e.after.* , e.op, e.source.ts_ms, e.source.lsn
    FROM `clone.dev.{cdc_topic}` e
    INNER JOIN valid_tenants t ON t.tenant_id = e.after.tenant_id
    UNION ALL
    -- Delete records (separate because tenant_id may be NULL)
    SELECT ... FROM `clone.dev.{cdc_topic}` e WHERE e.op = 'd'
),
prefinal AS (
    -- Deduplicate by LSN (latest write wins)
    SELECT *, ROW_NUMBER() OVER (PARTITION BY id, connector_name ORDER BY lsn DESC) AS rownum
    FROM source
),
final AS (
    SELECT ... FROM prefinal WHERE rownum = 1 AND op <> 'd'
)
SELECT * FROM final   -- shift-left injects tenant filter here in dev
```

#### Intermediates — Business Logic

```sql
INSERT INTO int_{product}_{name}
WITH dummy_rows AS (
    -- One NULL row per tenant (for outer-join safety downstream)
    SELECT CAST(NULL AS STRING) AS id, t.tenant_id, t.ts_ms
    FROM stage_tenant_dimension AS t
),
element_src AS (
    -- Business transforms: parent-child joins, JSON extraction, etc.
    SELECT child.*, parent.type AS parent_element_type,
           JSON_VALUE(parent.settings, '$.rowLabel') AS entity_label,
           GREATEST(child.ts_ms, parent.ts_ms) AS ts_ms
    FROM src_{product}_{table} AS child
    LEFT JOIN src_{product}_{table} AS parent ON parent.id = child.parent_id
),
final AS (
    SELECT ... FROM element_src
    UNION ALL
    SELECT 'NULL' AS id, ..., 'Missing Data' AS name, ... FROM dummy_rows
)
SELECT * FROM final
```

#### Dimensions & Facts — Surrogate Keys + Joins

```sql
INSERT INTO {product}_dim_{name}
WITH ... AS (...)
SELECT
    MD5(CONCAT_WS(',', src.id, src.tenant_id)) AS sid,   -- Surrogate key
    src.id AS element_id,
    ...,
    src.tenant_id,
    GREATEST(src.ts_ms, COALESCE(joined.ts_ms, 0)) AS ts_ms
FROM src
LEFT JOIN other_table ON ...
```

#### Views (MVs) — Multi-table Joins for BI

Views are the most complex: they join multiple dimensions and facts together to produce the final shape consumed by BI tools.

```sql
INSERT INTO {product}_mv_{name}
WITH ... AS (
    SELECT DISTINCT ... FROM {product}_fct_event_step_element
),
... AS (
    SELECT ..., ARRAY_JOIN(ARRAY_SORT(ARRAY_AGG(value)), ', ') AS element_value
    FROM {product}_dim_event_element GROUP BY ...
)
SELECT
    MD5(CONCAT_WS(',', ...)) AS sid,
    ...
FROM dim_step
LEFT JOIN fct_step_element ON ...
LEFT JOIN dim_event_element ON ...
LEFT JOIN fct_element_tag ON ...
LEFT JOIN dim_tag ON ... AND dim_tag.name <> 'Missing Tag Data'
```

---

## Key Design Patterns

### 1. Surrogate Keys (SIDs)

Every dimension and fact uses `MD5(CONCAT_WS(',', ...))` to generate deterministic surrogate keys. This ensures:
- Idempotent reprocessing (same input = same key)
- Consistent cross-table joins
- No dependency on auto-increment sequences

```sql
MD5(CONCAT_WS(',', element_id, tenant_id)) AS sid          -- Dimension SID
MD5(CONCAT_WS(',', element_id, tag_id, tenant_id)) AS sid  -- Fact SID
```

### 2. Dummy Rows for Outer-Join Safety

Intermediates produce a "dummy row" per tenant with `id = 'NULL'` and descriptive names like `'Missing Element Data'`. This ensures downstream INNER JOINs don't drop tenants that have no data yet — critical for streaming where data arrives asynchronously.

### 3. CDC Deduplication by LSN

Sources use `ROW_NUMBER() OVER (PARTITION BY id, connector_name ORDER BY lsn DESC)` to keep only the latest version of each record per CDC connector, handling out-of-order delivery.

### 4. Timestamp Lineage (`ts_ms`)

Every table carries a `ts_ms BIGINT` column. Downstream tables compute `GREATEST(parent1.ts_ms, parent2.ts_ms, ...)` to propagate the latest upstream change timestamp. This enables downstream consumers to know when data was last refreshed.

### 5. Multi-Tenant Isolation

All pipelines filter by `tenant_id`. In dev environments, the `shift_left_extensions` automatically inject:
```sql
WHERE tenant_id IN (SELECT tenant_id FROM tenant_filter_pipeline WHERE product = '{product}')
```

### 6. Environment-Aware SQL (shift-left-utils)

The `shift_left_extensions/sql_content_replace.py` transforms SQL at deploy time:
- **Topic names:** `.dev.` → `.stage.` or `.prod.`
- **Schema context:** `.flink-dev` → `.flink-stage` or `.flink-prod`
- **Region expansion:** `<<REGION_EXPAND>>` blocks become UNION ALL per region
- **Tenant filtering:** Added only in dev environments

### 7. Tag Inheritance & Pivoting (AQEM-specific)

Tags flow: `src_aqem_tag_link` → `aqem_fct_element_tag_relation` → `aqem_mv_dim_event_element_tag_for_pivot`. Tags can be:
- **Direct:** Applied to the element itself (`is_inherited = FALSE`)
- **Inherited:** From a parent element (`is_inherited = TRUE`)
- **MASTERCONTROL:** System-wide tags that match across all tenants
- **CUSTOMER:** Tenant-scoped custom tags

---

## Data Flow — End to End (AQEM Example)

```
PostgreSQL (AQEM DB)
    │
    │ Debezium CDC → Kafka topic: clone.dev.ap-tag-dev.state.link
    │
    ▼
┌──────────────────────────────────────────────────────────────┐
│  SOURCE: src_aqem_tag_link                                   │
│  • Reads Debezium envelope (before/after/op/source)          │
│  • Filters valid tenants                                     │
│  • Deduplicates by LSN                                       │
│  • Removes deletes (op <> 'd')                               │
└──────────────────────┬───────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────────────┐
│  INTERMEDIATE: int_aqem_recordconfiguration_form_element_dummy│
│  • Self-join (child ↔ parent elements)                       │
│  • Extracts JSON settings ($.rowLabel)                       │
│  • Adds dummy rows per tenant                                │
└──────────────────────┬───────────────────────────────────────┘
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
┌────────────────────┐  ┌─────────────────────────────────┐
│ DIMENSION:         │  │ FACT:                            │
│ aqem_dim_element   │  │ aqem_fct_element_tag_relation   │
│ • SID = MD5(id,    │  │ • Links elements → tags          │
│   tenant_id)       │  │ • Direct + inherited tags        │
│ • has_analytic_tag │  │ • Revision-level inheritance     │
│ • is_subform_child │  │ • SID = MD5(element, tag, tenant)│
└────────┬───────────┘  └─────────────┬───────────────────┘
         │                             │
         └──────────┬──────────────────┘
                    ▼
┌──────────────────────────────────────────────────────────────┐
│  VIEW: aqem_mv_dim_event_element_tag_for_pivot               │
│  • Joins: dim_step → fct_step_element → dim_event_element    │
│           → fct_element_tag → dim_tag                        │
│  • Aggregates multi-select: ARRAY_AGG → comma-delimited      │
│  • Classifies types: STRING / NUMBER / DATE                  │
│  • Output: one row per (event_element, tag, table_row)       │
└──────────────────────────────────────────────────────────────┘
                    │
                    ▼ Kafka topic → Kafka Connect → Snowflake
                    ▼ Sisense queries Snowflake for pivot reports
```

---

## Products Supported

| Product | Code Prefix | Description | Pipeline Layers |
|---------|-------------|-------------|-----------------|
| **AQEM** | `aqem_` | Quality Event Management (workflows, forms, approvals) | All 5 layers + 25 MVs |
| **QX** | `qx_` | Document Control (infocards, training, routing) | All 5 layers |
| **MX** | `mx_` | Manufacturing Execution (production records, captures) | Sources → Facts |
| **AMX** | `amx_` | Advanced Manufacturing (phases, steps, captures) | All 5 layers |
| **MOF** | `mof_` | Object Framework (catalogs, views, history) | Planned (PQI-7241) |

---

## Testing Strategy

### Unit Tests

Each pipeline has `tests/test_definitions.yaml` that declares:
1. **Foundations:** DDL for each input table (with `_ut` suffix)
2. **Test suites:** Named tests with input INSERT statements and validation SQL

Tests are run via `shift_left table init-unit-tests`:
- Creates `_ut` suffixed tables in Flink
- Inserts test data
- Runs the pipeline DML (rewritten to use `_ut` tables)
- Validates output against expected SQL

### Continuous Validation

Flink statements in `validation/continuous/` write to `*_streaming_validation` topics, checking:
- Volume monitoring (records per time window)
- Null checks on critical columns
- Duplicate detection

### Parity Testing

`validation/parity/` compares Databricks batch output vs Flink streaming output to verify migration correctness. Uses a Python toolkit with YAML configs per product.

---

## Deployment (CI/CD)

### GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `entrypoint.yml` | Push/PR to main | Security checks, linting, release tagging |
| `statement-deployment.yml` | `workflow_dispatch` | Deploy Flink statements to any environment |
| `flink-deploy-by-product.yml` | Manual | Deploy all tables for a product |
| `flink-deploy-by-table.yml` | Manual | Deploy specific table(s) |
| `flink-deploy-updates.yml` | Push to main | Auto-deploy changed SQL to dev |
| `flink-statement-test.yml` | Manual | Run unit tests |
| `flink-statement-check.yml` | PR | Validate SQL standards |
| `flink-prepare-tables.yml` | Manual | Create DDL tables |

### Deployment Flow

```
Developer writes SQL
    │
    ├── PR to main
    │   └── flink-statement-check (validates SQL standards)
    │
    ├── Merge to main
    │   └── flink-deploy-updates → dev-flink-us-west-2 (auto-deploy changed files)
    │
    └── Manual workflow_dispatch
        └── statement-deployment → target env (dev / stage / prod)
            ├── setup-shift-left (install Python, shift_left CLI)
            ├── build-inventory (collect pipeline metadata)
            ├── build-all-metadata (resolve dependencies)
            └── deploy (create DDL → create DML per table)
```

### Environments

| Environment | Confluent Cloud | Cluster Type | Purpose |
|-------------|-----------------|--------------|---------|
| **dev** | `env-jzrnmm` / `us-west-2` | `dev` | Development + unit tests |
| **stage** | `env-rv6xm9` / `us-west-2` | `stage` | Pre-production validation |
| **stage-b** | `env-25yngo` / `us-west-2` | `stageb` | Secondary staging |
| **prod** | (configured via CI/CD) | `prod` | Production |

---

## Key Concepts & Terminology

| Term | Meaning |
|------|---------|
| **shift-left-utils** | Internal Python CLI tool for Flink statement management (deploy, test, validate) |
| **SID** | Surrogate ID — deterministic MD5 hash used as primary key in dims/facts |
| **MV (mv_)** | "Materialized View" — actually a continuously-running Flink SQL INSERT statement writing to a Kafka topic (not a database view) |
| **CDC** | Change Data Capture via Debezium connectors |
| **LSN** | Log Sequence Number — used for deduplication ordering in sources |
| **Dummy Row** | A sentinel row per tenant with `id='NULL'` that prevents outer-join data loss |
| **tenant_filter_pipeline** | Seed table controlling which tenants process data in dev |
| **CFU** | Confluent Flink Units — compute pool sizing metric |
| **ts_ms** | Timestamp in milliseconds from the CDC source, propagated through all layers |
| **Compute Pool** | Confluent Cloud resource allocation for Flink statements |

---

## Pitfalls & Edge Cases

### 1. Topic Name Environment Mismatch
SQL files contain dev topic names like `clone.dev.ap-tag-dev.state.link`. The `MCReplaceEnvInSqlContent` class rewrites these at deploy time. If a new topic doesn't follow the `{prefix}.dev.{middle}` pattern, the regex won't match and the statement will fail in non-dev environments.

### 2. Dummy Row `'NULL'` vs SQL NULL
Dummy rows use the string `'NULL'` (not SQL NULL) for primary key columns because Flink primary keys can't be NULL. Downstream code must handle: `CASE WHEN id = 'NULL' THEN NULL ELSE id END`.

### 3. Schema Context Swap
DDL files use `.flink-dev` as schema context. The extension swaps this to `.flink-stage` or `.flink-prod`. If a DDL accidentally hardcodes the environment, schema resolution will fail.

### 4. Multi-Region UNION ALL
For topics spanning multiple AWS regions, DML files use `<<REGION_EXPAND>>` markers. The extension expands these into UNION ALL blocks per region. Forgetting these markers means only one region's data is consumed.

### 5. Tag Collision Cartesian Products
When the same tag is applied to multiple elements, the pivot view can produce Cartesian products. For an event with N colliding values × M multi-select values × K subform rows, output = N × M × K rows. Direct-only filtering (`is_inherited = FALSE`) mitigates this.

### 6. `GREATEST()` with NULLs
`GREATEST()` returns NULL if any argument is NULL. All optional joins must wrap in `COALESCE(table.ts_ms, 0)` to prevent NULL propagation breaking the timestamp chain.

### 7. Makefile TABLE_NAME Inconsistency
Some Makefiles have `TABLE_NAME` with double prefixes (e.g., `aqem_mv_mv_dim_event_element_tag_for_pivot`), while the actual SQL table uses a single prefix. This can cause statement name mismatches during manual deployment.

---

## Quick Reference — Common Tasks

### Deploy a single table to dev
```bash
cd pipelines/dimensions/aqem/dim_element
make create_aqem_dim_element
```

### Run unit tests
```bash
shift_left table init-unit-tests --table-name aqem_dim_element
```

### Check SQL standards
```bash
python tools/check_standards.py
```

### View pipeline dependency tree
```bash
python tools/pipeline_tree.py --table aqem_mv_dim_event_element_tag_for_pivot
```

### Deploy via CI/CD
Use GitHub Actions → `statement-deployment` workflow → Select environment, product, and process type.

### Start Flink shell (interactive SQL)
```bash
cd pipelines && make start_flink_shell
```

---

## File Counts by Area

| Area | Count | Description |
|------|-------|-------------|
| AQEM Sources | 26 pipelines | CDC ingestion from AQEM databases |
| AQEM Intermediates | 11 pipelines | Transforms, dummy rows, unnesting |
| AQEM Dimensions | 18 tables | Descriptive attributes |
| AQEM Facts | 22 tables | Event/relationship measures |
| AQEM Views | 25 MVs | BI-ready joined outputs |
| QX (all layers) | ~30 pipelines | Document control product |
| MX (all layers) | ~15 pipelines | Manufacturing execution |
| AMX (all layers) | ~20 pipelines | Advanced manufacturing |
| Seeds | 7 pipelines | Static reference data |
| Validation | ~10 per product | Streaming + parity checks |
| **Total** | **~200+ pipelines** | Registered in `inventory.json` |

---

## Suggested Next Steps for New Developers

1. **Read** `docs/dev-process.md` for the development workflow
2. **Read** `docs/implementation_decisions.md` for SQL pattern rationale
3. **Study** one complete pipeline chain: `src_aqem_tag_link` → `aqem_fct_element_tag_relation` → `aqem_mv_dim_event_element_tag_for_pivot`
4. **Run** `python tools/pipeline_tree.py` to visualize dependencies
5. **Run** unit tests on a simple dimension to understand the test harness
6. **Review** `shift_left_extensions/sql_content_replace.py` to understand environment transforms
7. **Check** `config/shift-left-config.yml` to understand deployment configuration
