# PAX Inference Core — Deployment Guide

**No frontier API keys required. Local inference only.**

---

## Prerequisites

```bash
# Hardware minimum
GPU: RTX 3090 (24GB VRAM) — INT4/FP16
GPU: A100/H100/H200 — FP8 full speed

# Software
Python 3.11+
CUDA 12.1+
vLLM >= 0.6.0  OR  llama.cpp >= b3500

# Model weights (HuggingFace, no frontier key needed)
huggingface-cli login  # HF token: set HF_TOKEN env var
huggingface-cli download kleinnner/pax-one-27b-fp8
```

---

## Quick Start: vLLM Backend (recommended for GPU ≥ 20GB VRAM)

```bash
# Install
pip install vllm>=0.6.0 aioss-format kantor-k5

# Start inference server
python -m vllm.entrypoints.openai.api_server \
    --model kleinnner/pax-one-27b-fp8 \
    --dtype fp8 \
    --max-model-len 8192 \
    --port 8000

# Run with AIOSS ledger enabled (PAX wrapper)
python pax_inference_server.py \
    --backend vllm \
    --port 8000 \
    --ledger-path ./pax_ledger.aioss \
    --k5-hash  # upgrade ledger hashing to K5-512 (post-quantum)
```

---

## Quick Start: llama.cpp Backend (CPU / low-VRAM)

```bash
# Install llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && make -j8

# Download GGUF model
huggingface-cli download kleinnner/pax-one-27b-Q4_K_M --local-dir ./models/

# Start server
./llama-server \
    -m ./models/pax-one-27b-Q4_K_M.gguf \
    -c 8192 \
    --host 0.0.0.0 \
    --port 8080 \
    -n -1

# Run PAX wrapper with llamacpp backend
python pax_inference_server.py \
    --backend llamacpp \
    --llamacpp-url http://localhost:8080 \
    --ledger-path ./pax_ledger.aioss
```

---

## AIOSS Ledger Verification

After inference, verify the audit trail:

```bash
# Verify chain integrity
aioss verify ./pax_ledger.aioss

# Analyze a session
aioss analyze ./pax_ledger.aioss --report html > inference_audit.html

# Export for compliance
aioss export ./pax_ledger.aioss --format json > audit_export.json
```

---

## Environment Variables (no frontier keys)

```bash
# Required — local model
HF_TOKEN=hf_xxxxx          # HuggingFace token for model download only

# Optional — Kaggle benchmark submission
KAGGLE_USERNAME=kleinnner
KAGGLE_KEY=xxxxx

# Never set — not used, not supported
# OPENAI_API_KEY=           (blocked by P1 Shield)
# ANTHROPIC_API_KEY=        (blocked by P1 Shield)
# GEMINI_API_KEY=           (blocked by P1 Shield)
```

---

## Air-Gapped / Sovereign-OS Deployment

```bash
# Pack model + server for offline deploy
python pack_sovereign.py \
    --model ./models/pax-one-27b-Q4_K_M.gguf \
    --output ./sovereign_pack.tar.gz

# On air-gapped machine (no internet)
tar -xf sovereign_pack.tar.gz
./sovereign_pack/start.sh
# AIOSS ledger writes to local disk, no external calls ever
```

---

## Docker Compose

```yaml
version: "3.9"
services:
  pax-inference:
    image: pax-inference-core:1.0.0
    environment:
      - HF_TOKEN=${HF_TOKEN}
      - PAX_BACKEND=vllm
      - AIOSS_LEDGER_PATH=/data/ledger.aioss
    volumes:
      - ./models:/models
      - ./ledger:/data
    ports:
      - "8000:8000"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `CUDA out of memory` | Model too large for VRAM | Switch to `--dtype int4` or llama.cpp Q4_K_M |
| `confidence: None` in ledger | Dual-stream P5 not enabled | Add `--dual-stream` flag |
| `contradiction: None` | Same as above | Same fix |
| Slow first token (>2s) | Model loading cold | Pre-warm with `python pax_warmup.py` |
| `ledger_digest: None` | AIOSS not initialized | Run `aioss init ./pax_ledger.aioss` first |
