# AI Agent-based Error Resolution System for SoC RTL Verification

**Authors:** Yonghyun Kwon, Hanna Jang, Seonghee Yim

## Abstract

SoC RTL verification workflows require considerable time due to manual analysis and resolution processes when errors occur. This paper proposes an AI agent-based system that automatically executes solutions for verification errors. The proposed system consists of two specialized agents: Solution Execution Agent and System Monitoring & Debug Agent, achieving both efficiency and safety through executability evaluation, safety verification, and human-in-the-loop approval mechanisms.

The system is designed based on 150,000 verification tasks collected from previous projects and 86 verified error-solution pairs. The executability determination logic comprehensively considers technical feasibility, safety, and business rules to decide whether each solution can be automatically executed. Error-solution pairs are classified into three groups according to execution complexity: simple automation possible (38 cases), parameter adjustment required (31 cases), and complex analysis required (17 cases).

Expected performance targets 70-80% of all tasks to be processed automatically or semi-automatically, with an average processing time reduction of 60-70%. When expanded to the entire organization, savings of 4,000-6,000 engineer-hours per month and 48,000-72,000 engineer-hours annually are anticipated. This research presents a practical and scalable solution that frees verification engineers from repetitive tasks and enables them to focus on more valuable work, establishing a safe deployment strategy through a five-phase verification methodology.

## I. INTRODUCTION

Modern SoC RTL verification requires increasing time and resources as designs become more complex and verification scope expands. During regression testing, numerous errors occur, and the process of identifying error causes, finding appropriate solutions, and applying them relies primarily on manual engineer effort. Engineers go through repetitive processes of checking error logs, referring to past experience or documentation to find solutions, applying solutions, and re-executing for verification.

This manual process causes several problems. First, it is time-consuming. Even simple environment configuration issues require an average of 20-30 minutes for log analysis, cause identification, and solution search, while complex problems can take hours to days. Second, there is inefficiency in repetitive work. Despite identical or similar errors occurring repeatedly, the same analysis and resolution process must be performed each time. Third, knowledge is fragmented. Error resolution methods depend on individual engineer experience or scattered documentation, making systematic knowledge management at the organizational level difficult. Fourth, nightly regression test delays occur. Errors that occur overnight remain unaddressed until the next morning, unnecessarily lengthening verification cycles.

Recent advances in AI technology, particularly Large Language Models, present new possibilities for solving these problems. LLMs demonstrate excellent capabilities in natural language understanding, pattern recognition, and contextual analysis, and are being utilized in automation systems. However, considering the complexity and safety requirements of verification environments, a systematic and safe approach is needed rather than simple AI application.

This paper proposes a practical AI agent-based system that automatically executes solutions for errors occurring in verification environments. The core ideas of the system are: (1) systematic management of verified error-solution pairs, (2) safe automatic execution mechanisms, (3) risk management through human-in-the-loop, and (4) continuous improvement through learning.

### Main Contributions

- Design of a safe and scalable framework for automatic error solution execution
- Development of executability evaluation and safety verification mechanisms
- Establishment of design and verification methodology based on 150,000 tasks from actual SoC verification projects
- Presentation of a practical automation approach through human-in-the-loop

The paper is organized as follows. Section 2 reviews related work, and Section 3 presents the system architecture. Section 4 describes implementation details, and Section 5 presents expected results and verification methodology. Section 6 discusses limitations and future research directions, and Section 7 concludes.

## II. RELATED WORK

### 2.1 Automated Software Error Correction

Research on automatic bug fixing has been actively conducted in the software development domain. AutoCodeRover is a system that automatically analyzes GitHub issues and proposes code modifications, using a three-stage pipeline of codebase search, bug localization, and patch generation. SWE-bench provides a benchmark for automatic fixing systems by constructing a dataset of real GitHub issues. While these studies leverage LLMs' code understanding capabilities, they primarily focus on source code-level bug fixing.

The key difference from our research is that verification environment errors stem from various causes such as environment configuration, tool issues, and resource shortage rather than source code bugs, and are resolved through environment adjustments or configuration changes rather than code modifications. Additionally, the safety and reproducibility of verification environments are critical, requiring a cautious approach.

### 2.2 Log-based Anomaly Detection

