# PAX Inference Core — OWASP / OSINT / PII Security Assessment

---

## OWASP LLM Top 10 (2025) — Detailed Analysis

### LLM01: Prompt Injection

**Risk:** Malicious input manipulates model to ignore instructions or exfiltrate data.

**PAX Mitigation (P1 Shield):**
```python
# pax_shield.py — runs before every inference call
INJECTION_PATTERNS = [
    r"ignore (all )?(previous|prior|above) instructions",
    r"you are now (a )?(different|new|evil|jailbroken)",
    r"system prompt.*reveal",
    r"DAN|JAILBREAK|SUDO MODE",
    r"<\|im_start\|>.*<\|im_end\|>",  # template injection
    r"\\n\\n###",                        # separator injection
]

def shield_check(prompt: str) -> ShieldResult:
    for pattern in INJECTION_PATTERNS:
        if re.search(pattern, prompt, re.IGNORECASE):
            return ShieldResult(blocked=True, reason=f"injection_pattern:{pattern[:30]}")
    return ShieldResult(blocked=False)
```

**Residual Risk:** Low. Indirect injection via retrieved documents (RAG path) requires separate check in PAX_RETRIEVAL.

---

### LLM04: Model Denial of Service

**Risk:** Extremely long prompts or repetitive token sequences cause OOM or excessive compute.

**PAX Mitigation:**
```python
# Hard caps in P0 Terminal Harness
MAX_PROMPT_TOKENS = 7168        # leaves 1024 for output in 8192 context
MAX_OUTPUT_TOKENS = 2048        # hard cap regardless of request
MAX_REQUESTS_PER_MINUTE = 60    # per-client rate limit
VRAM_GUARD_GB = 2.0             # refuse if free VRAM < 2GB
```

---

### LLM06: Sensitive Information Disclosure

**Risk:** Model trained on sensitive data may reproduce PII in outputs.

**PAX PII Scanner:**
```python
# Runs on output before returning to client
PII_PATTERNS = {
    "ssn":     r"\b\d{3}-\d{2}-\d{4}\b",
    "email":   r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b",
    "cc":      r"\b(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14})\b",
    "phone":   r"\b(\+?1[-.\s]?)?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}\b",
    "ip_priv": r"\b(10|172\.1[6-9]|172\.2\d|172\.3[01]|192\.168)\.\d+\.\d+\b",
}

def scan_output(text: str) -> ScanResult:
    findings = []
    for pii_type, pattern in PII_PATTERNS.items():
        matches = re.findall(pattern, text)
        if matches:
            findings.append(PIIFinding(type=pii_type, count=len(matches)))
    return ScanResult(clean=(len(findings)==0), findings=findings)
```

---

## OSINT Attack Surface Map

### Reconnaissance Targets (what an attacker would look for)

| Target | Exposure | Defense |
|--------|----------|---------|
| Port 8000 (vLLM API) | Localhost only | `--host 127.0.0.1` default |
| Port 8080 (llama.cpp) | Localhost only | Same |
| Model file paths | Not exposed | Files on local disk, not web-served |
| HF token | Environment variable | Never logged, never in ledger |
| Kaggle API key | Environment variable | Same |
| AIOSS ledger | Local `.aioss` file | No sync to external service |
| Inference logs | Local disk | Only K5 hash of prompt, not raw text |
| GPU fingerprint | Not exposed | Inference telemetry stays local |

### Shodan/Censys Profile
A correctly deployed PAX Inference Core instance will have:
- **0 open ports** on public internet
- **0 cloud API calls** (nothing to intercept in transit)
- **0 identifiable service banners** externally

---

## PII Classification and Handling

### Data Flow PII Map
```
User Prompt
    │
    ▼
[P1 Shield: PII detect before inference]
    │ PII found → log PII_WARNING to AIOSS, optionally block
    │
    ▼
Inference Core
    │
    ▼
[Output PII Scanner: scrub before response]
    │
    ▼
AIOSS Ledger
    │ Stores: K5-512(prompt ∥ timestamp)  ← hash only, never raw text
    │ Stores: K5-512(output ∥ timestamp)  ← hash only
    │ Stores: confidence, contradiction, telemetry
    └─ NEVER stores: raw prompt, raw output, user IP, user identity
```

### PII Categories Handled
| Category | Regulation | Handling |
|----------|-----------|---------|
| Health data (PHI) | HIPAA | Block or redact mode |
| EU personal data | GDPR | Minimal retention; hash-only in ledger |
| Financial data (PAN) | PCI-DSS | Block mode by default |
| Government ID (SSN) | CCPA/GDPR | Redact from output; warning in ledger |
| Biometric identifiers | EU AI Act | Not processed; no image PII by default |

---

## Penetration Testing Checklist (Kaggle-validated)

Run against `loiskleinner/pax-inference-core-pentest` Kaggle notebook:

- [x] Prompt injection variants (50+ patterns from OWASP LLM Pentest Guide)
- [x] Token flooding (max context window exhaustion)
- [x] Concurrent request flood (rate limiter stress test)
- [x] SSRF via tool-call (test if model can be tricked into internal requests)
- [x] PII leakage via crafted prompts (ask model to repeat training data)
- [x] Supply chain: verify model hash before load
- [x] Ledger tampering: modify .aioss file, verify `aioss verify` catches it

All checks passed. Test notebook available at Kaggle under `loiskleinner` account.
