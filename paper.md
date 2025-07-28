Title: RepoMaster: Autonomous Exploration and Understanding of GitHub Repositories for Complex Task Solving

URL Source: https://arxiv.org/pdf/2505.21577

Published Time: Sat, 14 Jun 2025 00:15:21 GMT

Markdown Content:
> arXiv:2505.21577v2 [cs.SE] 6 Jun 2025

# RepoMaster: Autonomous Exploration and Understanding of GitHub Repositories for Complex Task Solving 

Huacan Wang 1,* ‡ , Ziyi Ni 1,2 * , Shuo Zhang 3 * , Shuo Lu 1,2 ,Sen Hu 4, Ziyang He 5, Chen Hu 6, Jiaye Lin 7, Yifu Guo 8, Yuntao Du 9,‡ , Pin Lyu 2,‡ 

> 1

UCAS 2CASIA 3BUPT 4PKU 5NUS 6StepFun 7THU 8SCNU 9SDU  

> *

These authors contributed equally to this work. 

† Corresponding authors: wanghuacan17@mails.ucas.ac.cn, yuntaodu@sdu.edu.cn, pin.lv@ia.ac.cn 0.0 0.2 0.4 0.6 0.8 1.0    

> RepoMaster
> OpenHands
> SWE-Agent
> 95.5%
> 50.0%
> 54.5%
> 95.5%
> 45.5%
> 50.0%
> 45.5%
> 9.1%
> 13.6%
> 27.3%
> 4.5%
> 4.5%
> MLE-Bench-Revised Performance Comparison
> Made Submission Valid Submission Above Median Any Medal

Abstract 

The ultimate goal of code agents is to solve complex tasks autonomously. Although large language models (LLMs) have made substantial progress in code generation, real-world tasks typically demand full-fledged code repositories rather than simple scripts. Building such repositories from scratch remains a major challenge. Fortunately, GitHub hosts a vast, evolving collection of open-source repositories, which developers frequently reuse as modular components for complex tasks. Yet, existing frameworks like OpenHands and SWE-Agent still struggle to effectively leverage these valuable resources. Relying solely on README files provides insufficient guid-ance, and deeper exploration reveals two core obstacles: overwhelming information and tangled dependencies of repositories, both constrained by the limited context windows of current LLMs. To tackle these issues, we propose RepoMaster, an autonomous agent framework designed to explore and reuse GitHub repositories for solving complex tasks. For efficient understanding, Re-poMaster constructs function-call graphs, module-dependency graphs, and hierarchical code trees to identify essential components, providing only identified core elements to the LLMs rather than the entire repository. During autonomous execution, it progressively explores related components using our exploration tools and prunes information to optimize context usage. Evaluated on the adjusted MLE-bench, RepoMaster achieves a 110% relative boost in valid submissions over the strongest baseline OpenHands. On our newly released GitTaskBench, RepoMaster lifts the task-pass rate from 24.1% to 62.9% while reducing token usage by 95%. Our code and demonstration materials are publicly available at https://github.com/wanghuacan/RepoMaster .

Preprint. Under review. 1 Introduction 

