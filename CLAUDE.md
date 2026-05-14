# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本文件为 Claude Code (claude.ai/code) 提供在本仓库中工作的指引。

## 仓库概述

这是一个关于 **AI Agent 与 Agentic Operating System** 的 **Obsidian 知识库**，并非代码项目——没有构建系统、测试或包管理器。仓库内容包含 Markdown 笔记、PDF 附件和 Obsidian 配置文件。

## 内容领域

- **AgenticOS 研讨会**：ASPLOS 2026 关于面向 AI Agent 的操作系统设计的研讨会笔记，包含论文摘要和会议日程。
- **SWE-bench 基准测试**：SWE-bench、SWE-ReBench 和 Multi-SWE-bench 评估框架的文档与部署指南。
- **AgentCgroup**：关于面向 AI Agent 的基于 eBPF 的资源控制的研究笔记。
- **2025 AI Agent 指数**：MIT 关于已部署的 Agentic AI 系统的研究摘要。

## 知识库结构

```
AgenticOS/
├── Agentic OS - The 1st Workshop on Operating Systems Design for AI Agents.md
├── Linux kernel 仓库国内镜像加速.md
├── papers/
│   ├── 2025-AI-Agent-Index.pdf
│   └── AgenticOS2026/
│       └── AgenticOS_2026_paper_10.pdf
└── Agent Workload/
    ├── Benchmarks/
    │   ├── SWE-bench.md
    │   ├── SWE-rebench.md
    │   ├── Muilt-swe-bench.md
    │   ├── SWE-bench 本地测试部署指导手册.md
    │   └── SWE-bench vs SWE-ReBench 分析与国内部署指南.md
    └── AgenticOS2026/
        ├── The 1st Workshop onOperating Systems Design for AI Agents 会议总结.md
        ├── 2025 AI Agent 指数：部署型代理 AI 系统的技术与安全特性文档.md
        └── AgentCgroup Understanding and Controlling OS Resources of AI Agents.md
```

## Obsidian 配置

- **知识库根目录**：仓库根目录
- **链接格式**：Wikilinks (`[[笔记名称]]`)，启用 `alwaysUpdateLinks: true`
- **社区插件**：`pdf-plus`（PDF 批注）、`open-vscode`（在 VS Code 中打开知识库）、`obsidian-git`（Git 备份）

## Git 工作流

### 通过 Claude Code 提交（推荐）

当使用 Claude Code 编辑知识库时，可以直接通过命令行提交更改：

```bash
git add <文件>
git commit -m "<提交信息>" --author="Xie XiuQi <xiexiuqi@163.com>"
git push
```

**提交信息格式：** 使用符合 email 格式的描述性消息，例如：
- `add: SWE-bench 部署指南笔记`
- `update: AgentCgroup 论文摘要`
- `fix: 修正 SWE-rebench 链接`

### 通过 Obsidian 手动备份

本知识库也通过 obsidian-git 插件支持**手动 Git 备份**。

**备份步骤：**
1. 在 Obsidian 中，打开命令面板（`Ctrl/Cmd + P`）
2. 运行 "Obsidian Git: Commit all changes"
3. 运行 "Obsidian Git: Push"

**Obsidian 提交信息格式：** `vault backup: YYYY-MM-DD HH:mm:ss`

### 同步配置

- **同步方式：** rebase
- **推送前拉取：** 启用，以避免合并冲突
- **自动间隔：** 所有自动间隔（自动保存、自动提交、自动推送、自动拉取）均设置为 0（禁用）

## 编辑规范

- 在 `AgenticOS/` 下的适当子目录中创建 Markdown 文件
- 使用 `[[笔记名称]]` 语法链接到其他笔记——Obsidian 会在文件重命名时自动更新链接
- 对于 PDF 附件，将其放在引用笔记的同一目录中，并使用 `![[文件名.pdf]]` 语法
- 笔记主要使用**中文**，保留英文技术术语
- Markdown 文件名反映内容标题

## 引用的关键外部资源

- SWE-bench: https://github.com/swe-bench/SWE-bench
- AgenticOS 研讨会: https://os-for-agent.github.io/
- AgentCgroup: https://github.com/eunomia-bpf/agentcgroup
- 2025 AI Agent 指数: https://aiagentindex.mit.edu/
- Multi-SWE-bench: https://multi-swe-bench.github.io/
