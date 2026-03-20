# HSIL Hackathon 2026 – Team Idea Workspace

A shared space for our team to collect, compare, and refine ideas for the **HSIL Hackathon 2026 – Madrid Hub**.

## Event Overview

* **Theme:** Building High-Value Health Systems: Leveraging Artificial Intelligence
* **Dates:** April 10–11
* **Location:** María Zambrano Library
* **Team Size:** 3 people
* **Format:** Two-day, in-person hackathon

The goal is to move from an initial concept to a functional prototype or operational product during the event.

## Why This Repo Exists

We'll use this workspace to:

* Capture problem areas we care about
* Brainstorm solution ideas
* Evaluate ideas based on impact and feasibility
* Select one final concept
* Assign roles and next steps before the hackathon

## Sample Challenge Areas

* Remote patient monitoring
* Public health in urban settings and city organization
* Childhood mental health
* Hospital care and emergency management

---

## Team Members

| Name | Background | Skills | Preferred Role | Availability |
| ---- | ---------- | ------ | -------------- | ------------ |
| TBD  | DevOps / Platform Engineering | Infrastructure, CI/CD, cloud, pipelines | Data pipeline & anonymization engine | Full both days |
| TBD  | Software / Solutions Architecture | Backend APIs, system design, LLM integrations | AI agent & eval framework | Full both days |
| TBD  | Systems / Software Engineering | Frontend, dashboards, demo tooling | Demo UI & pitch | Full both days |

---

## Problems We Care About

List healthcare or health-system problems we think are urgent, interesting, and suitable for an AI-enabled solution.

### Brain Dump

* Clinical intake is slow, repetitive, and error-prone — patients wait, staff burn out
* Forms are filled manually, often with duplicate data entry across systems
* Patient data is siloed — clinical records can't easily be used for research
* Research datasets require expensive, manual anonymization before they're usable
* When intake data is wrong or incomplete, there's no systematic way to catch it before it enters the system
* Coordination between admin, triage, and clinical staff during intake is fragmented

### Prompts

Think about problems that are:

* Frequent and painful for patients, clinicians, hospitals, or public health systems
* Expensive or inefficient
* Managed today with fragmented data or manual workflows
* Suitable for decision support, risk prediction, triage, automation, monitoring, or communication improvements

---

## Idea Backlog

### Idea 1 – ED Triage Agent

**Problem**
Emergency department triage is high-pressure, inconsistent, and relies on brief human assessments under time constraints. Undertriage risks patient safety; overtriage wastes scarce resources.

**Who is affected?**
Patients arriving at the ED, triage nurses, emergency physicians.

**Why does this matter?**
Mistriage contributes to avoidable adverse events and ED overcrowding. Staff shortage makes the problem worse.

**Proposed solution**
An AI agent that conducts structured intake conversations with patients (or assists triage nurses), asks dynamic follow-up questions based on responses, and suggests a triage priority level with reasoning.

**Where AI is used**
Conversational intake (LLM), risk classification, guideline-aware reasoning (Manchester Triage / ESI).

**Users**
Triage nurses, ED physicians.

**Main benefit**
Faster, more consistent triage. Decision support that catches high-risk patterns a rushed nurse might miss.

**Prototype we could build in 2 days**
Chat interface where a simulated patient answers questions → agent produces a triage level + reasoning summary.

**Data needed**
Triage guidelines (Manchester / ESI), synthetic patient scenarios.

**Main risks / blockers**
Clinical validation, liability for AI-assisted triage decisions, adoption resistance from staff.

**Why our team can build it**
Strong backend and API integration skills. Can wire up LLM agents with structured output quickly.

---

### Idea 2 – Patient Support Agent

**Problem**
Patients constantly call hospitals with routine questions — appointment logistics, pre-visit instructions, post-discharge doubts, medication timing. These calls overwhelm admin staff and nursing lines.

**Who is affected?**
Patients, hospital admin staff, nurses handling phone lines.

**Why does this matter?**
Staff time is wasted on repetitive queries. Patients often can't get through, leading to missed instructions and avoidable readmissions.

**Proposed solution**
An AI-powered patient support agent (chat or voice) that handles common queries, provides personalized pre/post-visit guidance, and escalates complex cases to a human.

**Where AI is used**
Natural language understanding, retrieval-augmented generation over hospital knowledge bases, escalation logic.

**Users**
Patients directly, with oversight from hospital admin.

**Main benefit**
Reduces call volume, improves patient access to information, frees staff for higher-value tasks.