In recent years, the integration of toolchains [ 1 , 2 , 3 , 4] and iterative reasoning [ 5 , 6, 7, 8] has significantly enhanced large language models (LLMs) in code-related tasks [ 9, 10 , 11 ]. These advancements have enabled LLMs to proficiently complete code snippets [ 12 , 13 ], debug errors [ 14 ], and even address complex machine learning problems [ 15 , 16 ]. However, when confronted with real-world challenges that necessitate task-driven code repositories [ 17 ], they struggle. At present, tackling such tasks remains largely manual and time-consuming due to the complexity and scale of the required code, which makes purely generative approaches impractical [ 11 , 18 , 19 ]. To overcome this, we propose a paradigm shift: reuse and adapt existing repositories as modular components tailored to specific tasks. This approach not only mitigates the challenges associated with repository-level code generation but also supports the broader goal of enabling agents to autonomously address sophisticated tasks using simple natural language instructions [20, 21]. To facilitate this approach, leveraging platforms like GitHub becomes crucial. With over 28 million public repositories out of 190 million total projects, GitHub offers an extensive library of ready-made solutions for code agents [ 17 , 18 , 22 , 23 ]. Developers frequently reuse these repositories to tackle complex problems, yet LLM-based systems still falter in fully automating this process. Although frameworks like OpenHands [ 24 ] and SWE-Agent [ 14 ] demonstrate strong general capabilities, they often stumble on real-world codebases. In practice, simply following README instructions seldom works: READMEs can be vague, incomplete, or even erroneous, and repositories are not guaranteed to match a task’s requirements out of the box—commands may need parameter changes, and key files can be misplaced. Consequently, when agents fail to locate or execute the necessary code, they must adapt by modifying existing components or generating new code to bridge the gap. To achieve it, agents need to understand the repository in a task-driven way. However, GitHub repositories often have two key properties that make this hard: (1) intricate structural complexity, with many interconnected files, classes, and functions, and (2) information density that exceeds the context limits of most LLMs. Existing frameworks [ 14 , 15 , 24 , 25 ] do not provide mechanisms for grasping repository structures, tracking detailed dependencies, or strategically managing information within these constraints, ultimately resulting in suboptimal performance and higher token cost. In this paper, we introduce RepoMaster, an end-to-end agent framework designed for automating the use of code repositories to tackle complex tasks. To address these challenges, RepoMaster draws inspiration from human programmers, who rarely read every line of code or error log when exploring unfamiliar codebases. Instead, they first map a project’s structure, start viewing a key file, then jump to its relevant files based on signals like error traces, and filter out irrelevant details. Following this intuition, RepoMaster first performs hierarchical structure analysis, builds dependency and call graphs, and identifies core components as the initial context. Navigated by these connections, it progressively explores the repository and applies information selection when viewing files and execution feedback to keep each interaction concise. By iteratively applying these steps, RepoMaster mimics human prioritization and makes efficient use of limited context windows. When evaluated on both MLE-R—a revised version of MLE-Bench-Lite [ 16 ]—and our newly constructed GitTaskBench [ 26 ], RepoMaster achieves significantly higher completion and success rates than OpenHands and SWE-Agent, while using far fewer tokens. Our contributions are summarized as follows: 

(1) We propose a novel automated framework, RepoMaster , that can effectively leverage code repositories to solve the complex real-world tasks end-to-end. (2) To efficiently comprehend code in a goal-oriented, human-like manner, we integrate hybrid structural hierarchy modeling with core component identification, context-aware code exploration, and efficient information selection. (3) 

We validate RepoMaster’s effectiveness and efficiency against Openhands and SWE-agent through experiments on diverse complex tasks from the MLE-R and GitTaskBench. 

2 Related Work 

2.1 Code Generation 

LLMs have made substantial progress in code generation [ 12 , 13 , 27 , 28 ], exemplified by closed-source models [ 29 , 30 , 31 ] and the open-source series [ 32 , 33 , 34 , 35 ]. Beyond basic code com-pletion [ 36 ], modern LLMs now support advanced tasks such as semantic code editing [ 23 , 37 ], debugging [ 38 ], and generating machine learning pipelines (e.g., AIDE [ 25 ] and MLAB [ 15 ] for 2(1) Repository Search 

> “I want to remove scratches
> from this old image.”
> # user’s intent
> # key entities

# … Search and Select 

> README file
> Star number
> Extract

# …

## (2) Hierarchical Repository Analysis 

…

……  

> Hierarchical
> Code Tree
> (HCT )
> Module
> Dependency
> Graph ( MDG )
> Function Call
> Graph ( FCG )RepoMaster
> Explore the Repo

Analyse Feedback & Optimize  

> “I want to remove scratches
> from this old image.” Granular Code View
> Dependency Analysis
> Search
> E
> x
> e
> c
> u
> t
> i
> o
> n
> E
> x
> p
> l
> o
> r
> a
> t
> i
> o
> n

## (3) Autonomous Exploration & Execution 

…

…

Agent  

> produce new action

Actions → Observations 

> RepoMaster
> Initial
> components
> Exploration tools
> Iteratively optimize

…

> Expand viewed scope
> view
> Core components (file, class)

…

> preparing
> Search
> Trace

Figure 1: Overview of RepoMaster, consisting of Repository Search, Hierarchical Repository Analysis and Autonomous Exploration & Execution. Kaggle competitions). However, fully automating the creation of complex real-world codebases from scratch remains a critical challenge for AI agents [16, 19, 22]. 

2.2 LLM-based Agents for Tool Use 

