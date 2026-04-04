# MedFlow: AI Intake Agent + Compliance-Certified Anonymized Research Database

An AI agent that turns every patient intake conversation into a validated clinical form and a compliance-certified, research-ready anonymized record — automatically.

**HSIL Hackathon 2026 – Madrid Hub**

* **Theme:** Building High-Value Health Systems: Leveraging Artificial Intelligence
* **Dates:** April 10–11
* **Location:** Maria Zambrano Library
* **Team Size:** 4 people (3 engineers + 1 sales)

---

## Regulatory Tailwind: European Health Data Space (EHDS)

On 15 March 2024, the EU institutions reached a provisional political agreement on the European Health Data Space (EHDS) in trilogue. The European Parliament voted the agreed text on 24 April 2024, and the Council formally adopted it on 21 January 2025. The regulation — **Regulation (EU) 2025/327** — was published in the Official Journal and entered into force in **March 2025**. Implementation is staged: key implementing acts are due by March 2027, with major provisions for EHR systems and primary care applying from March 2029. The market this creates is already open.

As the first domain-specific regulation of its kind, the EHDS actively promotes the secondary use of already collected health data for AI applications. Through an opt-out procedure or sovereign consent, health data from more than **450 million European citizens** can become available for use. A basic prerequisite for any use of such data is that it must be processed in **secure processing environments**, as defined in Article 50 of the EHDS.

**MedFlow is an Article 50-compliant secure processing environment by design — not retrofitted compliance, purpose-built for the regulation.**

Organizations seeking to use health data within the EHDS — including pharmaceutical companies, medical technology companies, insurers, and research institutes — face regulatory, technical, and organizational barriers. Existing business models validated through academic research include pay-for-performance (P4P) for pharmaceutical companies in collaborative data use with health insurers for reimbursement-related purposes. MedFlow's architecture is designed to support exactly these models at scale.

> "We're not building ahead of the market. The EU just legislated this market into existence."

---

## Problem

Clinical intake is manual, slow, and error-prone. Patients repeat the same information multiple times. The resulting data lives in silos — clinical systems can't share it, and research teams can't access it because of privacy constraints. Anonymizing data for research is expensive and done ad hoc.

Billions of clinical encounters generate data that is effectively lost to research. Manual intake creates bottlenecks in every hospital. Poor data quality propagates downstream into clinical decisions. And with EHDS becoming binding law in 2026, hospitals now face a legal obligation to enable compliant secondary data use — without the infrastructure to do it.

**Hospitals are drowning in forms while researchers are starving for data. EHDS mandates a solution. The infrastructure doesn't exist yet.**

---

## Solution Overview

MedFlow is a conversational AI agent that conducts patient intake, auto-fills standardized clinical forms via structured LLM output, routes low-confidence entries to a human clinician for validation, and feeds every validated record into an anonymized research database — with no raw patient data ever stored.

A compliance gate validates every anonymized record before it enters the research DB. Records that fail are quarantined and re-anonymized. Every decision is logged to an immutable audit trail. An eval framework monitors agent quality continuously.

### Architecture

![MedFlow Architecture V2](./Medflow%20Architecture%20V2.svg)

---

## Anonymization-First Design

Traditional systems store raw patient data and then produce anonymized copies on demand. This creates two problems: (1) the raw data is a high-value target for breaches, and (2) anonymization is treated as an afterthought rather than a guarantee.

MedFlow inverts this. All data is anonymized at the point of intake validation. There is no persistent store of raw patient records. The anonymization pipeline is not a batch job that runs later — it is part of the core data flow, inline, before any write.

### What Gets Anonymized

All 18 HIPAA Safe Harbor identifiers and GDPR personal data categories:

* Names, geographic data smaller than state, dates (except year), phone numbers, fax numbers, email addresses, SSN/DNI, medical record numbers, health plan IDs, account numbers, certificate/license numbers, vehicle identifiers, device identifiers, URLs, IP addresses, biometric identifiers, full-face photographs, any other unique identifying code
* GDPR additions: location data, online identifiers, health/genetic/biometric data, racial/ethnic origin

