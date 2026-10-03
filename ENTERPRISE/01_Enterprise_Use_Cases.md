# PAX Inference Core — Enterprise Use Cases

**Local-first. No cloud dependencies. No frontier API keys.**

---

## Why Enterprise Chooses Local Inference

| Concern | Cloud LLM | PAX Inference Core |
|---------|-----------|-------------------|
| Data sovereignty | Prompts leave your network | **Zero data egress** |
| Cost at scale | $0.01–0.03/1K tokens | **$0.00** (hardware amortized) |
| Latency | 200ms–2s network round-trip | **50ms/token** on-prem |
| Compliance (HIPAA/GDPR) | Requires DPA, BAA | **No BAA needed** — data never leaves |
| Vendor lock-in | API key, rate limits, deprecation | **Apache 2.0 weights, yours forever** |
| Audit trail | Provider logs (opaque) | **AIOSS ledger, cryptographically verifiable** |

---

## Deployment Pattern 1: Internal Knowledge Base (RAG)

**Use case:** Legal team queries 50K internal documents.

```
Internal Docs → PAX_RETRIEVAL (chunking + embedding)
                    ↓
              Vector DB (local Qdrant/Chroma)
                    ↓
User Query → PAX_INFERENCE_CORE → grounded answer
                    ↓
           AIOSS ledger: query hash + answer hash + confidence
```

**Result:** Lawyers get grounded answers. Every query is auditable. Zero documents leave the building.

---

## Deployment Pattern 2: Code Review Automation

**Use case:** CI/CD pipeline automatically reviews PRs.

```
Git PR webhook → ANTICODE_AGENT
                     ↓
             PAX_INFERENCE_CORE (code understanding)
                     ↓
             Structured review (bugs, security, style)
                     ↓
             Post to GitHub PR via tool-call
                     ↓
             AIOSS ledger: code hash + review hash
```

**Result:** Every PR gets LLM review at commit time. No GitHub Copilot subscription.

---

## Deployment Pattern 3: Healthcare Decision Support

**Use case:** Hospital system provides clinical decision support.

```
Patient record (de-identified) → PII scanner (HIPAA mode)
                                       ↓
                             PAX_INFERENCE_CORE
                                       ↓
                    Clinical suggestion + confidence score
                                       ↓
                    AIOSS ledger (PHI-safe: hash only)
                                       ↓
                    Physician reviews (human-in-loop required)
```

**Compliance:** HIPAA BAA not required (no PHI leaves premise).  
**TRL:** 8.0 — MedQA 82.8%, MedMCQA 68.95% (see TECHNICAL/02_Benchmark_Results.md).

---

## Deployment Pattern 4: Air-Gapped Defense / Government

**Use case:** Classified document analysis.

```
SOVEREIGN_OS air-gapped node
    ├── PAX_INFERENCE_CORE (llama.cpp backend, no network)
    ├── AIOSS ledger (local disk, tamper-evident)
    ├── K-KANTOR K5-512 hash (post-quantum, classified audit trail)
    └── Zero internet connectivity required
```

**Compliance:** EU AI Act High-Risk, NIST AI RMF, FedRAMP-ready architecture.

---

## Pricing Model

| Tier | Hardware | Throughput | Per-month cost |
|------|---------|------------|---------------|
| Edge | 1× RTX 3090 (24GB) | ~30 tok/s | Hardware only (~$12/month electricity) |
| Mid | 2× RTX 3090 | ~60 tok/s | ~$24/month electricity |
| Performance | 1× H100 (80GB) | ~100 tok/s | Data center rates |
| Air-gap | Any, offline | Depends | Zero cloud spend |

No per-token charges. No API key costs. No rate limits.

---

## SLA / Uptime

- **Availability:** 99.5%+ (local hardware, no external dependency)
- **Degraded mode:** llama.cpp fallback if GPU unavailable (CPU inference)
- **Recovery:** Sub-60s restart via systemd/Docker health check
- **Data durability:** AIOSS ledger append-only; survives process restart
