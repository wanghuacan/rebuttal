# Reviewer1 TDN5
## 1. Weaknesses1【done】: a) The success of the entire workflow is highly dependent on the initial "Repository Search" stage. This stage relies on analyzing user intent, README files, and star counts. The paper could benefit from a deeper discussion of the limitations of this search phase and how the agent might recover if an unsuitable repository is initially selected.
### 1.1 ZH
感谢您的深刻见解。我们承认，存储库搜索阶段对于我们框架的成功至关重要。为了解决这一问题，我们实施了以下几种机制：
1. 基于“搜索与思考”范式的高级搜索策略 我们开发了一种复杂的深度搜索机制，它采用迭代的“搜索与思考”方法，而非依赖于单次搜索尝试。在每次搜索迭代中，代理：
a. 根据搜索和分析结果动态地制定和改进搜索查询
b. 在浏览存储库内容时执行实时相关性评估
c. 维护具有置信度得分的前 k 个候选存储库的排序列表
这种认知搜索过程模仿了人类开发人员探索 GitHub 的方式，通过主动探索而不是被动过滤来逐步加深理解。

2.通过全面的多代理架构实现强大的故障恢复 为了处理选择不合适的存储库的情况，我们设计了一个具有专门恢复代理的分层多代理系统：
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery and ranking
├── Code Agent: Task execution with repository
├──                             ...
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
When execution failures occur, our Schedule Agent coordinates these specialized agents.

3. When the Code Agent encounters execution failures (e.g., dependency conflicts, API mismatches, missing functionality), our Schedule Agent automatically:
(i) Analyzes failure patterns to generate structured feedback
(ii) Adjusts selection criteria based on the failure mode (e.g., if a computer vision task fails due to missing GPU support, it prioritizes CPU-compatible alternatives)
(iii) Seamlessly switches to the next candidate repository

这种自适应多代理方法确保系统能够从各种故障模式中恢复，从简单的依赖问题到基本的存储库不匹配，从而显著增强了框架的实际适用性。

### 1.2 EN

Thank you for your insightful feedback. We acknowledge that the repository search stage is critical to our framework's success. To address this concern, we have implemented several mechanisms:

1. **Advanced Search Strategy Based on "Search and Think" Paradigm**: We developed a sophisticated deep search mechanism that employs an iterative "search and think" approach rather than relying on single search attempts. In each search iteration, the agent:
   a. Dynamically formulates and refines search queries based on search and analysis results
   b. Performs real-time relevance assessment while browsing repository content
   c. Maintains a ranked list of top-k candidate repositories with confidence scores

This cognitive search process mimics how human developers explore GitHub, progressively deepening understanding through active exploration rather than passive filtering.

2. **Robust Recovery through Comprehensive Multi-Agent Architecture**: To handle cases of selecting unsuitable repositories, we designed a hierarchical multi-agent system with specialized recovery agents:
   - Schedule Agent (Orchestrator)
   - ├── DeepSearch Agent: Repository discovery and ranking
   - ├── Code Agent: Task execution with repository
   - ├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
   - ├── Dependency Agent: Handles environment setup and dependency conflicts

When execution failures occur, our Schedule Agent coordinates these specialized agents.

3. When the Code Agent encounters execution failures (e.g., dependency conflicts, API mismatches, missing functionality), our Schedule Agent automatically:
   (i) Analyzes failure patterns to generate structured feedback
   (ii) Adjusts selection criteria based on the failure mode (e.g., if a computer vision task fails due to missing GPU support, it prioritizes CPU-compatible alternatives)
   (iii) Seamlessly switches to the next candidate repository

This adaptive multi-agent approach ensures the system can recover from various failure modes, from simple dependency issues to fundamental repository mismatches, significantly enhancing the framework's practical applicability.


## 2. Weaknesses2【done】: b) The initial "Hierarchical Repository Analysis" involves parsing all source files to build ASTs, dependency graphs, and call graphs. While effective, the paper does not discuss the computational cost or time required for this preprocessing step.

### 2.1 ZH
感谢您的深刻见解。您说得完全正确，预处理步骤的计算成本值得更详细的讨论。我们将在修订版和附录中添加这方面的全面实验数据。
我们的静态分析预处理流程，利用高度优化的库（例如，用于 AST 解析的 tree-sitter）并支持多线程执行。对于典型的存储库（1-5k 个文件） ：整个预处理平均在 4.8 秒内完成，其中包括：
1. AST 构建：~2.1 秒（跨 CPU 核心并行）
2. 依赖/调用图生成：~1.9秒
3. 核心组件识别：~0.8秒

