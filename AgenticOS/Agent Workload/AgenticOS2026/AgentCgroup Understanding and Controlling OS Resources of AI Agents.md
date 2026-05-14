AgentCgroup：理解和控制AI代理的操作系统资源

Abstract

摘要

AI agents are increasingly deployed in multi-tenant cloud environments, where they execute diverse tool calls within sandboxed containers, each call with distinct resource demands and rapid ﬂuctuations. We present a systematic characterization of OS-level resource dynamics in sandboxed AI coding agents, analyzing 144 software engineering tasks from the [[SWE-rebench]] benchmark across two LLM models. Our measurements reveal that (1) OS-level execution (tool calls and container/agent initialization) accounts for 56–74% of end-to-end task latency; (2) memory, not CPU, is the concurrency bottleneck; (3) memory spikes are tool-call-driven with a 15.4× peak-to-average ratio; and (4) resource demands are highly unpredictable across tasks, runs, and models. Comparing these characteristics against [[serverless]], microservice, and batch workloads, we identify three mismatches in existing resource controls: a granularity mismatch (container-level policies vs. tool-calllevel dynamics), a responsiveness mismatch (user-space reaction vs. sub-second unpredictable bursts), and an adaptability mismatch (history-based prediction vs. non-deterministic stateful execution). We propose AgentCgroup, an intent-driven eBPF-based resource controller that exploits agents’ ability to declare resource needs and reconstruct execution strategies, using hierarchical cgroup structures aligned with tool-call boundaries, in-kernel enforcement via sched_ext and memcg_bpf_ops, and runtime-adaptive policies. Preliminary evaluation demonstrates improved multi-tenant isolation and reduced resource waste. AgentCgroup is open-source at [https://github.com/eunomia-bpf/agentcgroup](https://github.com/eunomia-bpf/agentcgroup).

人工智能代理越来越多地部署在多租户云环境中，在这些环境中，它们在沙箱容器内执行各种工具调用，每次调用都有不同的资源需求和快速波动。我们对沙箱化人工智能编码代理中的操作系统级资源动态进行了系统表征，分析了来自 SWE-rebench 基准测试的 144 个软件工程任务，涉及两个大型语言模型。我们的测量结果显示（1）操作系统级执行（工具调用和容器/代理初始化）占端到端任务延迟的 56-74%；（2）内存是并发瓶颈，而非 CPU；（3）内存峰值由工具调用驱动，峰均比为 15.4×；（4）跨任务、跨运行、跨模型的资源需求高度不可预测。将这些特性与无服务器、微服务和批处理工作负载进行比较，我们发现现有的资源控制存在三个不匹配：粒度不匹配（容器级策略与工具调用级动态）、响应性不匹配（用户空间反应与亚秒级不可预测的突发）和适应性不匹配（基于历史的预测与非确定性有状态执行）。我们提出了 AgentCgroup，一种意图驱动的基于 eBPF 的资源控制器，它利用代理声明资源需求和重建执行策略的能力，使用与工具调用边界对齐的分层 cgroup 结构，通过 sched_ext 和 memcg_bpf_ops 进行内核内强制执行，以及运行时自适应策略。初步评估表明，多租户隔离得到改善，资源浪费减少。[[AgentCgroup]] 是开源的，网址为 [https://github.com/eunomia-bpf/agentcgroup](https://github.com/eunomia-bpf/agentcgroup)。


关联资料：
slide
![[agentcgroup-slides.pdf]]
论文
![[AgenticOS_2026_paper_10.pdf]]
1. 论文
2. 