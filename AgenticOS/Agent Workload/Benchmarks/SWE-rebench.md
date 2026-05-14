# SWE-bench vs SWE-ReBench 分析与国内部署指南

  

## 一、SWE-bench vs SWE-ReBench 的关系与区别

  

### 核心关系

  

**SWE-ReBench 是 SWE-bench 的"去污染增强版"**，两者是**继承而非替代**关系：

  

| 维度 | SWE-bench (原始版, 2023) | SWE-ReBench (增强版, 2025) |

|------|------------------------|---------------------------|

| **作者** | Princeton (Jimenez et al.) | Nebius AI / TractoAI (Badertdinov et al.) |

| **任务数量** | ~2,300 个，12 个仓库 | **21,000+** 个，数千个仓库 |

| **构建方式** | 人工筛选 GitHub issue/PR | **全自动流水线**挖掘可执行任务 |

| **核心创新** | 首次提出执行式评测 | **时间门控去污染**（按模型训练截止日期过滤） |

| **防作弊能力** | 弱（模型可能记忆过旧 issue 的解决方案） | **强**（任务日期晚于模型 cutoff，无法靠记忆） |

| **Docker 镜像** | 官方提供 | **官方提供**（论文中使用的正是这些镜像） |

| **适用场景** | 基线评测 | 真实能力评测 + 支持 RL 训练 |

  

### 论文中的使用方式

  

论文 `main.tex:104` 同时引用了两者：

  

```latex

We collect 111 tasks with GLM and 33 tasks with Haiku from SWE-rebench~\cite{swe-rebench,swe-bench}

```

  

这表明论文**实际使用的是 SWE-ReBench 的 Docker 镜像和 HuggingFace 数据集**，但承认其源于 SWE-bench 的体系。具体使用的 18 个代表性任务定义在 `scripts/batch_test_swebench.py:32-134` 中。

  

### 关键差异：Benchmark Contamination（基准污染）

  

这是 SWE-ReBench 被提出的核心原因。某些模型在两个基准上的得分差距巨大：

  

| 模型 | SWE-bench | SWE-ReBench | 差距 |

|------|-----------|-------------|------|

| MiniMax M2.5 | ~80.2% | ~39.6% | **~40 个百分点** |

| Claude Opus 4.6 | ~80.8% | ~51.7% | ~29 个百分点 |

  

> 差距大的模型更可能是"背题"而非真正具备解决能力。DeepSeek-V3 在两个基准上表现一致，说明其泛化能力真实。

  

---

  

## 二、部署与测试架构（基于论文代码）

  

论文的测试架构分为三层：

  

```

┌─────────────────────────────────────────┐

│  Host OS (Ubuntu 24.04, 128GB RAM)      │

│  ├── Podman (rootless containers)       │

│  ├── Claude Code CLI (Node.js agent)    │

│  └── Resource Monitor (1秒采样)          │

└─────────────────────────────────────────┘

                    │

    ┌───────────────┼───────────────┐

    ▼               ▼               ▼

┌────────┐    ┌────────┐    ┌────────┐

│ Task 1 │    │ Task 2 │    │ Task N │  ← SWE-ReBench Docker images

│Container│    │Container│    │Container│    (2.9-17.3 GB each)

│/testbed │    │/testbed │    │/testbed │    (含 Python 项目 + issue)

└────────┘    └────────┘    └────────┘

```

  

### 核心脚本

  

| 脚本 | 功能 | 位置 |

|------|------|------|

| `run_swebench.py` | 单任务运行 + 资源监控 | `scripts/run_swebench.py` |

| `batch_test_swebench.py` | 18 任务批量运行（6 类别 x 3 难度） | `scripts/batch_test_swebench.py` |

| `verify_sample_tasks.py` | 验证 Docker 镜像存在性 | `scripts/verify_sample_tasks.py` |

| `plot_resources.py` | 资源使用可视化 | `scripts/plot_resources.py` |

  

### 单任务运行流程（`run_swebench.py`）

  

1. **拉取镜像**：`podman pull docker.io/swerebench/sweb.eval.x86_64.*`

2. **修复权限**：创建新镜像，将 `/testbed` 目录权限改为当前用户

3. **启动容器**：使用 `--userns=keep-id` 映射用户，挂载主机 `/usr`, `/lib`, `/home` 等目录

4. **运行 Agent**：在容器内执行 `claude --model haiku --print --dangerously-skip-permissions "<prompt>"`

5. **资源监控**：后台线程每秒执行 `podman stats` 采样内存/CPU

6. **收集 Trace**：从 `~/.claude/projects/-testbed/` 复制 JSONL trace 文件

7. **解析工具调用**：从 trace 中提取 Bash/Read/Edit/Task 等工具的调用时序

  

---

  

## 三、国内环境部署建议

  

### 挑战分析

  

在国内复现论文测试，主要面临以下障碍：

  

| 挑战 | 具体表现 | 影响程度 |

|------|---------|---------|

| **Docker 镜像拉取** | `docker.io/swerebench/*` 可能被墙或极慢 | 高 |

| **HuggingFace 数据集** | `nebius/SWE-rebench` 需翻墙访问 | 高 |

| **Claude Code CLI** | 需 Anthropic API key，国内访问受限 | 高 |

| **模型选择** | Haiku (云端) 或 GLM (本地 GPU) | 中 |