复杂存储库的附加处理：
对于核心组件超过上下文阈值（8k 个 token）的存储库，我们采用基于 LLM 的摘要方法，使用 Deepseek V3 时，耗时约 10 秒。即使对于复杂的存储库，整体预处理时间也保持在 15 秒以内。

### 2.2 EN

Thank you for your insightful feedback. You are absolutely correct that computational costs of preprocessing steps deserve more detailed discussion. We will add comprehensive experimental data on this aspect in the revision and appendix.

Our static analysis preprocessing pipeline leverages highly optimized libraries (e.g., tree-sitter for AST parsing) and supports multi-threaded execution. For typical repositories (1-5k files): the entire preprocessing completes in an average of 4.8 seconds, including:

1. AST construction: ~2.1 seconds (parallelized across CPU cores)
2. Dependency/call graph generation: ~1.9 seconds  
3. Core component identification: ~0.8 seconds

**Additional processing for complex repositories:**
For repositories where core components exceed context thresholds (8k tokens), we employ LLM-based summarization approaches, taking approximately 10 seconds when using Deepseek V3. Even for complex repositories, overall preprocessing time remains under 15 seconds.

This preprocessing cost is amortized across multiple task attempts within the same repository and represents a one-time investment that significantly improves subsequent exploration efficiency, as demonstrated by our 95% token reduction compared to baseline methods.

## 3. Weaknesses3【done】: c) The module-level importance scoring aggregates six features using equal weights. This seems somewhat arbitrary. The paper would be strengthened by a sensitivity analysis on these weights or a more principled method for determining them.

### 3.1 ZH
感谢建议，非常同意，我们确实应该展开介绍一下我们模块级重要性评分的实验分析过程，我们将在后续论文附录中补充介绍实验细节。

首先我们人为构建了几个仓库的核心文件模块作为我们的测试集，然后通过repomaster中的不同权重组合策略的表现来针对性优化，来提升重要文件集的召回率。
通过实验对比，我们去除了重合度比较高的几个评估维度的分数，然后我们对于这些每个单一维度的策略进行了简单的消融实验，最后保留了论文中所给出的六个评估维度，并给予相同的权重。
此外，我们逐步增加top-k的模块数量，最后核心模块的top20召回率达到70%，进一步增加topk，召回收益并不明显。

1. Top-k模块召回率实验结果
暂时无法在飞书文档外展示此内容
实验表明，Top-20能够在召回率和效率之间达到良好平衡，进一步增加k值带来的收益递减。

关于整体效果的权重敏感性分析，考虑到实验的时间成本较大，我们并没有系统地做一些定量分析，在后续的论文版本中我们可以补充更全面的实验分析结果。此外，我们想详细介绍的是：在我们整体算法框架设计中，评估得到的核心模块的作用是在有限的context窗口下，帮助agent对于整体仓库有一个全面的了解。这也是作为repomaster对整个仓库的自主探索入口，自主决策是否要从当前文件扩展搜索和阅读相邻节点的文件。定位到任务相关的代码片段或者文件信息后才送入Agent的context来完成搜索→理解→代码生成/编辑→执行→调试→生成可验证输出的端到端任务自主探索和执行。
### 3.2 EN

Thank you for the suggestion. We completely agree that we should provide more detailed discussion of our module-level importance scoring experimental analysis process, and we will supplement experimental details in the appendix of the subsequent paper.

First, we manually constructed core file modules for several repositories as our test set, then optimized through different weight combination strategies in RepoMaster to improve the recall rate of important file sets. Through experimental comparison, we removed scores from evaluation dimensions with high overlap, then conducted simple ablation experiments on each single-dimensional strategy, finally retaining the six evaluation dimensions presented in the paper with equal weights.

Additionally, we gradually increased the number of top-k modules, ultimately achieving 70% recall rate for core modules with top-20, with no significant recall improvement from further increasing top-k.

**Top-k Module Recall Rate Experimental Results:**
The experiments show that Top-20 achieves a good balance between recall and efficiency, with diminishing returns from further increasing k values.

