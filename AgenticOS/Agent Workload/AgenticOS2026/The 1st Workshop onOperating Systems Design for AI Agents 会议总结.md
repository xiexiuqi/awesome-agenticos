## Overview

AI 智能体正迅速从实验原型演变为**始终在线的服务**，它们能够自主规划、调用外部工具、协作，并持续与环境交互。这一转变对传统操作系统抽象——进程、线程、文件、套接字和资源控制器——构成了挑战，因为这些抽象从未为**动态的、语义丰富的、自适应的智能体工作负载**而设计。为了规模化支持 AI 智能体，操作系统本身必须变得**智能体化**，调整其抽象和资源管理策略以适应智能体的语义行为。

AgenticOS 研讨会征集原创观点论文和经验报告，探讨面向 AI 智能体工作负载的 OS 级机制。我们的目标是定义构建**面向智能体系统的操作系统**所必需的**原语、隔离模型、调度技术和可观测机制**。

## 感兴趣的主题

感兴趣的主题包括但不限于：

- 面向智能体执行的新型操作系统抽象（进程/容器/多内核增强）
- 用于安全执行智能体生成的代码和任务的动态沙箱与轻量级运行时
- 面向动态、多智能体工作负载的语义感知资源管理与调度
- 用于管理智能体上下文、提示词和情景记忆的长生命周期状态抽象
- 面向实时可观测性、自适应与约束强制的 eBPF 驱动扩展
- 面向自适应和智能体感知 JIT 执行策略的编译器-操作系统协同设计
- 面向智能体工作负载大规模部署的 GPU 与加速器虚拟化
- 面向智能体调用工具、代码和数据流的安全与隔离机制
- 智能体管理系统：内核调优、异常检测、资源编排、故障恢复与动态策略更新

## 投稿指南

我们征集两类投稿：

### 频道 1

**愿景论文** 1–2 页（不含参考文献）。

适用于早期想法、观点陈述、进行中项目、演示，以及来自生产系统的经验洞察。我们强烈欢迎**工业界从业者和开源社区**投稿，分享真实世界的经验与挑战。

### 频道 2

**研究论文** 最多 6 页（不含参考文献）。

适用于更完整的概念、研究成果和经验报告。

所有投稿必须遵循 **ACM 双栏会议格式**。审稿流程为**单盲审**，每篇投稿至少收到两份评审意见。
## Workshop Schedule

March 23, 2026 — Afternoon Session