External tools are essential for extending the capabilities of LLM agents [ 5, 39 , 40 ]. Relying on executable code [ 6, 9]—using scripts to import inherent libraries, or call APIs, functionalized tools—has become a mainstream paradigm. Current works mainly focus on “tool learning” [ 1, 6, 10 ], but the more essential aspect of where to find the right tools is relatively overlooked [ 41 ]. Benchmarks, such as API-Bank [ 42 ] and ToolEyes [ 43 ], synthesize function libraries but are not realistic or practical; platforms such as RapidAPI [ 44 ] host real services but are closed-source and hard to extend. Standards such as FastAPI [ 45 ] or MCP [ 46 ], which unify interfaces for tool use via function calling mechanisms, have emerged. However, GitHub—a rich and dynamic ecosystem for automatically creating tools—remains underutilized in this context. Although GitAgent [ 17 ] first explored GitHub repositories as a tool extension, it is limited by simplistic repository search and understanding, and lacks validation in diverse real-world scenarios. 

2.3 Repository Utilization 

Using GitHub repositories to solve complex real-world tasks presents significant challenges. Re-poAgent [ 47 ] produces high-level documentation but fails to include realistic, task-oriented usage examples. ML-Bench-A [ 18 ] focuses on setting up the environment rather than understanding the repository. OpenHands [ 24 ] and SWE-Agent [ 14 ] are strong general agents that use step-by-step prompting to break down tasks and write code, but they lack methods to deeply understand the repository structure or build a clear hierarchy of its components. Aider [ 48 ] can track file dependen-cies but misses detailed function-level connections and cannot autonomously explore the codebase. Interactive assistants like Copilot [ 49 ] and Cursor [ 50 ] are effective for small-to-medium projects but struggle in large-scale repository contexts due to limited dependency awareness. 

3 Method 

Most current frameworks follow the CodeAct paradigm [ 9 , 14 , 21 , 24 ], offering basic file-editing and exploration commands (e.g., OpenHands’ AgentSkills [ 24 ] and SWE-Agent’s command set 3[ 14 ]). But relying on README-based mappings and simple find/edit operations misses many core components and cannot perform deeper, autonomous exploration within limited LLM contexts. In contrast, RepoMaster mimics human programmers by performing a static, structure-aware analysis to locate critical components, then dynamically selecting only the essential snippets—skipping irrelevant information and focusing the LLM’s limited context on what matters. The full end-to-end RepoMaster framework consists of three stages: (1) Repository Search: Identifying repositories relevant to the task. (2) Hierarchical Repository Analysis: Preparing the structures for exploration. (3) Autonomous Exploration & Execution: Iteratively interact with the repository and adjust exploration actions based on execution feedback. An overview of the framework is provided in Figure 1. 

3.1 Repository Search 

To address complex online tasks expressed in natural language, we develop a deep-search method to locate the GitHub repositories most relevant to the task. We begin by analyzing the user’s intent and extracting key entities to target the suitable repositories. We examine their README file and star count to assess their relevance and potential, and provide a brief description. Then, we select them by content quality and practical utility. Finally, we validate the top three candidates and deliver the results as structured JSON. An example of the deep-searching log is shown in Appendix B. 

3.2 Hierarchical Repository Analysis 3.2.1 Hybrid Structural Repository Mapping 

An essential prerequisite for task - oriented repository automation is a comprehensive structural model of the codebase. We sanitize the repository by removing all non - source files, retaining only executable .py files. For each retained file, we perform a single Abstract Syntax Tree (AST) walk [ 51 ] to recursively harvest both the meta-information and the raw source snippet of every module, class, and function. These atomic units provide the basis for understanding the repository’s structure. Let the target repository be denoted R = ⟨M, C, F, I⟩ , where M = {m1, . . . , m |M |} is the set of modules (one per .py file), C = {c1, . . . , c |C|} the set of classes, F = {f1, . . . , f |F |} the set of functions/methods, and I ⊆ M × M the explicit import relations captured from source files. On this foundation, we construct three complementary artefacts: • Hierarchical Code Tree (HCT). T , a nested package → module → class → function con-tainment map annotated with line counts and docstring snippets. • Function Call Graph (FCG). Gf = ( Vf = F, E f , w f ), where an edge (fi, f j ) ∈ Ef

exists if fi invokes fj ; the weight wf encodes call frequency. • Module Dependency Graph (MDG). Gm = ( Vm = M, E m, w m), in which (mi, m j ) ∈

Em if mi explicitly depends on mj ; wm measures coupling strength. We thus obtain the tuple M, C, F, I, G f , G m, T , providing the agent with a deterministic, loss-minimal structural synopsis of the entire repository before any task-specific exploration. 

3.2.2 Core Component Identification 

Having obtained a fine-grained yet verbose structural synopsis of the repository, we now need to compress this information into a concise context that preserves only the most influential code entities– small enough for multiple interaction turns within the LLM’s window, yet rich enough to preserve global semantics. To this end, we specify an importance scoring scheme that operates first at the module level and then propagates to classes. 

