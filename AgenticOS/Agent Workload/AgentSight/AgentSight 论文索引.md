# AgentSight 论文索引

> 本目录收录与 AI Agent 可观测性、性能分析、资源调度、治理安全等相关的研究论文。
> 生成日期：2026-05-15

---

## 目录结构

```
AgentSight/
├── 01-Observability/                    # Agent 可观测性
├── 02-Benchmarking/                     # Agent 评测与基准测试
├── 03-Scheduling-Resource-Control/      # 调度与资源控制
├── 04-Governance-Safety/                # 治理与安全
├── 05-Failure-Analysis/                 # 故障分析与诊断
└── 06-Workload-Traces/                  # 工作负载追踪
```

---

## 01-Observability — Agent 可观测性

### AgentSight: System-Level Observability for AI Agents Using eBPF

- **作者**: Yusheng Zheng (UC Santa Cruz), Yanpeng Hu (ShanghaiTech), Tong Yu (eunomia-bpf), Andi Quinn (UC Santa Cruz)
- **文件**: ![[AgentSight_System_Level_Observability_for_AI_Agents_Using_eBPF.pdf]]
- **摘要**: 提出 AgentSight，一个基于 eBPF 的 AgentOps 可观测性框架，通过边界追踪（boundary tracing）弥合 Agent 高层意图与底层系统行为之间的语义鸿沟。该框架拦截 TLS 加密的 LLM 流量以提取语义意图，监控内核事件以观察系统级影响，并通过实时引擎将两个流跨进程边界关联。开销低于 3%，能有效检测提示注入攻击、识别资源浪费的推理循环，并揭示多 Agent 系统中的隐藏协调瓶颈。
- **关键词**: eBPF, 边界追踪, 提示注入检测, 系统级可观测性
- **Slides**: [[03-Scheduling-Resource-Control/slides/AgentCgroup/AgentCgroup_slides.pdf|AgentCgroup Slides]]

### AgentOps: Enabling Observability of LLM Agents

- **作者**: Liming Dong, Qinghua Lu, Liming Zhu (Data61, CSIRO, Australia)
- **文件**: ![[AgentOps_Enabling_Observability_of_LLM_Agents.pdf]]
- **摘要**: 提出 AgentOps 的全面分类体系，识别在 Agent 全生命周期中应被追踪的工件（artifacts）和相关数据，以实现有效可观测性。基于对现有 AgentOps 工具的系统映射研究，该分类体系为开发者设计监控、日志和分析基础设施提供参考模板。
- **关键词**: AgentOps, 可观测性分类, LLM Agent 监控

### A Survey on AgentOps: Categorization, Challenges, and Future Directions

- **作者**: Zexin Wang, Jingjing Li, Quan Zhou, Haotian Si, Yuanhao Liu, Jianhui Li, Gaogang Xie, Fei Sun, Dan Pei, Changhua Pei (中国科学院计算机网络信息中心等)
- **文件**: ![[AgentOps_Survey_Categorization_Challenges_Future_Directions.pdf]]
- **摘要**: 系统定义了 Agent 系统内的异常（intra-agent 和 inter-agent 异常），提出 Agent System Operations (AgentOps) 框架，涵盖监控、异常检测、根因分析和解决四个关键阶段。为 Agent 系统的运维提供清晰的框架定义和挑战分析。
- **关键词**: AgentOps, 异常分类, 运维框架, 综述

### AI Observability for Large Language Model Systems: A Multi-Layer Analysis

- **作者**: Twinkll Sisodia (Senior Software Engineer, Red Hat)
- **文件**: ![[AI_Observability_LLM_Systems_Multi_Layer_Analysis.pdf]]
- **摘要**: 提出五层 AI 可观测性分类体系（Model Internals → Confidence & Calibration → Behavioral Monitoring → Operational Intelligence → Infrastructure Tracing），综合比较五个代表性研究贡献，识别四个关键未解决挑战，为构建集成化、端到端可观测性系统提供方向。
- **关键词**: 多层可观测性, 置信度校准, 行为监控, 基础设施追踪

### Agentic AI Process Observability: Discovering Behavioral Variability

- **作者**: Fabiana Fournier, Lior Limonad, Yuval David (IBM Research, Israel)
- **文件**: ![[Agentic_AI_Process_Observability_Behavioral_Variability.pdf]]
- **摘要**: 探索将过程挖掘和因果发现应用于 Agent 执行轨迹，以增强开发者可观测性。结合 LLM 静态分析技术区分预期行为变异与意外变异，为开发者提供对进化中 Agent 规范的更大控制权。
- **关键词**: 过程挖掘, 行为变异, 因果发现, 静态分析