Regarding overall weight sensitivity analysis, considering the significant time cost of experiments, we did not systematically conduct quantitative analysis, but we can supplement more comprehensive experimental analysis results in future paper versions. Additionally, we want to detail that in our overall algorithm framework design, the core modules identified serve to help agents have comprehensive understanding of the entire repository within limited context windows. This also serves as RepoMaster's entry point for autonomous repository exploration, autonomously deciding whether to expand search and read adjacent node files from current files. Only after locating task-relevant code segments or file information are they fed into the Agent's context to complete the end-to-end autonomous exploration and execution of search→understand→code generation/editing→execution→debugging→generate verifiable output.


# Reviewer aqw3
## 4. Weaknesses1【done】：Many of RepoMaster's ideas e.g.repository graphs and context selection overlap with recent work. For example, the concurrent RepoGraph [1] system also builds a repository-level graph structure to guide LLM code agents. The CGM [2] approach similarly integrates a code graph into an LLM's attention. RepoMaster's novelty seems to lie mostly in the engineering of combining known techniques (AST traversal, call/dependency graphs, heuristic scoring) rather than introducing fundamentally new algorithms. As a result, its contribution feels somewhat incremental relative to these related methods.
### 4.1 ZH
感谢审稿人细致地指出与RepoGraph[1]和CGM[2]的关联。我们高度认可这些并行工作的贡献——RepoGraph作为可插拔模块有效提升了代码修复性能，CGM通过图注意力机制在SWE-bench-Lite上取得了优秀成果，我们也期待未来探索技术融合的可能性。
但我们的RepoMaster与 RepoGraph/CGM 在任务问题定义、方法设计和技术贡献上存在本质差异：
a.在任务层面我们面向真实、端到端任务，而非仅限代码修复。
b.在技术创新和贡献上，RepoMaster 的核心贡献不在于如何构建代码图，而在把结构化理解当作上下文感知和探索的工具，技术核心在于任务驱动的静态-动态协同与信息选择的自主探索和决策，并服务于真实世界端到端任务中的搜索—理解-生成—执行—调试—收敛闭环；实验结果证明，该方法框架带来了可量化的显著效果收益与token 效率提升。

1. 问题定义的根本差异
RepoGraph和CGM聚焦于单仓库内的代码修复（SWE-bench），而RepoMaster解决的是更有挑战性的任务场景：复用开源仓库复杂项目代码完成端到端真实任务，并基于 MLE-R / GitTaskBench进行评测，覆盖多域、可执行、可自动验证的任务
RepoGraph/CGM：给定GitHub issue，在仓库内定位并修改代码以修复bug
RepoMaster：给定自然语言任务（如"去除老照片划痕"），需要：检索合适仓库→理解其功能→配置环境→代码生成→执行调试→生成可验证输出
我们在GitTaskBench（18个仓库，54个任务）和MLE-R（22个ML任务）上的评测涵盖图像处理、视频分析、语音识别等多模态实际应用，而非仅限于代码修复。这种任务复杂度的差异直接驱动了不同的技术路线。

2. 核心技术贡献
RepoMaster 从一开始就以“复用开源仓库解决真实世界端到端任务”为目标，不仅需要理解仓库，还要在受限上下文下完成搜索→理解→代码生成/编辑→执行→调试→生成可验证输出的全流程：
(1) Hybrid Structural Repository Mapping：AST 遍历提取模块/类/函数，构建 HCT/FCG/MDG 三种互补结构，并在此基础上进行核心组件识别，以有限上下文提炼“该任务最关键的入口与骨干”。
(2) 自主探索-执行闭环：在有限上下文下，智能体在"代码查看↔依赖追踪↔代码生成/修改↔执行调试"间动态切换，而非静态地将图嵌入注意力。图2展示了这一迭代过程如何逐步定位并解决模型缺失、依赖错误等实际问题。

3. 显著的效果优势
GitTaskBench：执行完成率75.92%（vs OpenHands 48.15%），任务通过率62.96%（vs 24.07%），token消耗降低95%（154k vs 3094k）
MLE-R：有效提交率95.45%，获得奖牌率27.27%，相对最强基线提升110%
消融实验（表3）证明每个组件都有显著贡献，特别是移除"上下文感知探索"后性能下降最多（-5.56%），验证了我们的算法框架的有效性。