Research such as LogLLM utilizes LLMs to detect anomalies by analyzing system logs. While this approach is effective for pattern recognition and anomaly detection, it mainly stops at problem detection and does not address automatic resolution. Our research is differentiated in that it proposes an integrated system that goes beyond error detection to include solution execution.

### 2.3 Retrieval-Augmented Generation

RAG is a technique that augments LLM responses by retrieving external knowledge, and is effective when domain-specific knowledge is required. Our research applies the RAG concept to an error-solution database, establishing a system that retrieves verified solutions and executes them safely.

### 2.4 Hardware Verification Automation

Hardware verification automation research has primarily focused on test generation, coverage analysis, and formal verification. Research on error handling automation is limited, and most is confined to specific tools or environments. Our research is significant in that it presents a general-purpose solution execution framework applicable to various verification environments and tools.

## III. SYSTEM ARCHITECTURE

The proposed system consists of two specialized AI agents and operates based on a predefined error-solution database.

### 3.1 Overall Architecture

The system is designed to receive errors occurring in the verification environment as input, find appropriate solutions, automatically execute them, and monitor the results. The overall architecture consists of three core components.

The first is the **Error-Solution Database**, which stores and manages 86 verified error-solution pairs constructed from previous projects. Each pair includes error patterns, solution information, execution conditions, verification methods, and more.

The second is the **Solution Execution Agent**, which is responsible for searching for appropriate solutions from the database for errors that occur, evaluating executability, and executing them safely. This agent is the core of the system, with the goal of securing both automation efficiency and safety simultaneously.

The third is the **System Monitoring & Debug Agent**, which monitors the overall system performance, tracks execution results, and improves the system through continuous learning.

#### Workflow

The overall workflow proceeds as follows. When an error occurs in the verification environment, error information is collected, and the Solution Execution Agent receives it and searches for an appropriate solution in the database. Executability evaluation is performed on the retrieved solution, and based on the results, it is classified as automatic execution, execution after human review, or manual processing. Even when determined to be automatically executable, it is executed after obtaining human approval, and the execution process and results are tracked and recorded by the System Monitoring Agent. This information is reflected back into the database so that the system continuously improves.

### 3.2 Solution Execution Agent

The Solution Execution Agent is the core agent of the system, responsible for searching, evaluating, and executing error solutions. This agent consists of five main functions.

#### 1. Input Information Collection
It receives error information such as error messages, error types, occurrence locations, and timestamps from the verification environment. It also collects contextual information such as test environment, tool information, and system state. This information is essential for finding appropriate solutions and determining executability.

#### 2. Solution Retrieval
It analyzes the collected error information to find the most suitable solution from the Error-Solution Database. The search is performed on the 86 pre-constructed error-solution pairs and matches based on error pattern similarity. The search results provide detailed solution content along with metainformation such as past execution history and success rates.

#### 3. Executability Evaluation
It comprehensively evaluates whether the retrieved solution can be safely executed automatically in the current environment. The evaluation is conducted in three dimensions:

- **Technical Feasibility**: Verifies necessary system permissions, resources, and dependencies
- **Safety**: Calculates a risk score by comprehensively considering the scope of system changes, rollback difficulty, and past failure rates
- **Business Rules**: Determines the automatic execution allowance scope according to organizational policies

By synthesizing these three evaluations, it classifies into one of three categories: automatic execution possible, human review required, or manual processing required.

#### 4. Execution Strategy Determination
It establishes an appropriate execution strategy based on the evaluation results. When the risk is low and all conditions are satisfied, it recommends automatic execution; when the risk is medium level, it requests human review. When the risk is high or conditions are not met, it recommends manual processing.

#### 5. Solution Execution
Once human approval is complete, it actually executes the solution. Before execution, it creates a system state snapshot to enable rollback in case of problems. The execution process is logged in detail, and all changes occurring during execution are recorded. After execution, it automatically verifies success by comparing expected results with actual results. If verification fails or an exception occurs, it immediately restores to the original state using the snapshot.

### 3.3 System Monitoring & Debug Agent

The System Monitoring & Debug Agent is responsible for managing the entire system and supporting continuous improvement. This agent consists of three main functions.

#### 1. Performance Monitoring
It continuously tracks and evaluates the overall performance of the system. It measures solution execution success rates to understand the effectiveness of each solution and tracks processing time to quantify time-saving effects. It calculates automation rates to monitor the proportion of cases completely resolved without human intervention. These metrics are provided in real-time through a dashboard, allowing the overall health of the system to be understood at a glance.