### Taming Uncertainty via Automation: Observing, Analyzing, and Optimizing Agentic AI Systems

- **作者**: Dany Moshkovich, Sergey Zeltyn (IBM Research, Haifa, Israel)
- **文件**: ![[Taming_Uncertainty_Automation_Agentic_AI_Systems.pdf]]
- **摘要**: 提出 AgentOps Automation Pipeline，一个六阶段流程（行为观察 → 指标收集 → 问题检测 → 根因分析 → 优化建议 → 运行时自动化），面向开发者、测试人员、SRE 和业务用户四类角色，强调自动化在管理不确定性中的关键作用。
- **关键词**: AgentOps, 自动化流水线, 不确定性管理, 角色驱动分析

---

## 02-Benchmarking — Agent 评测与基准测试

### Efficient Benchmarking of AI Agents

- **作者**: Franck Ndzonga
- **文件**: ![[Efficient_Benchmarking_of_AI_Agents.pdf]]
- **摘要**: 研究是否可以用小规模任务子集保持 Agent 排名。发现排名预测在分布偏移下保持稳健，而分数预测迅速退化。提出 Mid-Range Difficulty Filter (MR)，通过保留历史通过率在 30-70% 的任务，将评估任务数量减少 44-70%，同时保持高排名保真度。
- **关键词**: 基准测试压缩, 排名保真度, 难度过滤, 成本优化

### Evaluation and Benchmarking of LLM Agents: A Survey

- **作者**: Mahmoud Mohammadi, Yipeng Li, Jane Lo, Wendy Yip (SAP Labs)
- **文件**: ![[LLM_Agent_Evaluation_Benchmarking_Survey.pdf]]
- **摘要**: 提出二维分类体系（评估目标 × 评估过程），涵盖 Agent 行为、能力、可靠性、安全性等目标维度，以及交互模式、数据集、指标计算、工具和环境等过程维度。强调企业级挑战（角色访问控制、可靠性保证、合规性）。
- **关键词**: 评估分类体系, 企业级挑战, 交互模式, 可靠性评估

---

## 03-Scheduling-Resource-Control — 调度与资源控制

### Towards Agentic OS: An LLM Agent Framework for Linux Schedulers (SchedCP)

- **作者**: Yusheng Zheng (UC Santa Cruz), Yanpeng Hu (ShanghaiTech), Wei Zhang (UConn), Andi Quinn (UC Santa Cruz)
- **文件**: ![[SchedCP_Agentic_OS_LLM_Agent_Framework_Linux_Schedulers.pdf]]
- **摘要**: 提出 SchedCP，首个使 LLM Agent 能够安全高效地优化 Linux 调度器的框架。核心创新是将优化问题分解为两个阶段：目标推断（"优化什么"）和策略合成（"如何执行"），通过解耦控制平面实现。在 kernel 编译上实现 1.79× 性能提升，13× 成本降低。
- **关键词**: Agentic OS, Linux 调度器, eBPF, MCP, 自动优化
- **代码**: https://github.com/eunomia-bpf/schedcp
- **Slides**: [[03-Scheduling-Resource-Control/slides/SchedCP/index.html|SchedCP Slides]]

### AgentCgroup: Understanding and Controlling OS Resources of AI Agents

- **作者**: Yusheng Zheng (UC Santa Cruz), Jiakun Fan, Quanzhi Fu (Virginia Tech), Yiwei Yang (UC Santa Cruz), Wei Zhang (UConn), Andi Quinn (UC Santa Cruz)
- **文件**: ![[AgentCgroup_Understanding_and_Controlling_OS_Resources_of_AI_Agents.pdf]]
- **摘要**: 系统刻画了沙箱化 AI 编码 Agent 的 OS 级资源动态，分析 144 个 SWE-rebench 任务。发现：(1) OS 级执行占端到端延迟 56-74%；(2) 内存（非 CPU）是多租户并发瓶颈；(3) 内存呈现工具调用驱动的双峰结构；(4) 资源需求高度不可预测。提出 AgentCgroup，基于 eBPF 的意图驱动资源控制器，实现 29% 高优先级 P95 延迟降低。
- **关键词**: eBPF, cgroup, 资源控制, 多租户, 意图驱动
- **代码**: https://github.com/eunomia-bpf/agentcgroup
- **Slides**: [[03-Scheduling-Resource-Control/slides/AgentCgroup/AgentCgroup_slides.pdf|AgentCgroup Slides]]

---

## 04-Governance-Safety — 治理与安全

### MI9: An Integrated Runtime Governance Framework for Agentic AI