[1] Ouyang, S., Yu, W., Ma, K., Xiao, Z., Zhang, Z., Jia, M., ... & Yu, D. (2024). RepoGraph: Enhancing AI Software Engineering with Repository-level Code Graph. arXiv preprint arXiv:2410.14684.
[2] Tao, Hongyuan, et al. "Code Graph Model (CGM): A Graph-Integrated Large Language Model for Repository-Level Software Engineering Tasks." arXiv preprint arXiv:2505.16901 (2025).
### 4.2 EN


## 5. Weaknesses2【done】：The experiments only compare with OpenHands and SWE-Agent, but do not include newer repository-level methods like RepoGraph [1] or other Agents works, such as Agentless [3]. Adding such baselines would better position RepoMaster within the current literature.

### 5.1 ZH
感谢审稿人的宝贵建议。我们完全认同可以与更多最新方法进行比较，以更全面地展示RepoMaster的定位。以下说明我们的基线选择理由，并承诺在修订版中加入与RepoGraph和Agentless的补充性实验结果与讨论。

我们选择OpenHands和SWE-Agent作为主要基线基于以下考虑：

SOTA Agent框架：SWE‑bench leaderboard榜单均显示，这两个agent框架在 Verified 上持续处于 SOTA水平，而 Agentless 变体整体排名靠后。因此我们将OpenHands和SWE-Agent作为“必须对齐”的强基线，以确保对 RepoMaster 的定位公平而具代表性。

端到端能力的完整性：我们的任务设定要求在受控环境中对完整仓库进行搜索—理解-代码生成—执行—调试的端到端执行，并完成真实世界端到端任务，这对于Agent的综合能力有比较大的要求和挑战。在这一任务设定下，OpenHands 与 SWE‑agent 是目前开源社区中最常用、最成熟且在SWE-bench Verified公开榜单上长期占据前列的通用Agent系统，而repograph主要聚焦在代码修复任务上：即给定GitHub issue，在仓库内定位并修改代码以修复bug。

### 5.2 EN

## 6. Weaknesses3 & Limitations【Pending】：The proposed GitTaskBench contains only 18 repositories and 54 tasks (line 230-231). While it covers diverse domains, the relatively small size may limit the generality of the conclusions. A larger benchmark would better demonstrate the robustness of the method.





## 7. Q1【done】: The implementation filters to Python files. Would the approach extend easily to, e.g., multi-language projects (C++, Java) on GitHub?

### 7.1 ZH
我们感谢审稿人提出关于多语言项目（如 C++、Java）可扩展性的深刻问题

我们的方法旨在与语言无关，并易于扩展到其他编程语言。 这里我们阐明了设计原理和可扩展性：

1. 与语言无关的架构 ：虽然我们选择 Python 进行评估是因为它在深度学习和机器学习领域占据主导地位（这与我们的基准测试任务一致），但我们的核心方法从根本上来说与语言无关。层次结构分析 (HCT)、函数调用图 (FCG) 和模块依赖图 (MDG) 的构建依赖于抽象语法树 (AST) 解析，该解析适用于大多数主流语言，包括 C++、Java、JavaScript 等。

2. 互补探索机制 ：我们的框架采用两种互补的探索策略：
2.1 基于图的探索， 针对具有丰富结构信息的语言（例如 Python、Java、C++）
2.2 基于树的层次结构探索作为脚本语言或结构更简单的项目的后备

这种双重方法确保了跨不同语言范式的稳健性能。根据我们的经验观察，RepoMaster 会自适应地选择最合适的策略：对于较简单的存储库使用基于树的搜索，而对于具有复杂依赖关系的复杂项目则使用基于图的分析。

我们认同这是未来工作的一个重要方向，并计划在下一版本中将评估范围扩展到多语言存储库。RepoMaster 的模块化设计使此扩展变得简单易用，我们预计不同编程语言的性能提升也将类似。

### 7.2 EN


## 8. Q2【done】: Will the authors release the GitTaskBench tasks and code for RepoMaster?

### 8.1 ZH
Hi, 感谢您的兴趣，。我们的GitTaskBench是已经开源的，且其的具体匿名链接时已经放入现在投稿的论文中的，作为参考文献【26】的形式给出。但是好像没有以footnote方式给出，可能让您没有注意到。详见参考文献【26】：GitTaskBench: Anonymous github repository. （关于GitTaskBench的所有tasks和code都在以上这个匿名的GitHub链接中。）
此外，我们已经在github仓库上传了我们的RepoMaster项目代码，我们会在后续论文版本中同步开源。
### 8.2 EN

