# vLLM on macOS (Apple Silicon) Setup Guide

This guide explains how to run vLLM on macOS with Apple Silicon (M1/M2/M3/M4).

## Prerequisites

- macOS with Apple Silicon (ARM64)
- Python 3.10 - 3.13
- At least 8GB RAM (16GB+ recommended for larger models)
- Git

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/vllm-project/vllm.git
cd vllm
```

### 2. Create Virtual Environment

```bash
python3 -m venv .venv
```

### 3. Get Your Full Path (Important!)

Run this command and **copy the output** - you'll need it to run vLLM from anywhere:

```bash
pwd
```

Example output: `/Users/yourname/projects/vllm`

Save this path! Your activate command will be:
```
source <YOUR_PATH>/.venv/bin/activate
```

### 4. Activate Virtual Environment

```bash
source .venv/bin/activate
```

> ⚠️ **Important**: Always activate the virtual environment before running any vLLM commands!
> 
> You can verify it's active by running:
> ```bash
> which python
> # Should show: <YOUR_PATH>/.venv/bin/python
> ```

### 5. Install Dependencies

```bash
# Install PyTorch
pip install torch==2.10.0

# Install vLLM in editable mode
pip install -e .
```

## Running the Server

### From the vllm Directory

```bash
# Make sure you're in the vllm directory with venv activated
source .venv/bin/activate

python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

### From Anywhere (Using Full Path)

Use the path you got from `pwd` earlier:

```bash
# Replace <YOUR_PATH> with your actual path from pwd
# Example: /Users/yourname/projects/vllm

source <YOUR_PATH>/.venv/bin/activate && \
cd <YOUR_PATH> && \
python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

**Example with real path:**
```bash
source /Users/john/projects/vllm/.venv/bin/activate && \
cd /Users/john/projects/vllm && \
python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

The server will:
1. Download the model (first run only)
2. Load and warm up the model
3. Start serving on `http://0.0.0.0:8000`

### One-Liner (from vllm directory)

```bash
source .venv/bin/activate && python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000 \
  --dtype float16
```

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

```bash
# Always activate the venv first
source .venv/bin/activate

# TinyLlama (small, good for testing)
python -m vllm.entrypoints.api_server \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --dtype float16

# Phi-2 (small, fast)
python -m vllm.entrypoints.api_server \
  --model microsoft/phi-2 \
  --dtype float16

# Mistral 7B
python -m vllm.entrypoints.api_server \
  --model mistralai/Mistral-7B-Instruct-v0.1 \
  --dtype float16

# Llama 2 (requires Hugging Face token)
python -m vllm.entrypoints.api_server \
  --model meta-llama/Llama-2-7b-chat-hf \
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
| `--tensor-parallel-size` | Number of tensor parallel workers | `1` |

## Troubleshooting

### Error: `ModuleNotFoundError: No module named 'torch'`

**Cause**: Virtual environment not activated or PyTorch not installed.

**Fix**:
```bash
# Activate the virtual environment
source .venv/bin/activate

# Install PyTorch
pip install torch==2.10.0
```

### Error: `Symbol not found` or C extension errors

**Cause**: PyTorch version mismatch with compiled vLLM extensions.

**Fix**:
```bash
source .venv/bin/activate
pip install torch==2.10.0
pip install -e .
```

### Error: Wrong Python being used

**Cause**: Virtual environment not activated.

**Fix**:
```bash
# Check which python is being used
which python

# If it doesn't point to .venv/bin/python, activate:
source .venv/bin/activate
```

### Slow First Run

The first run downloads the model from Hugging Face. Subsequent runs use the cached model from `~/.cache/huggingface/`.

## Shell Alias (Optional)

Add this to your `~/.zshrc` or `~/.bashrc` for quick access.

First, get your path:
```bash
cd /path/to/vllm
pwd
# Copy the output, e.g.: /Users/yourname/projects/vllm
```

Then add to `~/.zshrc` (replace with YOUR path from pwd):
```bash
# Add to ~/.zshrc - replace the path with your pwd output
alias vllm-start='source /Users/yourname/projects/vllm/.venv/bin/activate && cd /Users/yourname/projects/vllm && python -m vllm.entrypoints.api_server'

# Reload shell
source ~/.zshrc

# Usage:
vllm-start --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 --dtype float16
```

## Notes

- vLLM on macOS runs on **CPU** (MPS/GPU acceleration not yet available)
- Performance is slower than NVIDIA GPU setups
- Smaller models (1B-3B parameters) recommended for local development
- The `VLLM_USE_CUDA=0` flag is **not needed** on macOS

## Recommended Models for macOS

| Model | Size | Notes |
|-------|------|-------|
| `TinyLlama/TinyLlama-1.1B-Chat-v1.0` | 1.1B | Fast, good for testing |
| `microsoft/phi-2` | 2.7B | Good quality for size |
| `Qwen/Qwen2-1.5B-Instruct` | 1.5B | Multilingual |
| `google/gemma-2b-it` | 2B | Instruction tuned |

## Useful Links

- [vLLM Documentation](https://docs.vllm.ai/)
- [Hugging Face Model Hub](https://huggingface.co/models)
- [vLLM GitHub](https://github.com/vllm-project/vllm)
