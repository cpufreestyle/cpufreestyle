> 本文首发于 [GitHub](https://github.com/cpufreestyle/cpufreestyle/blob/main/blog/hybrid-gpu-scheduler-announcement.md) | [项目仓库](https://github.com/cpufreestyle/hybrid-gpu-scheduler) | [下载 Release](https://github.com/cpufreestyle/hybrid-gpu-scheduler/releases/tag/v1.1.0)

**标题：我做了一个异构 GPU 调度器——同时调度 NVIDIA + AMD GPU，训练推理自动分配**

---

### 背景

手头有两张卡：NVIDIA RTX 5070 Ti (16GB) + AMD RX 7900 XTX (24GB)。

问题来了：
- 训练任务更吃 CUDA 生态 → 应该跑在 NVIDIA 上
- 推理任务对显存要求大、AMD 性价比高 → 应该跑在 AMD 上
- 手动分配？太累了。多个任务同时跑？更头疼。

于是我写了一个 **Hybrid GPU Scheduler**——支持异构 GPU 的智能调度器。

---

### 核心功能

#### 1. 混合 GPU 调度
- 同时管理 NVIDIA + AMD GPU
- 根据任务类型自动分配：
  - `training` → NVIDIA（CUDA 生态优先）
  - `inference` → AMD（大显存、性价比高）
  - `compute` → 按打分分配

#### 2. 多因素智能打分
调度器根据 5 个维度打分（权重可调）：

| 因素 | 权重 | 说明 |
|------|------|------|
| 剩余显存 | 30% | 可用显存越多分数越高 |
| GPU 利用率 | 25% | 利用率越低分数越高 |
| GPU 类型匹配 | 20% | training→NVIDIA, inference→AMD |
| 任务优先级 | 15% | 高优先级优先调度 |
| 负载均衡 | 10% | 避免单 GPU 过载 |

#### 3. 三种调度策略
- **Binpack**：尽量用少的 GPU 承载更多任务（提高利用率）
- **Spread**：分散调度（避免单点过热）
- **GPU Type**：强制按任务类型匹配 GPU

#### 4. 真实任务执行器
- 支持真实进程启动、超时控制、日志收集
- Windows 下支持进程树清理（`taskkill /T /F`）
- 自动注入 GPU 隔离环境变量（`CUDA_VISIBLE_DEVICES` / `ROCR_VISIBLE_DEVICES`）

#### 5. REST API + Web Dashboard
- 完整的 RESTful API（任务提交、停止、查询、日志）
- 内置 Web 可视化面板（Chart.js 实时图表）
- Prometheus `/metrics` 端点（可接入 Grafana）

---

### 快速开始

```bash
# 下载 Release（Windows / Linux）
# https://github.com/cpufreestyle/hybrid-gpu-scheduler/releases/tag/v1.0.0

# 启动调度器
./hybrid-gpu-scheduler

# 提交任务
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "name": "training-job",
    "type": "training",
    "vram_mb": 4096,
    "command": "python train.py"
  }'

# 打开 Dashboard
open http://localhost:8080/dashboard
```

---

### 技术栈

- **语言**：Go 1.22+
- **Web 框架**：Gin
- **监控**：Prometheus client_golang
- **前端**：HTML + CSS + Chart.js
- **GPU 监控**：nvidia-smi / rocm-smi

---

### 适用场景

- 深度学习训练 + 推理混合部署
- 多 GPU 异构环境（NVIDIA + AMD）
- 小团队 GPU 资源共享
- 个人 AI 工作站

---

### 开源协议

MIT License

**欢迎 Star / Fork / 提 Issue！**

GitHub: https://github.com/cpufreestyle/hybrid-gpu-scheduler

---