# Phantom Layer: Agent-Powered Research Access for MedFlow

## What This Is

Phantom Layer is the access and monetization engine for MedFlow's anonymized research database. It sits on top of the compliance-certified Postgres DB and gives external researchers, pharma companies, and institutions a way to explore, query, and extract data — without ever touching raw infrastructure.

MedFlow solves **how data gets in** (intake → anonymization → compliance gate → research DB). Phantom Layer solves **how data gets out** — securely, compliantly, and profitably.

---

## Where It Fits in MedFlow's Architecture

```
Patient → AI Intake → Anonymization → Compliance Gate → Research DB (Postgres)
                                                              │
                                                    ┌─────────┴──────────┐
                                                    │   PHANTOM LAYER    │
                                                    │                    │
                                                    │  Virtual FS        │
                                                    │  Agent Interface   │
                                                    │  No-Code Builder   │
                                                    │  Audit + RBAC      │
                                                    └────────┬───────────┘
                                                             │
                                              ┌──────────────┼──────────────┐
                                              │              │              │
                                          Researchers    Pharma/Biotech   Public Health
```

MedFlow's existing layers handle everything up to and including the research DB. Phantom Layer is **Layer 5** — the external-facing access infrastructure that turns the DB into a product.

---

## The Problem Phantom Layer Solves

MedFlow's research DB is compliance-certified, multi-hospital, and continuously growing. But a database without an access layer is just storage. Today, MedFlow's pricing model lists "Research DB access" at $75K–$5M/year — but the document doesn't define **how** that access works.

Questions the current architecture doesn't answer:

- How does a pharma researcher in Basel explore what data MedFlow has?
- How does an academic in Madrid run a cohort analysis without writing SQL?
- How do we enforce that a Tier 1 subscriber only sees aggregates while a Tier 3 subscriber gets row-level de-identified data?
- How do we prove to regulators exactly what each external user accessed and when?
- How do we let institutions build reusable research artifacts (saved cohorts, dashboards) that justify their subscription?

Phantom Layer answers all of these.

---

## Core Concept: The Virtual Filesystem

Instead of giving researchers raw database access or building a rigid API, Phantom Layer presents MedFlow's Postgres as a **virtual filesystem**. This idea comes from a technique called ChromaFs, where a documentation company replaced expensive per-session sandboxes with a virtual filesystem over their existing database — cutting session creation from ~46 seconds to ~100ms at zero marginal cost.

We apply the same principle to MedFlow's research DB:

| Virtual Path | What It Maps To |
|---|---|
| `/hospitals/` | List of contributing hospitals (anonymized identifiers) |
| `/hospitals/h-0042/datasets/` | Available datasets from hospital h-0042 |
| `/datasets/cardiac-admissions/` | A cross-hospital dataset directory |
| `/datasets/cardiac-admissions/schema.md` | Auto-generated column definitions, types, valid ranges |
| `/datasets/cardiac-admissions/data-dictionary.md` | Field explanations, coding schemes, EHDS metadata |
| `/datasets/cardiac-admissions/sample.csv` | First 100 anonymized rows (preview) |
| `/cohorts/` | Saved cohort definitions from this researcher |
| `/cohorts/metformin-over-60.json` | A reusable filter definition |
| `/compliance/audit-log.md` | This researcher's own access history |

None of these are real files. Every `ls`, `cat`, or `grep` is intercepted and translated into a Postgres query against the research DB. The filesystem is **read-only** — no researcher can modify the underlying data.

### Why a Filesystem?

AI agents are converging on filesystem interfaces as their primary way to interact with data. Commands like browse, read, and search are universal. By presenting our DB as a filesystem:

- Any AI agent can navigate it immediately — no custom integration needed
- Researchers get a familiar mental model (directories and files)
- Access control becomes tree pruning — if you can't see a path, it doesn't exist for you
- It works over MedFlow's existing Postgres — no data duplication, no new infrastructure

### Key Design Decisions

**Instant boot**: The directory tree is built once from Postgres metadata (table names, column definitions, row counts) and cached. Subsequent sessions skip the build entirely. Session creation is <100ms.

**Lazy loading**: Schema files and data dictionaries are pre-generated from Postgres `information_schema` and cached globally. Sample data only materializes when a researcher actually reads the file. Large exports only generate on explicit request.

**Grep optimization**: When a researcher or agent searches across schemas (e.g., "which datasets contain HbA1c?"), the filesystem doesn't scan every virtual file. It queries Postgres metadata indexes directly, returning results in milliseconds even across hundreds of tables.

---

## The Agent Interface

On top of the virtual filesystem, researchers interact through an AI agent with two modes:

### Exploration Mode

The researcher asks: *"What cardiac datasets do you have from hospitals in Catalonia?"*