### How Anonymization Works

The anonymization pipeline has two layers:

1. **NER + LLM stripping:** A named entity recognition model (spaCy or LLM-based) identifies and removes PII from structured fields and free-text. An LLM pass handles contextual identifiers that NER misses (e.g., "the baker on Elm Street" implies a specific person).
2. **Compliance gate (supervised judge):** A second-pass validation that scores the anonymized output for residual PII risk. This is where the supervised model as a judge comes in.

---

## Supervised Model as a Judge

The compliance gate is the most critical component of MedFlow. If it fails, PII leaks into the research database. A general-purpose LLM prompted to "check for PII" is unreliable for this — it's non-deterministic, expensive per call, and hard to audit. A pure regex approach catches structured patterns but misses contextual leaks.

The solution: a supervised classifier trained specifically to judge anonymization quality.

### Why a Supervised Judge?

| Property | General LLM | Regex rules | Supervised judge |
| ---- | ---- | ---- | ---- |
| Catches structured PII (phone, SSN) | Yes | Yes | Yes |
| Catches contextual PII ("the teacher on Elm St") | Sometimes | No | Yes (trained on examples) |
| Deterministic | No | Yes | Yes |
| Fast at inference | Slow | Fast | Fast |
| Produces confidence score | No (or unreliable) | No | Yes |
| Auditable decision boundary | Hard | Easy | Easy (threshold + features) |
| Cost per record | High | Near zero | Near zero |
| Improves with more data | Not directly | No | Yes |

A supervised model gives you the contextual understanding of an LLM with the speed, determinism, and auditability of a rule-based system. For a compliance-critical component, these properties matter more than raw capability.

### How It Works

1. **Training data:** Synthetic anonymized records, some properly anonymized, some with deliberately leaked PII (direct identifiers, quasi-identifiers, contextual leaks). Each labeled as "clean" or "leaking" with the specific identifiers that leaked. This dataset also serves as the eval test suite.
2. **Model:** A fine-tuned text classifier (e.g., BERT-base, DeBERTa, or a distilled model). Input is the anonymized text; output is a PII-risk score between 0 and 1, plus flagged spans.
3. **Threshold:** Records scoring below the risk threshold (e.g., < 0.05) pass into the research DB. Records above are quarantined.
4. **Quarantine loop:** Quarantined records are re-anonymized (with the flagged spans as hints) and re-judged. After N failures, they go to manual review.
5. **Continuous improvement:** Every manual review decision becomes a new training example. The judge gets better over time as edge cases accumulate.

### For the Hackathon

For the prototype, the judge can start as an LLM-based classifier (faster to set up, no training needed). The key is to demonstrate the architecture: anonymize, judge, quarantine, re-anonymize. The supervised model is the production path — frame it as the evolution that makes the system reliable at scale. If there's time pre-hackathon, a small fine-tuned model on the synthetic test cases would make an excellent demo talking point.

---

## Three-Layer Defense

In production, MedFlow's compliance gate runs three layers in sequence:

1. **Regex rules** — fast scan for structured patterns (phone numbers, emails, ID formats, dates of birth, postal codes). Deterministic, zero cost, catches the obvious cases.
2. **Supervised judge** — trained classifier scores the full anonymized text for residual PII risk. Catches contextual and quasi-identifier leaks that regex misses.
3. **LLM fallback** — for records near the decision boundary (e.g., judge score between 0.03 and 0.07), a general-purpose LLM provides a third opinion. This layer runs only on edge cases, keeping costs low.

All three layers must pass before a record enters the research database. Any failure at any layer sends the record to quarantine.

---

## Where AI Is Used