#### 2. Debugging Support
For complex errors or cases where automatic resolution is not possible, it automatically collects and provides information to help engineers solve problems. It automatically finds and collects relevant log files, creates snapshots of the current environment, and identifies recent system changes. It also searches for similar cases that occurred in the past and their resolution methods to provide as reference materials. This information is systematically packaged and delivered to engineers, greatly reducing manual debugging time.

#### 3. Continuous Learning
It systematically analyzes all execution history to improve the system. It calculates the actual success rate of each solution and updates it in the database. Solutions whose success rate falls below a certain level are automatically deactivated and review is requested. Conversely, solutions showing high success rates have their priority elevated to be recommended more quickly. It also learns patterns where humans adjusted solution parameters, enabling more appropriate parameter suggestions in similar situations in the future. Risk scores and expected execution times are also continuously updated based on actual data.

### 3.4 Error-Solution Database

The Error-Solution Database is the core knowledge repository of the system. It systematically manages 86 verified error-solution pairs constructed from previous projects, and this system additionally extends execution-related information.

Each error-solution pair consists of multiple information categories:

#### Error Pattern Information
- Regular expression patterns for identifying errors
- Vector embeddings for semantic search
- Classification categories (environment configuration issues, tool-related issues, resource issues, design issues, script errors, and others)

#### Solution Information
- Actual commands or work content to be executed in parameterized form
- List of system permissions required for solution execution
- Amount and type of resources needed
- Expected execution time
- Risk scores based on past execution history
- Rollback feasibility and specific rollback methods

#### Applicability Condition Information
- Tool version requirements
- Operating system compatibility
- Project-specific special requirements

#### Verification Method Information
- What state should be checked
- What values are expected
- What the criteria for success are

#### Meta-information
- Creation time and last update time
- Total usage count
- Success rate based on the last 100 executions
- Average execution time

The database is constructed by combining two types of storage. Structured information is stored in a relational database to support accurate queries and transaction management. Vector embeddings for semantic search are stored in a vector database to efficiently perform similarity-based searches. These two storage systems are accessed through a unified interface, allowing users to use them like a single database.

## IV. IMPLEMENTATION DETAILS

This chapter conceptually describes the core implementation of the system.

### 4.1 Implementation Overview

This system utilizes error data from 150,000 verification tasks collected from previous projects. Error collection, parsing, classification, and solution matching have already been completed, and 86 verified error-solution pairs have been constructed in the database. Each pair includes clear error patterns and verified resolution methods.

This research focuses on the stage of actually executing these matched solutions safely. The system implementation consists of four core modules:

1. **Evaluation Engine**: Determines whether a matched solution can be safely executed in the current environment
2. **Approval Management System**: Efficiently obtains human approval for risky operations
3. **Execution Engine**: Safely executes approved solutions and verifies results
4. **Monitoring Mechanism**: Tracks execution results and monitors system performance

### 4.2 Executability Determination Logic

To determine whether a matched solution can be automatically executed, three evaluation criteria are applied.

#### 1. Technical Executability Evaluation
Verifies whether the system has the necessary permissions for solution execution, whether sufficient disk space and memory are available, and whether required tools or libraries are installed. If any of these are not met, it is determined that automatic execution is not possible.

#### 2. Safety Evaluation
Calculates a risk score by comprehensively considering:
- Scope of system changes (local or global change)
- Rollback difficulty
- Required permission level
- Past failure rate

Each element is weighted to calculate a final risk score, which is classified into three levels:
- **Safe**: Below 0.3
- **Medium**: 0.3 to below 0.6
- **High**: 0.6 or above

#### 3. Business Rule Verification
The scope of operations allowed for automatic execution is defined according to organizational policies:

**Generally Permitted Operations:**
- Environment variable settings
- Test re-execution
- Temporary file cleanup

**Requires Manual Processing:**
- Source code modifications
- System-wide configuration changes
- Security-related configuration changes
- Operations that incur costs

Only when all three criteria are passed is it determined to be automatically executable. Solutions that have low risk, are technically feasible, and comply with business rules are immediately recommended for automatic execution; when risk is medium, human review is requested; in other cases, manual processing is recommended.