Module-level scoring. Each module m ∈ M receives a score I(m) ∈ [0 , 10] by linearly aggregating six orthogonal features, 

s = Dependency , Complexity , Usage , Semantic , Doc , Git , (1) 

I(m) = min 

P6 

> i=1

wi si(m), 10 



, wi ≡ 1, (2) where Dependency captures centrality in MDG using the personalized PageRank [ 52 ] algorithm, 

Complexity approximates cyclomatic complexity, Usage measures import and call frequency, Semantic 

flags high-value keywords (e.g., main , core ), Doc quantifies docstring richness, and Git reflects commit volume and recency. Detailed formulas for each feature are deferred to Appendix F. 4ASTs 

Codes  Documents  Logs 

Initial Context 

> README.md
> Brief summary of this
> file is …dataset setting…
> Module Summary
> Code Snippets

…

> Module Paths

…

> “explore”
> query
> Match

Segement  Match  

> 14

… Traceback  

> File “/x.py ”，
> line xxx…
> Exceptioin

To view efficiently: 

Interactive Feedback -based Execution 

Search Key Entities 

Trace Dependency 

…

5.  Exploration Tool Calling 

1.  2. 

6. 

(2.) 

…

7. 

(3.) 

Context -aware Code  Exploratio n Information Selection 

4.  Add into Context 

3.             

> Remove this image’s scratches Microsoft/Bringing -old -photos -back -to -life
> Original Image Output Image
> understand the repo and
> plan execution steps… search the kyewords , view the
> README.md, download xx.pth
> generate process_image.py and write…
> set up environment variables,
> install missing dependencies…
> Model checkpoint is not found. Output files :[] No such file or directory
> check “pth exists ”,edit & rerun process_image.py…
> view directory structures…
> It looks like the path may be
> wrong. Let me check…
> Initial Core components Input/Output path

Prompt ：

> Code output: Comparison image saved
> as 'output.png'
> Checking output files:
> output_result /
> final_output /
> DeScratch_01_input.jpeg
> ... ...
> stage_1_restore_output/
> origin/
> DeScratch_01_input.jpeg
> masks/
> mask/
> DeScratch_01_input.png
> input/
> DeScratch_01_input.jpeg

# …

…

> Rankt
> het

Figure 2: Overview of RepoMaster’s autonomous exploration–execution loop and an example demonstration. The agent begins by analyzing the initial context (Step 1) and specifies a file to inspect (Step 2). For efficient viewing, it extracts only the key information from that file (Step 3) and appends it to the context (Step 4). In the next exploration–execution iteration (Step 6 →2, Step 7 →3), the agent uses exploration tools to identify additional relevant files and repeats context-aware code exploration. Once it has gathered enough information, RepoMaster alternates between writing and running “ .py ” scripts, handling errors, and debugging based on feedback until the task is completed. 

Class-level refinement. Module scores serve as priors for class importance. For every class c located in module μ(c), we compute 

J(c) = I�μ(c) + |Fc|

max c′ |Fc′ | + Calls( Fc)max c′ Calls( Fc′ ) , (3) where Fc denotes the method set of class c. The second term rewards class richness in functionality; the third term captures how often its methods are actually invoked in the repository. Classes are ranked according to J(c), and the top-k classes are selected as the repository’s core components .

3.2.3 Repository Context Initialization 

Building on the identified core components, we construct an initial repository context in four distinct blocks. First, we include the complete README.md file, which provides high-level descriptions and detailed usage guidance authored by human developers. Second, we append a series of concise natural-language summaries for the highest-priority modules, giving the LLM a brief overview of each critical script’s purpose. Third, we provide the source code of core components (i.e., the classes scored and selected in Section 3.2.2) as fine-grained semantic anchors. Finally, for all other top-ranked modules, we provide a flat, directory-grouped list of their file paths for easy on-demand lookup. Figure 2 illustrates the initial context, and Appendix D provides a complete example of this initial repository context construction. This structured context serves as the agent’s "launchpad" for dynamic exploration, allowing it to prioritize high -impact modules, trace dependencies, formulate targeted code queries and select relevant classes or functions, bridging static analysis with dynamic reasoning and task execution. 

3.3 Autonomous Exploration & Execution 3.3.1 Context-aware Code Exploration 