## 9. Q3【done】: How were the thresholds (e.g. top-20 modules, top-10 classes, context window sizes) chosen? Is performance sensitive to these?
### 9.1 ZH
感谢建议，非常同意，我们确实应该展开介绍一下我们模块级重要性评分的实验分析过程，我们将在后续论文附录中补充介绍实验细节。
1.首先我们人为构建了几个仓库的核心文件模块作为我们的测试集，然后通过repomaster中的不同权重组合策略的表现来针对性优化，来提升重要文件集的召回率。通过实验对比，我们去除了重合度比较高的几个评估维度的分数，然后我们对于这些每个单一维度的策略进行了简单的消融实验，最后保留了论文中所给出的六个评估维度，并给予相同的权重。此外，我们逐步增加top-k的模块数量，最后核心模块的top20召回率达到70%，进一步增加topk，召回收益并不明显。

Top-k模块召回率实验结果
Table1
实验表明，Top-20能够在召回率和效率之间达到良好平衡，进一步增加k值带来的收益递减。

关于整体效果的权重敏感性分析，考虑到实验的时间成本较大，我们并没有系统地做一些定量分析，在后续的论文版本中我们可以补充更全面的实验分析结果。此外，我们想详细介绍的是：在我们整体算法框架设计中，评估得到的核心模块的作用是在有限的context窗口下，帮助agent对于整体仓库有一个全面的了解。这也是作为repomaster对整个仓库的自主探索入口，自主决策是否要从当前文件扩展搜索和阅读相邻节点的文件。定位到任务相关的代码片段或者文件信息后才送入Agent的context来完成搜索→理解→代码生成/编辑→执行→调试→生成可验证输出的端到端任务自主探索和执行。

context window的设置是我们基于实验观察发现的经验值，当LLM总的context的上下文长度超过5w后，推理能力逐渐下降，这会影响到后续的代码生成、代码修改和代码调试，所以我们优先考虑高信息密度的context。此外整体性能对这些因素不是很敏感，因为我们在LLM的context超过一定长度后，我们会进行过往执行轨迹的反思，同时对已有的探索过程进行最优路径抽取，只保留最有效的执行轨迹信息后，让LLM思考一个更好的解决方案，进行新的探索。这部分我们会设置最大回退重试次数为3。
### 9.2 EN

# Reviewer 77DM
## 10. Weakness (3) & (4)【done】: How does RepoMaster deal with stale or broken dependencies found during exploration? What are the assumptions on the quality of README or internal documentation? What happens when the README is missing or incorrect?
### 10.1 ZH
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
### 10.2 EN

## 11. Q1 & Weakness(1)【done】：Why restrict analysis to .py files only? Many practical projects involve configurations (.yaml, .json), scripts (.sh), or compiled extensions (.cpp). Does this limit applicability? No mention of how the model adapts or generalizes to diverse languages or non-Python repositories.
### 11.1 ZH
我们感谢审稿人提出关于多语言项目（如 C++、Java）可扩展性的深刻问题

我们的方法旨在与语言无关，并易于扩展到其他编程语言。 这里我们阐明了设计原理和可扩展性：

1. 与语言无关的架构 ：虽然我们选择 Python 进行评估是因为它在深度学习和机器学习领域占据主导地位（这与我们的基准测试任务一致），但我们的核心方法从根本上来说与语言无关。层次结构分析 (HCT)、函数调用图 (FCG) 和模块依赖图 (MDG) 的构建依赖于抽象语法树 (AST) 解析，该解析适用于大多数主流语言，包括 C++、Java、JavaScript 等。

2. 互补探索机制 ：我们的框架采用两种互补的探索策略：
2.1 基于图的探索：针对具有丰富结构信息的语言（例如 Python、Java、C++）
2.2 基于树的层次结构探索：作为脚本语言(.sh)或结构简单的项目文件(.yaml, .json)的主要探索方式

这种双重方法确保了跨不同语言范式的稳健性能。根据我们的经验观察，RepoMaster 会自适应地选择最合适的策略：对于较简单的存储库使用基于树的搜索，而对于具有复杂依赖关系的复杂项目则使用基于图的分析。

我们认同这是未来工作的一个重要方向，并计划在下一版本中将评估范围扩展到多语言存储库。RepoMaster 的模块化设计使此扩展变得简单易用，我们预计不同编程语言的性能提升也将类似。

### 11.2 EN