The agent browses `/hospitals/`, filters by region metadata, lists matching datasets, reads their `schema.md` files, and returns a structured summary. No SQL is generated. This is discovery.

This maps directly to MedFlow's **Explorer tier** ($0 queries against metadata, pay for actual data access).

### Query Mode

The researcher asks: *"How many patients over 60 had HbA1c above 9 and were prescribed metformin in the last 12 months?"*

The agent translates this into SQL, executes it against Postgres **with the researcher's tier constraints injected**, and returns results. A Tier 1 (aggregate-only) researcher gets a count. A Tier 2 (de-identified) researcher gets row-level data. The researcher never sees SQL unless they ask.

**Critical**: The agent does NOT make access control decisions. The access control service injects tier constraints into every query before execution. Even a prompt-injected agent cannot bypass row-level or column-level restrictions because the constraints are applied at the database layer, not the prompt layer. This is consistent with MedFlow's existing philosophy of defense-in-depth (regex + judge + LLM fallback).

---

## Access Control: Tree Pruning + Query Constraints

MedFlow already enforces anonymization before data enters the research DB. Phantom Layer adds a second layer of access control for **who can see what** in the already-anonymized data.

### Tree Pruning

When a researcher authenticates, Phantom Layer evaluates:

- Their institution's data sharing agreement
- Their subscription tier (Explorer / Researcher / Institutional)
- Any dataset-specific restrictions (some hospitals may opt out of certain query types)
- EHDS compliance requirements for their jurisdiction

Based on this, the virtual filesystem tree is **pruned before the researcher sees anything**. Datasets they can't access don't appear in directory listings. The agent can't reference, search, or acknowledge paths that were pruned.

This mirrors MedFlow's existing anonymization-first philosophy: just as raw PII is never stored, unauthorized data paths are never visible.

### Query-Level Tier Enforcement

| Tier | What Gets Injected |
|---|---|
| Aggregate Only | Every SELECT is wrapped in COUNT/AVG/SUM. Any attempt to select individual rows is blocked. Minimum group size of 10 enforced (small-cell suppression). |
| De-identified | Row-level access permitted. Quasi-identifiers (age, zip) are generalized (age → age band, zip → first 3 digits). Date shifting applied per-hospital. |
| Full Research | Row-level access with original quasi-identifiers. Requires active IRB approval on file. |

Tier constraints are injected by the access control service, not by the agent. The agent generates the "intent" SQL; the service wraps it with constraints before execution.

---

## No-Code Builder

For researchers who want reusable artifacts without conversing with an agent:

### Cohort Builder

Visual drag-and-drop interface for defining patient populations. Select criteria (age range, ICD codes, lab values, medications), combine with AND/OR logic, preview the count, save. Each saved cohort becomes a JSON file in `/cohorts/` that the agent can reference in future queries.

### Dashboard Builder

Create visualizations from saved cohorts: distributions, trends, cross-tabulations. Dashboards auto-refresh as new compliant records enter the research DB through MedFlow's intake pipeline. This is the "living data" advantage — unlike batch retrospective datasets (IQVIA, Truven), MedFlow's data is continuously updated.

### Export Builder

Configure recurring data exports with specific fields, filters, and tier-appropriate de-identification. Exports can be scheduled (weekly, monthly) or triggered on-demand. Every export is logged to the audit trail.

**Key integration point**: Everything created in the no-code builder is stored as a structured definition in the virtual filesystem. When a researcher builds a cohort visually and later asks the agent *"run the same analysis but split by age band"*, the agent reads the saved cohort file and modifies the query. Visual tools and agent share a single source of truth.

---

## Audit Trail Integration

MedFlow already has an append-only audit trail for compliance decisions (anonymization pass/fail, quarantine events). Phantom Layer extends this to the access side:

| Event | What Gets Logged |
|---|---|
| Session start | Researcher identity, institution, tier, pruned tree snapshot |
| Directory browse | Path accessed, timestamp |
| File read | Path, content hash, timestamp |
| Query execution | Natural language question, generated SQL, tier constraints applied, row count returned, execution time |
| Export | Fields included, filters applied, de-identification level, file hash |
| Cohort save/modify | Definition before and after, timestamp |

This serves three purposes:

1. **EHDS compliance**: Article 46 of the EHDS requires logging of all secondary data use. Phantom Layer's audit trail satisfies this out of the box.
2. **Reproducibility**: Any published research finding can be traced back to the exact query, the exact data state, and the exact access tier that produced it.
3. **Hospital transparency**: Contributing hospitals can see (in aggregate) how their data is being used — which datasets are queried most, by what type of institution, for what research areas. This builds trust and encourages continued participation.

---

## How This Maps to MedFlow's Pricing

Phantom Layer operationalizes the pricing model that MedFlow already defines:

| MedFlow Tier | Phantom Layer Implementation |
|---|---|
| **Free (Hospitals)** | Hospitals contribute data through intake pipeline. Phantom Layer is invisible to them except for the transparency dashboard showing usage of their data. |
| **Explorer** | Agent access in exploration mode only. Aggregate-only tier. Virtual filesystem browsing, schema reading, sample previews. Limited queries/month. |
| **Researcher** | Full agent access (exploration + query mode). De-identified row-level data. No-code builder. API access. Higher query limits. |
| **Institutional** | Unlimited queries. Full research tier (with IRB). SSO integration. Custom data agreements. Export builder. Dedicated support. |
| **Custom Research** | Bespoke engagements: custom cohort construction, longitudinal tracking, linked datasets across hospitals. |

The revenue share model for hospitals also becomes concrete: Phantom Layer meters every query, attributes it to contributing hospitals based on which records were accessed, and calculates revenue share automatically.

---

## Technical Implementation

### What We Build on Top of Existing MedFlow

| Component | Technology | Notes |
|---|---|---|
| Virtual filesystem server | Node.js/TypeScript middleware | Translates FS operations → Postgres queries. Inspired by ChromaFs / just-bash. |
| Tree builder | Postgres `information_schema` + cached JSON | Builds directory tree from table metadata. Cached per-session. |
| Access control service | Auth middleware + session manager | Evaluates researcher permissions, prunes tree, injects tier constraints. |
| Agent runtime | Claude API with tool access (FS + SQL tools) | Same LLM stack MedFlow already uses for intake. |
| No-code UI | React (same stack as MedFlow frontend) | Cohort builder, dashboard builder, export builder. |
| Query engine | Parameterized SQL generation + tier wrapper | Agent generates intent SQL; service wraps with constraints. |
| Audit extension | Extends MedFlow's existing append-only log | Same table structure, new event types for access-side logging. |
| Metering + billing | Usage tracking per query per institution | Feeds into revenue share calculation and subscription billing. |

### What We Reuse From MedFlow

- **Postgres research DB** — no changes to schema
- **Anonymization pipeline** — data is already clean when Phantom Layer touches it
- **Compliance gate** — already enforced before data enters the DB
- **Audit trail infrastructure** — we extend, not replace
- **React frontend** — same stack, new views
- **Claude API integration** — same LLM, new tools (filesystem + SQL instead of intake forms)

---

## Phased Build Plan

### Phase 1: Virtual Filesystem + Exploration (Weeks 1–4)

- Build the virtual FS layer over MedFlow's Postgres research DB
- Auto-generate `schema.md` and `data-dictionary.md` from table metadata
- Implement tree pruning based on researcher permissions
- Agent integration in exploration mode only (browse, read, search — no SQL)
- Extend audit trail with access-side events
- Internal testing with synthetic data

### Phase 2: Query Mode + Tier Enforcement (Weeks 5–8)

- Add query mode: natural language → SQL with tier constraint injection
- Implement aggregate-only, de-identified, and full research tiers
- Small-cell suppression for aggregate queries
- Result caching for common queries
- Pilot with 1–2 friendly research institutions

### Phase 3: No-Code Builder + Monetization (Weeks 9–14)

- Cohort builder UI with saved definitions in virtual filesystem
- Dashboard builder for recurring visualizations
- Metering and billing integration
- Revenue share calculation engine for contributing hospitals
- API gateway for programmatic institutional access
- Self-service onboarding for new institutions

### Phase 4: Scale + Marketplace (Weeks 15–20)

- Export builder with scheduled deliveries
- Cross-institution collaboration (shared cohorts with access controls)
- Hospital transparency dashboard (usage analytics for contributing hospitals)
- Public launch of researcher-facing platform

---

## Why "Phantom Layer"

The data is real. The filesystem is not. Researchers interact with a structure that feels tangible — directories, files, schemas — but nothing exists on disk. It's a phantom interface over a live database. The data never moves, never duplicates, never leaves the compliant perimeter. The researchers see exactly what they're allowed to see, and nothing more. Everything else is invisible — as if it was never there.

---

## Competitive Advantage This Creates

MedFlow's current competitive landscape slide identifies four incumbent categories. Phantom Layer closes the remaining gap:

| Incumbent Gap | How Phantom Layer Closes It |
|---|---|
| EHR vendors don't monetize secondary use | Phantom Layer IS the monetization infrastructure |
| Health data exchanges use batch retrospective data | Phantom Layer queries live, continuously updated data |
| Research platforms require manual data curation | Virtual FS auto-generates metadata from Postgres schema |
| Hospital AI intake tools don't produce research-ready data | MedFlow handles intake; Phantom Layer handles access |

With Phantom Layer, MedFlow becomes the **only end-to-end pipeline** from patient conversation to researcher query — with compliance certification at every step.