### 4.3 Human-in-the-Loop Mechanism

To mitigate the risks of fully automatic execution, we implemented a human-in-the-loop approval process. Even solutions classified as automatic execution candidates receive approval from the responsible engineer before actual execution.

#### Approval Request Information
- Error information
- Detailed content of the solution to be executed
- Expected impact scope
- Expected execution time
- Risk factors
- Rollback methods
- Results of past similar cases

#### Approval Process
Approval requests are managed in a priority queue, with priority determined by error urgency and business impact. Immediate notifications are sent to responsible engineers via email, messaging systems, etc., and they can select approve, reject, or approve after parameter modification through a web interface. Each request has a timeout set, ranging from 1 hour to 24 hours depending on urgency.

#### Execution and Rollback
Once approval is complete, a system state snapshot is created before execution to enable rollback in case of problems. The execution process is logged in detail, and after execution, expected results are compared with actual results to automatically verify success. If verification fails or an exception occurs, the system is immediately restored to the original state using the snapshot.

### 4.4 Error-Solution Database Management

The database is the core knowledge repository of the system, managing 86 verified error-solution pairs constructed from previous projects. Each pair already includes error patterns and basic solution information, and this system additionally extends execution-related information.

#### Execution-related Extension Information
- Required permission lists
- Required resources (memory, disk, CPU)
- Expected execution time
- Risk scores
- Rollback feasibility and methods
- Pre-execution checklists
- Post-execution verification methods

#### Execution History Tracking
All execution attempts are recorded in detail, tracking:
- Execution time
- Duration
- Execution results
- System state before and after execution
- Human intervention status
- Causes of failure

Based on execution history, the success rate of each solution is recalculated and risk scores are updated. This allows the system to progressively improve and make more accurate decisions.

## V. EXPECTED RESULTS AND VALIDATION APPROACH

This chapter presents the expected performance of the proposed system and the validation methodology. The system design and preparation phase has been completed, and actual verification experiments are in progress. The final paper will include actual performance data and detailed analysis results obtained through experiments.

### 5.1 Error-Solution Database Classification

Experiments will be conducted on four major items of an ongoing SoC verification project. The system will be evaluated by analyzing a total of 150,000 verification tasks. This data collected from previous projects has already completed error analysis and solution matching, and 86 verified error-solution pairs have been constructed in the database.

#### Classification by Execution Complexity

**Group A - Simple Automation Possible (38 cases):**
- Environment variable settings
- License retries
- Path modifications
- Characteristics: No parameter changes required, easy rollback, low risk

**Group B - Parameter Adjustment Required (31 cases):**
- Memory allocation
- Timeout adjustment
- Server changes
- Characteristics: Require situation-specific parameter adjustments

**Group C - Complex Analysis Required (17 cases):**
- Configuration conflicts
- Complex problems
- Tool version issues
- Characteristics: High risk, require complex analysis

### 5.2 Expected Performance

#### Executability Determination Accuracy
- **Group A**: >95% accurately classified as automatically executable
- **Group B**: >85% classified as requiring human review
- **Group C**: >90% classified as requiring manual processing

#### Solution Execution Success Rates

**Group A:**
- Automatic execution: 93-96%
- With parameter adjustments: 97-99%

**Group B:**
- Initial automatic execution: 50-60%
- With human parameter adjustments: 85-90%

**Group C:**
- Manual resolution with system support: 70-80%

**Overall:**
- Automatic execution alone: 70-80%
- Including adjustments: 85-95%

#### Processing Time Reduction

**Group A:**
- Manual: 20-30 minutes → Automated: 3-5 minutes
- Time savings: 75-85%

**Group B:**
- Manual: 50-70 minutes → Automated: 15-25 minutes
- Time savings: 60-70%

**Group C:**
- Manual: 120-180 minutes → With support: 50-80 minutes
- Time savings: 50-60%

**Overall Average:**
- Manual: 60-90 minutes → Automated: 20-35 minutes
- Time savings: 60-70%

#### Automation Rate
Of the total 150,000 tasks:
- 80-85% expected to match the 86 patterns
- 55-60% immediately automatically executable
- Additional 30-35% executable after human review
- **Total: 70-80% processed automatically or semi-automatically**

### 5.3 Validation Methodology