## 12. q2 & q3【done】: Were the weights in the importance scoring scheme manually set or learned? How are the weights (and the corresponding contribution) determined for each feature? Did the authors consider using GNNs or unsupervised learning for structural importance prediction? Are there any negative insights or learnings from such experiments?
### 12.1 ZH

感谢建议，非常同意，我们确实应该展开介绍一下我们模块级重要性评分的实验分析过程，我们将在后续论文附录中补充介绍实验细节。
1. 首先我们人为构建了几个仓库的核心文件模块作为我们的测试集，然后通过repomaster中的不同权重组合策略的表现来针对性优化，来提升重要文件集的召回率。通过实验对比，我们去除了重合度比较高的几个评估维度的分数，然后我们对于这些每个单一维度的策略进行了简单的消融实验，最后保留了论文中所给出的六个评估维度，并给予相同的权重。此外，我们逐步增加top-k的模块数量，最后核心模块的top20召回率达到70%，进一步增加topk，召回收益并不明显。

2. Top-k模块召回率实验结果
实验表明，Top-20能够在召回率和效率之间达到良好平衡，进一步增加k值带来的收益递减。

3. 'GNNs or unsupervised learning for structural importance prediction', 这是一个非常有互补潜力的方向，我们也期待未来探索技术融合的可能性。但我们当前的考虑主要是从以下几个维度考虑：
a. 可解释性：我们的基于特征的方法清楚地解释了为什么某些模块被认为重要、这对于调试至关重要。
b. 计算效率：我们的方法以较小的时间开销运行，对于典型的存储库（1-5k 个文件）整个预处理平均在 4.8 秒内完成
c. 泛化：基于规则的特征在不同的编程语言中具有更好的泛化能力，包括 C++、Java、JavaScript 等。

### 12.2 EN

## 13.  q4【done】: What is the fallback strategy when errors persist (e.g., missing dependencies, ambiguous import paths)?

### 13.1 ZH
感谢您提出这个关于错误处理策略的重要问题。我们的 RepoMaster 框架实现了一套系统的多层回退机制，可以有效地处理持久性错误：

1.通过全面的多代理架构实现强大的故障恢复 为了处理选择不合适的存储库的情况，我们设计了一个具有专门恢复代理的分层多代理系统：
Schedule Agent (Orchestrator)
├── DeepSearch Agent: Repository discovery or issue solution search
├── Code Agent: Task exploration and execution with repository
├──                             ...
├── Issue Fix Agent: Analyzes GitHub issues for known problems/solutions
├── Dependency Agent: Handles environment setup and dependency conflicts
When execution failures occur, our Schedule Agent coordinates these specialized agents.

2. reflection 轨迹优化 当初始尝试失败时，我们实施基于reflection的恢复策略：
2.1. 执行轨迹分析 ：系统分析过去的执行轨迹以识别重复出现的故障模式
2.2. 最佳路径提取 ：我们提取探索历史，只保留最具信息量的执行轨迹，同时修剪冗余或失败的尝试
2.3. 上下文优化 ：这确保了 LLM 有限的上下文窗口在下一次尝试中专注于高价值信息

4.实证有效性 正如我们的案例研究（图 3）所示，这一策略对于复杂任务至关重要。例如，在 3D 姿态估计任务中，RepoMaster 通过智能回溯成功从多个依赖性错误中恢复，而基准方法要么完全失败，要么在徒劳的尝试中耗尽资源（OpenHands，约 140 次迭代）。

这种多层方法确保 RepoMaster 保持稳健性和效率，实现我们报告的 62.96% 的任务成功率，同时使用的令牌比基线少 95%。

### 13.2 EN

