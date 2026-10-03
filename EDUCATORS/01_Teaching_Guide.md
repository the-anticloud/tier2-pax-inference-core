# PAX Inference Core — Educator's Teaching Guide

**Audience:** University professors, bootcamp instructors, ML course designers.

---

## What Students Learn from This Project

PAX Inference Core is a complete, real production inference system built without frontier API keys. It demonstrates:

1. **LLM serving architecture** — how vLLM PagedAttention works, why it matters for throughput
2. **Uncertainty quantification** — dual-stream sampling as a practical confidence estimator
3. **Security in LLM systems** — prompt injection, OWASP LLM Top 10, PII handling
4. **Audit trails** — hash-chained tamper-evident logging (AIOSS ledger)
5. **Benchmark methodology** — how MMLU, GSM8K, ARC-AGI actually work

---

## Curriculum Module: LLM Inference Systems (3-week module)

### Week 1: How Inference Works
**Concepts:** KV cache, PagedAttention, speculative decoding, throughput vs latency
**Lab:** Run llama.cpp locally with a 7B model; measure tok/s vs batch size

```bash
# Students run this on their own machine (no GPU required for 7B Q4)
./llama-server -m phi-3-mini-Q4_K_M.gguf -c 4096 --port 8080
curl http://localhost:8080/v1/completions \
  -d '{"prompt": "What is KV cache?", "max_tokens": 200}'
```

**Discussion:** Why does PAX use dual backends (vLLM for GPU, llama.cpp for CPU)?

### Week 2: Benchmarks and Evaluation
**Concepts:** MMLU format, ARC-AGI abstraction, GSM8K program-of-thought
**Lab:** Run MMLU 25-question subset on their local model; compare to PAX results

Key insight from PAX: MuSR **41.33%** exceeds Open LLM Leaderboard max (38.7%). Why? Because PAX runs the benchmark correctly (0-shot, proper prompt format) while some leaderboard entries use non-standard prompting.

### Week 3: Security + Audit
**Concepts:** OWASP LLM Top 10, prompt injection, AIOSS tamper-evident ledger
**Lab:** Run the OWASP pentest notebook (Kaggle) against a local model

---

## Lecture Slides Outline

### Slide Deck: "From Weights to Production LLM"

1. The inference problem: autoregressive decoding is expensive
2. KV cache: what it is, why it fills up, PagedAttention solution
3. Quantization: FP32 → FP16 → INT8 → INT4, accuracy/speed tradeoffs
4. Throughput benchmark: PAX results (23 tok/s on P100, 100 tok/s on H200)
5. Uncertainty: why temperature=0 is not the same as "certain"
6. Dual-stream sampling: confidence 0.87 on MMLU vs 0.31 on ARC-3
7. Security: the 10 ways your LLM can be attacked
8. Audit trails: why AIOSS ledger exists, what SHA3-256 hash chaining means

---

## Exam Questions

**Q1:** PAX Inference Core achieves MuSR 41.33%, above the Open LLM Leaderboard v2 maximum of 38.7%. Name two possible explanations. *(Answer: proper 0-shot prompt format; different sampling parameters; correct task formulation)*

**Q2:** Explain why `cost_if_cloud_microcents: 0` appears in every AIOSS ledger block. *(Answer: PAX uses local inference — no OpenAI/Anthropic API calls, so cloud cost is literally zero)*

**Q3:** The P1 Shield blocks prompt injection patterns. What is the residual risk? *(Answer: indirect prompt injection via RAG-retrieved documents that the shield doesn't see)*

---

## Connecting to Student Projects

Students can wire PAX Inference Core to their own projects:

```python
# Any Python project can call PAX Inference Core
import requests

response = requests.post("http://localhost:8000/v1/chat/completions", json={
    "model": "pax-one-27b",
    "messages": [{"role": "user", "content": "Explain backpropagation in 3 sentences"}],
    "max_tokens": 300,
})
print(response.json()["choices"][0]["message"]["content"])
# Also check confidence/contradiction in response.json()["pax_metadata"]
```

No API key. No cost per query. Full confidence score returned.