System validation will proceed in five phases to ensure both safety and practicality.

#### Phase 1: System Design and Preparation (Completed)
- Executability evaluation algorithm development
- Human-in-the-loop approval process design
- Addition of execution-related information to 86 error-solution pairs
- Establishment of basic structure of execution engine and monitoring system

#### Phase 2: Simulation-based Evaluation
- Use error logs from past 150,000 tasks
- Predict system's judgment and classification results
- Measure expected success rates, processing time, and automation rates
- Identify potential risk factors

#### Phase 3: Limited Real Application
- Apply only the safest patterns of Group A to actual verification environment
- Human approval mandatory for all executions
- Intensive monitoring
- Track execution success rates, side effects, and time savings

#### Phase 4: Gradual Expansion
- Expand to all of Group A based on verified patterns
- Gradually expand to Group B
- Provide sufficient stabilization periods for each expansion stage
- Evaluate performance at each stage

#### Phase 5: Long-term Monitoring and Improvement
- Perform long-term monitoring after system stabilization
- Analyze execution history data
- Measure actual success rates and update database
- Evaluate possibility of expansion to other projects

## VI. DISCUSSION

This chapter discusses the limitations of the proposed system, considerations for practical deployment, and future research directions.

### 6.1 Limitations

#### Coverage Limitation
This system is based on 86 error-solution pairs constructed from previous projects. While this is expected to cover 80-85% of all occurring errors, the remaining 15-20% represent new types or cases without clear patterns. Considering the complexity and diversity of SoC verification environments, it is practically impossible to predefine all errors. Therefore, continuous expansion and updating of the database is essential.

#### Environment Change Adaptation
When tool version updates or verification environment changes occur, the validity of existing solutions may degrade. For example, if a simulator is updated, error message formats may change, and if environment variable paths change, existing solutions may not work. This is a fundamental limitation of automation systems, and the development of environment change detection and adaptation mechanisms will be an important future challenge.

#### Parameter Optimization
For Group B, basic solutions are clear but specific parameters vary depending on the situation. The current system proposes parameters based on past history, but may still suggest inappropriate values in new situations or exceptional cases. Automatically determining optimal parameters remains a difficult problem, and the introduction of advanced techniques such as reinforcement learning or Bayesian optimization may be necessary.

#### Complex Error Handling
Complex errors in Group C are almost impossible to resolve automatically with the current system. These errors involve multiple causes, complex contexts, or design flaws, and require root cause analysis and creative problem solving. The system can only provide limited support through presenting similar cases and collecting debugging information, reflecting the limitations of current AI technology.

### 6.2 Considerations for Practical Deployment

#### Organizational Culture and User Trust
The successful deployment of automation systems depends not only on technical performance but also significantly on organizational culture and user trust. Important considerations:
- Clearly educate about how the system works and its limitations
- Continuously share success cases and statistics
- Provide transparent decision-making processes
- Position the system as an intelligent assistance tool rather than perfect automation

#### Responsibility and Accountability
When automatically executed solutions cause unexpected side effects, responsibility becomes an important issue:
- Through human-in-the-loop, final responsibility should rest with the approving engineer
- Clear policies and guidelines are needed
- All execution history must be recorded in detail to enable audit trails
- System must be established to analyze causes when problems occur

#### Security Concerns
Automatic execution systems can make extensive changes to the verification environment:
- Strict permission management essential to prevent system damage
- Database modifications, approval of risky solutions, and system configuration changes should only be possible for users with restricted permissions
- Security audits and penetration testing necessary for actual deployment

#### Cost-Benefit Analysis
System construction and operation incur costs:
- Infrastructure costs
- Maintenance personnel
- Initial database construction costs
- Recovery of initial construction costs expected to take some time
- Long-term effectiveness will increase as database accumulates
- Clear ROI analysis along with phased investment plan needed

#### Scalability Challenges
When expanding from current 4 items to entire organization:
- 10-15 fold increase in database size can affect search and matching performance
- Database structure optimization and indexing strategies important
- Need separation of common patterns and project-specific patterns or hierarchical database structure
- Human approval burden can become bottleneck
- Strategies needed: approval authority delegation, expansion of automatic approval scope, priority-based queue management

### 6.3 Future Research Directions