**Prototype we could build in 2 days**
Chat interface answering common patient questions from a mock hospital knowledge base, with escalation trigger.

**Data needed**
Mock FAQ / knowledge base, sample patient scenarios.

**Main risks / blockers**
Hallucination risk for medical information, integration with real hospital systems, patient trust.

**Why our team can build it**
RAG pipelines and API orchestration are in our wheelhouse.

---

### Idea 3 – MedFlow: AI Intake Agent + Anonymized Research Data Lake

**Problem**
Clinical intake is manual, slow, and error-prone. Patients repeat the same information multiple times. The resulting data lives in silos — clinical systems can't share it, and research teams can't access it because of privacy constraints. Anonymizing data for research is expensive and done ad hoc.

**Who is affected?**
Patients (waiting, repeating info), intake staff (manual form-filling), clinicians (incomplete/incorrect data), researchers (no access to structured, anonymized datasets).

**Why does this matter?**
Billions of clinical encounters generate data that is effectively lost to research. Manual intake creates bottlenecks in every hospital. Poor data quality propagates downstream into clinical decisions.

**Proposed solution**
An AI agent that conducts patient intake conversations, auto-fills standardized clinical forms using structured LLM output, routes low-confidence entries to a human clinician for validation (human-in-the-loop), and stores data in two parallel tracks:
1. **Original records** — full patient data for clinical use / EHR integration
2. **Anonymized copy** — PII automatically stripped via NER, written to a unified research database

An eval framework runs continuously, scoring agent accuracy and flagging failure patterns.

**Where AI is used**
- Conversational intake (LLM with structured output)
- Confidence scoring and human-in-the-loop routing
- Named Entity Recognition for PII detection and anonymization
- Eval framework for model quality monitoring

**Users**
- Patients (chat interface for intake)
- Clinicians (review dashboard for flagged cases)
- Researchers (query interface over anonymized data lake)

**Main benefit**
Faster intake, fewer errors, zero manual anonymization, and a growing research dataset that updates automatically with every patient encounter. Turns operational data into a research asset.

**Prototype we could build in 2 days**
End-to-end demo: patient chats with the agent → form auto-fills → low-confidence fields get flagged for human review → validated form is stored as original + anonymized copy appears in the research DB → researcher can query the anonymized data.

**Data needed**
- Synthetic patient profiles and intake scenarios
- Standard clinical form templates (e.g., emergency intake, general consultation)
- NER model or LLM prompt for PII detection

**Main risks / blockers**
- Anonymization completeness (missed PII)
- Schema design for the unified research DB
- Regulatory compliance (GDPR, local health data laws) — out of scope for prototype but important for pitch narrative
- Edge cases in multilingual intake (Spanish / English)

**Why our team can build it**
Three infrastructure and platform engineers. We can build the data pipeline, the anonymization layer, the eval harness, and the agent orchestration. This is a systems problem with an AI layer — exactly our strength. We don't need to train models; we use LLM APIs with structured output and NER.

---

## Idea Scoring

Score each idea from 1 to 5.

| Idea | Importance of Problem | AI Fit | Feasibility in 2 Days | Prototype Potential | Pitch Strength | Total |
| ---- | --------------------: | -----: | --------------------: | ------------------: | -------------: | ----: |
| Idea 1 – ED Triage Agent | 5 | 4 | 3 | 3 | 4 | 19 |
| Idea 2 – Patient Support Agent | 3 | 3 | 4 | 3 | 3 | 16 |
| Idea 3 – MedFlow | 5 | 5 | 4 | 5 | 5 | 24 |

### Scoring Criteria

* **Importance of Problem:** Is this a meaningful health-system issue?
* **AI Fit:** Does AI add real value rather than being forced into the solution?
* **Feasibility in 2 Days:** Can we build a convincing demo quickly?
* **Prototype Potential:** Can judges clearly understand and experience it?
* **Pitch Strength:** Is the story compelling, specific, and memorable?

---

## Top 3 Concepts

### 1.

**Name:** MedFlow
**One-line summary:** AI intake agent that auto-fills clinical forms, routes uncertain cases to humans, and automatically generates an anonymized research database from every encounter.
**Why it stands out:** Solves two problems at once (intake efficiency + research data access), plays to our infra strengths, demo is visually clear and end-to-end.

### 2.

**Name:** ED Triage Agent
**One-line summary:** Conversational AI that conducts dynamic patient intake and suggests triage priority with clinical reasoning.
**Why it stands out:** High-stakes problem, strong AI fit, but harder to validate clinically in 2 days.

