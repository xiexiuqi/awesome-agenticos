# SWE 负载分析与 AI Agent 性能评测指导

## 目录

1. [SWE-bench 系列基准测试概览](#一-swe-bench-系列基准测试概览)
2. [AI Agent 观测与测试方法](#二-ai-agent-观测与测试方法)
3. [业界测试方法对比](#三-业界测试方法对比)
4. [评价指标体系](#四-评价指标体系)
5. [AI Agent 基础设施负载特征分析](#五-ai-agent-基础设施负载特征分析)
6. [负载选择与监测实践](#六-负载选择与监测实践)
7. [工具与资源汇总](#七-工具与资源汇总)

---

## 一、SWE-bench 系列基准测试概览

### 1.1 测试体系演进

| 基准测试 | 年份 | 任务数量 | 核心特点 | 适用场景 |
|---------|------|---------|---------|---------|
| **SWE-bench** | 2023 | ~2,294 | 首个执行式评测，真实 GitHub issue/PR | 基线评测 |
| **SWE-bench Lite** | 2024 | 300 | 精简子集，成本更低 | 快速验证 |
| **SWE-bench Verified** | 2024 | 500 | 人工筛选，质量更高 | 可信评测 |
| **SWE-bench Multimodal** | 2024 | 517 | 包含视觉元素 | 多模态评测 |
| **SWE-ReBench** | 2025 | 21,000+ | 时间门控去污染，自动挖掘 | 真实能力评测 + RL 训练 |
| **SWE-bench Pro** | 2025 | 1,865 | 企业级复杂任务，41 个仓库 | 长时程开发 |
| **SWE-Bench++** | 2025 | 自动扩展 | 4 层 AutoQA，环境自动合成 | 可扩展评测 |

### 1.2 核心评测指标

**主要指标：**

| 指标 | 说明 | 计算方式 |
|------|------|---------|
| **% Resolved (Pass@1)** | 首次尝试成功修复的百分比 | 通过所有测试的实例 / 总实例 |
| **Pass@k** | k 次尝试内找到正确解的比例 | 评估搜索效率 |
| **Patch Apply Rate** | 补丁语法应用成功率 | 成功应用的补丁 / 生成的补丁 |
| **Localization Success** | 修改文件与 gold patch 匹配率 | 文件级准确率 |

**测试验证机制：**

- **FAIL_TO_PASS**: 修复前失败、修复后通过的测试（验证修复有效性）
- **PASS_TO_PASS**: 修复前后均通过的测试（验证无回归）
- **Resolved**: 同时满足上述两项的综合指标

### 1.3 基准污染问题

SWE-ReBench 提出的核心问题：

| 模型 | SWE-bench | SWE-ReBench | 差距 | 分析 |
|------|-----------|-------------|------|------|
| MiniMax M2.5 | ~80.2% | ~39.6% | **~40 个百分点** | 可能存在"背题" |
| Claude Opus 4.6 | ~80.8% | ~51.7% | ~29 个百分点 | 部分记忆依赖 |
| DeepSeek-V3 | 一致 | 一致 | 接近 | 泛化能力真实 |

> 差距大的模型更可能是记忆而非真正具备解决能力。

### 1.4 典型运行时间参考

| 阶段 | 耗时 | 说明 |
|------|------|------|
| Base 镜像构建 | ~1 分钟 | 首次运行，后续复用 |
| Env 镜像构建 | ~5-7 分钟 | conda 环境 + 依赖安装 |
| Instance 镜像构建 | ~1-2 分钟 | 复制仓库 + checkout commit |
| 测试执行 | ~1-2 分钟 | 应用 patch + 运行测试 |
| **总计** | **~8-12 分钟/用例** | 首次运行 |

---

## 二、AI Agent 观测与测试方法

### 2.1 观测层次架构

```
┌─────────────────────────────────────┐
│  Layer 4: 业务指标层                 │  ← 输出质量、单次任务成本
│  (自定义仪表盘)                      │
├─────────────────────────────────────┤
│  Layer 3: LLM 专用监控               │  ← Token 使用、延迟、每次调用成本
│  (Langfuse, Helicone, LangSmith)    │
├─────────────────────────────────────┤
│  Layer 2: 基础设施监控               │  ← CPU、内存、磁盘、网络
│  (Datadog, Grafana, eBPF+cgroup)    │     + 增强型 cgroup 指标
├─────────────────────────────────────┤
│  Layer 1: Agent 原生监控             │  ← 心跳、任务状态、工具调用追踪
│  (AgentCenter, 自定义)               │     + AgentCgroup 式资源控制
└─────────────────────────────────────┘
```

### 2.2 AgentSight：系统级可观测性框架

**核心创新：** 基于 eBPF 的边界追踪（Boundary Tracing）

| 特性 | 说明 |
|------|------|
| **技术** | eBPF（extended Berkeley Packet Filter）内核级追踪 |
| **方法** | 从应用外部监控 Agent，零侵入 |
| **关键能力** | 拦截 TLS 加密的 LLM 流量，提取语义意图 |
| **系统监控** | 追踪内核事件、系统调用、文件操作、子进程执行 |
| **性能开销** | **<3%** 平均（工作流中实测 2.9%） |
| **插桩需求** | 零插桩（框架无关） |

**AgentSight 检测内容：**

- **Prompt 注入攻击** — 检测恶意重定向
- **资源浪费的推理循环** — Agent 陷入低效思考循环
- **多 Agent 协调瓶颈** — 分布式系统中的隐藏延迟
- **语义意图到系统动作的关联** — 桥接高层目标与底层操作

**性能基准：**

| 任务 | 基线 | AgentSight | 开销 |
|------|------|-----------|------|
| 理解仓库 | 127.98s | 132.33s | 3.4% |
| 代码编写 | 22.54s | 23.64s | 4.9% |
| 仓库编译 | 92.40s | 92.72s | 0.4% |

### 2.3 主流观测工具对比（2026）

| 平台 | 开销 | 特点 | 适用场景 |
|------|------|------|---------|
| **LangSmith** | ~0% | 最轻量追踪，最小同步工作 | LangChain 生态 |
| **Laminar** | ~5% | 无内联评估开销 | 通用追踪 |
| **AgentOps** | ~12% | 生命周期级监控 | 生产环境 |
| **Langfuse** | ~15% | 最深度的步骤级插桩 | 详细追踪需求 |
| **AgentSight** | ~3% | 内核级语义感知 | 安全 + 性能分析 |

---

## 三、业界测试方法对比

### 3.1 评测框架对比

| 框架 | 评测对象 | 环境 | 核心指标 | 特点 |
|------|---------|------|---------|------|
| **SWE-bench** | 代码修复 | Docker | Resolved, Pass@k | 真实 GitHub issue |
| **WebArena** | 网页操作 | 真实网站 | 任务完成率 | 端到端 Web 交互 |
| **AgentBench** | 通用 Agent | 多环境 | 成功率、效率 | 8 种环境（OS、DB、Web 等） |
| **OSWorld** | OS 操作 | 虚拟机 | 任务完成率 | 桌面操作系统交互 |
| **Terminal-Bench** | 终端操作 | 容器 | 命令准确率 | 命令行任务 |
| **Tau-Bench** | 多轮对话 | 模拟环境 | 用户满意度 | 对话式 Agent |

### 3.2 测试方法分类

**按评测深度：**

| 方法 | 描述 | 优点 | 缺点 |
|------|------|------|------|
| **黑盒测试** | 仅观察输入输出 | 简单、快速 | 无法定位问题 |
| **灰盒测试** | 观察工具调用序列 | 了解执行过程 | 需要 Agent 支持 |
| **白盒测试** | 全链路追踪（eBPF） | 精确定位瓶颈 | 开销较高、复杂 |

**按测试场景：**

| 场景 | 方法 | 关键指标 |
|------|------|---------|
| 功能正确性 | 单元测试 + 集成测试 | 通过率、覆盖率 |
| 性能基准 | 压力测试 + 负载测试 | 延迟、吞吐量、资源使用 |
| 稳定性 | 长时间运行测试 | 错误率、恢复时间 |
| 安全性 | 对抗测试 | 攻击成功率 |

### 3.3 关键研究发现

**AgentCgroup 论文核心发现（144 个 SWE-rebench 任务）：**

| 发现 | 数值 | 测试启示 |
|------|------|---------|
| OS 级执行占端到端延迟 | **56-74%** | 必须监控容器启动时间，不只是推理时间 |
| 并发瓶颈 | **内存，非 CPU** | 重点监控 `memory.peak` |
| 内存峰均比 | **15.4×** | 需要区分推理阶段和工具执行阶段 |
| Burst 持续时间 | **1-2 秒** | 采样频率必须 >= 1Hz |
| 同任务多次运行方差 | **1.8×** | 每个任务至少跑 3 次取统计分布 |
| 内存变化率 | **最高 3GB/s** | 需要亚秒级监控 |

**"The Cost of Dynamic Reasoning" 论文发现：**

| Agent 类型 | 相比单轮 LLM 延迟 | 相比单轮 LLM 能耗 |
|-----------|------------------|------------------|
| Reflexion (8B) | **153.7× 更慢** | **130.9× 更高** |
| LATS (8B) | **90.1× 更慢** | **17.7× 更高** |

---

## 四、评价指标体系

### 4.1 功能指标

| 指标 | 定义 | 测量方法 |
|------|------|---------|
| **任务成功率** | 成功完成任务的比例 | 自动化测试验证 |
| **步骤准确率** | 每个工具调用的正确率 | 与 gold trace 对比 |
| **代码质量** | 生成代码的可维护性 | 静态分析工具 |
| **回归率** | 引入新问题的比例 | PASS_TO_PASS 测试 |

### 4.2 性能指标

| 指标 | 定义 | 测量方法 |
|------|------|---------|
| **端到端延迟** | 从输入到输出的总时间 | 时间戳差 |
| **推理延迟** | LLM 生成时间 | API 返回时间 |
| **工具执行延迟** | 工具调用耗时 | 工具 wrapper 计时 |
| **首 token 延迟** | 第一个 token 返回时间 | Streaming API 计时 |
| **吞吐量** | 单位时间完成任务数 | 任务数 / 时间 |

### 4.3 资源指标

| 指标 | 定义 | 测量方法 |
|------|------|---------|
| **CPU 使用率** |  CPU 时间占比 | `cpu.stat` / `top` |
| **内存使用** |  RSS / Peak RSS | `memory.current` / `memory.peak` |
| **内存峰均比** | 峰值 / 平均值 | 连续采样计算 |
| **磁盘 I/O** | 读写带宽 | `iostat` / `blkio.throttle` |
| **网络 I/O** | 收发带宽 | `eth0` 统计 |

### 4.4 成本指标

| 指标 | 定义 | 测量方法 |
|------|------|---------|
| **Token 消耗** | 输入 + 输出 token 数 | API 返回或 tokenizer 计算 |
| **API 成本** | 单次调用费用 | 单价 × token 数 |
| **基础设施成本** | 计算资源费用 | 云厂商计费 |
| **单次任务成本** | 完整任务总费用 | 汇总所有调用 |

### 4.5 多维度评分卡

```
┌─────────────────────────────────────────────────┐
│           Agent 性能评分卡                        │
├─────────────────────────────────────────────────┤
│  功能正确性    ████████░░  80%  (Resolved)      │
│  执行效率      ██████░░░░  60%  (延迟 < 阈值)    │
│  资源效率      █████░░░░░  50%  (内存优化空间)    │
│  成本效益      ███████░░░  70%  (Token 效率)     │
│  稳定性        ████████░░  80%  (错误率 < 5%)    │
├─────────────────────────────────────────────────┤
│  综合评分      ███████░░░  68%                  │
└─────────────────────────────────────────────────┘
```

---

## 五、AI Agent 基础设施负载特征分析

### 5.1 负载特征概述

AI Agent 工作负载与传统工作负载有本质区别：

| 特征 | 传统工作负载 | AI Agent 工作负载 |
|------|------------|------------------|
| **执行模式** | 持续运行 | Burst-Silence 交替 |
| **资源需求** | 相对平稳 | 剧烈波动 |
| **可预测性** | 历史可预测 | 非确定性 |
| **状态管理** | 无状态或可丢弃 | 有状态（LLM 上下文） |
| **错误处理** | 重启即可 | 重启丢失上下文 |
| **粒度** | 进程级 | 工具调用级 |

### 5.2 关键负载特征

**1. Burst-Silence 模式**

```
内存使用
  │
4GB├                    ╱╲
  │                   ╱  ╲
  │    ╱╲            ╱    ╲
2GB├───╱  ╲──────────╱      ╲─────
  │  ╱    ╲        ╱        ╲
1GB├─╱      ╲──────╱          ╲───
  │╱        ╲    ╱
  ├──────────╲──╱──────────────────
  │  推理    工具   推理    工具
  └─────────────────────────────────→ 时间
```

- 98.5% 的内存 burst 发生在工具调用期间
- 工具调用仅占 50.6% 的执行时间
- Burst 持续 1-2 秒，变化率可达 3GB/s

**2. 内存主导瓶颈**

| 模型 | 平均 CPU | 基线内存 | Burst 内存 | 峰均比 |
|------|---------|---------|-----------|--------|
| Haiku | 13.2% | 185 MB | 500 MB - 4 GB | 15.4× |
| GLM | 7.6% | 185 MB | 500 MB - 4 GB | 15.4× |

**3. 非确定性**

- 同任务多次运行：1.8× 方差
- 不同任务间：20× 峰值内存差异
- 不同模型：资源曲线完全不同

### 5.3 与传统工作负载对比

| 工作负载类型 | 峰均比 | 可预测性 | 状态敏感度 |
|-----------|--------|---------|-----------|
| **Serverless** | ~1.5× | 中 | 低 |
| **Microservice** | 2-3× | 高 | 中 |
| **Batch** | 5-10× | 高 | 低 |
| **AI Agent** | **15.4×** | **低** | **高** |

### 5.4 三大不匹配问题

AgentCgroup 论文识别的核心问题：

| 不匹配 | 问题描述 | 传统方法 | Agent 现实 |
|--------|---------|---------|-----------|
| **粒度不匹配** | 容器级策略 vs 工具调用级动态 | 每个容器一个策略 | 每次工具调用资源不同 |
| **响应性不匹配** | 用户空间反应太慢 | 毫秒到分钟级 | 亚秒级不可预测 burst |
| **适应性不匹配** | 基于历史的预测失效 | 杀死重启可接受 | 破坏累积的 LLM 上下文 |

---

## 六、负载选择与监测实践

### 6.1 负载选择策略

**选择原则：**

1. **代表性**：覆盖不同复杂度、不同领域
2. **可扩展性**：支持从小规模到大规模
3. **可重复性**：固定种子，多次运行取平均
4. **多样性**：包含成功、失败、边界案例

**SWE-bench 任务选择方法：**

```python
# 论文中的 18 任务选择（6 类别 × 3 难度）
task_categories = {
    "sympy": ["sympy__sympy-11232", "sympy__sympy-12419", "sympy__sympy-13480"],
    "django": ["django__django-10087", "django__django-11011", "django__django-12209"],
    "matplotlib": ["matplotlib__matplotlib-22232", "matplotlib__matplotlib-23314", "matplotlib__matplotlib-24149"],
    "scikit": ["scikit__scikit-10297", "scikit__scikit-12471", "scikit__scikit-13142"],
    "pytest": ["pytest__pytest-5631", "pytest__pytest-6197", "pytest__pytest-7356"],
    "httpx": ["encode__httpx-2701", "encode__httpx-852", "encode__httpx-1182"]
}
```

**分层采样策略：**

| 层级 | 采样目的 | 任务数 |
|------|---------|--------|
|  smoke test | 快速验证 | 1-2 |
|  回归测试 | 防止退化 | 10-20 |
|  性能基准 | 资源分析 | 50-100 |
|  全量测试 | 完整评估 | 300+ |

### 6.2 监测指标体系

**端到端延迟分解：**

```
总延迟 = 容器启动 + Agent 初始化 + 推理时间 + 工具执行 + 后处理
        │           │            │          │          │
        │           │            │          │          └─ 5%
        │           │            │          └─ 30-50% (工具调用)
        │           │            └─ 20-40% (LLM 推理)
        │           └─ 5-10% (Agent 框架)
        └─ 10-20% (容器冷启动)
```

**资源监测维度：**

| 维度 | 指标 | 采样频率 | 工具 |
|------|------|---------|------|
| **时间** | 各阶段延迟 | 事件驱动 | 应用内计时 |
| **CPU** | 使用率、调度延迟 | 1-10 Hz | `cpu.stat`, `perf` |
| **内存** | RSS、Peak、Swap | >= 1 Hz | `memory.current`, `memory.peak` |
| **磁盘** | IOPS、带宽 | 1 Hz | `iostat`, `blkio` |
| **网络** | 带宽、延迟 | 1 Hz | `ss`, `nstat` |
| **工具** | 调用次数、耗时 | 事件驱动 | Agent trace |
| **Token** | 输入/输出数量 | 每次调用 | API 返回 |

### 6.3 监测工具配置

**方案 1：Podman stats（论文使用）**

```bash
# 1Hz 采样
watch -n 1 podman stats --no-stream --format "{{.Name}}\t{{.MemUsage}}\t{{.CPUPerc}}"

# 输出到文件
while true; do
    podman stats --no-stream --format "json" >> stats.json
    sleep 1
done
```

**方案 2：cgroup v2 直接读取（更精确）**

```bash
# 内存
CONTAINER_ID="your-container-id"
echo "timestamp,memory.current,memory.peak,memory.swap.current"
while true; do
    TS=$(date +%s.%N)
    MEM=$(cat /sys/fs/cgroup/machine.slice/libpod-$CONTAINER_ID.scope/memory.current)
    PEAK=$(cat /sys/fs/cgroup/machine.slice/libpod-$CONTAINER_ID.scope/memory.peak 2>/dev/null || echo 0)
    SWAP=$(cat /sys/fs/cgroup/machine.slice/libpod-$CONTAINER_ID.scope/memory.swap.current)
    echo "$TS,$MEM,$PEAK,$SWAP"
    sleep 1
done > memory.csv
```

**方案 3：eBPF 高级追踪（AgentSight 风格）**

```bash
# 使用 bpftrace 追踪系统调用
bpftrace -e '
tracepoint:syscalls:sys_enter_execve {
    printf("execve: pid=%d, comm=%s, arg=%s\n", pid, comm, str(args->argv[0]));
}

tracepoint:syscalls:sys_enter_clone {
    printf("clone: pid=%d, comm=%s\n", pid, comm);
}
'
```

**方案 4：综合监控脚本**

```python
#!/usr/bin/env python3
"""Agent 资源监控器"""

import time
import json
import subprocess
from datetime import datetime
from pathlib import Path

class AgentMonitor:
    def __init__(self, container_id: str, output_dir: str = "./metrics"):
        self.container_id = container_id
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
        self.samples = []
        
    def sample(self) -> dict:
        """采集一次资源样本"""
        ts = time.time()
        
        # Podman stats
        result = subprocess.run(
            ["podman", "stats", "--no-stream", "--format", "json", self.container_id],
            capture_output=True, text=True
        )
        stats = json.loads(result.stdout)[0] if result.stdout else {}
        
        # cgroup v2 直接读取（更精确）
        cgroup_path = f"/sys/fs/cgroup/machine.slice/libpod-{self.container_id}.scope"
        
        sample = {
            "timestamp": ts,
            "datetime": datetime.now().isoformat(),
            "memory": {
                "current": self._read_cgroup(f"{cgroup_path}/memory.current"),
                "peak": self._read_cgroup(f"{cgroup_path}/memory.peak"),
                "swap": self._read_cgroup(f"{cgroup_path}/memory.swap.current"),
            },
            "cpu": {
                "usage": self._read_cgroup(f"{cgroup_path}/cpu.stat"),
            },
            "podman_stats": stats
        }
        
        self.samples.append(sample)
        return sample
    
    def _read_cgroup(self, path: str) -> int:
        try:
            with open(path) as f:
                return int(f.read().strip())
        except:
            return 0
    
    def run(self, interval: float = 1.0, duration: float = None):
        """持续监控"""
        start = time.time()
        try:
            while True:
                self.sample()
                if duration and (time.time() - start) >= duration:
                    break
                time.sleep(interval)
        except KeyboardInterrupt:
            pass
        finally:
            self.save()
    
    def save(self):
        """保存到文件"""
        output_file = self.output_dir / f"metrics_{int(time.time())}.jsonl"
        with open(output_file, "w") as f:
            for sample in self.samples:
                f.write(json.dumps(sample) + "\n")
        print(f"Saved {len(self.samples)} samples to {output_file}")

if __name__ == "__main__":
    import sys
    monitor = AgentMonitor(sys.argv[1])
    monitor.run(interval=1.0)
```

### 6.4 Token 消耗监测

**API 级别追踪：**

```python
import functools
import time

class TokenTracker:
    def __init__(self):
        self.calls = []
    
    def track(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            latency = time.time() - start
            
            # 从 API 响应中提取 token 信息
            usage = result.get("usage", {})
            self.calls.append({
                "timestamp": start,
                "model": result.get("model", "unknown"),
                "prompt_tokens": usage.get("prompt_tokens", 0),
                "completion_tokens": usage.get("completion_tokens", 0),
                "total_tokens": usage.get("total_tokens", 0),
                "latency_ms": latency * 1000,
                "cost_usd": self._calculate_cost(usage, result.get("model"))
            })
            return result
        return wrapper
    
    def _calculate_cost(self, usage: dict, model: str) -> float:
        # 基于 OpenAI/Anthropic 定价
        pricing = {
            "gpt-4": {"input": 0.03, "output": 0.06},
            "claude-3-haiku": {"input": 0.00025, "output": 0.00125},
        }
        p = pricing.get(model, {"input": 0, "output": 0})
        return (usage.get("prompt_tokens", 0) * p["input"] + 
                usage.get("completion_tokens", 0) * p["output"]) / 1000
    
    def summary(self) -> dict:
        total_calls = len(self.calls)
        total_tokens = sum(c["total_tokens"] for c in self.calls)
        total_cost = sum(c["cost_usd"] for c in self.calls)
        avg_latency = sum(c["latency_ms"] for c in self.calls) / total_calls if total_calls else 0
        
        return {
            "total_calls": total_calls,
            "total_tokens": total_tokens,
            "total_cost_usd": total_cost,
            "avg_latency_ms": avg_latency,
            "tokens_per_call": total_tokens / total_calls if total_calls else 0
        }
```

### 6.5 数据分析方法

**时序分析：**

```python
import pandas as pd
import numpy as np

def analyze_resource_trace(metrics_file: str) -> dict:
    """分析资源追踪数据"""
    df = pd.read_json(metrics_file, lines=True)
    
    # 内存分析
    mem_current = df["memory.current"] / (1024**2)  # MB
    
    analysis = {
        "memory": {
            "baseline_mb": mem_current.quantile(0.1),  # 10% 分位数作为基线
            "peak_mb": mem_current.max(),
            "mean_mb": mem_current.mean(),
            "peak_to_mean_ratio": mem_current.max() / mem_current.mean(),
            "burst_count": len(find_bursts(mem_current)),
        },
        "latency": {
            "total_duration_s": df["timestamp"].max() - df["timestamp"].min(),
        }
    }
    
    return analysis

def find_bursts(series: pd.Series, threshold_std: float = 2.0) -> list:
    """识别 burst 事件"""
    mean = series.mean()
    std = series.std()
    threshold = mean + threshold_std * std
    
    bursts = []
    in_burst = False
    burst_start = None
    
    for idx, value in series.items():
        if value > threshold and not in_burst:
            in_burst = True
            burst_start = idx
        elif value <= threshold and in_burst:
            in_burst = False
            bursts.append({
                "start": burst_start,
                "end": idx,
                "duration": idx - burst_start,
                "peak": series[burst_start:idx].max()
            })
    
    return bursts
```

---

## 七、工具与资源汇总

### 7.1 开源工具

| 工具 | 用途 | 链接 |
|------|------|------|
| **SWE-bench** | 代码修复基准测试 | https://github.com/swe-bench/SWE-bench |
| **SWE-ReBench** | 去污染增强版 | https://github.com/nebius/SWE-rebench |
| **AgentSight** | eBPF 系统级观测 | https://github.com/eunomia-bpf/agentsight |
| **AgentCgroup** | eBPF 资源控制 | https://github.com/eunomia-bpf/agentcgroup |
| **AgentLedger** | 成本追踪 | https://github.com/WDZ-Dev/agent-ledger |
| **bpftrace** | eBPF 追踪 | https://github.com/iovisor/bpftrace |
| **Helicone** | LLM 可观测性 | https://github.com/Helicone/helicone |
| **Langfuse** | 开源 LLM 追踪 | https://github.com/langfuse/langfuse |

### 7.2 商业工具

| 工具 | 特点 | 定价 |
|------|------|------|
| **LangSmith** | LangChain 原生 | $39/seat/月 |
| **AgentOps** | 生命周期监控 | 按量计费 |
| **Datadog** | 全栈可观测性 | $8-12/10K 请求 |
| **Arize AI** | ML 可观测性 | 企业定制 |

### 7.3 关键论文

| 论文 | 作者 | 年份 | 核心贡献 |
|------|------|------|---------|
| AgentCgroup: Understanding and Controlling OS Resources of AI Agents | Zheng et al. | 2026 | 首次系统表征 Agent OS 资源动态 |
| AgentSight: System-Level Observability for AI Agents Using eBPF | Cardoz et al. | 2025 | eBPF 边界追踪 |
| The Cost of Dynamic Reasoning | - | 2025 | Agent 能耗分析 |
| SWE-bench: Can Language Models Resolve Real-World GitHub Issues? | Jimenez et al. | 2023 | 首个代码修复基准 |
| SWE-rebench: Automated Pipeline for Decontaminated Evaluation | Badertdinov et al. | 2025 | 去污染评测 |

### 7.4 快速参考：监控命令速查

```bash
# 容器资源监控
podman stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# cgroup v2 内存
cat /sys/fs/cgroup/$CGROUP_PATH/memory.current
cat /sys/fs/cgroup/$CGROUP_PATH/memory.peak
cat /sys/fs/cgroup/$CGROUP_PATH/memory.stat

# cgroup v2 CPU
cat /sys/fs/cgroup/$CGROUP_PATH/cpu.stat

# 进程级监控
pidstat -r -u -p $PID 1

# eBPF 系统调用追踪
bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s\n", str(args->argv[0])); }'

# 网络连接
ss -tanp | grep $PID

# 磁盘 I/O
iotop -p $PID
```

---

## 附录：实验设计模板

### A.1 单任务资源分析实验

```yaml
experiment:
  name: "swe-bench-single-task-resource-analysis"
  task: "sympy__sympy-11232"
  model: "claude-3-haiku"
  
  monitoring:
    sample_rate_hz: 1
    metrics:
      - memory.current
      - memory.peak
      - cpu.usage
      - disk.io
      - network.io
      - tool_calls
      - token_usage
  
  repetitions: 5
  
  analysis:
    - baseline_memory
    - peak_memory
    - burst_patterns
    - latency_breakdown
    - token_efficiency
```

### A.2 多租户并发实验

```yaml
experiment:
  name: "multi-tenant-concurrency-test"
  tasks: ["task1", "task2", "task3"]
  concurrency_levels: [1, 2, 4, 8]
  
  metrics:
    - throughput_tasks_per_minute
    - p50_latency
    - p95_latency
    - p99_latency
    - memory_contention_ratio
    - oom_events
```

---

*本文档基于 AgenticOS 知识库中的 SWE-bench、SWE-ReBench、AgentCgroup、AgentSight 等资料整理，持续更新中。*
