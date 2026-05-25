> This article was first published on [GitHub](https://github.com/cpufreestyle/cpufreestyle/blob/main/blog/hybrid-gpu-scheduler-en.md) | [Project Repo](https://github.com/cpufreestyle/hybrid-gpu-scheduler) | [Download Release](https://github.com/cpufreestyle/hybrid-gpu-scheduler/releases/tag/v1.1.0)

**Title: I built a hybrid GPU scheduler for NVIDIA + AMD – automatically routes training to CUDA and inference to AMD**

---

Hey folks!

I have two GPUs in my workstation: **NVIDIA RTX 5070 Ti (16GB)** and **AMD RX 7900 XTX (24GB)**.

The problem: training jobs want CUDA, inference jobs want lots of VRAM (AMD is great for this). Manually assigning tasks was annoying, 
So I built **Hybrid GPU Scheduler** – an open-source scheduler that manages **heterogeneous GPU environments** (NVIDIA + AMD together).

### What it does

- **Smart scheduling** based on 5 factors (VRAM 30%, utilization 25%, GPU type 20%, priority 15%, load balance 10%)
- **Auto routing**: `training` → NVIDIA, `inference` → AMD, `compute` → best available
- **3 strategies**: `binpack` (consolidate), `spread` (distribute), `gpu_type` (type-first)
- **Real process execution** with timeout control, log collection, GPU isolation (`CUDA_VISIBLE_DEVICES` / `ROCR_VISIBLE_DEVICES`)
- **REST API** + **Web Dashboard** (Chart.js real-time charts)
- **Prometheus `/metrics`** endpoint

### Quick start

```bash
# Download binary (Windows / Linux)
# https://github.com/cpufreestyle/hybrid-gpu-scheduler/releases/tag/v1.0.0

# Run
./hybrid-gpu-scheduler

# Submit a task
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"name":"training","type":"training","vram_mb":4096,"command":"python train.py"}'

# Open Dashboard
http://localhost:8080/dashboard
```

### Tech stack
Go 1.22+ · Gin · Prometheus · Chart.js

### Use cases
- DL training + inference on same machine
- Mixed NVIDIA + AMD environments
- Small team GPU sharing
- Personal AI workstation

**GitHub (MIT)**: https://github.com/cpufreestyle/hybrid-gpu-scheduler

Feedback welcome! 🚀

---

**注**：此公告可用于知乎、Reddit (r/MachineLearning, r/golang, r/selfhosted, r/homelab)