| **Podman 替代 Docker** | 论文使用 Podman，国内 Docker 更常见 | 低 |

  

### 具体部署方案

  

#### 方案 A：全量复现（需要海外环境或代理）

  

```bash

# 1. 环境准备（Ubuntu 24.04 推荐）

sudo apt update && sudo apt install -y podman podman-compose jq

  

# 2. 配置镜像加速（解决 docker.io 拉取问题）

# 编辑 /etc/containers/registries.conf，添加镜像站

[[registry]]

prefix = "docker.io"

location = "docker.mirrors.sjtug.sjtu.edu.cn"  # 或 mirrors.tuna.tsinghua.edu.cn

  

# 3. 验证单个任务

python scripts/run_swebench.py \

    swerebench/sweb.eval.x86_64.encode_1776_httpx-2701 \

    --memory 8g --cpus 4 --model haiku

  

# 4. 批量运行 18 个代表性任务

python scripts/batch_test_swebench.py

```

  

#### 方案 B：国内友好版（推荐）——替换为国产模型

  

论文中使用了两种模型配置，**GLM 本地部署方案更适合国内**：

  

```python

# scripts/run_swebench.py 中模型参数

# --model haiku    # 云端 API，需翻墙

# 改为本地模型，如 GLM-4、Qwen、DeepSeek 等

```

  

**修改点**：

  

1. **替换 Claude Code 的模型后端**：

   - Claude Code 默认只支持 Anthropic 模型

   - 需要改用支持国产模型的 Agent 框架，如：

     - **AutoCode** / **CodeFuse**（阿里）

     - **DevOpsGPT**（开源）

     - 或自行基于 LangChain/LlamaIndex 搭建

  

2. **使用国内镜像源拉取 Docker 镜像**：

   ```bash

   # 配置 Podman 使用国内镜像站

   mkdir -p ~/.config/containers

   cat > ~/.config/containers/registries.conf << 'EOF'

   [[registry]]

   prefix = "docker.io"

   location = "docker.mirrors.ustc.edu.cn"

   insecure = true

   EOF

   ```

  

3. **数据集下载**：

   ```python

   # 使用 HF-Mirror 下载 SWE-ReBench 数据集

   # 修改 scripts/verify_sample_tasks.py:165

   # 原：load_dataset("nebius/SWE-rebench", split="filtered")

   # 改为使用镜像：

   import os

   os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"

   dataset = load_dataset("nebius/SWE-rebench", split="filtered")

   ```

  

#### 方案 C：最小化验证（仅测资源占用，不关注修复正确性）

  

如果你**只关心 Agent 对操作系统资源的占用模式**（这正是论文的核心贡献），可以大幅简化：

  

```bash

# 1. 只拉取 1-2 个镜像验证环境

podman pull docker.io/swerebench/sweb.eval.x86_64.encode_1776_httpx-2701

  

# 2. 在容器内运行一个简单的"伪 Agent"脚本，模拟工具调用模式

# 即：循环执行 {sleep(模拟推理) -> bash 命令(模拟工具) -> sleep -> bash...}

  

# 3. 用 podman stats 或 cgroup v2 接口采集资源数据

# 论文的核心发现（burst-silence 模式、memory 两层结构）不依赖 LLM 是否真的能修复 bug

```

  

---

  

## 四、资源监控核心要点

  

论文的关键发现对你设计测试非常有价值：

  

| 发现 | 对你测试的启示 |

|------|--------------|

| **56-74% 时间花在 OS 执行**（容器启动 + 工具执行） | 测试时必须监控容器启动时间，不只是 Agent 推理时间 |

| **内存是并发瓶颈**（非 CPU） | 重点监控 `memory.peak`，CPU 采样间隔可以放宽 |

| **两层内存结构：185MB 基线 + 工具触发 burst** | 需要区分"推理阶段"和"工具执行阶段"的内存 |

| **Burst 仅持续 1-2 秒，变化率可达 3GB/s** | **采样频率必须 >= 1Hz**，否则捕获不到峰值 |

| **同任务多次运行资源曲线不同（1.8x 方差）** | 每个任务至少跑 3 次取统计分布 |

  

### 推荐的监控命令

  

```bash

# 方案 1：Podman stats（论文使用）

watch -n 1 podman stats --no-stream --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}"

  

# 方案 2：直接读 cgroup v2（更精确，无 Podman 开销）

cat /sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/app.slice/container_id/memory.current

cat /sys/fs/cgroup/.../memory.stat

cat /sys/fs/cgroup/.../cpu.stat

  

# 方案 3：使用论文的 ResourceMonitor 类（已封装）

# 见 scripts/run_swebench.py:67-178

```

  

---

  

## 五、总结建议

  

| 你的目标 | 推荐方案 | 关键动作 |

|---------|---------|---------|

| **完全复现论文** | 方案 A + 海外服务器/代理 | 配置 Podman、拉镜像、申请 Anthropic API key |

| **评估国产 Agent 资源占用** | 方案 B | 替换模型后端、用 HF-Mirror、保持监控逻辑不变 |

| **快速验证资源模式假设** | 方案 C | 只跑 1-2 个镜像、用伪 Agent 模拟工具调用循环 |

| **大规模并发测试** | 基于论文发现优化 | 关注内存 burst、避免静态 limit、考虑动态扩缩容 |