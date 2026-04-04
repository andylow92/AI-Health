# MedFlow — Internal Planning

> This document is for team coordination. See [readme.MD](./readme.MD) for the project overview.

---

## Workstreams

| Workstream | Owner | Support | Notes |
| ---- | ---- | ---- | ---- |
| Problem research | All | — | Pre-hackathon |
| Clinical logic / health validation | Person 1 | Person 2 | Standard intake form templates |
| AI / model workflow (agent + evals) | Person 2 | Person 1 | LLM API, structured output, confidence scoring |
| Anonymization pipeline | Person 1 | Person 2 | NER, compliance gate, DB schema |
| Supervised judge (compliance) | Person 2 | Person 1 | Training data, model selection, threshold tuning |
| Audit trail | Person 1 | Person 2 | Immutable log, quarantine logic |
| Product / UX | Person 3 | — | Chat UI, dashboard, compliance panel, research explorer |
| Front-end prototype | Person 3 | Person 2 | React |
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

* **Supervised judge candidates:** BERT-base, DeBERTa-v3-small, or a distilled model fine-tuned on PII detection. Input: anonymized text. Output: risk score + flagged spans. Training set: 50-100 synthetic examples (clean + deliberately leaked) should be enough for a prototype.
* **Compliance gate architecture:** regex layer (structured identifiers) → supervised judge (contextual PII) → LLM fallback (edge cases only). All three must pass.
* **Audit trail schema:** `record_id`, `timestamp`, `check_type` (regex/judge/llm), `result` (pass/fail), `risk_score`, `identifiers_found` (list), `action_taken` (approved/quarantined/re-anonymized), `reviewer` (system/human)
* **Eval strategy:** Build a test suite of 10-20 synthetic patient cases with known correct form outputs. Include 5+ cases with deliberately incomplete anonymization to test compliance gate catch rate. Score field-level accuracy, track confidence calibration, and measure compliance gate precision/recall.

---

## Hackathon Prototype Notes

For the prototype, the supervised judge can start as an LLM-based classifier (faster to set up, no training needed). The key is to demonstrate the architecture: anonymize → judge → quarantine → re-anonymize. The supervised model is the production path — frame it as the evolution that makes the system reliable at scale. If there's time pre-hackathon, a small fine-tuned model on the synthetic test cases would make an excellent demo talking point.

---

## EHDS Regulatory Detail

On 15 March 2024, the EU institutions reached a provisional political agreement on the European Health Data Space (EHDS) in trilogue. The European Parliament voted the agreed text on 24 April 2024, and the Council formally adopted it on 21 January 2025. The regulation — **Regulation (EU) 2025/327** — was published in the Official Journal and entered into force in **March 2025**. Implementation is staged: key implementing acts are due by March 2027, with major provisions for EHR systems and primary care applying from March 2029.

As the first domain-specific regulation of its kind, the EHDS actively promotes the secondary use of already collected health data for AI applications. Through an opt-out procedure or sovereign consent, health data from more than 450 million European citizens can become available for use. A basic prerequisite for any use of such data is that it must be processed in **secure processing environments**, as defined in Article 50 of the EHDS.

MedFlow is an Article 50-compliant secure processing environment by design — not retrofitted compliance, purpose-built for the regulation.

Organizations seeking to use health data within the EHDS — including pharmaceutical companies, medical technology companies, insurers, and research institutes — face regulatory, technical, and organizational barriers. Existing business models validated through academic research include pay-for-performance (P4P) for pharmaceutical companies in collaborative data use with health insurers for reimbursement-related purposes.