# Reviewer LTVq
## 14. q1【done】: How was the subset use for MLE-R chosen? Why was a subset taken and not the entire MLE-Bench?
### 14.1 ZH
感谢审稿人提出这个重要问题。我来详细说明MLE-R子集选择的理由和方法：
1. 计算资源和时间考虑：我们经过初步测试发现，单个任务的平均运行时长超过10小时（包括repository搜索、结构分析、代码生成和多次迭代调试）。考虑到需要在3个不同的LLM（GPT-4o、Claude 3.5、DeepSeek V3）和3个框架（RepoMaster、OpenHands、SWE-Agent）上进行完整评估，完成全部75个任务需要约6,750小时的计算时间和大量的API资源消耗，因此我们选择了OpenAI官方提供的MLE-Bench-lite作为实验评估测试集(22 Task)。
采用标准化的子集：MLE-Bench-lite这是一个经过精心设计的代表性子集，包含了不同领域（CV、NLP、ASR等）和不同难度级别的任务，能够充分反映agent在机器学习工程任务上的能力。这种选择确保了我们的评估具有可比性和标准化。
数据可用性调整：在使用MLE-Bench-lite的过程中，我们发现有2个任务（detecting-insults-in-social-commentary和the-icml-2013-whale-challenge-right-whale-redux）由于Kaggle数据访问权限问题无法下载。为保持评估的完整性，我们从完整的MLE-Bench中选择了2个替代任务（chaii-hindi-and-tamil-question-answering和tgs-salt-identification-challenge），这两个任务在领域覆盖和难度上与被替换任务相当，从而保持了总数22个任务不变。
代表性保证：尽管是子集，我们的MLE-R仍然覆盖了多样化的ML任务类型，包括图像分类、文本分类、时间序列预测等，能够全面评估agent的repository利用能力。具体的Competition ID列表已在附录Table 6-10中详细展示。
我们相信这种选择在实验可行性和评估全面性之间达到了良好的平衡，同时保持了与现有工作的可比性, 在future work中我们愿意扩展到更多任务进行评估。

## 15. q2【done】: As for other baselines, why wasn't AIDE [1] used? It holds the current state of the art on MLE-Bench.

### 15.1 ZH
感谢您提出这个关于 AIDE 的重要问题。我们承认 AIDE 在原始 MLE-Bench 上达到了最佳性能，并且很高兴有机会解释为什么它没有被纳入我们的评估基准。

1.基本任务差异： 关键原因是 AIDE 和 RepoMaster 针对的是根本不同的问题：

1.1. AIDE 专为 Kaggle 竞赛从头开始生成代码而设计，具有高度定制的工作流程，可满足Kaggle任务的特定要求
1.2. RepoMaster 从一开始就以“复用开源仓库端到端解决真实世界任务”为目标，不仅需要理解仓库，还要在受限上下文下完成搜索→理解→代码生成/编辑→执行→调试→生成可验证输出的全流程。

2. 这种区别导致了架构上的不兼容：AIDE 缺乏基于存储库的任务的基本功能，包括：a)现有代码库的结构分析;b）跨文件的依赖关系跟踪和导航；c)选择性代码修改和生成而非单纯代码生成

3. 而SWE-Agent和OpenHands和我们的RepoMaster，都是通用Code Agent架构，从而确保公平比较。此外在代码修复任务的任务SWE‑bench leaderboard榜单均显示，这两个agent框架在 Verified 上持续处于 SOTA水平。因此我们将OpenHands和SWE-Agent作为“必须对齐”的强基线，以确保对 RepoMaster 的定位公平而具代表性。

最后，参考您对可比性的担忧和建议，我们使用 DeepSeek V3 对mle-bench上相同的 22 个任务进行了 AIDE 的额外实验, AIDE的整体任务提交成功率是63.4%，而RepoMaster的是86.36%，我们的优势非常明显,我们后续也将在论文中添加这些说明

Table1

## 16. q3: I would have liked to see a more comprehensive description of GitTaskBench in the paper. Looking through the Appendix/repo, I found the tasks to be interesting, and I think a comprehensive description in the paper is warranted. I'm interested in the tasks in GitTaskBench, and would like to know more. For example, in some of the tasks, it seems like there could be several correct answers (image coloring, style transfer). How are correct solutions determined? 

### 16.1 ZH
Hi, 感谢您对GitTaskBench的兴趣~但想友好提醒您，事实上我们的GitTaskBench是已经开源的，且已经放入投稿的论文中，作为参考文献【26】的形式给出。
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


考虑到篇幅有限，且这篇文章确实是更偏向于算法和框架的构造，我们非常抱歉没有添加对其“comprehensive description” in this version. 不过我们承诺，如果我们有幸中稿，之后增加的一页，将着重介绍这部分内容。
再次感谢您的兴趣。

## 17. Else【done】: In general I like the style of the figures, but I think they're too information-dense. They would be easier to understand if they were simpler in my opinion.
### 17.1 ZH
Thanks for your insightful suggestion and your like. We are sorry 这个确实有一些information-dense 考虑到篇幅的有限，将几个图的信息放到了一起，以及对构图进行过努力的压缩。我们非常同意您的观点，或许直接将图的面积增大会改善这一点，如果中稿会增加一页，我们将立刻调整为 simpler and 更宽松的图。


