# Reviewer1 TDN5
## 1. Weaknesses1【done】: a) The success of the entire workflow is highly dependent on the initial "Repository Search" stage. This stage relies on analyzing user intent, README files, and star counts. The paper could benefit from a deeper discussion of the limitations of this search phase and how the agent might recover if an unsuitable repository is initially selected.
### 1.1 ZH

感谢您的深刻见解。我们承认，存储库搜索阶段对于我们框架的成功至关重要。为了解决这一问题，我们实施了以下几种机制：

**1. 基于"搜索与思考"范式的高级搜索策略**

我们开发了一种复杂的深度搜索机制，它采用迭代的"搜索与思考"方法，而非依赖于单次搜索尝试。在每次搜索迭代中，代理：

- 根据搜索和分析结果动态地制定和改进搜索查询
- 在浏览存储库内容时执行实时相关性评估  
- 维护具有置信度得分的前k个候选存储库的排序列表

这种认知搜索过程模仿了人类开发人员探索GitHub的方式，通过主动探索而不是被动过滤来逐步加深理解。

**2. 通过全面的多代理架构实现强大的故障恢复**

为了处理选择不合适的存储库的情况，我们设计了一个具有专门恢复代理的分层多代理系统：

```
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery and ranking
├── Code Agent: Task execution with repository
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
```

当执行失败发生时，我们的Schedule Agent会协调这些专门的代理。

**3. 自动故障恢复机制**

当Code Agent遇到执行失败时（例如，依赖冲突、API不匹配、缺少功能），我们的Schedule Agent会自动：

- (i) 分析失败模式以生成结构化反馈
- (ii) 根据失败模式调整选择标准（例如，如果计算机视觉任务因缺少GPU支持而失败，它会优先选择CPU兼容的替代方案）
- (iii) 无缝切换到下一个候选存储库

这种自适应多代理方法确保系统能够从各种故障模式中恢复，从简单的依赖问题到基本的存储库不匹配，从而显著增强了框架的实际适用性。

### 1.2 EN

Thank you for your insightful observation. We acknowledge that the repository search stage is crucial to the success of our framework. To address this concern, we have implemented several mechanisms:

**1. Advanced Search Strategy Based on "Search and Think" Paradigm**

We have developed a sophisticated deep search mechanism that employs an iterative "search and think" approach, rather than relying on a single search attempt. In each search iteration, the agent:

- Dynamically formulates and refines search queries based on search and analysis results
- Performs real-time relevance assessment while browsing repository content
- Maintains a ranked list of top-k candidate repositories with confidence scores

This cognitive search process mimics how human developers explore GitHub, gradually deepening understanding through active exploration rather than passive filtering.

**2. Robust Failure Recovery through Comprehensive Multi-Agent Architecture**

To handle cases of selecting unsuitable repositories, we designed a hierarchical multi-agent system with specialized recovery agents:

```
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery and ranking
├── Code Agent: Task execution with repository
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
```

When execution failures occur, our Schedule Agent coordinates these specialized agents.

**3. Automatic Failure Recovery Mechanisms**

When the Code Agent encounters execution failures (e.g., dependency conflicts, API mismatches, missing functionality), our Schedule Agent automatically:

- (i) Analyzes failure patterns to generate structured feedback
- (ii) Adjusts selection criteria based on failure patterns (e.g., if a computer vision task fails due to lack of GPU support, it prioritizes CPU-compatible alternatives)
- (iii) Seamlessly switches to the next candidate repository

This adaptive multi-agent approach ensures the system can recover from various failure modes, from simple dependency issues to fundamental repository mismatches, significantly enhancing the framework's practical applicability.


## 2. Weaknesses2【done】: b) The initial "Hierarchical Repository Analysis" involves parsing all source files to build ASTs, dependency graphs, and call graphs. While effective, the paper does not discuss the computational cost or time required for this preprocessing step.

### 2.1 ZH

感谢您提出的宝贵意见。您关注的**预处理步骤的计算成本**的确需要我们作更为详尽的阐述。我们将在修订版和附录中添加这方面的全面实验数据。

#### 1. 静态分析预处理性能

我们的静态分析预处理流程，利用高度优化的库（例如，用于AST解析的**tree-sitter**）并支持多线程执行。

**对于典型的存储库（1-5k个文件）：**
整个预处理平均在**4.8秒**内完成，其中包括：

1. **AST构建**：~2.1秒（跨CPU核心并行）
2. **依赖/调用图生成**：~1.9秒
3. **核心组件识别**：~0.8秒

#### 2. 复杂存储库的附加处理

对于核心组件超过上下文阈值（**8k个token**）的存储库，我们采用基于**LLM的摘要方法**：
- 使用**Deepseek V3**时，耗时约**10秒**
- 即使对于复杂的存储库，整体预处理时间也保持在**15秒以内**

### 2.2 EN

Thank you for your valuable feedback. We acknowledge that the **computational cost of the preprocessing step** indeed requires more thorough elaboration. We will add comprehensive experimental data on this aspect in the revision and appendix.

#### 1. Hierarchical Repository Analysis

Our static analysis preprocessing pipeline utilizes highly optimized libraries (e.g., **tree-sitter** for AST parsing) and supports multi-threaded execution.

**For typical repositories (1-5k files):**
The entire preprocessing completes on average within **4.8 seconds**, including:

1. **AST construction**: ~2.1 seconds (parallelized across CPU cores)
2. **Dependency/call graph generation**: ~1.9 seconds
3. **Core component identification**: ~0.8 seconds

#### 2. Additional Processing for Complex Repositories

For repositories where core components exceed the context threshold (**8k tokens**), we employ an **LLM-based summarization approach**:
- Using **Deepseek V3**, this takes approximately **10 seconds**
- Even for complex repositories, overall preprocessing time remains **within 15 seconds**


## 3. Weaknesses3【done】: c) The module-level importance scoring aggregates six features using equal weights. This seems somewhat arbitrary. The paper would be strengthened by a sensitivity analysis on these weights or a more principled method for determining them.

### 3.1 ZH

感谢您的建设性建议。我们同样认为有必要更系统地阐述**模块级重要性评分**的实验分析流程，我们将在后续论文附录中补充介绍实验细节。

#### 实验设计过程

**1. 测试集构建与权重优化**
- 首先我们人为构建了几个仓库的核心文件模块作为我们的测试集
- 然后通过**RepoMaster**中的不同权重组合策略的表现来针对性优化来提升重要文件集的召回率

**2. 维度筛选与消融实验**
- 通过实验对比，我们去除了重合度比较高的几个评估维度的分数
- 对每个单一维度的策略进行了简单的**消融实验**
- 最后保留了论文中所给出的**六个评估维度**，并给予相同的权重

**3. Top-k参数优化**
- 我们逐步增加top-k的模块数量
- 最后核心模块的**top-20召回率达到70%**
- 进一步增加top-k，召回收益并不明显

#### Top-k模块召回率实验结果

| Top-k  | Recall Rate (%) | Relative Improvement |
|--------|----------------|---------------------|
| Top-5  | 46.6           | -                   |
| Top-10 | 60.0           | +13.4%              |
| Top-20 | 70.0           | +10.0%              |
| Top-30 | 73.3           | +3.3%               |

实验表明，**Top-20**能够在召回率和效率之间达到良好平衡，进一步增加k值带来的收益递减。

#### 关于权重敏感性分析

关于整体效果的权重敏感性分析，考虑到实验的时间成本较大，我们并没有系统地做一些定量分析，在后续的论文版本中我们可以补充更全面的实验分析结果。

此外，我们想详细介绍的是：在我们整体算法框架设计中，评估得到的**核心模块的作用**是在有限的context窗口下，帮助agent对于整体仓库有一个全面的了解。这也是作为**RepoMaster**对整个仓库的自主探索入口，自主决策是否要从当前文件扩展搜索和阅读相邻节点的文件。定位到任务相关的代码片段或者文件信息后才送入Agent的context来完成端到端任务自主探索和执行：

```
搜索 → 理解 → 代码生成/编辑 → 执行 → 调试 → 生成可验证输出
```

### 3.2 EN

Thank you for the suggestion. We completely agree that we should indeed expand on the experimental analysis process of our **module-level importance scoring**. We will supplement the experimental details in the appendix of the subsequent paper.

#### Experimental Design Process

