# vLLM on macOS (Apple Silicon) Setup Guide

This guide explains how to run vLLM on macOS with Apple Silicon (M1/M2/M3/M4).

## Prerequisites

- macOS with Apple Silicon (ARM64)
- Python 3.10 - 3.13
- At least 8GB RAM (16GB+ recommended for larger models)

## Quick Start

### 1. Create and Activate Virtual Environment

```bash
# Navigate to the vllm directory
cd /Users/denair/testing/tryVllm/vllm

# Create virtual environment (if not exists)
python3 -m venv .venv

# Activate virtual environment
source .venv/bin/activate
```

> ⚠️ **Important**: Always activate the virtual environment before running any vLLM commands!

### 2. Install Dependencies

```bash
# Install PyTorch (required version for this vLLM build)
pip install torch==2.10.0

# Install vLLM in editable mode
pip install -e .
```

### 3. Run the API Server

```bash
# Start the vLLM API server
python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --tensor-parallel-size 1 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

The server will:
1. Download the model (first run only, ~75 seconds)
2. Load and warm up the model (~25 seconds)
3. Start serving on `http://0.0.0.0:8000`

## Testing the Server

### Health Check
```bash
curl http://localhost:8000/health
```

### Generate Text
```bash
curl -X POST http://localhost:8000/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello, how are you?", "max_tokens": 50}'
```

### API Documentation
Open in browser: http://localhost:8000/docs

## Available Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/generate` | POST | Generate text completion |
| `/docs` | GET | Swagger API documentation |
| `/redoc` | GET | ReDoc API documentation |

## Using Different Models

Replace the model name with any Hugging Face model:

```bash
# Llama 2 (requires HF token)
python -m vllm.entrypoints.api_server \
  --model meta-llama/Llama-2-7b-chat-hf \
  --dtype float16

# Mistral
python -m vllm.entrypoints.api_server \
  --model mistralai/Mistral-7B-Instruct-v0.1 \
  --dtype float16

# Phi-2 (smaller, faster)
python -m vllm.entrypoints.api_server \
  --model microsoft/phi-2 \
  --dtype float16
```

## Common Options

| Option | Description | Default |
|--------|-------------|---------|
| `--model` | Hugging Face model name or path | Required |
| `--host` | Host to bind | `localhost` |
| `--port` | Port to bind | `8000` |
| `--dtype` | Data type (`float16`, `float32`, `auto`) | `auto` |
| `--max-model-len` | Maximum sequence length | Model default |
| `--tensor-parallel-size` | Number of GPUs for tensor parallelism | `1` |

## Troubleshooting

### Error: `ModuleNotFoundError: No module named 'torch'`

**Cause**: Virtual environment not activated or PyTorch not installed.

**Fix**:
```bash
source .venv/bin/activate
pip install torch==2.10.0
```

### Error: `Symbol not found` or C extension errors

**Cause**: PyTorch version mismatch with compiled vLLM extensions.

**Fix**:
```bash
# Ensure correct PyTorch version
pip install torch==2.10.0

# Reinstall vLLM
pip install -e .
```

### Error: `python` points to wrong Python

**Cause**: Virtual environment not activated.

**Fix**:
```bash
# Check which python is being used
which python
# Should show: /Users/denair/testing/tryVllm/.venv/bin/python

# If not, activate venv
source /Users/denair/testing/tryVllm/.venv/bin/activate
```

### Slow Model Loading

First run downloads the model from Hugging Face. Subsequent runs will be faster as the model is cached in `~/.cache/huggingface/`.

## One-Liner Quick Start

Copy and paste this to start the server:

```bash
cd /Users/denair/testing/tryVllm/vllm && \
source .venv/bin/activate && \
python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

## Notes

- vLLM on macOS runs on **CPU** (not MPS/GPU acceleration yet)
- Performance will be slower than NVIDIA GPU setups
- Smaller models (1B-3B parameters) work best for local development
- The `VLLM_USE_CUDA=0` flag is not needed on macOS

## Useful Links

- [vLLM Documentation](https://docs.vllm.ai/)
- [Hugging Face Model Hub](https://huggingface.co/models)
- [vLLM GitHub](https://github.com/vllm-project/vllm)