## 18. Else【done】: The descriptions sometimes lack detail. In Table 3's description, it should specify that the ablation study was done on RepoMaster + 4o on GitTaskBench.
### 18.1 ZH
我们衷心感谢审稿人对表 3 描述的清晰度提供宝贵的反馈。我们完全同意，实验设置细节对于读者正确解释消融研究结果至关重要。当前我们这部分实验细节是在4.4节中说明的，但是在后续论文版本中我们会在table3描述中明确指出消融研究是在GitTaskBench上对RepoMaster + 4o进行的


## 19. Else【done】: I think that seeing RepoMaster results on SWE-Bench would be very interesting. Many of the repos that make up SWE-Bench contain many files and lines of code, and it seems like an agent that uses Hybrid Hierarchical Analysis + Code Exploration could achieve high performance.

### 19.1 ZH
Thank you for your suggestion and interest in our work! Your point about testing RepoMaster on SWE-Bench is very insightful. We completely agree that the large codebases in SWE-Bench (with numerous files and lines of code) are exactly the scenarios our Hybrid Hierarchical Analysis + Code Exploration approach is designed to address.
We have added SWE-Bench validation to our near-term roadmap and will submit our results once completed. Additionally, our ongoing work includes training RepoMaster's agentic reasoning capabilities into foundation models through reinforcement learning. We hope that RepoMaster's complex repository understanding and exploration capabilities, particularly the Hybrid Hierarchical Analysis + Code Exploration mechanisms, will outperform existing systems like Claude Code on complex codebase development tasks. These efforts are part of our continuous work series, with experiments already underway, and we look forward to sharing more progress in the future.

## 20. q4【done】: In terms of GitTaskBench -- how would the results change if the underlying models were changed to o4-mini or Gemini 2.5 Pro? These are competitive cost-wise with the models used in the paper, and would probably yield scores of 70-75% on GitTaskBench, given that the current state of the art gets nearly 63%. In this case, GitTaskBench would be very close to being solved, already on release. How would you address this? I really like the premise of GitTaskBench, but practically speaking, it seems close to being solved.

### 20.1 ZH
感谢审稿人对GitTaskBench可扩展性的深入思考。我们认真考虑了使用o4-mini和Gemini 2.5 Pro等新模型的影响，以下是我们的实验分析：
1. 任务难度的长尾分布特性 GitTaskBench的设计反映了真实世界任务的难度分布——从简单的PDF解析到图像风格迁移任务再到复杂的VideoPose3D姿态估计，难度呈指数级增长。即使整体完成率达到70%，剩余的30%任务代表着需要深度代码理解、复杂依赖管理和端到端任务解决的挑战。这种长尾分布确保了benchmark的持续相关性。且对于真实世界的端到端任务执行需求来说，70%仅仅是一个及格线。
2. 性能提升的边际递减效应 我们的实验分析揭示了一个关键洞察：当我们将模型从GPT-4o升级到Claude 3.5时，任务通过率从40.74%提升到62.96%。然而，深入分析这22%的提升发现：
a. >50%的提升来自环境配置和依赖安装的成功率提高
b. <20%的提升来自核心的代码库探索和任务执行能力的提升
这表明模型在处理复杂代码库的自主探索和端到端执行方面仍有巨大改进空间，而这正是GitTaskBench的核心评估目标。
3. 算法设计的关键作用 即使使用相同的Claude 3.5模型，不同框架的表现差异巨大：
RepoMaster: 62.96% (154k tokens)
OpenHands: 24.07% (3094k tokens)
这种显著的token效率差异和近3倍的性能差距表明，即使底层模型能力提升，Agent算法框架设计与效率约束仍然至关重要。
4. 超越代码修复的综合评估 参考SWE-Bench的发展轨迹：Gemini 2.5 Pro达到63.8%的代码修复率，而最新模型已接近80%。这说明纯代码修复任务正在被快速解决。GitTaskBench通过要求agents处理完整的代码库理解、依赖管理、错误诊断和端到端任务解决，提供了更全面的评估维度。这也会是Code Agent下一阶段的核心优化方向之一，同时我们后续也会定期引入反映最新开发实践的任务，来保持GitTaskBench的挑战性。
