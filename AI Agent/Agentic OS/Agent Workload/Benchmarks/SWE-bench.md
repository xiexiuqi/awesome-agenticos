#大模型评测基准

SWE-bench is a benchmark for evaluating large language models on real world software issues collected from GitHub. Given a _codebase_ and an _issue_, a language model is tasked with generating a _patch_ that resolves the described problem. [^4]

SWE-bench provides:

- ✅ **Real-world GitHub issues** - Evaluate LLMs on actual software engineering tasks
- ✅ **Reproducible evaluation** - Docker-based evaluation harness for consistent results
- ✅ **Multiple datasets** - SWE-bench, SWE-bench Lite, SWE-bench Verified, and SWE-bench Multimodal

SWE-bench是一个用于评估大型语言模型解决真实世界软件工程问题能力的基准测试，由[普林斯顿大学](https://baike.baidu.com/item/%E6%99%AE%E6%9E%97%E6%96%AF%E9%A1%BF%E5%A4%A7%E5%AD%A6/400839?fromModule=lemma_inlink)和[芝加哥大学](https://baike.baidu.com/item/%E8%8A%9D%E5%8A%A0%E5%93%A5%E5%A4%A7%E5%AD%A6/514980?fromModule=lemma_inlink)的研究人员于2024年提出。其初始版本从12个流行的[Python](https://baike.baidu.com/item/Python/407313?fromModule=lemma_inlink)开源项目中收集了2294个真实的GitHub Issue-Pull Request对作为测试样本。 [3] [9]

2024年，[OpenAI](https://baike.baidu.com/item/OpenAI/19758408?fromModule=lemma_inlink)与SWE-bench原作者合作发布了改进版本SWE-bench Verified，该版本通过人工筛选构建了包含500个已验证样本的子集。 [3] [5]此后，该基准测试衍生出[SWE-bench Lite](https://baike.baidu.com/item/SWE-bench%20Lite/67165626?fromModule=lemma_inlink)、SWE-bench Pro等版本。 [1] [4] [6]2026年，因发现SWE-bench Verified中近60%的问题存在测试缺陷，其权威性受到挑战，OpenAI宣布停止将其作为核心评估标准。 [6]同年，其评估框架被指出存在安全漏洞。[^1]

# 官方安装文档
https://www.swebench.com/SWE-bench/installation/

# 开源仓库
https://github.com/swe-bench/SWE-bench

# SWE-Bench Pro - Public

Scale AI 于 2025 年 9 月 21 日发布了 SWE-Bench Pro，这是一个针对 AI 代理在软件工程任务上的评估基准。该基准包含 1,865 个问题，来源于 41 个活跃维护的代码仓库，聚焦企业级复杂任务。现有模型在该基准上的表现显示出显著差距，顶级模型的通过率低于 25%，而最近的榜单更新显示部分模型已超过 40%。这一发布旨在推动 AI 在长时程软件开发中的应用研究.[^2]

# SWE-bench series[^3]

SWE-bench **Verified** is a human-filtered subset of 500 instances; use the Agent dropdown to compare LMs with [mini-SWE-agent](https://github.com/SWE-agent/mini-swe-agent) or view all agents [[Post](https://www.swebench.com/verified.html)].  
SWE-bench **Multilingual** features 300 tasks across 9 programming languages [[Post](https://www.swebench.com/multilingual-leaderboard.html)].  
SWE-bench **Lite** is a subset curated for less costly evaluation [[Post](https://www.swebench.com/lite.html)].  
SWE-bench **Multimodal** features issues with visual elements [[Post](https://www.swebench.com/multimodal.html)].  

Each entry reports the **% Resolved** metric, the percentage of instances solved (out of 2294 Full, 500 Verified, 300 Lite & Multilingual, 517 Multimodal).

# 相关论文
[SWE-BENCH: CAN LANGUAGE MODELS RESOLVE REAL-WORLD GITHUB ISSUES?](https://openreview.net/pdf?id=VTF8yNQM66)

# SWE bench 文档
https://www.swebench.com/SWE-bench/

[^1]: 百度百科 - SWE Bench: https://baike.baidu.com/item/SWE%20bench/67614096

[^2]: SWE-Bench Pro:https://www.datalearner.com/benchmarks/swe-bench-pro

[^3]: https://www.swebench.com/

[^4]: https://www.swebench.com/SWE-bench/