### 3.

**Name:** Patient Support Agent
**One-line summary:** AI chatbot that handles routine patient queries and escalates complex cases to staff.
**Why it stands out:** Easy to build, but less differentiated — many teams will do chatbots.

---

## Final Selected Concept

**Project Name:** MedFlow

**One-sentence pitch:** An AI agent that turns every patient intake conversation into a validated clinical form and a research-ready anonymized record — automatically.

**Problem statement:** Clinical intake is manual, slow, and error-prone. The data it generates is siloed and inaccessible for research because anonymization is expensive and ad hoc. Hospitals are drowning in forms while researchers are starving for data.

**Target user:** Patients (intake), clinicians (validation), researchers (data access).

**Solution summary:** A conversational AI agent conducts patient intake, auto-fills standardized clinical forms via structured LLM output, routes low-confidence entries to a human reviewer, and stores every validated record in two parallel tracks: original (for clinical use) and anonymized (for a unified research database). An eval framework monitors agent quality continuously.

**What the demo will show:** A patient chatting with the agent → form auto-populating in real time → a flagged field being sent to the clinician dashboard → the clinician approving it → the validated record appearing in both the clinical store and the anonymized research DB → a researcher querying the research DB.

**What makes it innovative:** Most intake tools stop at digitizing forms. MedFlow treats every intake as a data pipeline — clinical value AND research value from a single conversation, with built-in quality control and privacy by design.

**Why it matters for health systems:** Reduces intake time and errors (operational value), eliminates manual anonymization (cost savings), creates a continuously growing research dataset (scientific value), and builds trust through human-in-the-loop validation and eval transparency.

**Expected impact:** Faster patient flow, fewer data entry errors, lower anonymization costs, and a unified research database that grows with every patient encounter instead of requiring separate data collection efforts.

---

## Demo Plan

### User Journey

1. Patient opens the MedFlow chat interface and starts an intake conversation
2. The AI agent asks structured questions, adapts follow-ups based on answers, and fills a clinical form in real time
3. High-confidence fields are auto-approved; low-confidence fields are routed to a clinician dashboard with the agent's reasoning
4. Clinician reviews flagged fields, corrects if needed, and approves
5. Validated form is stored as original record AND anonymized copy appears in the research DB
6. Researcher queries the anonymized DB and sees the new record (without PII)

### Demo Components

* **Front end / interface:** Patient chat UI (React or simple HTML), clinician review dashboard, research DB explorer
* **AI component:** LLM API (Claude or GPT) with structured output for form-filling, confidence scoring prompt, NER-based PII detection for anonymization
* **Data input:** Synthetic patient scenarios (pre-scripted + free-form)
* **Output / dashboard / recommendation:** Auto-filled clinical form, flagged-field alerts, anonymized record viewer, eval metrics dashboard
* **Pitch deck visuals:** Architecture diagram, live demo walkthrough, before/after comparison of manual vs MedFlow intake

### What We Can Fake vs. What We Should Actually Build

## **Actually build:**

* Conversational AI agent with structured form output (LLM API)
* Confidence scoring and human-in-the-loop routing logic
* Clinician review dashboard (approve / edit flagged fields)
* Anonymization pipeline (NER-based PII stripping)
* Dual storage: original records + anonymized research DB
* Eval framework: accuracy scoring, failure tracking, test suite
* End-to-end data flow from chat → form → both databases

## **Mock or simulate:**

