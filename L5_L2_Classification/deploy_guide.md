# Deploy Guide — PAX_INFERENCE_CORE
**Stack:** Python 3.11, PyTorch 2.10+, llama.cpp C-extension, CUDA, AIOSS_FORMAT | Air-gap capable

## Prerequisites
Anticloud core stack installed. PAX 27B weights (pax-27b-q4.gguf). AIOSS_FORMAT.

## Install
```bash
pip install anticloud-pax-inference-core
```

## AIOSS Integration
```bash
aioss init --module PAX_INFERENCE_CORE --output ./pax_inference_core.aioss
```

## Air-Gap
```bash
pip download -r requirements.txt -d ./wheels/
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(model_path="./pax-27b-q4.gguf", module="PAX_INFERENCE_CORE",
                     aioss_chain="./pax_inference_core.aioss",
                     classification="L5_NARROW_L2_GENERAL")
```

## Verification
```bash
aioss verify --chain ./pax_inference_core.aioss --verbose
python -m pax_inference_core.tests.smoke
```