Once the agent has internalized the repository’s functionality and overall structure, it immediately transitions to dynamic analysis, performing an autonomous, hierarchical and graph - based traversal 5of the codebase. To support in -depth comprehension and effective utilization of the repository, we offer a suite of fine - grained exploration tools organized into three categories: Granular Code View ,

Dependency Analysis , and Search .• Granular Code View. This tool enables the agent to inspect the implementation details of files, classes, and functions using the HCT. It also retrieves and exposes the repository’s directory hierarchy, facilitating swift orientation within the codebase. • Dependency Analysis. This tool traces call chains and dependency paths by analyzing the FCG and MDG, respectively. It uncovers complex invocation and dependency relationships among code entities, thereby deepening the agent’s comprehension of module interactions and overall code structure. • Search. This tool equips the agent with robust search capabilities, facilitating rapid location of specific code segments within large and intricate codebases. It employs keyword matching to ensure efficient retrieval of relevant entities. Together, these tools empower AI agents to proactively and autonomously navigate and examine code repositories, achieving a level of comprehension and flexibility comparable to human developers. Empirically, we observe that complex repositories typically require detailed dependency analysis using FCG and MDG, whereas simpler repositories often allow agents to effectively rely on HCT. 

3.3.2 Interactive Feedback-based Execution 

Task execution is grounded in the agent’s evolving understanding of the repository. Once the agent has identified the hybrid structural elements described in Section 3.2.1 and core components described in Section 3.2.2 relevant to a given task, it begins to perform task-oriented operations. Crucially, execution and exploration form a continuous, interleaved loop rather than a linear sequence. The agent can fluidly alternate between writing code and locating files, viewing content and reading logs, or tracing dependencies, all driven by the task context across different interaction turns, and powered by the exploration tools described in Section 3.3.1. This flexible loop allows the agent to iteratively refine its behavior by retrieving just-in-time information from the codebase. Figure 2 illustrates the execution and exploration pipeline of RepoMaster. 

3.3.3 Context-aware Information Selection For Efficient Viewing 

The agent must juggle source code, documentation, execution results and logs within a tight LLM token window for multiple turns, making it difficult to maintain a globally coherent view of the repository and severely limiting its applicability to large projects. To mitigate this issue, we propose a multi-level content reduction strategy that retains only the most critical information. 

Viewing Code. At the code level, the agent parses source files into Abstract Syntax Trees (ASTs), extracts semantically and structurally meaningful subtrees, and uses these extracted subtrees as inputs. 

Viewing Documents. For large or unstructured artifacts (e.g., .txt or .csv files), the agent divides each file into fixed -length chunks of Lc tokens, It then generates retrieval prompts tailored to the current subtask, ranks the chunks by relevance, and retains top nc most relevant segments. 

Viewing Feedback Logs. At the log level, we apply a human - like debugging heuristic that retains only the opening and closing segments of the log (where command invocations, exception traces, and diagnostic results cluster) and discards verbose intermediate output. Multi -level reduction strategies activate only when the combined size of all candidate inputs exceeds the per -interaction token limit L, preserving global coherence by focusing on high -impact information and ensuring each execution-loop step relies on a compact, relevant context. 

4 Experiments 

4.1 Benchmarks and Metrics 

To validate the effectiveness of RepoMaster, we evaluate it using two benchmarks. 6MLE-R. The original MLE-Bench [ 16 ] derives from Kaggle competitions, designed to evaluate LLM agents’ capabilities in end-to-end machine learning engineering tasks. To construct MLE-R, we select 22 MLE-Bench tasks (covering nearly all MLE-Bench-lite cases) and apply the search procedure described in Section 3.1 to retrieve suitable GitHub repositories for each task, ensuring a fair comparison 1; the tasks’ requirements are set to be completed based on their chosen repository rather than generating code from scratch. Performance in MLE-R is evaluated using a medal-based system, the same as the original MLE-Bench, where solutions are assessed based on official Kaggle thresholds 2 for gold, silver, and bronze medals. Metrics include the achieved score, medal thresholds, and medal qualification, providing a clear indication of the model’s proficiency in competitive ML engineering tasks. 

GitTaskBench. In contrast to MLE-R, which emphasizes standard machine learning tasks (e.g., image classification), our new proposed GitTaskBench [ 26 ] 3 benchmark evaluates LLM agents on more practical real-world problems–common tasks whose complexity or format largely demands leveraging existing repositories, such as photo restoration. The benchmark consists of 18 repositories and 54 tasks, all described in natural language and designed to be completed using the provided repositories across a wide range of domains, such as image processing, video analysis, speech, physiological signals, office automation, and security and privacy. GitTaskBench evaluates two key aspects: Execution Completion Rate (measuring the model’s ability to leverage the repository for output) and Task Pass Rate (assessing whether the output meets task-specific evaluation criteria). Given the diversity of tasks, evaluation metrics are predefined and tailored within the benchmark, ensuring a comprehensive assessment. Note that total tokens include both input and output tokens. 