|Time|Session|Details|
|---|---|---|
|1:30pm – 1:35pm EDT|Opening|Workshop Opening Remarks by [Prof. Dong Li](https://faculty.ucmerced.edu/dong-li/) ([slides](https://os-for-agent.github.io/slides/opening-slides.pdf))|
|1:35pm – 2:00pm EDT|Keynote|Keynote Talk by [Prof. Dan Williams](https://website.cs.vt.edu/people/faculty/dan-williams.html) (25 min) ([slides](https://os-for-agent.github.io/slides/keynote-slides.pdf))|
|2:00pm – 2:15pm EDT|Research Paper|**[AgentCgroup: Understanding and Controlling OS Resources of AI Agents](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_10.pdf)** ([slides](https://os-for-agent.github.io/slides/agentcgroup-slides.pdf))<br><br>Yusheng Zheng, Jiakun Fan, Quanzhi Fu, Yiwei Yang, Wei Zhang and Andi Quinn|
|2:15pm – 2:30pm EDT|Research Paper|**[Rethinking OS Interfaces for LLM Agents](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_9.pdf)** ([slides](https://os-for-agent.github.io/slides/rethinking-os-interfaces-slides.pdf))<br><br>Yuan Wang, Mingyu Li and Haibo Chen|
|2:30pm – 2:45pm EDT|Research Paper|**[Skills are the new Apps — Now It's Time for Skill OS](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_13.pdf)**<br><br>Le Chen, Zichang Wang, Wenxin Zheng, Erhu Feng, Dong Du, Yubin Xia and Haibo Chen|
|2:45pm – 3:00pm EDT|Research Paper|**[Toward LLM-Driven Rule Generation for Enforcement Systems: An Exploratory Study on WAF](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_24.pdf)** ([slides](https://os-for-agent.github.io/slides/waf-slides.pdf))<br><br>Quanzhi Fu and Dan Williams|
|3:00pm – 3:10pm EDT|Vision Paper|**[Mobile-MCP: Implementing the Model Context Protocol via Android's Inter-Application Communication Mechanisms](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_18.pdf)**<br><br>Xiheng Li, Mengting He, Chengcheng Wan and Linhai Song|
|3:10pm – 3:20pm EDT|Vision Paper|**[pMVX: Policy-Level Multi-Version Execution for Agentic OS Kernel Self-Tuning](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_17.pdf)** ([slides](https://os-for-agent.github.io/slides/pmvx-slides.pdf))<br><br>Sujot Singh, Eddie Federmeyer, Kenan Alghythee and Xiaoguang Wang|
|3:20pm – 3:30pm EDT|Vision Paper|**[Execute-Only Agents: Architectural Defense Against Prompt Injection for AI Agents](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_21.pdf)** ([slides](https://os-for-agent.github.io/slides/execute-only-agents-slides.pptx))<br><br>Rahul Tiwari and Dan Williams|
|3:30pm – 4:00pm EDT|Break|Coffee Break (30 min)|
|4:00pm – 4:15pm EDT|Invited Talk|**Agents Are Not Just a Model Problem. They Are an Execution Problem.** ([slides](https://os-for-agent.github.io/slides/execution-problem-slides.pdf))<br><br>Guanlan Dai|
|4:15pm – 4:30pm EDT|Research Paper|**[Fork, Explore, Commit: OS Primitives for Agentic Exploration](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_8.pdf)** ([slides](https://os-for-agent.github.io/slides/branchcontext-slides.pdf))<br><br>Cong Wang and Yusheng Zheng|
|4:30pm – 4:45pm EDT|Research Paper|**[It is Time to Virtualize Foundation Models with a Self-evolving Operating System Layer](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_26.pdf)** ([slides](https://os-for-agent.github.io/slides/vfmos-slides.pdf))<br><br>Suparna Bhattacharya, Tarun Kumar, Cong Xu, Satish Kumar Mopur, Jiahao Li, Ashish Mishra, Aalap Tripathy, Annmary Justine Koomthanam, Martin Foltin and Ian Foster|
|4:45pm – 5:00pm EDT|Research Paper|**[Fuyun: Bridging the Semantic Gap in Serverless Resource Provisioning via LLM Agents](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_16.pdf)** ([slides](https://os-for-agent.github.io/slides/fuyun-slides.pdf))<br><br>Qingwen Li, Kai Lv, Mingxuan Yang, Zhengyu Lei, Cunchi Lv, Xiao Shi and Xiaofang Zhao|
|5:00pm – 5:10pm EDT|Vision Paper|**[Towards Agentic Performance Management](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_11.pdf)** ([slides](https://os-for-agent.github.io/slides/agentic-performance-mgmt-slides.pdf))<br><br>Jingyuan Chen, Gongqi Huang and Amit Levy|
|5:10pm – 5:20pm EDT|Vision Paper|**[Grimlock: Guarding High-Agency Systems with eBPF and Attested Channels](https://os-for-agent.github.io/papers/AgenticOS_2026_paper_23.pdf)** ([slides](https://os-for-agent.github.io/slides/grimlock-slides.pdf))<br><br>Qiancheng Wu, Wenhui Zhang, Gan Fang, Sheng Mao, Biao Gao, David Levitsky, Shawna Murphy Butterworth and Rob Cameron|
|5:20pm – 5:25pm EDT|Awards|**[Best Paper Awards + Closing](https://os-for-agent.github.io/slides/closing-slides.pdf)** (5 min)<br><br>Prof. Dong Li|
|5:25pm – 6:00pm EDT|Social|**Networking** (35 min)|
|6:30pm EDT|Social|Social Dinner|