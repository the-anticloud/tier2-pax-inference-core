# PAX Inference Core — Multi-Backend Model Serving

**Status:** Production | **Version:** 1.0.0 | **Author:** PAX Engine Team  
**Domain:** 0-1.gg/pax/inference-core

---

## What Is PAX Inference Core?

PAX Inference Core is the foundational inference engine powering the Pax One L5 Narrow 27B model. It abstracts over multiple backend serving frameworks (vLLM, TensorRT, llama.cpp) to provide a unified, OpenAI-compatible API with streaming, tool-calling, and multi-modal support.

The engine serves as the backbone for all specialized PAX solvers (math, semantic, vision, code) and integrates seamlessly with the broader Anticloud ecosystem.

```
User Request (OpenAI format)
    ↓
PAX Inference Core (routing layer)
    ↓
[vLLM Backend | TensorRT Backend | llama.cpp Backend]
    ↓
Pax One L5 Narrow 27B (FP8 or FP16)
    ↓
Streaming/structured output response
```

---

## Key Specifications

| Aspect | Details |
|--------|---------|
| **Model** | Pax One L5 Narrow 27B (Apache 2.0 open-weight base) |
| **Quantization** | FP8 (E4M3, dynamic scaling) / FP16 / INT4/INT8 available |
| **Context Window** | 8192 tokens (extendable to 32K via rope scaling) |
| **Throughput** | ~100 tok/sec on single H100, ~30 tok/sec on dual RTX 3090 |
| **Latency (P95)** | ~500ms first-token, ~50ms/token streaming |
| **Backends Supported** | vLLM (primary), TensorRT (optimization), llama.cpp (edge) |
| **API Protocol** | OpenAI Chat Completions, text-embedding-ada-002 compatible |
| **Tool Support** | Function calling, web+shell integration, vision (Qwen2-VL) |

---

## Architecture

### Layer 1: API Gateway
- OpenAI-compatible `/v1/chat/completions` and `/v1/embeddings`
- Request validation, schema checking, rate limiting
- Streaming (SSE) and non-streaming response paths
- Tool-calling request preprocessing

### Layer 2: Router & Cache
- Per-request backend selection (vLLM for throughput, TensorRT for latency)
- KV cache management across concurrent requests
- Embedding cache for repeated queries
- Result memoization for deterministic queries

### Layer 3: Inference Backends
- **vLLM:** High-throughput batched inference, paged attention
- **TensorRT:** Low-latency optimized execution, graph fusion
- **llama.cpp:** CPU-only inference, Raspberry Pi compatibility

### Layer 4: Model Layer
- Pax One L5 Narrow 27B weights (quantized variants available)
- LoRA adapter composition for fine-tuned behaviors
- Attention mechanisms: Multi-query attention (MQA) for speed
- Rope scaling for context extension

---

## Performance (Verified from PAX_RESULTS.md)

### Reasoning Benchmarks
- **GSM8K:** 72% (baseline), 100% (with SymPy-PoT)
- **MMLU:** 84.28% (0-shot), 83.42% (5-shot)
- **MMLU-Pro:** 58.57% (5-shot)
- **MuSR:** 41.33% (0-shot, **above leaderboard max**)

### Specialized Tasks
- **ARC-Challenge:** 58.79% (commonsense reasoning)
- **HellaSwag:** 82.87% (instruction following)
- **WinoGrande:** 76.48% (pronoun resolution)

### Hardware Efficiency
- H100: ~0.37/hr (Vast.ai, peak performance)
- 2x RTX 3090: ~30 tok/sec (optimal cost/performance ratio)
- CPU-only (llama.cpp): ~5 tok/sec on modern CPU

---

## Quick Start

### Installation (vLLM Backend)
```bash
pip install vllm transformers torch

# Download model
python -c "from transformers import AutoTokenizer; AutoTokenizer.from_pretrained('pax-one-l5-narrow-27b')"

# Start server
python -m vllm.entrypoints.openai.api_server \
  --model pax-one-l5-narrow-27b \
  --quantization fp8 \
  --dtype auto \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.9 \
  --port 8000
```

### API Usage
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="not-needed")

# Simple completion
response = client.chat.completions.create(
    model="pax-one-l5-narrow-27b",
    messages=[{"role": "user", "content": "What is 42 * 37?"}],
    temperature=0.7
)
print(response.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="pax-one-l5-narrow-27b",
    messages=[{"role": "user", "content": "Explain quantum computing briefly"}],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")
```

### Docker Deployment
```bash
docker run --gpus all -p 8000:8000 \
  pax-inference-core:latest \
  --model pax-one-l5-narrow-27b \
  --quantization fp8
```

---

## Integration Points

### Primary Consumers (Tier 2)
- **PAX_MATH_SOLVER** — Uses inference core for reasoning chain
- **PAX_SEMANTIC_ENGINE** — Backbone for pattern recognition
- **PAX_ARC_SOLVER** — Powers all reasoning lanes
- **PAX_VISION_ENGINE** — Multi-modal inference

### Complementary (Tier 1 & 3)
- **ANTICODE_AGENT** — AI coding automation backend
- **api-oss-gateway** — Enterprise API wrapper
- **PAX_GATEWAY** — OpenAI-compatible proxy
- **KAZCADE** — SIMD acceleration for embeddings

### Deployment (Tier 3)
- **api-oss-monitor** — Health and latency monitoring
- **PAX_SCHEDULER** — Request queuing and priority dispatch
- **PAX_CACHE_LAYER** — KV cache coordination
- **SOVEREIGN_OS** — Air-gapped inference containers

---

## Security & Compliance

- **Input validation:** Schema checking, injection prevention
- **Output filtering:** No model weights leakage, PII masking option
- **Audit trail:** AIOSS ledger integration (via K5 hashing)
- **Privacy:** No model API calls to external providers
- **Air-gap capable:** Full self-hosted deployment option

---

## Configuration Reference

```yaml
# config.yaml
inference:
  model: pax-one-l5-narrow-27b
  quantization: fp8
  tensor_parallel: 1
  gpu_memory_utilization: 0.9
  max_model_len: 8192
  
api:
  port: 8000
  host: 0.0.0.0
  streaming_timeout: 300
  max_concurrent_requests: 64
  
cache:
  type: redis  # or in-memory
  ttl_seconds: 3600
  
monitoring:
  prometheus_port: 9090
  log_level: info
```

---

## Roadmap

- **Q4 2026:** Rope scaling to 32K context
- **Q1 2027:** MoE (mixture-of-experts) variants for specialization
- **Q2 2027:** Speculative decoding for 2x speedup
- **Q3 2027:** Multi-modal (audio, video) support

---

## References

- **Benchmark Evidence:** `/pax-one-evidence/`
- **PAX_RESULTS.md:** Official results (H100 session)
- **PAX_RESULTS_2.md:** Hardware portability (RTX 3090 validation)
- **GitHub:** github.com/0-1-gg/pax-inference-core
- **Docs:** 0-1.gg/pax/inference-core

---

**Next:** See APPENDIX/ for integration patterns and architectures