**1. Test Set Construction and Weight Optimization**
- First, we manually constructed the core file modules of several repositories as our test set
- Then we performed targeted optimization through the performance of different weight combination strategies in **RepoMaster** to improve the recall rate of important file sets

**2. Dimension Screening and Ablation Experiments**
- Through experimental comparison, we removed scores from several evaluation dimensions with high overlap
- We conducted simple **ablation experiments** on each single dimension strategy
- Finally, we retained the **six evaluation dimensions** presented in the paper and gave them equal weights

**3. Top-k Parameter Optimization**
- We gradually increased the number of top-k modules
- Finally, the **top-20 recall rate of core modules reached 70%**
- Further increasing top-k did not yield significant recall benefits

#### Top-k Module Recall Rate Experimental Results

| Top-k  | Recall Rate (%) | Relative Improvement |
|--------|----------------|---------------------|
| Top-5  | 46.6           | -                   |
| Top-10 | 60.0           | +13.4%              |
| Top-20 | 70.0           | +10.0%              |
| Top-30 | 73.3           | +3.3%               |

Experiments show that **Top-20** achieves a good balance between recall rate and efficiency, with diminishing returns from further increasing k values.

#### Regarding Weight Sensitivity Analysis

Regarding overall effectiveness weight sensitivity analysis, considering the significant time cost of experiments, we did not systematically conduct quantitative analysis. In subsequent paper versions, we can supplement more comprehensive experimental analysis results.

Additionally, we want to detail that in our overall algorithm framework design, the role of the evaluated **core modules** is to help the agent have a comprehensive understanding of the entire repository under limited context windows. This also serves as the **RepoMaster**'s autonomous exploration entry point for the entire repository, autonomously deciding whether to expand the search and read adjacent node files from the current file. Only after locating task-relevant code snippets or file information are they fed into the Agent's context to complete end-to-end task autonomous exploration and execution:

```
Search → Understand → Code Generation/Editing → Execute → Debug → Generate Verifiable Output
```




# Reviewer aqw3
## 4. Weaknesses1【done】：Many of RepoMaster's ideas e.g.repository graphs and context selection overlap with recent work. For example, the concurrent RepoGraph [1] system also builds a repository-level graph structure to guide LLM code agents. The CGM [2] approach similarly integrates a code graph into an LLM's attention. RepoMaster's novelty seems to lie mostly in the engineering of combining known techniques (AST traversal, call/dependency graphs, heuristic scoring) rather than introducing fundamentally new algorithms. As a result, its contribution feels somewhat incremental relative to these related methods.
### 4.1 ZH

感谢审稿人对RepoGraph[1]与CGM[2]关联性的细致指正。我们高度认可这些并行工作的贡献——**RepoGraph**作为可插拔模块有效提升了代码修复性能，**CGM**通过图注意力机制在SWE-bench-Lite上取得了优秀成果，我们也期待未来探索技术融合的可能性。

但我们的**RepoMaster**与**RepoGraph/CGM**在任务问题定义、方法设计和技术贡献上存在本质差异：

- **任务层面**：我们面向真实、端到端任务，而非仅限代码修复
- **技术创新和贡献**：RepoMaster的核心贡献不在于如何构建代码图，而在把结构化理解当作上下文感知和探索的工具，技术核心在于**任务驱动的静态-动态协同与信息选择的自主探索和决策**，并服务于真实世界端到端任务中的**搜索→理解→生成→执行→调试→收敛闭环**

实验结果证明，该方法框架带来了可量化的显著效果收益与token效率提升。

#### 1. 问题定义的根本差异

**RepoGraph和CGM**聚焦于单仓库内的代码修复（SWE-bench），而**RepoMaster**解决的是更有挑战性的任务场景：复用开源仓库复杂项目代码完成端到端真实任务，并基于MLE-R/GitTaskBench进行评测，覆盖多域、可执行、可自动验证的任务。

**对比分析：**
- **RepoGraph/CGM**：给定GitHub issue，在仓库内定位并修改代码以修复bug
- **RepoMaster**：给定自然语言任务（如"去除老照片划痕"），需要：
  ```
  检索合适仓库 → 理解其功能 → 配置环境 → 代码生成 → 执行调试 → 生成可验证输出
  ```

Our evaluation on **GitTaskBench** (18 repositories, 54 tasks) and **MLE-R** (22 ML tasks) covers: Image processing, Video analysis, Speech recognition and other multimodal practical applications. Rather than just code repair. This difference in task complexity directly drives different technical approaches.

#### 2. 核心技术贡献

RepoMaster从一开始就以**"复用开源仓库解决真实世界端到端任务"**为目标，不仅需要理解仓库，还要在受限上下文下完成完整的任务执行流程：

**(1) Hybrid Structural Repository Mapping**

AST遍历提取模块/类/函数，构建HCT/FCG/MDG三种互补结构，并在此基础上进行核心组件识别，以有限上下文提炼"该任务最关键的入口与骨干"。

**(2) 自主探索-执行闭环**

在有限上下文下，智能体在以下环节间动态切换：
- 代码查看 ↔ 依赖追踪
- 代码生成/修改 ↔ 执行调试

而非静态地将图嵌入注意力。图2展示了这一迭代过程如何逐步定位并解决模型缺失、依赖错误等实际问题。

#### 3. 显著的效果优势

**GitTaskBench实验结果：**
- 执行完成率：**75.92%**（vs OpenHands 48.15%）
- 任务通过率：**62.96%**（vs 24.07%）
- token消耗降低：**95%**（154k vs 3094k）

**消融实验**（表3）证明每个组件都有显著贡献，特别是移除"上下文感知探索"后性能下降最多（**-5.56%**），验证了我们算法框架的有效性。

### 4.2 EN

Thank you for your meticulous observations regarding the connections with RepoGraph[1] and CGM[2]. We highly acknowledge the contributions of these parallel works—**RepoGraph** as a pluggable module effectively improved code repair performance, and **CGM** achieved excellent results on SWE-bench-Lite through graph attention mechanisms. We also look forward to exploring the possibilities of future technical integration.

However, our **RepoMaster** differs fundamentally from **RepoGraph/CGM** in task problem definition, method design, and technical contributions:

- **Task Level**: We focus on real, end-to-end tasks, not just code repair
- **Technical Innovation and Contribution**: Our work is application-driven. All algorithms and approaches are designed to serve practical use
  - RepoMaster's contribution **is not** in how to construct code graphs, **but** in using structured understanding as a tool for context-aware exploration. 
  - The technical core is **task-driven static-dynamic collaboration, and autonomous exploration, and decision-making for information selection**, serving the **search→understand→generate→execute→debug→convergence closed loop** in real-world end-to-end tasks. 

Experimental results prove that this methodological framework brings quantifiable significant effectiveness benefits and token efficiency improvements.

#### 1. Fundamental Differences in Problem Definition

**RepoGraph and CGM** focus on code repair within single repositories (SWE-bench), while **RepoMaster** addresses more challenging task scenarios: reusing open-source repository complex project code to complete end-to-end real tasks, evaluated based on MLE-R/GitTaskBench, covering multi-domain, executable, automatically verifiable tasks.

- **RepoGraph/CGM**: Given GitHub issues, locate and modify code within repositories to fix bugs
- **RepoMaster**: Given natural language tasks (e.g., "remove scratches from old photos"), requires:
  ```
  Retrieve suitable repository → Understand its functionality → Configure environment → Code generation → Execute debugging → Generate verifiable output
  ```

Our evaluation on **GitTaskBench** (18 repositories, 54 tasks) and **MLE-R** (22 ML tasks) covers: Image processing, Video analysis, Speech recognition and other multimodal practical applications. Rather than just code repair. This difference in **task complexity** directly drives different technical approaches.

#### 2. Core Technical Contributions

RepoMaster was designed from the beginning with the goal of **"reusing open-source repositories to solve real-world end-to-end tasks"**, requiring not only repository understanding but also **completing the entire task execution process under constrained context**:

**(1) Hybrid Structural Repository Mapping**

AST traversal extracts modules/classes/functions, constructs three complementary structures (HCT/FCG/MDG), and identifies core components on this basis, distilling "the most critical entry points and backbone for this task" with limited context.

**(2) Autonomous Exploration-Execution Closed Loop**

Under limited context, agents dynamically switch between the following phases:
- Code viewing ↔ Dependency tracking
- Code generation/modification ↔ Execution debugging

Rather than statically embedding graphs into attention. Figure 2 shows how this iterative process gradually locates and resolves practical issues like missing models and dependency errors.

