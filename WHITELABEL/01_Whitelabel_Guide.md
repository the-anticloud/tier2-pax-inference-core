# PAX Inference Core — Whitelabel & OEM Guide

**Apache 2.0 license. Rebrand, resell, embed freely.**

---

## What You Can Do (Apache 2.0)

- Rebrand as "YourCompany AI Engine"
- Embed in a commercial product
- Sell deployments to your customers
- Modify the source without disclosure obligation
- Keep your modifications proprietary

---

## Whitelabel Checklist

```bash
# 1. Clone and rename
git clone [this repo] my-company-inference
cd my-company-inference

# 2. Replace branding
find . -name "*.py" -exec sed -i 's/PAX Inference Core/MyCompany AI/g' {} +
find . -name "*.md" -exec sed -i 's/PAX/MyCompany/g' {} +

# 3. Configure your model endpoint
# In config.yaml:
model_name: "mycompany-base-27b"
model_path: "s3://my-bucket/models/mycompany-base-27b-fp8"

# 4. Customize the AIOSS ledger company tag
# In pax_ledger.py:
LEDGER_ISSUER = "MyCompany AI Platform v1.0"
```

---

## OEM Integration Patterns

### Pattern A: Embedded Inference SDK
```python
# Your product embeds PAX as a library
from pax_inference_core import InferenceEngine, LedgerConfig

engine = InferenceEngine(
    model="mycompany-27b",
    ledger=LedgerConfig(path="./audit.aioss", issuer="MyCompany"),
    shield=True,
    pii_mode="redact",
)

result = engine.chat([{"role": "user", "content": "Analyze this contract..."}])
print(result.text)          # The answer
print(result.confidence)    # 0.0–1.0
print(result.ledger_block)  # Audit record
```

### Pattern B: Docker Sidecar
```yaml
# docker-compose.yml for your product
services:
  your-product:
    image: mycompany/product:latest
    environment:
      - INFERENCE_URL=http://pax-inference:8000

  pax-inference:
    image: mycompany/pax-inference-core:1.0
    volumes: ["./models:/models", "./audit:/data"]
    environment:
      - PAX_LEDGER_ISSUER=MyCompany
```

### Pattern C: Air-Gapped Appliance
```
Physical hardware box (your brand):
  ├── PAX Inference Core (rebranded)
  ├── 2× RTX 3090 or 1× A100
  ├── AIOSS ledger (tamper-evident, your company's issuer tag)
  └── Management UI (your design)

Customer receives: plug in, no internet, full LLM capability
```

---

## Compliance Inheritance

When you whitelabel PAX Inference Core, you inherit:
- TRL 8.0 qualification (document as part of your product's TRL)
- OWASP LLM Top 10 mitigations (cite PAX architecture)
- AIOSS ledger for SOC2/HIPAA audit trail
- PII scanner for GDPR/CCPA compliance

You still need your own:
- SOC2 Type II audit (your company's processes)
- DPA/BAA agreements with your customers
- Customer-facing privacy policy

---

## Pricing Guidance for Resellers

Suggested OEM pricing tiers:

| Package | What's included | Suggested MSRP |
|---------|----------------|----------------|
| Core Engine | Inference + ledger + shield | $15K/year/deployment |
| Compliance Pack | + OWASP report, PII scanner, TRL docs | $25K/year/deployment |
| Sovereign Pack | + Air-gap deploy, K5 PQ hash, GDPR/HIPAA docs | $45K/year/deployment |

PAX Inference Core costs you nothing in perpetual licensing (Apache 2.0).