* **Conversational intake** — LLM with structured output for form-filling
* **Confidence scoring** — LLM-based assessment of extraction certainty, routing low-confidence fields to human review
* **PII detection and anonymization** — NER model + LLM for stripping identifiers from structured and free-text fields
* **Compliance judge** — Supervised classifier scoring anonymized records for residual PII risk
* **LLM fallback judge** — General-purpose LLM for edge-case second opinions on borderline records
* **Eval framework** — Automated scoring of agent accuracy, anonymization completeness, and compliance gate precision/recall

---

## Users

* **Patients** — chat interface for intake
* **Clinicians** — review dashboard for flagged low-confidence cases
* **Researchers** — query interface over the anonymized research database

---

## Demo Plan

### User Journey

1. Patient opens the MedFlow chat interface and starts an intake conversation
2. The AI agent asks structured questions, adapts follow-ups based on answers, and fills a clinical form in real time
3. High-confidence fields are auto-approved; low-confidence fields are routed to a clinician dashboard with the agent's reasoning
4. Clinician reviews flagged fields, corrects if needed, and approves
5. Validated form is anonymized immediately — PII stripped before storage
6. Anonymized record passes through the compliance gate (regex + supervised judge)
7. Compliant records enter the research DB; non-compliant records are quarantined and re-anonymized
8. Every compliance decision is logged to the audit trail
9. Researcher queries the anonymized DB and sees the new record (without PII)

### Demo Power Move

During the pitch, show a record with a deliberately leaked phone number. The compliance gate catches it, the record goes to quarantine, it gets re-anonymized, and now it passes. This makes the compliance layer tangible for judges.

### Demo Components

* **Front end:** Patient chat UI, clinician review dashboard, compliance status panel, audit trail viewer, research DB explorer
* **AI component:** LLM API (Claude) with structured output for form-filling, confidence scoring, NER-based PII detection, supervised judge for compliance validation
* **Data input:** Synthetic patient scenarios (pre-scripted + free-form)
* **Output:** Auto-filled clinical form, flagged-field alerts, compliance pass/fail status with reasoning, quarantine queue, audit trail log, anonymized record viewer

### What We Actually Build vs. Mock

**Actually build:**

* Conversational AI agent with structured form output (LLM API)
* Confidence scoring and human-in-the-loop routing logic
* Clinician review dashboard (approve / edit flagged fields)
* Anonymization pipeline (NER-based PII stripping)
* Compliance gate (supervised judge or LLM-based judge for prototype + regex rules)
* Quarantine queue for non-compliant records with re-anonymization loop
* Audit trail logging every compliance decision (pass, fail, retry, manual override)
* Anonymized research database (single store — no raw patient data persisted)
* Eval framework: accuracy scoring, failure tracking, test suite
* End-to-end data flow from chat to form to anonymization to compliance gate to database

**Mock or simulate:**

