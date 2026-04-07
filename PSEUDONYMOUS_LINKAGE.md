# Pseudonymous Linkage: Enabling Longitudinal Research in MedFlow

> This document describes how MedFlow supports longitudinal (multi-visit) research queries while preserving its anonymization-first architecture. See [readme.MD](./readme.MD) for the core anonymization pipeline and [PHANTOM_LAYER.md](./PHANTOM_LAYER.md) for how researchers access the data.

---

## How the Pseudonymous Linkage Works

Here's what happens during MedFlow's anonymization step today vs. with the change:

### Today (fully anonymous)

```
Patient: Maria Garcia, DNI 12345678A, DOB 1958-03-15
                    |
                    v  anonymization
Record: female, age_band 60-65, HbA1c 9.4, metformin...
```

No link. If Maria comes back 6 months later with a cardiac event, that's a completely separate record with no connection.

### With pseudonymous linkage

```
Patient: Maria Garcia, DNI 12345678A, DOB 1958-03-15
                    |
                    v  hash function
pseudo_id = SHA-256("MEDFLOW_SALT_2026" + "12345678A" + "1958-03-15")
          = "a7f3b2c8d91e..."
                    |
                    v  anonymization (same as today)
Record: pseudo_id a7f3b2c8..., female, age_band 60-65, HbA1c 9.4, metformin...
```

The name, DNI, and date of birth are still discarded — never stored. But the hash is deterministic: same inputs always produce the same `pseudo_id`. So when Maria returns 6 months later, her new record gets the same hash and a researcher can query *"patients whose HbA1c was above 9 who later had a cardiac event."*

The salt is a secret key held only by MedFlow. Without it, the hash is irreversible. You can add a second layer by splitting the salt into two parts held by separate services so no single component can reconstruct the link.

---

## EHDS Compliance

This aligns directly with how the EHDS is structured:

- Under the EHDS, data users can in exceptional cases seek access to **pseudonymized data** instead of anonymous data, provided they identify an appropriate legal basis under GDPR such as legitimate interest.
- The regulation explicitly envisions both models — anonymized and pseudonymized.
- Researchers may only access pseudonymized data if anonymized data is **insufficient for their purpose**, and it is forbidden to re-identify data subjects or even attempt to do so.
- The EHDS requires the use of **secure processing environments** and data minimization techniques, including anonymization and pseudonymization, as prerequisites for secondary data access.
- For secondary use, data must be properly anonymized or pseudonymized and handled in accordance with strict data security requirements, and access is only permitted within secure processing environments.

This is exactly MedFlow's model: longitudinal research requires pseudonymized linkage because anonymous data cannot support it. The regulation was written to allow this exact use case.

---

## What This Means for MedFlow's Architecture

The three-layer compliance gate stays exactly the same — it just operates on a record that now includes a `pseudo_id` field. The gate still checks for leaked PII (names, phone numbers, DNI). The `pseudo_id` itself is not PII because it's an irreversible hash that cannot be traced back to an individual without the salt.

### Salt Architecture Options

The key architectural decision is where the salt lives:

| Option | How It Works | Trade-off |
|---|---|---|
| **Option A — MedFlow holds the salt** | Simpler. MedFlow is the pseudonymization controller. If a court order or regulatory action requires re-identification (e.g., a safety signal traced to a specific patient), MedFlow can comply. This is actually a feature — EHDS anticipates scenarios where re-linkage may be necessary under strict conditions. | Simpler infrastructure, single trust boundary |
| **Option B — Split salt across two services** | Neither service alone can re-identify. Re-identification requires both parties to cooperate, which can be restricted to court-ordered scenarios. | Stronger privacy guarantee, more complex infrastructure |

For the hackathon, **Option A** is the clear choice. Option B is a production hardening step worth mentioning in the pitch.

---

## What Changes in the Codebase

The change is small. In the anonymization pipeline, before PII is stripped, you add one step:

```python
import hashlib

def generate_pseudo_id(dni: str, dob: str, salt: str) -> str:
    """One-way hash. Irreversible without the salt."""
    raw = f"{salt}:{dni}:{dob}"
    return hashlib.sha256(raw.encode()).hexdigest()[:16]

# During anonymization (before PII is stripped):
pseudo_id = generate_pseudo_id(patient.dni, patient.dob, MEDFLOW_SALT)
record.pseudo_id = pseudo_id

# Then proceed with existing anonymization pipeline...
# DNI, name, DOB are discarded as before
```

The `pseudo_id` goes into the research DB as just another column. The compliance gate treats it as non-PII (it's an opaque hash). Researchers can `GROUP BY` or `JOIN` on `pseudo_id` to link records longitudinally, but they can never reverse it to identify a patient.

---

## Why This Matters for Revenue

Without pseudonymous linkage, MedFlow sells **cross-sectional snapshots** — point-in-time data with no way to track patient journeys. With it, MedFlow sells **longitudinal real-world evidence** — the ability to follow outcomes over time across visits, hospitals, and treatments.

This is what unlocks the $75K–$5M pricing tier. This is what Flatiron sold to Roche for $1.9B.