**(3) Context-aware Information Selection**

To address the key challenge of limited context in agent applications, we propose programmer-inspired, multi-level information selection with reduction strategies for code, documentation, and feedback logs.


#### 3. Significant Performance Advantages

**GitTaskBench Experimental Results:**
- Execution completion rate: **75.92%** (vs OpenHands 48.15%)
- Task pass rate: **62.96%** (vs 24.07%)
- Token consumption reduction: **95%** (154k vs 3094k)

**Ablation experiments** (Table 3) prove each component has significant contribution, particularly after removing "context-aware exploration" performance dropped most (**-5.56%**), validating the effectiveness of our algorithmic framework.

*Additionally,  as a minor note, reference CGM[2] is a concurrent work that was made public on arXiv after our submission (22 May 2025).*

## 5. Weaknesses2【done】：The experiments only compare with OpenHands and SWE-Agent, but do not include newer repository-level methods like RepoGraph [1] or other Agents works, such as Agentless [3]. Adding such baselines would better position RepoMaster within the current literature.

### 5.1 ZH

感谢审稿人的宝贵建议。我们完全认同可以与更多最新方法进行比较，以更全面地展示**RepoMaster**的定位。以下说明我们选择**OpenHands**和**SWE-Agent**作为主要基线的理由：

**1. SOTA Agent框架地位**

SWE-bench leaderboard榜单均显示，这两个agent框架在Verified上持续处于**SOTA水平**（OpenHands达到70.4%，SWE-Agent达到66.4%），而Agentless变体整体排名靠后（任务解决率仅为50.8%），而RepoGraph则不在公开榜单上（其论文报告的效果仅为30.33%）。因此我们将OpenHands和SWE-Agent作为**"必须对齐"的强基线**，以确保对RepoMaster的定位公平而具代表性。

**2. 端到端任务解决的能力**

我们的任务设定要求在受控环境中对完整仓库进行以下端到端执行：

```
搜索 → 理解 → 代码生成 → 执行 → 调试 → 任务验证输出
```

并完成真实世界端到端任务，这对于Agent的综合能力有比较大的要求和挑战。相比之下，**RepoGraph缺乏这种端到端任务解决的能力**。

在这一任务设定下：
- **RepoGraph**：主要聚焦在代码修复任务上，即给定GitHub issue，在仓库内定位并修改代码以修复bug，**无法拓展到完整的端到端任务解决场景**
- **OpenHands与SWE-agent**：目前开源社区中最常用、最成熟且在SWE-bench Verified公开榜单上长期占据前列的通用Agent系统，**具备完整的端到端任务解决能力**



### 5.2 EN


Thank you for your suggestion. We fully agree that comparing with more recent methods could further clarify RepoMaster's positioning. 
However, for completing real-world end-to-end tasks in our benchmarks—which poses significant requirements and challenges for agent's comprehensive capabilities—many frameworks, including **RepoGraph**,  **lack the ability** to support full pipeline resolution.

In fact, **RepoGraph[1] is a technique which could be integrated into general agent frameworks, and it lacks true end-to-end task-solving capability**. Meanwhile, **Agentless[3] is an earlier work categorized as "procedural framework" instead of "agent framwork" in [1], and is less powerful and general than our primary baselines: OpenHands and SWE-Agent (identified as agent frameworks in [1])**.

Below, we detail our baseline selection rationale:

**1. SOTA Agent Framework Status**

SWE-bench leaderboard consistently ranks OpenHands and SWE-Agent at **SOTA levels** on Verified (OpenHands: 70.4%, SWE-Agent: 66.4%), in contrast, Agentless variants rank substantially lower (with only 50.8% task resolution rate), and RepoGraph is not on the public leaderboard (and, in fact, can not handle such task).
In their own paper [1], Agentless + RepoGraph only achieved 29.67% (the best performance) on SWE-Bench-Lite. 
Therefore, we positioned OpenHands and SWE-Agent as **"must-align" strong baselines** for fair and representative positioning of RepoMaster.

**2. End-to-End Task Resolution Capabilities**

Our task setting requires full end-to-end execution on complete repositories:

```
Search → Understand → Code Generation → Execute → Debug → Task Verification Output
```

Under this task setting:
- RepoGraph: Primarily focuses on code repair tasks, i.e., given GitHub issues, locate and modify code within repositories to fix bugs, **unable to extend to complete end-to-end task resolution scenarios**
- OpenHands and SWE-agent: **Currently the most commonly used, daily-updated, mature general Agent systems** in the open-source community, long dominating the SWE-bench Verified public leaderboard, **with proven complete end-to-end task resolution capabilities**


## 6. Weaknesses3 & Limitations【Pending】：The proposed GitTaskBench contains only 18 repositories and 54 tasks (line 230-231). While it covers diverse domains, the relatively small size may limit the generality of the conclusions. A larger benchmark would better demonstrate the robustness of the method.

### 6.2 EN
Thanks for your comment. 

First, we'd like to kindly remind you that our RepoMaster evaluation covers **not only** GitTaskBench but also **MLE-R**—a revision of MLE-Bench-Lite—comprising 22 ML Kaggle tasks and 22×3 repositories. 

**In total, we evaluate 76 comprehensive, real-world tasks across 120 repositories.**

Second, beyond task domain diversity, **the properties of our selected full-stack repositories in GitTaskBench are also highly varied and general:**
- Repo Files: 7–1157 (Avg. 204)
- Intra-repo Dependencies: 33–6979 (Avg. 1242.7)
- Intra-repo Calls: 180–40,552 (Avg. 8,651)
- Functions: 25–4,915 (Avg. 1,274.8)
- Lines of Code: 0.58k–351.42k (Avg. 52.63k)

These statistics demonstrate that GitTaskBench is diverse not only in task domains, but also in the repositories themselves—a critical aspect for evaluating repository-centric tasks.

Third, we acknowledge your concern about task/scale. However, **existing comparable benchmarks at this complexity and comprehensiveness level are similarly scoped**:
- Full MLE-Bench[1]: 72 ML tasks
- ML-Bench-A[2]: 55 (for script) + 13 (for code) repo-level ML-related tasks
- MLAgentBench[3]: 13 ML experiments
- SWE-Bench[4] (smaller-granularity tasks, repair program): 12 Python repos
- M3ToolEval[5] (multi-tool, multi-turn calling): 82 tasks, 5 domains
While more tasks can further enhance generality, our method’s capabilities are rigorously validated on two strong, diverse benchmarks, supporting robust and generalizable conclusions.
- PaperBench[6]: 20 tasks/repos/papers.

>[1] Jun Shern Chan, et al. MLE-bench: Evaluating Machine Learning Agents on Machine Learning Engineering. 2024  
[2] Tang, Xiangru, et al. ML-Bench: Evaluating Large Language Models and Agents for Machine Learning Tasks on Repository-Level Code. 2023  
[3] Huang, Qian, et al. MLAgentBench: Evaluating Language Agents on Machine Learning Experimentation. 2023  
[4] Yang, John, et al. SWE-bench: Can Language Models Resolve Real-World GitHub Issues? 2023  
[5] Wang, Xingyao, et al. Executable Code Actions Elicit Better LLM Agents. 2024  
[6] Starace, Giulio, et al. PaperBench: Evaluating AI's Ability to Replicate AI Research. 2025


## 7. Q1【done】: The implementation filters to Python files. Would the approach extend easily to, e.g., multi-language projects (C++, Java) on GitHub?

### 7.1 ZH

感谢审稿人就**多语言项目**（如C++、Java）可扩展性提出的深刻问题。

首先我们抱歉论文中的描述表达有些误导，事实上，我们并不是不对其他类型文件进行操作，而仅仅是为了强调对于文件内部内容的图的构建时候，是面向“Python files”————因为我们的仓库在收集、选取时候就是选取的python/pytorch-based的仓库，任务也领域也是机器学习相关。然而，可以丝滑无痛取消这一限制，或者这一限制并不是必须的。

其次，我们的方法本身就旨在**与语言无关**，并确实易于扩展到其他编程语言。这里我们阐明了设计原理和可扩展性：

#### 1. 与语言无关的架构

虽然我们选择**Python**进行评估是因为它在深度学习和机器学习领域占据主导地位（这与我们的基准测试任务一致），但我们的核心方法从根本上来说与语言无关。
**层次结构分析（HCT）**、**函数调用图（FCG）**和**模块依赖图（MDG**的构建依赖于抽象语法树（AST）解析, 该解析适用于大多数主流语言，包括C++、Java、JavaScript等

