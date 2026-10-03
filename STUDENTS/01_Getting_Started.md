# PAX Inference Core — Student Getting Started Guide

**No GPU required to start. No API keys. No credit card.**

---

## What You're Building

You're running a production LLM inference server locally — the same kind that powers commercial AI products, but 100% on your own machine.

PAX Inference Core scored:
- **84.28% on MMLU** (university-level knowledge across 57 subjects)
- **41.33% on MuSR** (above the leaderboard max for open models)
- **90.75% on GSM8K** (grade-school math word problems)

You're going to run benchmarks yourself and verify these numbers.

---

## Step 1: Run a 7B Model (CPU, any laptop)

```bash
# 1. Install llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && make -j4

# 2. Download a small model (1.6GB, works on any modern CPU)
# Phi-3 Mini is good for learning
wget "https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf/resolve/main/Phi-3-mini-4k-instruct-q4.gguf"

# 3. Start the server
./llama-server -m Phi-3-mini-4k-instruct-q4.gguf -c 4096 --port 8080

# 4. Chat with it
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "What is the Poseidon hash function?"}], "max_tokens": 200}'
```

---

## Step 2: Run a Benchmark (Kaggle, free GPU)

Go to [kaggle.com](https://www.kaggle.com), create free account, open a new notebook with GPU T4×2.

Paste this to run MMLU on a 7B model:

```python
# Install lm-eval
!pip install lm-eval -q

# Run 5 MMLU subjects (fast, ~10 minutes on T4)
!python -m lm_eval \
    --model hf \
    --model_args pretrained=microsoft/phi-3-mini-4k-instruct,dtype=float16 \
    --tasks mmlu_anatomy,mmlu_astronomy,mmlu_biology,mmlu_chemistry,mmlu_math \
    --num_fewshot 5 \
    --output_path ./my_results.json \
    --batch_size 4
```

Compare your numbers to PAX_RESULTS.md. The PAX model (27B) should score higher — you're running a 3.8B model.

---

## Step 3: Understand Dual-Stream Uncertainty

```python
# This is what PAX does internally — you can reproduce it
import requests, json

def dual_stream(prompt, server_url="http://localhost:8080"):
    # Stream A: low temperature (confident, deterministic)
    a = requests.post(f"{server_url}/v1/completions",
        json={"prompt": prompt, "max_tokens": 100, "temperature": 0.1}).json()

    # Stream B: high temperature (exploratory)
    b = requests.post(f"{server_url}/v1/completions",
        json={"prompt": prompt, "max_tokens": 100, "temperature": 0.9}).json()

    text_a = a["choices"][0]["text"].strip()
    text_b = b["choices"][0]["text"].strip()

    # Simple word overlap as confidence proxy
    words_a = set(text_a.lower().split())
    words_b = set(text_b.lower().split())
    confidence = len(words_a & words_b) / max(len(words_a | words_b), 1)

    print(f"Stream A (confident): {text_a[:100]}")
    print(f"Stream B (exploratory): {text_b[:100]}")
    print(f"Confidence estimate: {confidence:.2%}")

dual_stream("What is 127 × 83?")
dual_stream("Who will win the 2028 election?")  # should show lower confidence
```

High confidence = both streams agree. Low confidence = model is uncertain. This is how PAX generates the `confidence` field in every AIOSS ledger entry.

---

## Step 4: Write to the AIOSS Ledger

```python
# Install
pip install aioss-format kantor-k5

from aioss_format import Ledger, Block
from kantor import k5_512
import time

ledger = Ledger.create("./my_learning_ledger.aioss")

# Record your benchmark run
block = Block(payload={
    "task": "mmlu_anatomy",
    "model": "phi-3-mini-4k",
    "score": 0.72,
    "prompt_hash": k5_512(b"my test prompt"),
    "confidence": 0.85,
    "contradiction": 0.12,
    "telemetry": {
        "tokens_in": 150,
        "tokens_out": 1,
        "wall_time_ms": 340,
        "cost_if_cloud_microcents": 0,  # free local inference!
    }
})
ledger.append(block)
print("Block appended. Verifying chain...")
assert ledger.verify()
print("Chain integrity: OK")
```

---

## What to Explore Next

| Project | What You Learn |
|---------|---------------|
| PAX_MATH_SOLVER | How SymPy + LLM achieves 100% GSM8K |
| PAX_SEMANTIC_ENGINE | How 321 zero-LLM detectors beat neural approaches on ARC-1 |
| K-KANTOR | How Poseidon hash works (post-quantum cryptography) |
| K-AIOSS | How hash-chained ledgers provide tamper evidence |
| PAX_ARC_SOLVER | The hardest ML benchmark — abstract visual reasoning |