- **作者**: Charles L. Wang (Barclays / Columbia University), Trisha Singhal, Ameya Kelkar, Jason Tuo (Barclays)
- **文件**: ![[MI9_Runtime_Governance_Framework_Agentic_AI.pdf]]
- **摘要**: 提出 MI9，面向 Agentic AI 运行时安全的集成框架，在实时行为序列上强制执行安全属性。提供六种协调机制：Agency-Risk Index、Agent-semantic Telemetry、Goal-aware Authorization、Finite-state Conformance、Goal-conditioned Drift Detection、Graded Containment。在 1,000+ 多域合成场景中实现高检测率和低误报率。
- **关键词**: 运行时治理, 安全属性, 行为监控, 漂移检测, 分级遏制

---

## 05-Failure-Analysis — 故障分析与诊断

### AgentFixer: From Failure Detection to Fix Recommendations in LLM Agentic Systems

- **作者**: Hadar Mulian, Sergey Zeltyn, Ido Levy, Liane Galanti, Avi Yaeli, Segev Shlomov (IBM Research, Israel)
- **文件**: ![[AgentFixer_Failure_Detection_Fix_Recommendations.pdf]]
- **摘要**: 提出面向 LLM Agent 系统的综合验证框架，包含 15 种故障检测工具和两种根因分析模块，联合发现输入处理、提示设计和输出生成中的弱点。集成轻量级规则检查与 LLM-as-a-judge 评估，支持结构化事件检测、分类和修复。在 IBM CUGA 和 AppWorld/WebArena 基准上验证。
- **关键词**: 故障检测, 根因分析, LLM-as-a-judge, 验证框架

### SysOM-AI: Continuous Cross-Layer Performance Diagnosis for Production AI Training

- **作者**: Yusheng Zheng, Wenan Mao, Shuyi Cheng, Fuqiu Feng, Guangshui Li, Zhaoyan Liao, Yongzhuo Huang, Zhenwei Xiao, Yuqing Li, Andi Quinn, Tao Ma (Alibaba Group / UC Santa Cruz)
- **文件**: ![[SysOM_AI_Continuous_Cross_Layer_Performance_Diagnosis.pdf]]
- **摘要**: 提出 SysOM-AI，面向生产级 AI 训练的性能剖析系统，通过 eBPF 持续集成 CPU 栈剖析、GPU 内核追踪和 NCCL 事件采集。采用自适应混合栈展开和分层差分诊断方法，在阿里巴巴 80,000+ GPU 上部署一年以上，诊断 94 个确认生产问题，将中位诊断时间从天缩短到约 10 分钟。
- **关键词**: 性能诊断, eBPF, 跨层分析, GPU 训练, 生产环境

---

## 06-Workload-Traces — 工作负载追踪

### RAGPulse: An Open-Source RAG Workload Trace to Optimize RAG Serving Systems

- **作者**: Zhengchao Wang, Yitao Hu, Jianing Ye, Zhuxuan Chang, Jiazheng Yu, Youpeng Deng, Keqiu Li (Tianjin University, China)
- **文件**: ![[RAGPulse_RAG_Workload_Trace.pdf]]
- **摘要**: 发布 RAGPulse，一个开源 RAG 工作负载追踪数据集，收集自 2024 年 4 月以来服务 40,000+ 师生的大学级 Q&A 系统。揭示真实 RAG 工作负载具有显著的时间局部性和高度偏斜的热文档访问模式，为内容感知批处理和检索缓存等优化策略提供基础。
- **关键词**: RAG, 工作负载追踪, 时间局部性, 缓存优化, 开源数据集
- **代码**: https://github.com/flashserve/RAGPulse

---

## 按主题快速导航

| 主题 | 论文数量 | 目录 |
|------|---------|------|
| 可观测性 (Observability) | 6 | [[01-Observability]] |
| 评测基准 (Benchmarking) | 2 | [[02-Benchmarking]] |
| 调度与资源控制 | 2 | [[03-Scheduling-Resource-Control]] |
| 治理与安全 | 1 | [[04-Governance-Safety]] |
| 故障分析 | 2 | [[05-Failure-Analysis]] |
| 工作负载追踪 | 1 | [[06-Workload-Traces]] |

---

## 相关笔记

- [[SWE-bench]] — SWE-bench 基准测试
- [[SWE-rebench]] — SWE-ReBench 评测框架
- [[AgentCgroup Understanding and Controlling OS Resources of AI Agents]] — AgentCgroup 论文笔记
- [[The 1st Workshop onOperating Systems Design for AI Agents 会议总结]] — AgenticOS 2026 研讨会总结