#### 2. 互补探索机制

我们的框架采用两种互补的探索策略：

- **2.1 基于图的探索**：针对具有丰富结构信息的语言（例如Python、Java、C++）
- **2.2 基于树的层次结构探索**：作为脚本语言（.sh）或结构简单的项目文件（.yaml, .json）的主要探索方式

#### 3. 自适应策略选择

这种双重方法确保了跨不同语言范式的稳健性能。根据我们的经验观察，**RepoMaster**会自适应地选择最合适的策略：
- 对于较简单的存储库使用**基于树的搜索**
- 对于具有复杂依赖关系的复杂项目则使用**基于图的分析**

#### 4. 未来扩展计划

我们认同这是未来工作的一个重要方向，并计划在下一版本中将评估范围扩展到多语言存储库。**RepoMaster的模块化设计**使此扩展变得简单易用，我们预计不同编程语言的性能提升也将类似。

### 7.2 EN

Thank you for raising this insightful question about **multi-language project** (such as C++, Java) extensibility.

First, we apologize for the somewhat misleading description in the paper. In fact, we do not exclude operating on other types of files; all types of files are parsed for tree-based structure construction; we only emphasized “Python files” because the graph construction for file content is currently focused on them—this is simply because our selected repositories are all Python/PyTorch-based, and the tasks themselves are in the machine learning domain. However, this restriction can be easily and seamlessly removed, as it is not fundamental to our approach.

Second, our approach is designed to be **language-agnostic** and **can readily extend to additional programming languages**. Here we clarify the design principles and extensibility:

#### 1. Language-Agnostic Architecture

While we chose **Python** for evaluation because it dominates the deep learning and machine learning fields (consistent with our benchmark tasks), our core approach is fundamentally language-agnostic. 
Hierarchical Component Tree (HCT), Function Call Graph (FCG), and Module Dependency Graph (MDG) construction relies on Abstract Syntax Tree (AST) parsing, which applies to most mainstream languages, including C++, Java, JavaScript, etc.

#### 2. Complementary Exploration Mechanisms

Our framework employs two complementary exploration strategies:

- **2.1 Graph-based Exploration**: For languages with rich structural information (e.g., Python, Java, C++)
- **2.2 Tree-based Hierarchical Exploration**: As the primary exploration method for scripting languages (.sh) or structurally simple project files (.yaml, .json)

#### 3. Adaptive Strategy Selection

This dual approach ensures robust performance across different language paradigms. Based on our empirical observations, **RepoMaster** adaptively selects the most appropriate strategy:
- Uses **tree-based search** for simpler repositories
- Uses **graph-based analysis** for complex projects with intricate dependencies

#### 4. Future Extension Plans

We acknowledge this as an important direction for future work and plan to extend the evaluation scope to multi-language repositories in the next version. **RepoMaster's modular design** makes this extension straightforward, and we anticipate similar performance improvements across different programming languages.


## 8. Q2【done】: Will the authors release the GitTaskBench tasks and code for RepoMaster?

### 8.1 ZH
Hi, 感谢您的兴趣，。我们的GitTaskBench是已经开源的，且其的具体匿名链接时已经放入现在投稿的论文中的，作为参考文献【26】的形式给出。但是好像没有以footnote方式给出，可能让您没有注意到。详见参考文献【26】：GitTaskBench: Anonymous github repository. （关于GitTaskBench的所有tasks和code都在以上这个匿名的GitHub链接中。）
此外，我们已经在github仓库上传了我们的RepoMaster项目代码，我们会在后续论文版本中同步开源。
### 8.2 EN

Hi, thank you for your interest. Our GitTaskBench is already open source, and its specific anonymous link has been placed in the currently submitted paper as reference [26]. However, it seems it was not provided as a footnote, which may have caused you to miss it. Please refer to reference [26]: GitTaskBench: Anonymous github repository. (All tasks and code for GitTaskBench are available in the above anonymous GitHub link.)

Additionally, we have uploaded our RepoMaster project code to the github repository, and we will synchronously open source it in subsequent paper versions.

## 9. Q3【done】: How were the thresholds (e.g. top-20 modules, top-10 classes, context window sizes) chosen? Is performance sensitive to these?
### 9.1 ZH

感谢建议，非常同意，我们确实应该展开介绍一下我们模块级重要性评分的实验分析过程，我们将在后续论文附录中补充介绍实验细节。

#### 实验设计过程

**1. 测试集构建与权重优化**
- 首先我们人为构建了几个仓库的核心文件模块作为我们的测试集
- 然后通过**RepoMaster**中的不同权重组合策略的表现来针对性优化来提升重要文件集的召回率

**2. 维度筛选与消融实验**
- 通过实验对比，我们去除了重合度比较高的几个评估维度的分数
- 对每个单一维度的策略进行了简单的**消融实验**
- 最后保留了论文中所给出的**六个评估维度**，并给予相同的权重

**3. Top-k参数优化**
- 我们逐步增加top-k的模块数量
- 最后核心模块的**top-20召回率达到70%**
- 进一步增加top-k，召回收益并不明显

#### Top-k模块召回率实验结果

| Top-k  | Recall Rate (%) | Relative Improvement |
|--------|----------------|---------------------|
| Top-5  | 46.6           | -                   |
| Top-10 | 60.0           | +13.4%              |
| Top-20 | 70.0           | +10.0%              |
| Top-30 | 73.3           | +3.3%               |

实验表明，**Top-20**能够在召回率和效率之间达到良好平衡，进一步增加k值带来的收益递减。

#### 关于权重敏感性分析

关于整体效果的权重敏感性分析，考虑到实验的时间成本较大，我们并没有系统地做一些定量分析，在后续的论文版本中我们可以补充更全面的实验分析结果。

#### Context Window
context window的设置是我们基于实验观察发现的经验值，当LLM总的context的上下文长度超过50k token后，推理能力逐渐下降，这会影响到后续的代码生成、代码修改和代码调试，所以我们优先考虑高信息密度的context。此外整体性能对这些因素不是很敏感，因为我们在LLM的context超过一定长度后，我们会进行过往执行轨迹的反思，同时对已有的探索过程进行最优路径抽取，只保留最有效的执行轨迹信息后，让LLM思考一个更好的解决方案，进行新的探索。这部分我们会设置最大回退重试次数为3。
### 9.2 EN

Thank you for the suggestion. 这部分细节的参数设置是如何选取的以及关于他们的sensitivity analysis我们因为篇幅原因没有放入论文的正文中。We agree that 他们是非常重要的 并且indeed expand on the experimental analysis process of our module-level importance scoring. We will supplement the experimental details in the appendix of subsequent paper versions.

#### Experimental Design Process

**1. Test Set Construction and Weight Optimization**
- First, we manually constructed the core file modules of several repositories as our test set
- Then we performed targeted optimization through the performance of different weight combination strategies in **RepoMaster** to improve the recall rate of important file sets

**2. Dimension Screening and Ablation Experiments**
- Through experimental comparison, we removed scores from several evaluation dimensions with high overlap
- We conducted simple **ablation experiments** on each single dimension strategy
- Finally, we retained the **six evaluation dimensions** presented in the paper and gave them equal weights

**3. Top-k Parameter Optimization**
- We gradually increased the number of top-k modules
- Finally, the **top-20 recall rate of core modules reached 70%**
- Further increasing top-k did not yield significant recall benefits

#### Top-k Module Recall Rate Experimental Results

| Top-k  | Recall Rate (%) | Relative Improvement |
|--------|----------------|---------------------|
| Top-5  | 46.6           | -                   |
| Top-10 | 60.0           | +13.4%              |
| Top-20 | 70.0           | +10.0%              |
| Top-30 | 73.3           | +3.3%               |

Experiments show that **Top-20** achieves a good balance between recall rate and efficiency, with diminishing returns from further increasing k values.

#### Regarding Weight Sensitivity Analysis

Regarding overall effectiveness weight sensitivity analysis, considering the significant time cost of experiments, we did not systematically conduct quantitative analysis. In subsequent paper versions, we can supplement more comprehensive experimental analysis results.