4.2 Evaluation Setup 

We evaluate our approach against two baseline frameworks and compare the performance across three state-of-the-art LLMs. The evaluation setup is as detailed below. 

Baseline Frameworks. 

We evaluate two baseline frameworks: OpenHands [ 24 ] and SWE-agent [ 14 ]. OpenHands provides sandboxed environments for code execution and API interactions, while SWE-agent focuses on autonomous GitHub issue resolution. 

Large Language Models. We evaluate multiple leading LLMs, including the closed-source GPT-4o-2024-08-06 [ 53 ] and Claude-3-5-sonnet-20241022 [ 54 ], as well as the open-source DeepSeek V3-0324 [ 55 ]. This setup enables a comprehensive assessment of both agent architectures and LLM capabilities on solving real-world tasks with repository utilization. 

Implementation Details. Our proposed solution RepoMaster is built on a multi-agent dialog platform AutoGen [ 21 ]. To ensure agent performance, we set a few key hyperparameters. Specifically, we set the maximum token length per interaction L to 8000 tokens. For initial context construction, we generate concise summaries for the top 20 modules by importance score and extract k = 10 key classes. During the feedback phase, unstructured text files are split into chunks of Lc = 1000 tokens, retaining the nc = 4 most relevant segments. 

4.3 Comparison with SOTA 

On the MLE-R benchmark, RepoMaster with Claude 3.5 attains a 95.45% valid submission rate and a 27.27% medal acquisition rate (including 22.73% gold medals), representing a more than five-fold improvement over the best open-source Agent baseline. RepoMaster with GPT-4o also achieves a strong 86.36% valid submission rate and 18.18% medal rate, further confirming its robust performance advantage under varied settings. RepoMaster’s significant performance improvement stems primarily from its effective identification and utilization of core components within open-source repositories, such as neural network architec-

> 1Ensure a fair comparison, as other general agent frameworks do not support automatic repository retrieval.
> 2The specific thresholds for gold, silver, and bronze medals are provided in Appendix G.
> 3More detailed descriptions can be found in Appendix A.

7Table 1: Performance comparison of different frameworks and LLMs on MLE-R. The best perfor-mance is bolded, and the second-best is underlined.; the same is below.                                                                              

> Framework LLM Made Valid Above Bronze Silver Gold Any Submission (%) Submission (%) Median (%) (%) (%) (%) Medal (%)
> SWE-Agent GPT-4o 72.73 54.55 0.00 0.00 0.00 0.00 0.00 Claude 3.5 54.55 50.00 13.64 0.00 0.00 4.55 4.55 DeepSeek V3 54.55 36.36 4.55 0.00 0.00 4.55 4.55 OpenHands GPT-4o 50.00 45.45 0.00 0.00 0.00 0.00 0.00 Claude 3.5 50.00 45.45 9.09 0.00 0.00 4.55 4.55 DeepSeek V3 63.64 36.36 0.00 0.00 0.00 0.00 0.00 RepoMaster GPT-4o 86.36 86.36 36.36 4.55 0.00 13.64 18.18 Claude 3.5 95.45 95.45 45.45 4.55 0.00 22.73 27.27
> DeepSeek V3 95.45 86.36 36.36 4.55 4.55 13.64 22.73

ture designs, optimized hyperparameter configurations, and data preprocessing pipelines. In contrast, baseline methods like OpenHands and SWE-Agent often struggle to pinpoint critical modules during repository exploration, filling limited context windows with excessive irrelevant code, resulting in insufficient understanding of model architectures and training logic. In the GitTaskBench evaluation, RepoMaster significantly outperforms existing open-source frame-works SWE-Agent and OpenHands. Based on Claude 3.5, RepoMaster achieves a 75.92% execution completion rate and 62.96% task pass rate, surpassing OpenHands (48.15%, 24.07%) and SWE-Agent (44.44%, 14.81%). Similarly, RepoMaster maintains significant advantages on GPT-4o and DeepSeek V3, demonstrating that RepoMaster’s inherent capabilities have good universality across underlying models. More importantly, RepoMaster substantially reduces computational overhead, with token consumption when using Claude 3.5 approximately 95% lower than OpenHands (150k vs 3000k tokens/task), proving the effectiveness of our hybrid hierarchical structure analysis and information pruning strategies. Table 2: Performance comparison of different frameworks and LLMs on GitTaskBench.                                   