#### Root Cause Analysis
The current system matches error symptoms with existing solutions. Future functionality needed:
- Automatically analyze root causes of errors
- Determine whether memory shortage is due to test requirements, memory leak, or unnecessary parallel execution
- Propose more fundamental solutions
- Requires deeper contextual analysis, time series data analysis, and integrated analysis of multiple log sources

#### Proactive Error Prevention
Currently reactive approach that responds after errors occur. Future improvements:
- Proactive approach that detects and prevents errors before they occur
- Analyze system monitoring data to warn of memory or disk space shortages in advance
- Take preventive measures to prevent unnecessary interruptions
- Requires time series prediction models, anomaly detection algorithms, and resource usage pattern learning
- Integration with verification schedulers

#### Advanced Parameter Optimization
To automate parameter determination for Group B:
- Introduce reinforcement learning or Bayesian optimization
- Learn from past execution history
- Automatically propose optimal parameters for current situation
- Reflect execution results back into learning
- Create virtuous cycle

#### Explainable AI
More detailed explanations needed:
- Why specific solutions were proposed
- Why judged to be automatically executable
- Enhances human understanding and trust
- Helps with failed case analysis
- Methods: leverage LLM's explanation generation capabilities or visualize decision trees

#### Extension to Other Verification Stages
Currently focused on RTL simulation verification. Future applications:
- Build and compilation errors
- Formal verification errors
- System-level verification
- Each stage has unique error types and resolution methods
- Core architecture should be reusable: executability evaluation, human-in-the-loop approval, safe execution mechanisms

## VII. CONCLUSION

This paper proposed an AI agent-based system that automatically executes solutions for errors occurring in the SoC RTL verification process. To address the inefficiencies of traditional manual error handling, we designed a system consisting of two specialized agents and established a comprehensive implementation methodology.

### Main Contributions

1. A safe and scalable framework for automatic solution execution based on 150,000 tasks and 86 verified error-solution pairs
2. An executability evaluation mechanism that balances efficiency and safety by considering technical feasibility, safety, and business rules
3. A practical automation approach through human-in-the-loop mechanisms
4. A phased verification methodology for safe deployment

### Expected Impact

The system targets:
- 70-80% automation rate
- 60-70% processing time reduction
- When expanded to entire organization: savings of 4,000-6,000 engineer-hours per month (48,000-72,000 annually)
- Significantly improve verification organization productivity
- Free engineers from repetitive tasks to focus on more complex and valuable verification work

### Current Limitations

- 86 patterns cover only 80-85% of errors, requiring continuous database expansion
- Challenges remain in parameter optimization for Group B and root cause analysis for Group C
- Environment change adaptation mechanisms needed to maintain solution validity during tool updates

### Future Work

The system design and preparation phase is complete. Future phased verification experiments will collect actual performance data to empirically demonstrate the system's practicality and effectiveness. This research presents a concrete and achievable direction for digital transformation of verification workflows, and with advanced features such as root cause analysis, predictive error prevention, and automatic parameter optimization, AI agents are expected to evolve into intelligent verification partners beyond simple automation tools.

## REFERENCES

[1] G. Eason, B. Noble, and I.N. Sneddon, "On certain integrals of Lipschitz-Hankel type involving products of Bessel functions," Phil. Trans. Roy. Soc. London, vol. A247, pp. 529-551, April 1955.

[2] J. Clerk Maxwell, A Treatise on Electricity and Magnetism, 3rd ed., vol. 2. Oxford: Clarendon, 1892, pp.68-73.

[3] I.S. Jacobs and C.P. Bean, "Fine particles, thin films and exchange anisotropy," in Magnetism, vol. III, G.T. Rado and H. Suhl, Eds. New York: Academic, 1963, pp. 271-350.

[4] K. Elissa, "Title of paper if known," unpublished.

[5] R. Nicole, "Title of paper with only first word capitalized," J. Name Stand. Abbrev., in press.

[6] Y. Yorozu, M. Hirano, K. Oka, and Y. Tagawa, "Electron spectroscopy studies on magneto-optical media and plastic substrate interface," IEEE Transl. J. Magn. Japan, vol. 2, pp. 740-741, August 1987 [Digests 9th Annual Conf. Magnetics Japan, p. 301, 1982].

[7] M. Young, The Technical Writer's Handbook. Mill Valley, CA: University Science, 1989.