* EHR integration (show the API contract, don't connect to a real EHR)
* Voice input (text chat is sufficient; mention voice as roadmap)
* Multi-language support (demo in English; mention Spanish as roadmap)
* Large-scale research DB queries (use SQLite with 10-20 synthetic records)
* Full legal review of compliance rules (implement the technical checks; note that production needs legal sign-off)
* Production supervised judge (use LLM-based judge for prototype; frame supervised model as production evolution)

---

## Business Model

MedFlow's model flips the traditional dynamic: **hospitals do not pay — they get paid.**

### How It Works

| Layer | Who | What |
| --- | --- | --- |
| **Free intake software** | Hospitals | MedFlow provides the AI intake agent at no cost in exchange for contributing anonymized records to the shared research database |
| **Revenue share** | Hospitals | Hospitals receive a share of revenue generated from queries against their anonymized data |
| **Research DB access** | Pharma, biotech, research institutes, public health agencies | Pay-per-query or subscription access to the compliance-certified, multi-hospital anonymized database |
| **P4P models** | Pharma + insurers | Pay-for-performance arrangements for reimbursement-related collaborative data use, as validated in existing EHDS research |
| **SaaS premium tier** | Hospitals (optional) | Advanced analytics dashboard, EHR integration, custom compliance reports for regulators |

### Why Hospitals Say Yes

They get two things they don't have today:
1. **Free intake automation** — staff time savings, fewer errors, better data quality, reduced compliance risk
2. **A new passive revenue stream** from data they were already generating but couldn't legally monetize

The EHDS creates the legal framework that makes the revenue share possible. MedFlow provides the technical infrastructure that makes it compliant.

### The Paying Customers

Pharmaceutical companies, biotech firms, academic research institutions, and public health agencies need EHDS-compliant access to real-world health data. They face high regulatory, technical, and organizational barriers to accessing it independently. MedFlow removes those barriers and charges for the access.

This is the proven playbook: Flatiron Health aggregated oncology records from hospitals, monetized access to pharma, and was acquired by Roche for $1.9B.

---

## Madrid Hub Alignment

MedFlow addresses three of the four Madrid Hub challenge areas:

**Hospital care (primary):** MedFlow directly transforms the hospital intake process — the entry point of every clinical encounter. Faster, structured, lower-error intake improves care quality and reduces administrative burden across every hospital department.

**Public health in urban environments:** When MedFlow scales across Madrid hospitals, the anonymized research database becomes a real-time population health surveillance layer for the city. Outbreak patterns, chronic disease prevalence by district, and the health impact of urban pollution on respiratory admissions all become automatically detectable — from data that was always being generated but never accessible.

**Emergency management:** The intake form is explicitly designed for *urgencias* (emergency) workflows. Structured AI-assisted triage in emergency settings accelerates time-to-treatment and produces better-quality clinical records under pressure.

---

## What Makes MedFlow Different

* **Anonymization-first, not anonymization-after:** PII is stripped before any data hits persistent storage. There is no database of raw patient records to breach.
* **Supervised judge for compliance:** Instead of trusting a general-purpose LLM or simple regex to catch PII leaks, a trained classifier provides deterministic, auditable, and improvable quality control.
* **Three-layer defense:** Regex + supervised judge + LLM fallback. Any failure at any layer quarantines the record.
* **Continuous improvement loop:** Every manual review of a quarantined record becomes training data for the judge. The system gets better with use.
* **Audit trail as a feature:** Every compliance decision is logged. Institutions can prove to regulators exactly what was checked, when, and what the result was.
* **Research as a byproduct:** Every patient intake conversation automatically produces a compliance-certified research datapoint. No separate data collection effort needed.

**Pitch angle:** "In March 2024, the EU agreed the European Health Data Space — binding law by 2026. It mandates secure processing environments for health data secondary use and unlocks 450 million patient records for AI research. MedFlow is the infrastructure layer EHDS requires. Every hospital intake becomes a compliance-certified research record, automatically. Hospitals get free intake automation and a new revenue stream. Researchers get compliant data. The regulation created the market. We built the infrastructure."

---

## Team

| Name | Background | Skills | Preferred Role | Availability |
| ---- | ---------- | ------ | -------------- | ------------ |
| TBD  | DevOps / Platform Engineering | Infrastructure, CI/CD, cloud, pipelines | Data pipeline & anonymization engine | Full both days |
| TBD  | Software / Solutions Architecture | Backend APIs, system design, LLM integrations | AI agent & eval framework | Full both days |
| TBD  | Systems / Software Engineering | Frontend, dashboards, demo tooling | Demo UI & pitch | Full both days |
| TBD  | Sales / Business Development | Go-to-market, hospital and pharma partnerships, EHDS regulatory context | Business model, pitch narrative, stakeholder mapping | Full both days |

### Roles

* **Person 1 (Platform / DevOps):** Data pipeline owner — anonymization engine, compliance gate, quarantine logic, audit trail, database schema, infrastructure.
* **Person 2 (Solutions Architect):** AI agent owner — LLM integration, structured output, confidence scoring, supervised judge training/setup, eval framework.
* **Person 3 (Systems Engineer):** Demo owner — patient chat UI, clinician dashboard, compliance status panel, audit trail viewer, research DB explorer, pitch deck.
* **Person 4 (Sales / BD):** Business model owner — hospital and pharma go-to-market strategy, EHDS regulatory narrative, stakeholder mapping, pitch delivery.

### Workstreams

| Workstream | Owner | Support | Notes |
| ---- | ---- | ---- | ---- |
| Problem research | All | — | Pre-hackathon |
| Clinical logic / health validation | Person 1 | Person 2 | Standard intake form templates |
| AI / model workflow (agent + evals) | Person 2 | Person 1 | LLM API, structured output, confidence scoring |
| Anonymization pipeline | Person 1 | Person 2 | NER, compliance gate, DB schema |
| Supervised judge (compliance) | Person 2 | Person 1 | Training data, model selection, threshold tuning |
| Audit trail | Person 1 | Person 2 | Immutable log, quarantine logic |
| Product / UX | Person 3 | — | Chat UI, dashboard, compliance panel, research explorer |
| Front-end prototype | Person 3 | Person 2 | React or HTML |
| Business model & EHDS narrative | Person 4 | All | Hospital + pharma go-to-market, regulatory context |
| Stakeholder & market mapping | Person 4 | — | Hospital procurement, pharma buyers, EHDS timelines |
| Pitch deck | Person 3 | Person 4 | Architecture diagram, demo script, impact story, business model slide |
| Final presentation | All | — | Practice run evening of Day 1 |

---

## Pre-Hackathon Action List

| Task | Owner | Deadline | Status |
| ---- | ---- | ---- | ---- |
| Finalize team members | All | 1 week before | -- |
| Define synthetic patient scenarios (5-10 cases) | Person 2 | 3 days before | -- |
| Choose clinical form template (ED intake or general consult) | Person 1 | 3 days before | -- |
| Set up repo + project board | Person 1 | 1 week before | -- |
| Prototype LLM structured output prompt | Person 2 | 3 days before | -- |
| Prototype NER anonymization pass | Person 1 | 3 days before | -- |
| Build HIPAA Safe Harbor 18-identifier checklist as validation rules | Person 1 | 3 days before | -- |
| Build GDPR personal data category checklist as validation rules | Person 1 | 3 days before | -- |
| Create test cases with deliberately leaked PII (to test compliance gate) | Person 2 | 2 days before | -- |
| Label training data for supervised judge (clean vs. leaking examples) | Person 2 | 2 days before | -- |
| Prototype supervised judge or LLM-based judge fallback | Person 2 | 2 days before | -- |
| Draft pitch storyline (with anonymization-first + supervised judge angle) | Person 3 | 2 days before | -- |
| Prepare design references for UI (including compliance panel + audit trail) | Person 3 | 2 days before | -- |
| Set up eval test cases | Person 2 | 2 days before | -- |
| Choose tech stack (frontend framework, DB, hosting) | All | 1 week before | -- |

---

## Technical Notes

* **Tech stack candidates:** React (frontend), FastAPI or Node (backend), Claude API (LLM), SQLite or Postgres (DB), spaCy or LLM-based NER (anonymization), HuggingFace Transformers (supervised judge)
* **Supervised judge candidates:** BERT-base, DeBERTa-v3-small, or a distilled model fine-tuned on PII detection. Input: anonymized text. Output: risk score + flagged spans. Training set: 50-100 synthetic examples (clean + deliberately leaked) should be enough for a prototype.
* **Compliance gate architecture:** regex layer (structured identifiers) -> supervised judge (contextual PII) -> LLM fallback (edge cases only). All three must pass.
* **Audit trail schema:** record_id, timestamp, check_type (regex/judge/llm), result (pass/fail), risk_score, identifiers_found (list), action_taken (approved/quarantined/re-anonymized), reviewer (system/human)
* **Eval strategy:** Build a test suite of 10-20 synthetic patient cases with known correct form outputs. Include 5+ cases with deliberately incomplete anonymization to test compliance gate catch rate. Score field-level accuracy, track confidence calibration, and measure compliance gate precision/recall.
