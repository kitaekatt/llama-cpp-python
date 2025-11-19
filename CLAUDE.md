# llama-cpp-python Local Build

## Project Context

This is a **local development fork of llama-cpp-python** that supports the **llm-dev project** at `~/Dev/llm-dev/`.

**Relationship:**
- **llama-cpp-python repository** (you are here): Python bindings for llama.cpp with RTX 5090 optimizations
- **llm-dev project** (~/Dev/llm-dev/): Multi-modal AI platform that uses llama-cpp-python for GGUF model inference
- **Configuration**: llm-dev configures GGUF models to use this local build via CMake arguments

## Purpose

This local llama-cpp-python build exists to:
1. **Support GGUF inference** on RTX 5090 with optimized CUDA kernels
2. **Apply SM 12.0 optimizations** for Hopper GPU architecture
3. **Enable custom patches** and features not yet in stable releases
4. **Maintain consistent fork architecture** with vllm and cchooks

Currently configured for:
- **RTX 5090 SM 12.0 (Hopper)** GPU architecture
- **CUDA-optimized GGUF inference** via llama.cpp C library

## Fork & Upstream Management

This is a **public fork** with SM 12.0 optimizations:

```
origin:   git@github.com:kitaekatt/llama-cpp-python.git  (your fork - push here)
upstream: https://github.com/abetlen/llama-cpp-python.git (official - pull from here)
```

**Branch Strategy:**
- `main`: Clean, synced with upstream (no local modifications)
- `sm12.0-optimizations`: Local RTX 5090 optimizations (ready for PRs upstream)

**Syncing with upstream:**
```bash
cd ~/Dev/git/llama-cpp-python
git fetch upstream
git merge upstream/main main
git push origin main
```

**Contributing back:**
```bash
git push origin sm12.0-optimizations
gh pr create --base abetlen/llama-cpp-python:main --head kitaekatt/llama-cpp-python:sm12.0-optimizations
```

## Build System

### Build with RTX 5090 Optimization

```bash
cd ~/Dev/git/llama-cpp-python
export CMAKE_CUDA_ARCHITECTURES="120"  # Force SM 12.0 (RTX 5090) compilation
pip install -e . --no-build-isolation
```

**Key Configuration**:
- `CMAKE_CUDA_ARCHITECTURES="120"` - Forces CUTLASS kernels for Hopper architecture
- Enables custom CUDA kernels optimized for RTX 5090 compute patterns
- Installed as editable package (`-e .`) for development iteration

### Build from Within llm-dev

When llm-dev needs to build GGUF models, it references this local fork:

```bash
cd ~/Dev/llm-dev
source .venv/bin/activate
# llm-dev scripts automatically detect and use local llama-cpp-python build
```

## Relationship to llm-dev

### How llm-dev Uses This Build

1. **GGUF Model Support**:
   - Scripts detect available GGUF models in `~/.cache/huggingface/hub/`
   - Use this local build for inference server
   - Fallback to PyPI if local build not available

2. **Integration Points**:
   - Server startup: `python llm/bin/server.py --model [gguf-model]`
   - Chat interface: `python llm/bin/chat.py --model [gguf-model]`
   - Configuration: `llm/config/models.json` specifies GGUF models

3. **Performance**:
   - CUDA-optimized kernels reduce inference latency
   - SM 12.0 support ensures RTX 5090 compatibility
   - Measured performance stored in `benchmark_data/llama-cpp-performance.json`

## Architecture Decisions

### Conservative Dynamic Calculation (llm-dev principle)

llama-cpp parameters (context_length, n_gpu_layers, threads) are **not hard-coded** in configuration. Instead:
- **Phase 1**: Calculate dynamically from available VRAM using conservative defaults
- **Phase 2**: Run benchmarks to measure actual usage (stored in llm-dev benchmark data)
- **Phase 3**: Use measured data for hardware-specific optimization (future work)

This ensures the same build works across different GPU VRAM sizes (8GB → 32GB).

## Common Tasks

### Rebuild after code changes
```bash
cd ~/Dev/git/llama-cpp-python
export CMAKE_CUDA_ARCHITECTURES="120"
pip install -e . --no-build-isolation
```

### Test a specific GGUF model
```bash
cd ~/Dev/llm-dev
source .venv/bin/activate
python llm/bin/benchmark.py [gguf-model-name]
```

### Check build status
```bash
cd ~/Dev/git/llama-cpp-python
pip show llama-cpp-python
```

### Force clean rebuild
```bash
cd ~/Dev/git/llama-cpp-python
pip uninstall llama-cpp-python -y
export CMAKE_CUDA_ARCHITECTURES="120"
pip install -e . --no-build-isolation
```

## Current Status

**Setup**: ✅ Local fork created and integrated with llm-dev
- Fork created at kitaekatt/llama-cpp-python
- Remotes configured (origin = fork, upstream = official)
- Ready for SM 12.0 optimizations and GGUF support

## References

- **llm-dev project**: ~/Dev/llm-dev/CLAUDE.md (parent project documentation)
- **llama.cpp official**: https://github.com/ggerganov/llama.cpp
- **llama-cpp-python upstream**: https://github.com/abetlen/llama-cpp-python
- **GGUF Format**: https://github.com/ggerganov/gguf
