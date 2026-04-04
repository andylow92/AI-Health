# MedFlow: AI-Powered Hospital Intake & Compliance-Certified Research Data

> **In March 2025, the EU enacted the European Health Data Space — binding law that mandates secure processing environments for health data secondary use and unlocks 450 million patient records for AI research. MedFlow is the infrastructure layer EHDS requires. Every hospital intake becomes a compliance-certified research record, automatically. Hospitals get free intake automation and a new revenue stream. Researchers get compliant data. The regulation created the market. We built the infrastructure.**

**HSIL Hackathon 2026 – Madrid Hub** | April 10–11 | Maria Zambrano Library

---

## Problem

Clinical intake is manual, slow, and error-prone. Patients repeat the same information multiple times. The resulting data lives in silos — clinical systems can't share it, and research teams can't access it because of privacy constraints.

Billions of clinical encounters generate data that is effectively lost to research. With EHDS becoming binding law, hospitals now face a legal obligation to enable compliant secondary data use — without the infrastructure to do it.

**Hospitals are drowning in forms while researchers are starving for data. EHDS mandates a solution. The infrastructure doesn't exist yet.**

---

## Solution

MedFlow is a conversational AI agent that conducts patient intake, auto-fills standardized clinical forms via structured LLM output, routes low-confidence entries to a clinician for validation, and feeds every validated record into an anonymized research database — with no raw patient data ever stored.

A compliance gate validates every anonymized record before it enters the research DB. Records that fail are quarantined and re-anonymized. Every decision is logged to an immutable audit trail.

### Architecture

![MedFlow Architecture V2](./Medflow%20Architecture%20V2.svg)

### User Journey

1. Patient opens MedFlow chat and starts an intake conversation
2. AI agent asks structured questions, adapts follow-ups, and fills a clinical form in real time
3. High-confidence fields are auto-approved; low-confidence fields route to a clinician dashboard
4. Clinician reviews flagged fields, corrects if needed, approves
5. Validated form is anonymized immediately — PII stripped before storage
6. Anonymized record passes through the compliance gate (regex + supervised judge + LLM fallback)
7. Compliant records enter the research DB; non-compliant records are quarantined and re-anonymized
8. Every compliance decision is logged to the audit trail
9. Researcher queries the anonymized DB and gets compliant, structured data

---

## Business Model

MedFlow flips the traditional dynamic: **hospitals do not pay — they get paid.**

| Layer | Customer | Value |
| --- | --- | --- |
| **Free intake software** | Hospitals | AI intake agent at no cost, in exchange for contributing anonymized records to the shared research DB |
| **Revenue share** | Hospitals | Share of revenue generated from queries against their anonymized data |
| **Research DB access** | Pharma, biotech, research institutes, public health agencies | Pay-per-query or subscription access to the compliance-certified, multi-hospital anonymized database |
| **P4P models** | Pharma + insurers | Pay-for-performance arrangements for reimbursement-related collaborative data use (validated in EHDS research) |
| **SaaS premium tier** | Hospitals (optional) | Advanced analytics, EHR integration, custom compliance reports |

### Why Hospitals Say Yes

1. **Free intake automation** — staff time savings, fewer errors, better data quality, reduced compliance risk
2. **A new passive revenue stream** from data they were already generating but couldn't legally monetize

The EHDS creates the legal framework. MedFlow provides the technical infrastructure.

### Who Pays

Pharmaceutical companies, biotech firms, academic research institutions, and public health agencies need EHDS-compliant access to real-world health data. They face high regulatory, technical, and organizational barriers to accessing it independently. MedFlow removes those barriers and charges for access.

---

## Market Opportunity

### Total Addressable Market

- **450 million** EU citizens covered under EHDS
- **~15,000** hospitals across the EU-27
- **~1 billion** clinical encounters per year in EU hospitals
- The global Real-World Evidence (RWE) market is valued at **$2.2B** (2024) and projected to reach **$5B+** by 2030

### Why Now

The EHDS (Regulation (EU) 2025/327) entered into force in March 2025. Key implementing acts are due by March 2027, with major provisions for EHR systems applying from March 2029. The compliance infrastructure gap is open now — hospitals need to prepare, and the vendors who establish trust early will own the market.

### Comparable Exit

Flatiron Health aggregated oncology records from hospitals, monetized access to pharma, and was acquired by Roche for **$1.9B**. MedFlow applies the same playbook across all specialties, with a stronger regulatory mandate.

### Competitive Landscape

| Incumbent | Gap MedFlow Fills |
| --- | --- |
| **EHR vendors** (Epic, Cerner) | Store raw data, don't anonymize or monetize secondary use. Anonymization is not their core business. |
| **Health data exchanges** (IQVIA, Truven) | Rely on batch retrospective data. Not integrated into intake. Not built for EHDS compliance. |
| **Research data platforms** (TriNetX, Flatiron) | Specialty-specific. Require manual data curation. Don't automate intake or offer real-time anonymization. |
| **Hospital AI intake tools** | Focus only on intake efficiency. Don't produce research-ready data. No compliance certification layer. |

MedFlow is the only solution that combines **intake automation + inline anonymization + compliance certification + research monetization** in a single pipeline.

---

## How Anonymization Works

Traditional systems store raw patient data and produce anonymized copies on demand. This creates two problems: the raw data is a high-value breach target, and anonymization is an afterthought.

**MedFlow inverts this.** All data is anonymized at the point of intake validation. There is no persistent store of raw patient records.

### What Gets Anonymized

All 18 HIPAA Safe Harbor identifiers and GDPR personal data categories — names, geographic data, dates, phone/fax/email, SSN/DNI, medical record numbers, health plan IDs, account numbers, license numbers, device/vehicle identifiers, URLs, IPs, biometrics, photographs, plus GDPR additions (location data, online identifiers, health/genetic/biometric data, racial/ethnic origin).

