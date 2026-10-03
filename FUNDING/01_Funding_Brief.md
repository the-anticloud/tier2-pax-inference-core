# PAX Inference Core — Funding Brief

**Stage:** TRL 8.0 — Production-qualified, seeking deployment capital

---

## The Problem

Enterprise AI today means paying $0.01–$0.03 per 1,000 tokens to OpenAI/Anthropic, sending proprietary data to foreign servers, with no audit trail and no sovereignty. At scale (1M tokens/day), that is **$300–$900/day** in perpetual vendor fees plus compliance risk.

---

## The Solution

PAX Inference Core: local-first LLM serving with cryptographic audit trail.

- **Zero per-token cost** — hardware amortized, not pay-per-query
- **Zero data egress** — prompts never leave the building
- **Tamper-evident audit** — AIOSS ledger, SHA3-256/K5-512 hash chain
- **Sovereignty** — Apache 2.0 model weights, runs air-gapped
- **SOTA performance** — MuSR 41.33% (above Open LLM Leaderboard max)

---

## Traction

| Evidence | Detail |
|---------|-------|
| Benchmark results | 40+ tasks verified across H200 + 2× RTX 3090 |
| MuSR SOTA | 41.33% — above published leaderboard maximum (38.7%) |
| ARC-AGI | 51.75% zero-LLM (ARC-1 eval), top class for non-LLM approach |
| Kaggle submissions | ARC-1 + ARC-2 submitted under `loiskleinner` |
| AIOSS ledger | Live, append-only, Ed25519-signed audit trail |
| TRL | 8.0 — system complete and qualified |

---

## Market Size

- **Enterprise AI software market:** $50B by 2028 (IDC)
- **On-premise / private AI segment:** $8B (fastest growing, CAGR 38%)
- **Target buyer:** Companies with >100 employees handling regulated data (healthcare, legal, finance, government)

---

## Revenue Model

| Stream | Mechanism |
|--------|----------|
| Enterprise license | $50K/year per deployment (unlimited tokens) |
| AIOSS compliance module | $20K/year add-on (SOC2/GDPR/HIPAA audit package) |
| Sovereign-OS integration | $30K one-time setup + $10K/year |
| Training / fine-tuning service | $5K per LoRA adapter (industry-specific) |

---

## Use of Funds

| Allocation | % | Purpose |
|-----------|---|---------|
| Engineering | 50% | vLLM optimization, SGLang integration, ARM/Apple Silicon port |
| Sales | 25% | Enterprise direct sales, pilot programs |
| Compliance | 15% | SOC2 Type II audit, FedRAMP assessment |
| Operations | 10% | Infrastructure, legal, admin |

---

## Competitive Moat

| Competitor | Weakness | PAX Advantage |
|-----------|---------|--------------|
| OpenAI API | Data leaves your network, $0.03/1K tokens | Zero egress, zero per-token cost |
| Azure OpenAI | Still Microsoft's cloud, vendor lock-in | Fully sovereign, Apache 2.0 |
| Ollama | No audit trail, no compliance module | AIOSS ledger + K5 hash = compliance-ready |
| vLLM standalone | No security layer, no PII handling | PAX adds P1 Shield + AIOSS + PII scanner |

---

## Ask

**$500K seed** for 18-month runway to:
1. First 10 enterprise pilot customers
2. SOC 2 Type II certification
3. FedRAMP-ready architecture (government market entry)
4. Team: 2 senior ML engineers, 1 enterprise sales

**Contact:** quazakeido@gmail.com | Lois-Kleinner Alpasan
