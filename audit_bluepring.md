# Repository Audit Blueprint: PROD_NUM_DB

## 1. Executive Summary
This blueprint defines the target state for the `PROD_NUM_DB` repository. The architecture is designed to serve a dual purpose: supporting **human data engineers** through clear operational boundaries, and empowering **Agentic AI** through strictly separated, machine-readable semantic mapping and headless tooling.

---

## 2. Target Directory Architecture
An audited, compliant repository must adhere to the following structure:

.github/
├── copilot-instructions.md        # Repository-wide AI rules for IDE assistants
└── agents/
    ├── tools/                     # Python MCP tools (e.g., query_snowflake.py)
    ├── data_ops_agent.yaml        # Headless configuration for Ops Agent
    ├── data_product_agent.yaml    # Headless configuration for Product Agent
    └── prompts/                   # Markdown cognitive instructions for agents
catalogs/                          
└── lineage.yaml                   # Maps dynamic runtime dependencies 
docs/
├── README.md                      # Unified Table of Contents
├── architecture/                  # Orchestration state machine details
├── governance/                    # RBAC and role request protocols
└── operations/                    # Incident response and Sev-1 runbooks
semantic_layer/                    # Machine-readable JSON business logic
├── tables/                        # Logical columns and metadata
├── joins/                         # Permitted entity relationships
└── metrics/                       # Business logic and aggregations
sql/                               # Physical Declarative State
├── EXTERNAL_PROXIES/              # Views masking external dependencies
├── PROCEDURES/                    # Orchestration logic
├── SEMANTIC_VIEWS/                # Downstream presentation models
├── TASKS/                         # Snowflake schedule declarations
└── WORKING_TABLES/                # Staging and intermediate structures

---

## 3. Core Principles & System Invariants
Every pull request and agentic action must be audited against these four unbreakable rules:

1. **Strict Namespace Symmetry:** Subdirectories under `/sql/` must map 1:1 to Snowflake schemas. Physical views in `SEMANTIC_VIEWS` must have a corresponding JSON definition in `/semantic_layer/tables/`.
2. **One Object Per File:** A `.sql` file must contain exactly one statement matching the filename.
3. **Idempotent Execution:** All physical code must utilize `CREATE OR REPLACE` or `CREATE TABLE IF NOT EXISTS`.
4. **Zero Embedded Secrets:** No external cloud credentials or API keys may exist in plain text within `/sql/` or `/docs/`.

---

## 4. Component Deep-Dive

### 4.1 The Physical Layer (`/sql/`)
This is the execution territory. It manages the physical data flow pattern:
* **The State Machine:** Orchestration `/sql/PROCEDURES/` manage the lifecycle of data moving from `TMP_` (session-level temporary tables) to `WRK_` (working staging tables) to `TGT_` (physical target tables).
* **The Boundary:** External source tables are never referenced directly in business logic. They are queried exclusively through `/sql/EXTERNAL_PROXIES/`.

### 4.2 The Cognitive Layer (`/semantic_layer/` & `/catalogs/`)
This is the reasoning map for autonomous agents.
* **Semantic Decoupling:** Business metrics, join keys, and column descriptions live in structured `.json` files, completely separated from the complex CTEs used to physically build the views.
* **Lineage Bridge:** Because standard DDL parsing cannot trace data through runtime control tables and temporary staging layers, `lineage.yaml` explicitly maps these invisible "hops" so agents can perform root-cause analysis on stale pipelines.

### 4.3 AI & Operations Control (`.github/` & `/docs/`)
* **Human-in-the-Loop (`copilot-instructions.md`):** Drives IDE-based AI (like GitHub Copilot) to enforce directory rules and coding standards. 
* **Headless Automation (`/agents/`):** YAML files define the tools and LLM parameters, pointing to `/prompts/*.md` for Standard Operating Procedures (SOPs).
* **Operations:** ServiceNow routing, alerting thresholds, and feature intake templates are isolated in `/docs/operations/` to prevent context pollution during agentic RAG searches.

---

## 5. Migration & Audit Checklist

Use this phased plan to refactor your existing repository to the target state.

### Phase 1: Structural Baselining
- [ ] Move all existing DDL/DML scripts into `/sql/<SCHEMA_NAME>/` directories.
- [ ] Audit `.sql` files to ensure only one `CREATE` statement exists per file.
- [ ] Ensure all DDL is idempotent (`CREATE OR REPLACE`).
- [ ] Scan for and remove any hardcoded credentials; replace with Snowflake Integrations.

### Phase 2: Orchestration & Lineage Mapping
- [ ] Identify all control-table-driven pipelines (Temp -> Working -> Target).
- [ ] Ensure all external system dependencies are routed through an `EXTERNAL_PROXY` view.
- [ ] Draft `catalogs/lineage.yaml` to explicitly document the runtime hops between external proxies, working tables, and target tables.

### Phase 3: Semantic Extraction
- [ ] Migrate existing Google-style Open Knowledge JSON files into `/semantic_layer/`.
- [ ] Cross-reference `/semantic_layer/tables/` against `/sql/SEMANTIC_VIEWS/` to ensure no orphaned definitions exist.
- [ ] Document approved relationships in `/semantic_layer/joins/`.

### Phase 4: AI & Operational Wiring
- [ ] Draft `01_incident_response.md` (defining Sev-1 vs Sev-3 and ServiceNow assignment groups).
- [ ] Draft `01_rbac_model.md` detailing Snowflake role assignments.
- [ ] Create `.github/copilot-instructions.md` to feed repo invariants into developer IDEs.
- [ ] Create YAML configuration and Markdown SOP files for the **Data Ops Agent** and **Data Product Agent** in `.github/agents/`.