### Three-Layer Compliance Gate

Every record must pass all three layers before entering the research database:

1. **Regex rules** — fast scan for structured patterns (phone numbers, emails, ID formats, postal codes). Deterministic, zero cost.
2. **Supervised judge** — trained classifier scores the full anonymized text for residual PII risk. Catches contextual and quasi-identifier leaks that regex misses.
3. **LLM fallback** — for records near the decision boundary, a general-purpose LLM provides a third opinion. Runs only on edge cases, keeping costs low.

Any failure at any layer sends the record to quarantine for re-anonymization.

### Why a Supervised Judge?

| Property | General LLM | Regex | Supervised Judge |
| ---- | ---- | ---- | ---- |
| Catches structured PII | Yes | Yes | Yes |
| Catches contextual PII | Sometimes | No | Yes |
| Deterministic | No | Yes | Yes |
| Fast at inference | Slow | Fast | Fast |
| Auditable decision boundary | Hard | Easy | Easy |
| Cost per record | High | ~Zero | ~Zero |
| Improves with data | No | No | Yes |

---

## Where AI Is Used

| Component | AI Technology | Purpose |
| --- | --- | --- |
| Conversational intake | LLM (Claude) with structured output | Form-filling from natural conversation |
| Confidence scoring | LLM-based assessment | Route low-confidence fields to human review |
| PII detection | NER model + LLM | Strip identifiers from structured and free-text fields |
| Compliance judge | Supervised classifier | Score anonymized records for residual PII risk |
| LLM fallback judge | General-purpose LLM | Edge-case second opinions on borderline records |
| Eval framework | Automated scoring | Agent accuracy, anonymization completeness, compliance gate precision/recall |

---

## Users

| User | Interface |
| --- | --- |
| **Patients** | Chat interface for intake |
| **Clinicians** | Review dashboard for flagged low-confidence cases |
| **Researchers** | Query interface over the anonymized research database |

---

## Madrid Hub Alignment

MedFlow addresses three of the four Madrid Hub challenge areas:

- **Hospital care (primary):** Transforms the intake process — faster, structured, lower-error intake across every department.
- **Public health in urban environments:** At scale, the anonymized DB becomes a real-time population health surveillance layer — outbreak patterns, chronic disease prevalence by district, pollution impact on respiratory admissions.
- **Emergency management:** The intake form supports *urgencias* workflows. Structured AI triage in emergency settings accelerates time-to-treatment. (Our intake model is based on the [standard Spanish hospital admission form](./formulario_preguntas.md).)

---

## What Makes MedFlow Different

- **Anonymization-first, not anonymization-after** — no database of raw patient records to breach
- **Supervised judge for compliance** — deterministic, auditable, improvable quality control
- **Three-layer defense** — regex + supervised judge + LLM fallback; any failure quarantines the record
- **Continuous improvement loop** — every manual review becomes training data for the judge
- **Audit trail as a feature** — institutions can prove to regulators exactly what was checked, when, and what the result was
- **Research as a byproduct** — every intake conversation automatically produces a compliance-certified research datapoint

---

## Demo Highlights

### Power Move

During the pitch, we show a record with a deliberately leaked phone number. The compliance gate catches it, the record goes to quarantine, it gets re-anonymized, and now it passes. This makes the compliance layer tangible for judges.

### What We Build vs. Mock

**Built end-to-end:**
- Conversational AI agent with structured form output
- Confidence scoring and human-in-the-loop routing
- Clinician review dashboard
- Anonymization pipeline (NER-based PII stripping)
- Compliance gate (regex + judge + quarantine loop)
- Audit trail logging every compliance decision
- Anonymized research database (no raw patient data persisted)
- Eval framework with test suite

**Mocked / roadmap:**
- EHR integration (API contract shown, not connected)
- Voice input (text chat for demo; voice on roadmap)
- Multi-language support (English demo; Spanish on roadmap)
- Large-scale queries (SQLite with 10-20 synthetic records)
- Production supervised judge (LLM-based for prototype; supervised model is the production path)

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | React |
| Backend | FastAPI |
| LLM | Claude API (structured output) |
| Database | SQLite (prototype) / Postgres (production) |
| Anonymization | spaCy NER + LLM pass |
| Compliance judge | HuggingFace Transformers (BERT-base / DeBERTa-v3-small) |
| Audit trail | Append-only log table |

---

## Team

| Role | Focus Area | Hackathon Responsibility |
| ---- | ---------- | ------------------------ |
| Platform / DevOps | Infrastructure, CI/CD, pipelines | Anonymization engine, compliance gate, quarantine logic, audit trail, DB schema |
| Solutions Architect | Backend APIs, system design, LLM integrations | AI agent, structured output, confidence scoring, supervised judge, eval framework |
| Systems Engineer | Frontend, dashboards, demo tooling | Patient chat UI, clinician dashboard, compliance panel, pitch deck |
| Sales / BD | Go-to-market, partnerships, EHDS regulatory context | Business model, pitch narrative, stakeholder mapping |

---

## Getting Started

```bash
# Clone the repository
git clone https://github.com/andylow92/ai-health.git
cd ai-health

# Setup instructions (coming during hackathon)
# ...
```

---

## Project Structure

```
├── readme.MD                       # This file
├── formulario_preguntas.md         # Standard Spanish hospital intake form (reference)
├── Medflow Architecture V2.svg     # System architecture diagram
└── PLANNING.md                     # Internal team planning (workstreams, pre-hackathon tasks)
```

---

## License

TBD