#### Context Window
The context window setting is an empirical value we discovered based on experimental observations. When the LLM's total context length exceeds 50k token, reasoning ability gradually declines, which affects subsequent code generation, code modification, and code debugging. Therefore, we prioritize high information density context. Additionally, overall performance is not very sensitive to these factors because when the LLM's context exceeds a certain length, we perform reflection on past execution trajectories, simultaneously extracting optimal paths from existing exploration processes, retaining only the most effective execution trajectory information, then having the LLM think of a better solution for new exploration. We set the maximum rollback retry count to 3 for this part.

# Reviewer 77DM
## 10. Weakness (3) & (4)【done】: How does RepoMaster deal with stale or broken dependencies found during exploration? What are the assumptions on the quality of README or internal documentation? What happens when the README is missing or incorrect?
### 10.1 EN
We thank the reviewer for raising these important practical concerns. Indeed, handling stale dependencies and unreliable documentation are critical challenges for real-world repository utilization.
Handling Stale/Broken Dependencies: RepoMaster incorporates robust error recovery mechanisms specifically designed for such scenarios. As demonstrated in our GitTaskBench experiments (Section E.2), when encountering missing dependencies (e.g., "ModuleNotFoundError"), RepoMaster:
1. Automatically identifies and installs missing packages through iterative error analysis
2. Adapts execution strategies when dependencies fail (e.g., switching from GPU to CPU versions)
3. Leverages our hierarchical repository analysis to find alternative implementations when primary paths fail

README Quality Assumptions: While READMEs provide valuable initial guidance, RepoMaster is explicitly designed NOT to rely solely on documentation. Our hierarchical repository analysis (Section 3.2) constructs three complementary structural representations (HCT, FCG, MDG) that provide ground-truth understanding independent of documentation quality. This is why RepoMaster significantly outperforms baseline methods that primarily follow README instructions.
Missing/Incorrect Documentation: When documentation is absent or misleading, RepoMaster's exploration tools (Section 3.3.1) enable autonomous discovery through:
1. Structural analysis to identify entry points and core components
2. Dependency tracing to understand component relationships
3. Iterative execution with feedback-based learning

The case study in Figure3 and Figure2 illustrates this capability - despite incomplete documentation, RepoMaster successfully completed the task through structural understanding and adaptive exploration, while baselines failed.

## 11. Q1 & Weakness(1)【done】：Why restrict analysis to .py files only? Many practical projects involve configurations (.yaml, .json), scripts (.sh), or compiled extensions (.cpp). Does this limit applicability? No mention of how the model adapts or generalizes to diverse languages or non-Python repositories.
### 11.1 ZH

我们感谢审稿人提出关于多语言项目（如**C++、Java**）可扩展性的深刻问题。

我们的方法旨在**与语言无关**，并易于扩展到其他编程语言。这里我们阐明了设计原理和可扩展性：

1. 与语言无关的架构 ：虽然我们选择 Python 进行评估是因为它在深度学习和机器学习领域占据主导地位（这与我们的基准测试任务一致），但我们的核心方法从根本上来说与语言无关。层次结构分析 (HCT)、函数调用图 (FCG) 和模块依赖图 (MDG) 的构建依赖于抽象语法树 (AST) 解析，该解析适用于大多数主流语言，包括 C++、Java、JavaScript 等。

**2. 互补探索机制**

我们的框架采用两种互补的探索策略：

- **2.1 基于图的探索**：针对具有丰富结构信息的语言（例如Python、Java、C++）
- **2.2 基于树的层次结构探索**：作为脚本语言（.sh）或结构简单的项目文件（.yaml, .json）的主要探索方式

**3. 自适应策略选择**

这种双重方法确保了跨不同语言范式的稳健性能。根据我们的经验观察，**RepoMaster**会自适应地选择最合适的策略：
- 对于较简单的存储库使用**基于树的搜索**
- 对于具有复杂依赖关系的复杂项目则使用**基于图的分析**

**4. 未来扩展计划**

我们认同这是未来工作的一个重要方向，并计划在下一版本中将评估范围扩展到多语言存储库。**RepoMaster**的模块化设计使此扩展变得简单易用，我们预计不同编程语言的性能提升也将类似。

### 11.2 EN

We thank the reviewer for raising this insightful question about **multi-language project** (such as C++, Java) extensibility.

Our approach is designed to be **language-agnostic** and easily extensible to other programming languages. Here we clarify the design principles and extensibility:

**1. Language-Agnostic Architecture:**

While we chose Python for evaluation because it dominates the deep learning and machine learning fields (consistent with our benchmark tasks), our core approach is fundamentally language-agnostic. Hierarchical Component Tree (HCT), Function Call Graph (FCG), and Module Dependency Graph (MDG) construction relies on Abstract Syntax Tree (AST) parsing, which applies to most mainstream languages, including C++, Java, JavaScript, etc.

**2. Complementary Exploration Mechanisms**

Our framework employs two complementary exploration strategies:

- **2.1 Graph-based Exploration**: For languages with rich structural information (e.g., Python, Java, C++)
- **2.2 Tree-based Hierarchical Exploration**: As the primary exploration method for scripting languages (.sh) or structurally simple project files (.yaml, .json)

**3. Adaptive Strategy Selection**

This dual approach ensures robust performance across different language paradigms. Based on our empirical observations, **RepoMaster** adaptively selects the most appropriate strategy:
- Uses **tree-based search** for simpler repositories
- Uses **graph-based analysis** for complex projects with intricate dependencies

**4. Future Extension Plans**

We acknowledge this as an important direction for future work and plan to extend the evaluation scope to multi-language repositories in the next version. **RepoMaster's modular design** makes this extension straightforward, and we anticipate similar performance improvements across different programming languages.

## 12. q2 & q3【done】: Were the weights in the importance scoring scheme manually set or learned? How are the weights (and the corresponding contribution) determined for each feature? Did the authors consider using GNNs or unsupervised learning for structural importance prediction? Are there any negative insights or learnings from such experiments?

### 12.1 EN

Thank you for your valuable suggestion. We agree on the necessity to systematically elaborate on the experimental analysis process of our **module-level importance scoring**. We will supplement the experimental details in the appendix of subsequent paper versions.

#### Experimental Design Process

**1. Test Set Construction and Weight Optimization**
- First, we manually constructed the core file modules of several repositories as our test set
- Then we performed targeted optimization through the performance of different weight combination strategies in **RepoMaster** to improve the recall rate of important file sets

**2. Dimension Screening and Ablation Experiments**
- Through experimental comparison, we removed scores from several evaluation dimensions with high overlap
- We conducted simple **ablation experiments** on each single dimension strategy
- Finally, we retained the **six evaluation dimensions** presented in the paper and gave them equal weights

**3. Top-k Parameter Optimization**
- We gradually increased the number of top-k modules
- Finally, the **top-20 recall rate of core modules reached 70%**
- Further increasing top-k did not yield significant recall benefits

#### Top-k Module Recall Rate Experimental Results

| Top-k  | Recall Rate (%) | Relative Improvement |
|--------|----------------|---------------------|
| Top-5  | 46.6           | -                   |
| Top-10 | 60.0           | +13.4%              |
| Top-20 | 70.0           | +10.0%              |
| Top-30 | 73.3           | +3.3%               |

Experiments show that **Top-20** achieves a good balance between recall rate and efficiency, with diminishing returns from further increasing k values.

#### GNNs or unsupervised learning for structural importance prediction
It is a direction with great complementary potential, and we also look forward to exploring the possibilities of future technical integration. However, our current considerations are mainly from the following dimensions:
**a. Interpretability:** Our feature-based approach clearly explains why certain modules are considered important, which is crucial for debugging.
**b. Computational Efficiency:** Our method runs with small time overhead, completing the entire preprocessing on average within 4.8 seconds for typical repositories (1-5k files)
**c. Generalization Ability:** Rule-based features have better generalization across different programming languages, including C++, Java, JavaScript, etc.

## 13.  q4【done】: What is the fallback strategy when errors persist (e.g., missing dependencies, ambiguous import paths)?

### 13.1 ZH

感谢您提出这个关于**错误处理策略**的重要问题。我们的**RepoMaster**框架实现了一套系统的多层回退机制，可以有效地处理持久性错误：

#### 1. 通过全面的多代理架构实现强大的故障恢复

为了处理选择不合适的存储库的情况，我们设计了一个具有专门恢复代理的**分层多代理系统**：

```
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery or issue solution search
├── Code Agent: Task exploration and execution with repository
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
```

当执行失败发生时，我们的**Schedule Agent**会协调这些专门的代理。

