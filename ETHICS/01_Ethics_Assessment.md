# PAX Inference Core — Ethics Assessment

**Framework:** EU AI Act, NIST AI RMF, IEEE Ethically Aligned Design

---

## Risk Classification (EU AI Act)

**Category: General-Purpose AI System**
- Not inherently high-risk (inference engine, not a decision-making system by itself)
- Becomes high-risk if deployed in: medical diagnosis, credit scoring, hiring, critical infrastructure
- High-risk deployment requires: human oversight, transparency obligations, accuracy documentation

**PAX Inference Core provides the foundation for compliant high-risk deployment:**
- Confidence/contradiction scores → quantified uncertainty (Art. 13 transparency)
- AIOSS ledger → audit trail (Art. 17 record-keeping)
- PII scanner → data governance (Art. 10 data and data governance)

---

## Transparency

### What the system discloses:
- Every response includes `confidence` (0–1) and `contradiction` (0–1) scores
- Users can query the AIOSS ledger to see the hash of any past inference
- Model identity (`pax-one-27b-fp8`) is disclosed in response headers
- System does not pretend to be human

### What it does NOT disclose:
- Raw prompt content (hashed only in ledger — privacy protection, not opacity)
- Internal reasoning steps (chain-of-thought is optional, not default)

---

## Fairness Analysis

| Dimension | Assessment |
|-----------|-----------|
| Training data bias | Inherits base model biases (Apache 2.0 27B base) |
| MMLU subject variance | Math 92% vs Chemistry 40% — significant subject imbalance |
| Language equity | Primarily English; non-English performance not benchmarked |
| Demographic bias | TruthfulQA MC2: 54.14% — room for improvement on bias-adjacent tasks |
| CrowS-Pairs | 4.08 likelihood_diff — within acceptable range |

**Mitigation:** Users deploying in high-stakes contexts should evaluate subject-specific accuracy before production use. Subject scores are documented in TECHNICAL/02_Benchmark_Results.md.

---

## Safety Analysis

### Capabilities (what this system CAN do)
- Answer knowledge questions, write code, reason about math, analyze documents
- Generate plausible text on most topics

### Designed-out capabilities (PAX P1 Shield)
- Prompt injection attacks → blocked
- Jailbreak patterns → blocked
- Excessive agency (shell commands without allow-list) → blocked

### Known limitations
- **Hallucination:** System can generate confident wrong answers. Contradiction score partially indicates this but is not a perfect signal.
- **Temporal staleness:** Knowledge cutoff of base model; no real-time retrieval unless PAX_RETRIEVAL is wired in.
- **Math errors on complex problems:** AIME 16%, Minerva-Math 36.57% — significant gap from human expert performance.

---

## Human Oversight Requirements

PAX Inference Core is designed for **human-in-the-loop** deployment:
- Medical, legal, financial outputs: always require human review before acting
- Confidence < 0.5: system should flag for human review (configurable threshold)
- Contradiction > 0.4: system should present both interpretations to user

These are not enforced by default — deployers must configure thresholds appropriate for their use case.

---

## Environmental Impact

| Metric | Value |
|--------|-------|
| Inference power (RTX 3090) | ~350W per GPU |
| Tokens per kWh (RTX 3090) | ~300K tokens |
| Tokens per kWh (H200) | ~1.2M tokens |
| Carbon vs cloud baseline | Depends on grid — renewable energy recommended |

Local inference eliminates data center round-trip but shifts energy responsibility to deployer. Recommend renewable energy where possible.

---

## Responsible Disclosure

Security vulnerabilities in PAX Inference Core: report to quazakeido@gmail.com.  
Response time target: 48 hours for critical, 7 days for non-critical.
