# PAX Inference Core — Development Guide

---

## Repository Structure

```
PAX_INFERENCE_CORE/
├── pax_inference_server.py      # Main entry point — P0 terminal harness
├── pax_shield.py                # P1 prompt injection + rate limiting
├── pax_router.py                # P1 VRAM-aware backend selection
├── pax_dual_stream.py           # P5 dual-stream confidence/contradiction sampling
├── pax_ledger.py                # P6 AIOSS integration — append/verify
├── backends/
│   ├── vllm_backend.py          # vLLM PagedAttention adapter
│   ├── llamacpp_backend.py      # llama.cpp server adapter
│   └── sglang_backend.py        # SGLang RadixAttention adapter (experimental)
├── tests/
│   ├── test_shield.py           # Injection pattern unit tests (50+ patterns)
│   ├── test_dual_stream.py      # Confidence/contradiction calibration tests
│   ├── test_ledger.py           # AIOSS append/verify/tamper-detect tests
│   └── test_backends.py         # Backend mock integration tests
├── benchmarks/
│   ├── kaggle_owasp_pentest.ipynb    # Kaggle: OWASP LLM pentest suite
│   ├── kaggle_benchmark_suite.ipynb  # Kaggle: MMLU/GSM8K/ARC eval
│   └── kaggle_arc_submission.ipynb   # Kaggle: ARC-AGI submission builder
└── TECHNICAL/  COMPLIANCE/  ENTERPRISE/  (docs — you are here)
```

---

## Running Tests

```bash
# Unit tests (no GPU needed)
pip install pytest pytest-asyncio
pytest tests/ -v

# Integration test (requires running inference server)
python pax_inference_server.py --backend llamacpp --test-mode &
pytest tests/test_backends.py -v --integration

# OWASP pentest suite
python -m pytest tests/test_owasp.py -v --html=owasp_report.html
```

---

## Kaggle Benchmark Notebooks

Three notebooks under `benchmarks/` are runnable on Kaggle GPU T4×2:

```bash
# Push to Kaggle
kaggle kernels push -p benchmarks/kaggle_owasp_pentest/
kaggle kernels push -p benchmarks/kaggle_benchmark_suite/
kaggle kernels push -p benchmarks/kaggle_arc_submission/
```

All notebooks use only:
- `HF_TOKEN` for model loading
- `KAGGLE_KEY` for submission
- No frontier API keys

---

## Adding a New Backend

```python
# backends/my_backend.py
from pax_inference_server import BackendBase, InferenceResult

class MyBackend(BackendBase):
    def __init__(self, url: str, model: str):
        self.url = url
        self.model = model

    def infer(self, prompt: str, max_tokens: int = 512, temperature: float = 0.1) -> InferenceResult:
        # Call your local inference server
        response = requests.post(f"{self.url}/v1/completions", json={
            "model": self.model,
            "prompt": prompt,
            "max_tokens": max_tokens,
            "temperature": temperature,
        })
        data = response.json()
        return InferenceResult(
            text=data["choices"][0]["text"],
            tokens_in=data["usage"]["prompt_tokens"],
            tokens_out=data["usage"]["completion_tokens"],
        )
```

Register in `pax_router.py`:
```python
BACKENDS["my_backend"] = lambda: MyBackend(url=os.getenv("MY_BACKEND_URL"), model="my-model")
```

---

## AIOSS Ledger — Developer Notes

```python
from aioss_format import Ledger, Block

ledger = Ledger.open("./pax_ledger.aioss")

# Append inference result
block = Block(
    payload={
        "model": "pax-one-27b-fp8",
        "prompt_hash": k5_512(prompt + timestamp),
        "output_hash": k5_512(output + timestamp),
        "confidence": confidence_score,
        "contradiction": contradiction_score,
        "telemetry": {
            "tokens_in": tokens_in,
            "tokens_out": tokens_out,
            "wall_time_ms": wall_time,
            "cost_if_cloud_microcents": 0,
        }
    }
)
ledger.append(block)

# Verify chain at any point
assert ledger.verify(), "Ledger chain broken — tampering detected"
```

---

## K5 Hash Integration

```python
from kantor import k5_512

# Hash a prompt without storing its content
prompt_hash = k5_512(prompt.encode() + timestamp.encode())

# Upgrade AIOSS ledger to post-quantum (replace SHA3-256)
# In ledger config:
# hash_algorithm = "k5-512"  # instead of "sha3-256"
```

---

## Code Standards

- Type hints on all public functions
- No `print()` in production paths — use structured logging
- No frontier API keys anywhere (CI check enforces this: `grep -r "OPENAI_API_KEY\|ANTHROPIC" . --include="*.py" && exit 1`)
- Every new backend must pass `test_backends.py` mock tests
- Ledger append must be transactional — never leave a partial block
