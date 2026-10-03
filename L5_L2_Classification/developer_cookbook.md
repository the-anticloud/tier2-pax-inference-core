# Developer Cookbook — PAX_INFERENCE_CORE
**Stack:** Python 3.11, PyTorch 2.10+, llama.cpp C-extension, CUDA, AIOSS_FORMAT

## Basic Usage
```python
from pax_inference_core import Inferencecore
module = Inferencecore(pax_model="./pax-27b-q4.gguf",
                               aioss_chain="./pax_inference_core.aioss")
result = module.process(input_data)
print(result.output, result.chain_hash)
```

## Batch Processing
```python
results = module.process_batch(inputs, batch_size=4)
for r in results:
    print(r.chain_hash)
```

## AIOSS Append
```python
import hashlib, time
def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

chain_hash = aioss_append("./pax_inference_core.aioss", result.to_bytes(), "PAX_INFERENCE_CORE")
```

## Integration with Anticloud TIER_2
```python
# Chain with PAX_INFERENCE_CORE
from pax_inference_core import PAXInferenceCore
from pax_inference_core import Inferencecore

core = PAXInferenceCore(model="./pax-27b-q4.gguf")
module = Inferencecore(inference_core=core)
```

## Domain: Core inference engine: token generation, sampling, streaming
This module specializes in: core inference engine: token generation, sampling, streaming.
AIOSS entry type: inference result (prompt hash + completion hash + tokens generated + latency).

## Performance
Use module.benchmark() to measure throughput on your hardware.
Pre-warm: module.warmup() before serving production requests.