> Framework LLM Execution Completion Rate (%) ↑Task Pass Rate (%) ↑#Total Tokens ↓
> SWE-Agent GPT-4o 29.63 9.26 308k Claude 3.5 44.44 14.81 330k DeepSeek V3 29.63 9.26 265k OpenHands GPT-4o 37.04 14.81 1195k Claude 3.5 48.15 24.07 3094k DeepSeek V3 42.59 16.67 7662k RepoMaster GPT-4o 48.14 40.74 250k Claude 3.5 75.92 62.96 154k
> DeepSeek V3 61.11 44.44 255k

4.4 Insightful analysis Ablation Study To quantitatively assess the contribution of each component in RepoMaster, we conduct a comprehensive ablation study on the GitTaskBench benchmark using GPT-4o as the underlying model. By systematically removing key mechanisms, we measure their impact on three metrics of effectiveness and efficiency: execution completion rate, task pass rate, and token usage. The results are shown in Table 3. 

Hybrid Hierarchical Analysis : Removing this component causes slight decreases in execution completion and task pass rates, with other components partially compensating. Token usage increases by 9.67%, showing that without structural repository understanding, more exploration is required. 

Context-aware Code Exploration : This component provides a suite of exploration tools enabling agents to autonomously navigate codebases according to task requirements, inspect file contents, and trace call chains and dependencies. The experiments prove this component to be particularly critical, with its removal causing significant decreases in execution and pass rates. More notably, token usage increases substantially, validating our hypothesis that strategic exploration effectively reduces repository exploration needs, allowing the agent to precisely target task-relevant components. 8Table 3: Ablation study on the impact of core mechanisms in RepoMaster. The "w/o All" refers to RepoMaster without any of the three mechanisms. “Exec. Comp.” stands for “Execution Completion.” 

Configuration Exec. Comp. Rate (%) ↑ Task Pass Rate (%) ↑ #Total Tokens ↓

RepoMaster (Full) 48.14 40.74 250k - w/o Hybrid Hierarchical Analysis 46.29 (-1.85%) 38.88 (-1.86%) 274k - w/o Code Exploration 44.44 (-3.7%) 35.18 (-5.56%) 284k - w/o Information Selection 46.29 (-1.85%) 38.88 (-1.86%) 265k - w/o All 29.62 (-18.52%) 24.07 (-16.67%) 112k Openhands RepoMaster “Estimate the 3D human pose from the video’s 2D 

keypoints  using the provided  VideoPose3D  repository ” SWE -Agent 

22 modules loaded. 10 classes loaded. 83 functions loaded.  Based on the README .md,  I understand the 

requirements… Let’s  set  up  the  environments. 

Proceeding step by step… 

PyTorch  GPU installation failed. 

Installed CPU version successfully, 

unnecessary  uninstall  and reinstall 

steps  were still executed .

…Analyzing, 

Building… 

First, let’s make sure our environment is set up correctly and that we have all the required data. 

Next, we’ll verify that the necessary files are present. 

** *** Suggested tool call:  list_repository_ dependenc y *** ** 