#### 2. Reflection轨迹优化

当初始尝试失败时，我们实施基于**reflection**的恢复策略：

**2.1 执行轨迹分析:** 系统分析过去的执行轨迹以识别重复出现的故障模式

**2.2 最佳路径提取:** 我们提取探索历史，只保留最具信息量的执行轨迹, 同时修剪冗余或失败的尝试

**2.3 上下文优化:** 这确保了LLM有限的上下文窗口在下一次尝试中专注于高价值信息

#### 3. 实证有效性

正如我们的**案例研究（图3）**所示，这一策略对于复杂任务至关重要。例如，在3D姿态估计任务中：
- **RepoMaster**：通过智能回溯成功从多个依赖性错误中恢复
- **基准方法**：要么完全失败，要么在徒劳的尝试中耗尽资源（OpenHands，约140次迭代）

这种多层方法确保**RepoMaster**保持稳健性和效率：实现我们报告的**62.96%**的任务成功率, 同时使用的token比基线少**95%**

### 13.2 EN

Thank you for raising this important question about **error handling strategies**. Our **RepoMaster** framework implements a systematic multi-layer fallback mechanism that can effectively handle persistent errors:

#### 1. Robust Failure Recovery through Comprehensive Multi-Agent Architecture

To handle cases of selecting unsuitable repositories, we designed a **hierarchical multi-agent system** with specialized recovery agents:

```
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery or issue solution search
├── Code Agent: Task exploration and execution with repository
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
```

When execution failures occur, our **Schedule Agent** coordinates these specialized agents.

#### 2. Reflection Trajectory Optimization

When initial attempts fail, we implement **reflection**-based recovery strategies:

**2.1 Execution Trajectory Analysis:** The system analyzes past execution trajectories to identify recurring failure patterns

**2.2 Optimal Path Extraction:** We extract exploration history, retaining only the most informative execution trajectories while pruning redundant or failed attempts

**2.3 Context Optimization:** This ensures the LLM's limited context window focuses on high-value information in the next attempt

#### 3. Empirical Effectiveness

As demonstrated in our **case study (Figure 3)**, this strategy is crucial for complex tasks. For example, in the 3D pose estimation task:
- **RepoMaster**: Successfully recovered from multiple dependency errors through intelligent backtracking
- **Baseline methods**: Either failed completely or exhausted resources in futile attempts (OpenHands, ~140 iterations)

This multi-layer approach ensures **RepoMaster** maintains both robustness and efficiency: Achieving our reported **62.96%** task success rate while using **95%** fewer tokens than baselines

# Reviewer LTVq
## 14. q1【done】: How was the subset use for MLE-R chosen? Why was a subset taken and not the entire MLE-Bench?
### 14.1 ZH

感谢审稿人提出这个重要问题。我来详细说明**MLE-R子集选择**的理由和方法：

#### 1. 计算资源和时间考虑

我们经过初步测试发现，单个任务的平均运行时长超过**10小时**（包括repository搜索、结构分析、代码生成和多次迭代调试）。考虑到需要在**3个不同的LLM**（GPT-4o、Claude 3.5、DeepSeek V3）和**3个框架**（RepoMaster、OpenHands、SWE-Agent）上进行完整评估，完成全部75个任务需要约**6,750小时**的计算时间和大量的API资源消耗，因此我们选择了**OpenAI官方提供的MLE-Bench-lite**作为实验评估测试集（22个任务）。

#### 2. 采用标准化的子集

**MLE-Bench-lite**这是一个经过精心设计的代表性子集，包含了不同领域（CV、NLP、ASR等）和不同难度级别的任务，能够充分反映agent在机器学习工程任务上的能力。这种选择确保了我们的评估具有可比性和标准化。

在使用MLE-Bench-lite的过程中，我们发现有2个任务（detecting-insults-in-social-commentary和the-icml-2013-whale-challenge-right-whale-redux）由于Kaggle数据访问权限问题无法下载。为保持评估的完整性，我们从完整的MLE-Bench中选择了2个替代任务（chaii-hindi-and-tamil-question-answering和tgs-salt-identification-challenge），这两个任务在领域覆盖和难度上与被替换任务相当，从而保持了总数22个任务不变。

#### 3. 代表性保证

尽管是子集，我们的MLE-R仍然覆盖了多样化的ML任务类型，包括图像分类、文本分类、时间序列预测等，能够全面评估agent的repository利用能力。具体的Competition ID列表已在附录Table 6-10中详细展示。

我们相信这种选择在实验可行性和评估全面性之间达到了良好的平衡，同时保持了与现有工作的可比性, 在future work中我们愿意扩展到更多任务进行评估。

### 14.2 EN

Thank you for raising this important question. Let me detail the rationale and methodology for **MLE-R subset selection**:

#### 1. Computational Resources and Time Considerations

Through preliminary testing, we found that the average runtime for a single task exceeds **10 hours** (including repository search, structural analysis, code generation, and multiple iterative debugging sessions). Considering the need for complete evaluation across **3 different LLMs** (GPT-4o, Claude 3.5, DeepSeek V3) and **3 frameworks** (RepoMaster, OpenHands, SWE-Agent), completing all 75 tasks would require approximately **6,750 hours** of computation time and substantial API resource consumption. Therefore, we selected the **OpenAI officially provided MLE-Bench-lite** as our experimental evaluation test set (22 tasks).

#### 2. Adopting Standardized Subset

**MLE-Bench-lite** is a carefully designed representative subset containing tasks from different domains (CV, NLP, ASR, etc.) and different difficulty levels, capable of fully reflecting agent capabilities in machine learning engineering tasks. This choice ensures our evaluation has comparability and standardization.

During the use of MLE-Bench-lite, we found that 2 tasks (detecting-insults-in-social-commentary and the-icml-2013-whale-challenge-right-whale-redux) could not be downloaded due to Kaggle data access permission issues. To maintain evaluation completeness, we selected 2 alternative tasks from the complete MLE-Bench (chaii-hindi-and-tamil-question-answering and tgs-salt-identification-challenge). These two tasks are comparable to the replaced tasks in domain coverage and difficulty, thus maintaining the total of 22 tasks unchanged.

#### 3. Representativeness Guarantee

Despite being a subset, our MLE-R still covers diverse ML task types, including image classification, text classification, time series prediction, etc., capable of comprehensively evaluating agent repository utilization capabilities. The specific Competition ID list has been detailed in appendix Tables 6-10.

We believe this choice achieves a good balance between experimental feasibility and evaluation comprehensiveness, while maintaining comparability with existing work. In future work, we are willing to extend to more tasks for evaluation.

## 15. q2【done】: As for other baselines, why wasn't AIDE [1] used? It holds the current state of the art on MLE-Bench.

### 15.1 ZH

感谢审稿人就**AIDE**提出的关键问题。虽然AIDE在原始MLE-Bench上表现卓越，下文将说明其未纳入本评估基准的原因。

**1. 基本任务差异**

关键原因是**AIDE**和**RepoMaster**针对的是根本不同的问题：

- **1.1 AIDE**：专为Kaggle竞赛从头开始生成代码而设计，具有高度定制的工作流程，可满足Kaggle任务的特定要求
- **1.2 RepoMaster**：从一开始就以"复用开源仓库端到端解决真实世界任务"为目标，不仅需要理解仓库，还要在受限上下文下完成**搜索→理解→代码生成/编辑→执行→调试→生成可验证输出**的全流程

**2. AIDE架构能力受限**

**AIDE**缺乏基于复杂代码项目端到端解决任务的基本功能，包括：

- **a)** 现有代码库的结构分析
- **b)** 跨文件的依赖关系跟踪和导航
- **c)** 选择性代码修改和生成而非单纯代码生成

**3. 基线选择的合理性**

而**SWE-Agent**和**OpenHands**和我们的**RepoMaster**，都是通用Code Agent架构，从而确保公平比较。

此外在代码修复任务的**SWE-bench leaderboard**榜单均显示，这两个agent框架在Verified上持续处于SOTA水平。因此我们将**OpenHands**和**SWE-Agent**作为"必须对齐"的强基线，以确保对RepoMaster的定位公平而具代表性。

**4. 补充实验结果**

最后，参考您对可比性的担忧和建议，我们使用**DeepSeek V3**对mle-bench上相同的**22个任务**进行了AIDE的额外实验：