* EHR integration (show the API contract, don't connect to a real EHR)
* Voice input (text chat is sufficient for demo; mention voice as roadmap)
* Multi-language support (demo in English; mention Spanish as roadmap)
* Large-scale research DB queries (use a small SQLite or Postgres with 10-20 synthetic records)
* Regulatory compliance documentation (mention GDPR/local law in pitch, don't implement)

---

## Team Roles

| Workstream | Owner | Support | Notes |
| ---- | ---- | ---- | ---- |
| Problem research | All | — | Pre-hackathon |
| Clinical logic / health validation | Person 1 | Person 2 | Use standard intake form templates |
| AI / model workflow (agent + evals) | Person 2 | Person 1 | LLM API, structured output, confidence scoring |
| Data pipeline + anonymization | Person 1 | Person 2 | NER, dual storage, DB schema |
| Product / UX | Person 3 | — | Patient chat, clinician dashboard, research explorer |
| Front-end prototype | Person 3 | Person 2 | React or HTML, 3 views |
| Pitch deck | Person 3 | All | Architecture diagram, demo script, impact story |
| Final presentation | All | — | Practice run evening of Day 1 |

### Role Summary

* **Person 1 (Platform / DevOps):** Data pipeline owner — anonymization engine, dual storage, DB schema, infrastructure. Supports clinical logic.
* **Person 2 (Solutions Architect / Software Engineer):** AI agent owner — LLM integration, structured output, confidence scoring, eval framework. Supports pipeline and frontend.
* **Person 3 (Systems / Software Engineer):** Demo owner — patient chat UI, clinician dashboard, research DB explorer, pitch deck.

---

## Questions to Answer Early

* ~~What exact user pain point are we solving?~~ ✅ Manual intake + inaccessible research data
* ~~What makes our solution different from existing tools?~~ ✅ Dual-track storage (clinical + research) from a single intake, with built-in quality control
* ~~Why is AI necessary here?~~ ✅ Dynamic conversation, structured extraction, confidence scoring, NER anonymization
* ~~What can we realistically demo in the hackathon timeframe?~~ ✅ Full end-to-end flow with synthetic data
* How will we explain impact in a simple, convincing way? → "Every intake conversation becomes two assets: a clinical record and a research datapoint — automatically."
* What are the ethical, privacy, or implementation concerns? → Anonymization completeness, GDPR compliance, patient consent for research use, clinician trust in AI-filled forms

---

## Pre-Hackathon Action List

| Task | Owner | Deadline | Status |
| ---- | ---- | ---- | ---- |
| Finalize team members | All | 1 week before | ⬜ |
| ~~Choose top 3 ideas~~ | All | — | ✅ |
| ~~Select final concept~~ | All | — | ✅ |
| Define synthetic patient scenarios (5-10 cases) | Person 2 | 3 days before | ⬜ |
| Choose clinical form template (ED intake or general consult) | Person 1 | 3 days before | ⬜ |
| Set up repo + project board | Person 1 | 1 week before | ⬜ |
| Prototype LLM structured output prompt | Person 2 | 3 days before | ⬜ |
| Prototype NER anonymization pass | Person 1 | 3 days before | ⬜ |
| Draft pitch storyline | Person 3 | 2 days before | ⬜ |
| Prepare design references for UI | Person 3 | 2 days before | ⬜ |
| Set up eval test cases | Person 2 | 2 days before | ⬜ |
| Choose tech stack (frontend framework, DB, hosting) | All | 1 week before | ⬜ |

---

## Fast Brainstorming Prompts

* ~~What hospital workflow is still too manual?~~ → Intake form-filling
* ~~Where do patients fall through the cracks?~~ → Between intake and research — their data is captured but never reused
* ~~What clinical decision is repetitive and data-heavy?~~ → Filling the same form fields from patient answers
* ~~What public health issue could benefit from earlier detection?~~ → Trends in anonymized intake data (e.g., seasonal patterns, emerging conditions)
* ~~What could help overworked staff save time without compromising quality?~~ → AI-assisted intake with human-in-the-loop for edge cases
* ~~Where is there poor coordination between professionals, services, or systems?~~ → Between clinical operations and research teams sharing the same data
* ~~What problem could be demonstrated clearly in a simple prototype?~~ → Chat → form → two databases

---

## Notes

* Tech stack candidates: React (frontend), FastAPI or Node (backend), Claude API or OpenAI (LLM), SQLite or Postgres (DB), spaCy or LLM-based NER (anonymization)
* Eval strategy: build a test suite of 10-20 synthetic patient cases with known correct form outputs. Run the agent against all cases, score field-level accuracy, track confidence calibration (did low-confidence flags actually correspond to errors?)
* Pitch angle: "Hospitals generate data every day that could advance research — but it's locked behind forms and privacy barriers. MedFlow unlocks it automatically."
* Consider adding a simple analytics view in the research DB explorer showing aggregate stats (e.g., most common chief complaints, age distribution) — makes the research value tangible for judges

---

## Reminder

Multidisciplinary teams are strongly encouraged. A strong project will likely combine:

* A clear health-system problem ✅ — manual intake + data silos
* A realistic AI use case ✅ — conversational agent, structured extraction, NER anonymization, eval framework
* A demo judges can understand quickly ✅ — chat → form → two databases, 30-second walkthrough
* A compelling story about impact, implementation, and value ✅ — operational savings + research data as a byproduct