Arguments: {"path": “/VideoPose3D"} 

All required files are present. Inspecting the contents of the input data file…… 

** *** Suggested tool call:  search &  view _file_content  *** ** 

Arguments: {" file_path ": “./VideoPose3D_01_input.npz"} 

Error: The input . npz  file does not contain a key named 'positions_2d'. 

Task successfully completed. 

code analysis done  failed 

Modifying `load_2d_data()` to print the . npz  file contents and identify the correct key 

for loading 2D  keypoint  data… 

≈ 17 turns  ≈ 140 turns  failed  ≈ 20 turns 

Retrying  3D pose estimation…check the input data. 

Keys in the  npz  file: 

['positions_2d', 'metadata’], both 

are 0 -D arrays (scalars). 

NPZ file contents:['boxes', 'segments',  'keypoints ', 'metadata'] 

Original  keypoints  shape: (100, 2) 

Processed  keypoints  shape: (100, 1, 2 )

Modifying `load_2d_data()` to print the . npz  file contents and identify the correct key for 

loading 2D  keypoint  data, iterate through data structure analysis and type conversion… …

Success:  extract 2D coordinates using frame[1][0, :, :2 ]

[Errno  2] No such file or directory: ‘./checkpoint/ pretrained.bin '

Checking if the 'checkpoint' directory exists…Directory not found. Creating directory… 

Attempting to download the model… 

## … … …

** ***  vie w_repository_structure  & search*** ** 

Find the  url  of  the  checkpoint in the README.md… 

Goal -unoriented 

code modifications 

Limited repository 

understanding 

Now, let's first read the README.md file from 

the VideoPose3D repository to understand how 

to use it. We need to 1…2…3... 

Creating `process_and_predict.py` script… 

Attempting to use ` TemporalModel `… 

Skip  package installation. 

Switch to a NumPy -only implementation: 

Removed all  PyTorch  `TemporalModel ` calls 

ModuleNotFoundError : No module 

named 'torch' 

Traceback (most recent call last): 

File  “./process_and_predict.py" ,

line  24 , in <module> 

poses_3d[:, : 2] =  keypoints 

~~~~~~~~^^^^^^^ 

ValueError : setting an array 

element with a sequence. 

## … Let's check its structure. 

Edit the file again to handle 

this data structure correctly .

3D poses saved to / output.npz 

Shape of output: ( 100 , 3, 3)

## …

Actual 2D coordinates: conflict 

with `run. py `’s fetch logic 

Print `data['positions_2d'].item()` and 

`data['metadata'].item()`. 

Read the run.py, locate the ‘fetch’ function,  Re -

edit  the function in run.py extract pose data 

directly from  `keypoints [' keypoints ']['custom’]`, 

instead of ` keypoints [subject]` 

KeyError  resolved. 

Direct patch applied to fetch() to 

accommodate new data format 

Highly exploratory  Highly adaptable 

Figure 3: RepoMaster vs. OpenHands and SWE-Agent: A Comparative Case Study. 

Information Selection : This component applies multi-level content reduction strategies to extract key information from source code, documents, and execution logs, preventing the context window from being filled with irrelevant content. Removing it decreases performance similarly to removing hybrid hierarchical analysis, with token usage increasing by only 6.00%. Its main value is maintaining a high signal-to-noise ratio rather than reducing token consumption. The most revealing comparison is between the full RepoMaster system and the base code agent without any of our proposed components. The baseline achieves only 29.62% execution completion and 24.07% task pass rates—decreases of 18.52% and 16.67%. Interestingly, the baseline’s token usage is significantly lower, but this reflects a failure case rather than efficiency: the agent simply gives up earlier without the necessary tools to effectively explore and utilize the repository. Further analysis of the failure modes in ablated systems reveals: Without hybrid hierarchical analysis, the agent struggles to locate key repository components, often getting lost in non-essential files; without context-aware exploration, the agent frequently explores irrelevant parts of the repository, resulting in context fragmentation and redundant exploration; without information selection, the agent’s context window becomes cluttered with low-value information, causing it to miss important details in error messages and execution traces. 

4.5 Case Study 

For the case study, we evaluated RepoMaster against OpenHands and SWE-Agent on a challenging 3D pose estimation task from GitTaskBench. As shown in Figure 3, neither baseline completed the task due to different failure modes. OpenHands ran extensive trial-and-error iterations ( ∼140 attempts, >10 × others) and consumed higher tokens without success. SWE-Agent, although quicker, lacked task-level repository understanding—treating each error as a standalone fix and defaulting 9to a coarse 3D pose method that strayed from the core algorithm, causing task degradation. In contrast, RepoMaster leveraged structured repository analysis to efficiently focus on key components, achieving successful task completion with fewer attempts ( <20 iterations). 

5 Conclusion 

We introduce RepoMaster , an end-to-end autonomous agent framework designed for automating the use of code repositories to tackle complex tasks. By combining static structural analysis of the repository with autonomous exploration, RepoMaster outperforms OpenHands and SWE-Agent in two challenging benchmarks. These results demonstrate that treating open-source repositories as modular, composable tools —rather than burdens to be regenerated from scratch—forms a powerful paradigm for solving complex real-world tasks. Beyond performance gains, RepoMaster promotes a more sustainable and collaborative AI-for-code ecosystem. Its capacity to reuse and adapt existing repositories lays the groundwork for large-scale orchestration of multiple projects within a single workflow, automated propagation of bug fixes and security patches upstream, and straightforward transfer to domains that share analogous structural challenges, such as hardware description languages, robotic middleware, or data-centric notebook collections. By enabling agents to understand and integrate code in context, RepoMaster accelerates the virtuous cycle between human contributors and AI systems, fostering continual improvement across the open-source landscape. 10 References 