- **Valid Submissions:** RepoLearner **(86.36%)** vs AIDE **(63.64%)**
- **Gold Medals:** RepoLearner **(13.64%)** vs AIDE **(4.55%)**

Our advantage is very obvious, and we will subsequently add these explanations to the paper.

#### 实验结果对比

| Model | Make submission | Valid Submissions | Above Median | Bronze Medals | Silver Medals | Gold Medals | Total Medals |
|-------|----------------|-------------------|--------------|---------------|---------------|-------------|--------------|
| RepoLearner (deepseek) | 95.45% | 86.36% | 36.36% | 4.55% | 4.55% | 13.64% | 22.73% |
| AIDE (deepseek) | 63.64% | 63.64% | 18.18% | 0% | 0% | 4.55% | 4.55% |


### 15.2 EN

Thank you for raising this critical question regarding **AIDE**. While AIDE demonstrates superior performance on the original MLE-Bench, we would like to explain the rationale behind its exclusion from our evaluation baseline.

**1. Fundamental Task Differences**

The key reason is that **AIDE** and **RepoMaster** address fundamentally different problems:

- **1.1 AIDE**: Designed specifically for generating code from scratch for Kaggle competitions, with highly customized workflows to meet the specific requirements of Kaggle tasks
- **1.2 RepoMaster**: Aimed from the beginning at "reusing open-source repositories to solve real-world tasks end-to-end", requiring not only repository understanding but also completing the full process of **search→understand→code generation/editing→execute→debug→generate verifiable output** under constrained context

**2. Limitations in AIDE's Architecture Capabilities**

**AIDE** lacks the fundamental functions necessary for solving tasks end-to-end for complex code projects, including:

- **a)** Structural analysis of existing codebases
- **b)** Cross-file dependency tracking and navigation
- **c)** Selective code modification and generation rather than pure code generation

**3. Rationality of Baseline Selection**

While **SWE-Agent**, **OpenHands**, and our **RepoMaster** are all general-purpose Code Agent architectures, ensuring fair comparison.

Additionally, the **SWE-bench leaderboard** for code repair tasks consistently shows these two agent frameworks at SOTA levels on Verified. Therefore, we positioned **OpenHands** and **SWE-Agent** as "must-align" strong baselines to ensure fair and representative positioning of RepoMaster.

**4. Supplementary Experimental Results**

Finally, in reference to your concerns and suggestions about comparability, we conducted additional experiments with AIDE using **DeepSeek V3** on the same **22 tasks** on mle-bench:

- **Valid Submissions:** RepoLearner **(86.36%)** vs AIDE **(63.64%)**
- **Gold Medals:** RepoLearner **(13.64%)** vs AIDE **(4.55%)**

Our advantage is very obvious, and we will subsequently add these explanations to the paper.

#### 实验结果对比

| Model | Make submission | Valid Submissions | Above Median | Bronze Medals | Silver Medals | Gold Medals | Total Medals |
|-------|----------------|-------------------|--------------|---------------|---------------|-------------|--------------|
| RepoLearner (deepseek) | 95.45% | 86.36% | 36.36% | 4.55% | 4.55% | 13.64% | 22.73% |
| AIDE (deepseek) | 63.64% | 63.64% | 18.18% | 0% | 0% | 4.55% | 4.55% |


## 16. q3: I would have liked to see a more comprehensive description of GitTaskBench in the paper. Looking through the Appendix/repo, I found the tasks to be interesting, and I think a comprehensive description in the paper is warranted. I'm interested in the tasks in GitTaskBench, and would like to know more. For example, in some of the tasks, it seems like there could be several correct answers (image coloring, style transfer). How are correct solutions determined? 

### 16.1 ZH
感谢您对 GitTaskBench 表现出的浓厚兴趣，我们的GitTaskBench是已经开源的，且已经放入投稿的论文中，作为参考文献【26】的形式给出。
详见参考文献【26】：GitTaskBench: Anonymous github repository. 

关于GitTaskBench的所有tasks和code都在以上这个匿名的GitHub链接中。

此外，很高兴您想了解更多，我们可以给您提供我们的对于这些任务的execution completion（EC）（finish）和task pass（TP）（correct）的评测方法：
EC measures the proportion of cases where the agent successfully executes the target code repository and generates outputs in an acceptable format (e.g., \texttt{.jpg} or \texttt{.png} for image processing tasks). This metric reflects the agent's compatibility with the code repository and its basic operational capability. It ensures that: (1) the output file(s) exist, (2) the output file(s) are not empty, and (3) the output format can be correctly processed by the testing scripts.

TP quantifies the agent's actual performance quality in task completion. It is determined by formulating evaluation test functions and defining concrete success and failure criteria using established metrics tailored to each task, drawing on standards recognized within the domain developer community. TPR requires the agent's outputs to satisfy predefined quality standards, such as functional correctness, result completeness, or achieving specific task objectives. 
For example, 
Image coloring's success criteria is " CIEDE2000 colour difference \le 2.0 and NIQE \le 7.0 on the output image.
比如 Style Transfer in Video Processing, correct solutions determined if the average SSIM between input and output frames is $\ge 0.7$, and the FID score is $\le 400$.
In speech enhancement tasks, success might be defined by achieving a PESQ $\ge 2.0$ (indicating acceptable perceptual quality) and a SNR $\ge 15 dB$ (suggesting good suppression of noise). 
Tasks failing to meet these thresholds are marked as failures.


考虑到篇幅有限，且这篇文章确实是更偏向于算法和框架的构造，我们非常抱歉没有添加对其"comprehensive description" in this version. 不过我们承诺，如果我们有幸中稿，之后增加的一页，将着重介绍这部分内容。
再次感谢您的兴趣。

### 16.2 EN

Hi, thank you for your interest in GitTaskBench~ But I would like to friendly remind you that our GitTaskBench is already open source and has been included in the submitted paper as reference [26].
Please see reference [26]: GitTaskBench: Anonymous github repository.

All tasks and code for GitTaskBench are in the above anonymous GitHub link.

Additionally, we are happy that you want to learn more. We can provide you with our evaluation methods for execution completion (EC) (finish) and task pass (TP) (correct) for these tasks:

EC measures the proportion of cases where the agent successfully executes the target code repository and generates outputs in an acceptable format (e.g., .jpg or .png for image processing tasks). This metric reflects the agent's compatibility with the code repository and its basic operational capability. It ensures that: (1) the output file(s) exist, (2) the output file(s) are not empty, and (3) the output format can be correctly processed by the testing scripts.

TP quantifies the agent's actual performance quality in task completion. It is determined by formulating evaluation test functions and defining concrete success and failure criteria using established metrics tailored to each task, drawing on standards recognized within the domain developer community. TPR requires the agent's outputs to satisfy predefined quality standards, such as functional correctness, result completeness, or achieving specific task objectives.

For example, Image coloring's success criteria is "CIEDE2000 colour difference ≤ 2.0 and NIQE ≤ 7.0 on the output image.
For example, Style Transfer in Video Processing, correct solutions determined if the average SSIM between input and output frames is ≥ 0.7, and the FID score is ≤ 400.
In speech enhancement tasks, success might be defined by achieving a PESQ ≥ 2.0 (indicating acceptable perceptual quality) and a SNR ≥ 15 dB (suggesting good suppression of noise).
Tasks failing to meet these thresholds are marked as failures.

Considering the limited space and that this article is indeed more focused on algorithm and framework construction, we sincerely apologize for not adding a "comprehensive description" in this version. However, we promise that in the revised version of this article we will add an extra page specifically to introduce this content.
Thank you again for your interest.

## 17. Else【done】: In general I like the style of the figures, but I think they're too information-dense. They would be easier to understand if they were simpler in my opinion.
### 17.1 ZH
Thanks for your insightful suggestion and your like. We are sorry 这个确实有一些information-dense 考虑到篇幅的有限，将几个图的信息放到了一起，以及对构图进行过努力的压缩。我们非常同意您的观点，或许直接将图的面积增大会改善这一点，如果中稿会增加一页，我们将立刻调整为 simpler and 更宽松的图。

### 17.2 EN

Thanks for your insightful suggestion and your appreciation. We apologize that this indeed has some information-dense content. Considering the limited space, we placed information from several figures together and made efforts to compress the composition. We completely agree with your viewpoint. Perhaps directly increasing the figure area would improve this issue. If accepted, with the additional page, we will immediately adjust to simpler and more spacious figures.


## 18. Else【done】: The descriptions sometimes lack detail. In Table 3's description, it should specify that the ablation study was done on RepoMaster + 4o on GitTaskBench.
### 18.1 ZH
我们衷心感谢审稿人对表 3 描述的清晰度提供宝贵的反馈。我们完全同意，实验设置细节对于读者正确解释消融研究结果至关重要。当前我们这部分实验细节是在4.4节中说明的，但是在后续论文版本中我们会在table3描述中明确指出消融研究是在GitTaskBench上对RepoMaster + 4o进行的

### 18.2 EN

We sincerely thank the reviewer for providing valuable feedback on the clarity of Table 3's description. We completely agree that experimental setup details are crucial for readers to correctly interpret ablation study results. Currently, these experimental details are explained in Section 4.4, but in subsequent paper versions, we will explicitly state in the table 3 description that the ablation study was conducted on RepoMaster + 4o on GitTaskBench.


## 19. Else【done】: I think that seeing RepoMaster results on SWE-Bench would be very interesting. Many of the repos that make up SWE-Bench contain many files and lines of code, and it seems like an agent that uses Hybrid Hierarchical Analysis + Code Exploration could achieve high performance.

### 19.1 ZH

感谢您的建议和对我们工作的兴趣！您关于在**SWE-Bench**上测试RepoMaster的观点非常有见地。我们完全同意，SWE-Bench中的大型代码库（具有众多文件和代码行）正是我们的**Hybrid Hierarchical Analysis + Code Exploration**方法旨在解决的场景。

**未来工作规划：**

我们已将**SWE-Bench验证**添加到我们的近期路线图中，并将在完成后提交我们的结果。

**持续研究方向：**

此外，我们正在进行的工作包括：
- 通过**强化学习**将RepoMaster的代理推理能力训练到基础模型中
- 我们希望RepoMaster的复杂存储库理解和探索能力，特别是**Hybrid Hierarchical Analysis + Code Exploration**机制，将在复杂代码库开发任务上优于现有系统（如Claude Code）

这些努力是我们持续工作系列的一部分，实验已经在进行中，我们期待在未来分享更多进展。

### 19.2 EN

Thank you for your suggestion and interest in our work! Your perspective on testing RepoMaster on **SWE-Bench** is very insightful. We completely agree that the large codebases in SWE-Bench (with numerous files and lines of code) are exactly the scenarios our **Hybrid Hierarchical Analysis + Code Exploration** method aims to address.

**Future Work Planning:**

We have added **SWE-Bench validation** to our near-term roadmap and will submit our results upon completion.

**Ongoing Research Directions:**

Additionally, our ongoing work includes:
- Training RepoMaster's agent reasoning capabilities into foundation models through **reinforcement learning**
- We hope RepoMaster's complex repository understanding and exploration capabilities, particularly the **Hybrid Hierarchical Analysis + Code Exploration** mechanism, will outperform existing systems (such as Claude Code) on complex codebase development tasks

These efforts are part of our continuous work series, with experiments already underway. We look forward to sharing more progress in the future.

## 20. q4【done】: In terms of GitTaskBench -- how would the results change if the underlying models were changed to o4-mini or Gemini 2.5 Pro? These are competitive cost-wise with the models used in the paper, and would probably yield scores of 70-75% on GitTaskBench, given that the current state of the art gets nearly 63%. In this case, GitTaskBench would be very close to being solved, already on release. How would you address this? I really like the premise of GitTaskBench, but practically speaking, it seems close to being solved.

### 20.1 ZH

感谢审稿人就**GitTaskBench可扩展性**所做的深入思考。我们已系统评估了采用o4-mini和Gemini 2.5 Pro等新模型的潜在影响，现呈现我们的实验分析：

#### 1. 任务难度的长尾分布特性

**GitTaskBench**的设计反映了真实世界任务的难度分布：
- 从简单的**PDF解析**
- 到**图像风格迁移任务**
- 再到复杂的**VideoPose3D姿态估计**

难度呈**指数级增长**。即使整体完成率达到70%，剩余的30%任务代表着需要：
- 深度代码理解
- 复杂依赖管理
- 端到端任务解决的挑战

这种**长尾分布**确保了benchmark的持续相关性。且对于真实世界的端到端任务执行需求来说，**70%仅仅是一个及格线**。

#### 2. 性能提升的边际递减效应

我们的实验分析揭示了一个关键洞察：

当我们将模型从**GPT-4o**升级到**Claude 3.5**时：
- 任务通过率从**40.74%**提升到**62.96%**

然而，深入分析这**22%**的提升发现：
- **>50%**的提升来自环境配置和依赖安装的成功率提高
- **<20%**的提升来自核心的代码库探索和任务执行能力的提升

这表明模型在处理复杂代码库的自主探索和端到端执行方面仍有巨大改进空间，而这正是**GitTaskBench的核心评估目标**。

#### 3. 算法设计的关键作用

即使使用相同的**Claude 3.5**模型，不同框架的表现差异巨大：

| 框架 | 任务通过率 | Token消耗 |
|------|------------|-----------|
| **RepoMaster** | **62.96%** | **154k tokens** |
| **OpenHands** | **24.07%** | **3094k tokens** |

这种显著的**token效率差异**和近**3倍的性能差距**表明，即使底层模型能力提升，**Agent算法框架设计与效率约束**仍然至关重要。

参考**SWE-Bench**代码修复任务评估的发展轨迹：**Gemini 2.5 Pro**达到**63.8%**的代码修复率，而最新的Claude4模型已接近**80%**，这说明纯代码修复任务正在被快速解决。

**GitTaskBench的独特价值：**
通过要求agents处理完整的：
- 代码库理解
- 依赖管理
- 错误诊断
- 端到端任务解决

提供了更全面的评估维度。这也会是**Code Agent下一阶段的核心优化方向**之一，同时我们后续也会定期引入反映最新开发实践的任务，来保持GitTaskBench的挑战性。

### 20.2 EN

Thank you for your thoughtful consideration of **GitTaskBench's scalability**. We have systematically evaluated the potential impact of employing newer models such as o4-mini and Gemini 2.5 Pro. Our experimental analysis reveals:

#### 1. Long-tail Distribution Characteristics of Task Difficulty

**GitTaskBench**'s design reflects the difficulty distribution of real-world tasks:
- From simple **PDF parsing**
- To **image style transfer tasks**
- To complex **VideoPose3D pose estimation**

The difficulty exhibits **exponential growth**. Even if the overall completion rate reaches 70%, the remaining 30% of tasks represent challenges requiring:
- Deep code understanding
- Complex dependency management
- End-to-end task resolution challenges

This **long-tail distribution** ensures the benchmark's continued challenge. For real-world end-to-end task execution demands, **70% is merely a passing grade**.

#### 2. Diminishing Returns of Performance Improvements

Our experimental analysis reveals a key insight:

When we upgraded the model from **GPT-4o** to **Claude 3.5**:
- Task pass rate improved from **40.74%** to **62.96%**

However, in-depth analysis of this **22%** improvement revealed:
- **>50%** of the improvement came from increased success rates in environment configuration and dependency installation
- **<20%** of the improvement came from enhanced core codebase exploration and task execution capabilities

This indicates that models still have enormous room for improvement in handling autonomous exploration and end-to-end execution of complex codebases, which is precisely **GitTaskBench's core evaluation objective**.

#### 3. Critical Role of Algorithm Design

Even using the same **Claude 3.5** model, different frameworks showed dramatic performance differences:

| Framework | Task Pass Rate | Token Consumption |
|-----------|---------------|-------------------|
| **RepoMaster** | **62.96%** | **154k tokens** |
| **OpenHands** | **24.07%** | **3094k tokens** |

This significant **token efficiency difference** and nearly **3x performance gap** demonstrate that even as underlying model capabilities improve, **Agent algorithm framework design and efficiency constraints** remain crucial.

Referencing the development trajectory of **SWE-Bench** code repair task evaluation: **Gemini 2.5 Pro** achieved **63.8%** code repair rate, while the latest Claude4 model approaches **80%**, indicating that pure code repair tasks are being rapidly solved.

**GitTaskBench's Unique Value:** By requiring agents to handle complete: Codebase understanding, Dependency management, Error diagnosis, End-to-end task resolution

It provides more comprehensive evaluation dimensions. This will also be one of **Code Agent's core optimization directions in the next phase**, and we will subsequently regularly introduce tasks reflecting the latest development practices to maintain GitTaskBench's challenge level.
