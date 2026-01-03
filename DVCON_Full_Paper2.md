# Multi-Agent System for Automated Error Triage in SoC RTL Verification

**Abstract**—RTL verification for modern SoC designs generates thousands of errors during regression testing, requiring significant engineer time for triage and information gathering. This paper presents a multi-agent system that automates error information gathering for RTL verification workflows. The system employs five specialized agents orchestrated via LangChain/LangGraph with domain-specific prompt engineering for 12 RTL verification error categories. Preliminary case studies from a flagship SoC project with over 30,000 regression tests demonstrate potential time reduction from approximately 24 minutes to 3 minutes per error for information gathering tasks. The system leverages RAG-based retrieval from an 86-pattern database and integrates with existing verification infrastructure. When deployed across the common verification domain, the system is expected to save over 1,700 engineer-hours per regression cycle. The key contribution of this work is the systematic approach to encoding RTL verification expertise into agent instructions through comprehensive prompt engineering, enabling practical deployment in production verification environments.

**Index Terms**—RTL verification, multi-agent systems, prompt engineering, error triage, LangChain, RAG

---

## I. INTRODUCTION

Modern System-on-Chip verification workflows generate thousands of errors during overnight regression testing. In a flagship SoC project with over 30,000 regression tests, verification engineers routinely spend 15 to 25 minutes per error manually gathering information before they can begin actual problem-solving. This process involves reviewing error logs, searching for similar historical cases, consulting documentation, and collecting system state information. While these tasks are systematic and follow predictable patterns, they remain a significant bottleneck that consumes engineering resources better spent on complex design analysis.

The challenge of automating RTL verification error handling stems from several factors. First, verification errors require specialized knowledge of HDL semantics, tool configurations, and design constraints that generic automation approaches fail to capture. Second, the information needed for resolution is scattered across multiple sources including log files, specification databases, historical case records, and system configuration files. Third, verification errors span a wide spectrum from simple configuration issues that can be resolved automatically to complex design conflicts requiring deep RTL expertise. Finally, production verification environments demand conservative automation with clear boundaries between information gathering and decision-making.

This paper presents a multi-agent system focused on automating the information gathering phase of RTL verification error triage. Rather than attempting complete error resolution, which would require extensive RTL design knowledge, the system automates the systematic collection of information that engineers need to make informed decisions. The approach separates what can be safely automated from what requires human judgment.

The contributions of this work are fourfold. First, we present comprehensive domain-specific prompt engineering that encodes RTL verification expertise into agent instructions for 12 error categories. This represents our primary contribution and enables the practical application of large language models to verification workflows. Second, we describe a five-agent architecture specifically designed for error triage, with clear separation between automated information gathering and human decision-making. Third, we introduce a 12-category error taxonomy derived from analysis of production regression tests, classifying errors based on their automation potential and information requirements. Fourth, we provide validation through case studies from a flagship SoC project, demonstrating the practical effectiveness of the approach.

The remainder of this paper is organized as follows. Section II reviews related work in automated debugging and agent-based systems. Section III presents the system architecture including the agent design and error taxonomy. Section IV describes the prompt engineering methodology. Section V reports implementation details and case study results. Section VI discusses practical considerations and future directions, and Section VII concludes.

---

## II. RELATED WORK

Research on automated software debugging has advanced significantly with the emergence of large language models. Systems like AutoCodeRover [1] demonstrate that LLM-based agents can analyze issues and propose code modifications, achieving notable success rates on benchmarks such as SWE-bench [2]. These approaches typically employ multi-stage pipelines combining code search, bug localization, and patch generation. However, their focus on source code modification differs fundamentally from verification error triage, where the goal is information gathering rather than code changes, and where safety requirements necessitate human oversight of any modifications.

Log-based anomaly detection represents another relevant research direction. Approaches using deep learning and language models for log analysis [3] have shown effectiveness in identifying system anomalies through pattern recognition. While these methods excel at detection, they typically stop short of providing the actionable information gathering that verification engineers require. The gap between detecting that something is wrong and collecting the specific information needed for resolution remains significant.

Hardware verification automation research has primarily concentrated on test generation, coverage analysis, and formal verification techniques. Work on HDL debugging assistance using retrieval-augmented generation [4] provides conversational support for general debugging tasks. Research on assertion failure resolution [5] demonstrates that domain-specific language models can suggest fixes for RTL verification issues. However, these efforts target specific aspects of the verification process rather than the broader challenge of error triage and information gathering.

Multi-agent collaboration frameworks have emerged as a promising approach for complex software tasks. Systems employing role-based agents for software development [6] demonstrate that specialization and structured communication protocols can improve outcomes. The concept of encoding standard operating procedures into agent instructions [7] has proven particularly relevant, as it provides a mechanism for capturing domain expertise in a form that agents can execute consistently.

Our work differs from prior approaches in its specific focus on information gathering automation for RTL verification. Rather than attempting to fix errors automatically, we automate the tedious but systematic process of collecting the information engineers need. This conservative approach aligns with the safety requirements of production verification environments while still providing substantial time savings.

---

## III. SYSTEM ARCHITECTURE

### A. Design Philosophy

The architecture follows a fundamental principle: automate information gathering while preserving human decision-making. Analysis of standard operating procedures for RTL verification errors reveals a consistent pattern across error types. Engineers first locate and open relevant log files, then identify error patterns using keywords and line numbers, followed by collecting context from configuration files, specifications, and environment state. Only after this information gathering is complete do they interpret findings and determine corrective action.

The first three steps are systematic and follow predictable patterns that can be automated effectively. The final step requires RTL design knowledge, understanding of project-specific constraints, and engineering judgment that cannot be safely automated. The architecture reflects this boundary explicitly, with agents handling information collection while presenting synthesized results to engineers for decision-making.

### B. Error Taxonomy

We classify verification errors into 12 categories based on analysis of regression test failures from a flagship SoC project. These categories group errors by their characteristics and information requirements rather than by superficial symptoms.

Environment and configuration errors comprise three categories. Option and value setting errors occur when configuration files contain missing or invalid parameters. Path and filename mismatches arise from version inconsistencies, often related to source control synchronization. File not found errors indicate missing specification or HDL files at expected locations.

Design errors span seven categories reflecting the complexity of RTL verification. Type and name definition errors involve undefined identifiers or type mismatches. Multiple destination errors occur when signals have conflicting drivers. Port-related errors include missing port names, empty specification fields, and bit-width mismatches between specifications and implementations. Tie value errors arise from inconsistencies in constant signal assignments. Hierarchy constraint violations occur when module instantiation depth exceeds defined limits.

Tool and file errors include two categories covering specification value mismatches between documentation and implementation, and file generation or parsing failures in automated flows.

This taxonomy emerged from analyzing common domain regression tests, where approximately 75 percent of failures fell into these 12 categories. The remaining errors typically require specialized investigation beyond systematic information gathering.

### C. Five-Agent Architecture

The system employs five specialized agents, each with distinct responsibilities in the error triage workflow. Figure 1 illustrates the architecture and data flow between components.

The Error Analyzer agent combines error classification with SOP retrieval. When receiving an error log, it classifies the error into one of 12 categories using pattern matching and contextual analysis. Simultaneously, it generates embeddings of the error context and performs similarity search against the 86-pattern database to retrieve relevant standard operating procedures. The output includes error classification with confidence scores and matched SOP steps when similarity exceeds the threshold.

The Data Collector agent performs category-specific information gathering based on the error classification and SOP steps. For configuration errors, it locates and reads configuration files, searches documentation for valid parameters, and queries the pattern database for similar historical cases. For design errors, it extracts relevant portions of HDL source files, queries specification databases for expected values, and traces signal connections through the hierarchy. All operations are read-only to ensure safety.

The Decision Maker agent synthesizes collected information and formulates recommendations. It evaluates information completeness, assesses whether the matched SOP applies to the current situation, and calculates risk scores based on the complexity and potential impact of recommended actions. The output classifies each case as either suitable for automated execution or requiring manual review, with clear rationale for the classification.

The Auto Executor agent handles only low-risk actions that meet strict criteria. It operates under a whitelist of permitted operations, creates mandatory backups before any modifications, and includes automatic rollback capability. Actions with risk scores above the threshold or without clear rollback paths are never executed automatically regardless of other factors.

The Notification agent formats all gathered information and recommendations into structured reports optimized for quick engineer review. Reports highlight actionable items, include relevant file locations with line numbers, present similar historical cases, and provide clear guidance on required decisions.

### D. Technology Stack

The system uses LangChain and LangGraph for agent orchestration, providing workflow management and inter-agent communication. Retrieval-augmented generation employs embedding models for error pattern similarity search against the 86-pattern database. The pattern database stores verified error-solution pairs along with historical case records. A separate specification database contains RTL design data including port definitions and signal characteristics. Custom protocol servers provide standardized access to databases and file system operations on verification servers.

---

## IV. PROMPT ENGINEERING METHODOLOGY

Effective automation of verification error triage requires prompts that encode domain expertise in a form agents can execute consistently. Through iterative development and testing, we identified four critical design principles that govern prompt construction across all agents.

The first principle is domain specificity. Generic instructions fail in RTL verification contexts because agents lack the background knowledge to interpret them correctly. Effective prompts must include specific error patterns for each category, common file locations in verification environments, tool-specific terminology, and expected value formats. Rather than instructing an agent to analyze an error, prompts specify exactly which patterns to search for, which files to examine, and how to interpret the findings.

The second principle is structured output. Agents must produce outputs in consistent formats that downstream agents can parse reliably. We enforce strict schemas specifying required fields, null handling conventions, and nested structures for complex data. This consistency enables reliable workflow orchestration without manual intervention between stages.

The third principle concerns safety constraints. Production verification requires explicit boundaries on agent behavior. Prompts include both whitelists of permitted operations and blacklists of prohibited actions. Specifying only permitted operations proved insufficient, as agents would sometimes combine allowed commands in unintended ways. Explicit prohibition of dangerous operations like file deletion or database modification provides defense in depth.

The fourth principle addresses context preservation. Each agent must receive sufficient context to operate correctly without requiring information from agents it cannot communicate with directly. Prompts specify that error context should include surrounding log lines, file paths with line numbers, timestamps, and relevant environment state. Previous agent outputs flow through the workflow to maintain continuity.

These principles manifest differently across agents based on their roles. The Error Analyzer prompt includes detailed descriptions of all 12 error categories with example patterns, keywords, and typical locations. It specifies RAG search parameters including similarity thresholds and result limits, and defines the output schema for classification results and matched SOPs.

The Data Collector prompt contains category-specific collection instructions. Each of the 12 categories has tailored guidance on what information to gather and where to find it. The prompt explicitly lists permitted read operations and prohibited write operations, with instructions to set missing data fields to null rather than failing the collection process.

The Decision Maker prompt provides a risk assessment framework distinguishing low, medium, and high risk actions with specific criteria for each level. It includes rules for classifying actions as automatically executable versus requiring manual review, with conservative defaults that favor human oversight when uncertainty exists.

The Auto Executor prompt implements strict authorization checks, mandatory backup procedures, and step-by-step validation requirements. It specifies exact command whitelists and details the automatic rollback process triggered by any execution failure.

The Notification prompt defines the report structure optimized for quick review, including guidelines for highlighting actionable items, presenting file locations, and summarizing similar cases. The goal is enabling engineers to understand the situation and make decisions within five minutes.

---

## V. IMPLEMENTATION AND CASE STUDY VALIDATION

### A. Experimental Context

The system was developed and tested in the context of a flagship SoC verification project containing over 30,000 regression tests. Analysis focused on the common verification domain comprising approximately 6,600 tests, selected because errors in this domain affect multiple IP blocks and design teams, providing broad applicability for automation efforts.

Within the common domain, approximately 75 percent of errors fell into the 12 categories defined in our taxonomy, representing roughly 4,900 potentially automatable cases. The remaining errors involved unique situations requiring specialized investigation beyond systematic information gathering.

The pattern database was constructed from historical error records, containing 86 verified error-solution pairs. Each pattern includes error identification criteria, required information for resolution, solution steps, and historical success metrics. Pattern quality directly impacts system effectiveness, as retrieval accuracy depends on comprehensive and accurate pattern definitions.

### B. Case Study Results

Preliminary validation was conducted using representative cases from the flagship project. Due to the early stage of deployment, comprehensive evaluation across all potentially automatable errors has not yet been completed. The results presented here demonstrate the approach's feasibility rather than definitive performance metrics.

For a representative configuration error case involving a missing simulation option, manual information gathering typically required approximately 25 minutes. This included 8 minutes reviewing error logs, 12 minutes locating configuration files and searching for similar cases, and 5 minutes consulting documentation. The agent system completed equivalent information gathering in approximately 2.5 minutes, with error analysis and SOP retrieval taking 30 seconds, data collection requiring 90 seconds, and report generation completing in 30 seconds.

A representative path mismatch case showed similar results. Manual information gathering averaged 22 minutes, while the agent system completed the task in under 3 minutes. Information completeness assessment indicated the system collected 7 to 8 of the 9 to 10 items engineers would typically gather, with missing items generally being optional context rather than essential information.

Table I summarizes the case study measurements. These results suggest potential time reduction of approximately 85 to 90 percent for the information gathering phase of error triage.

**Table I: Case Study Validation Results**

| Metric | Config Error | Path Error | Average |
|--------|--------------|------------|---------|
| Manual Process Time | 25 min | 22 min | 23.5 min |
| Agent Process Time | 2.5 min | 2.8 min | 2.65 min |
| Time Reduction | 90% | 87% | 88.5% |
| Information Completeness | 8/10 items | 7/9 items | ~80% |

### C. Expected Impact

Based on the case study results, we estimate potential impact when the system is deployed across the common verification domain. With approximately 4,900 automatable errors and average time savings of 21 minutes per error, full deployment could save over 1,700 engineer-hours per regression cycle. These estimates assume the case study results generalize to the broader population of errors within the 12 categories, which requires validation through expanded deployment.

The primary limitation of our current validation is sample size. Case studies demonstrate feasibility but do not constitute comprehensive evaluation. Manual baseline times were estimated from engineer interviews rather than controlled measurement, introducing potential variability. Results from a single project may not generalize to other verification environments with different tools, processes, or error distributions.

---

## VI. DISCUSSION

The case study results suggest that information gathering automation for verification error triage is practically achievable. The systematic nature of information collection lends itself to automation in ways that decision-making does not. Engineers found the structured reports useful for quickly understanding error situations, and the inclusion of similar historical cases provided valuable context for resolution decisions.

Several factors proved critical for achieving useful results. Domain-specific prompts encoding verification expertise significantly outperformed generic instructions. The 12-category taxonomy provided sufficient granularity for tailored information gathering while remaining manageable for prompt engineering. Conservative safety constraints, while limiting automation scope, built confidence in system reliability.

Deploying this type of system in production environments requires addressing several practical considerations. The pattern database requires ongoing maintenance as verification tools evolve and new error patterns emerge. Currently, patterns must be manually curated by domain experts, representing a significant initial investment and continuing maintenance burden. Future work should explore automated pattern extraction from successful engineer resolutions to reduce this manual effort.

The current implementation provides information gathering and recommendations but does not include automated execution with human approval workflows. Implementing such capability would require careful attention to approval interfaces, timeout handling, and audit logging. The conservative approach of requiring manual action for all but the lowest-risk operations reflects the early stage of deployment and organizational comfort with automation.

Integration with existing verification infrastructure presents technical challenges around authentication, access control, and system compatibility. The modular architecture using standardized protocols for database and file system access facilitates integration, but each deployment environment has unique requirements.

Trust building with engineering teams is essential for adoption. Demonstrating reliability through progressive expansion from simple cases to more complex scenarios helps establish confidence. Transparency about system limitations and clear communication when manual investigation is needed maintains realistic expectations.

Future development directions include expanding coverage beyond the current 12 categories to address a larger fraction of verification errors. Dynamic learning from engineer feedback and successful resolutions could improve pattern matching over time. Predictive capabilities that identify error-prone configurations before regression runs would shift the approach from reactive triage to proactive prevention.

---

## VII. CONCLUSION

This paper presented a multi-agent system for automating information gathering in RTL verification error triage. The system employs five specialized agents with domain-specific prompt engineering for 12 error categories, achieving preliminary case study results showing potential time reduction from approximately 24 minutes to 3 minutes per error for information gathering tasks.

The primary contribution is the systematic approach to encoding RTL verification expertise into agent prompts. The prompt engineering methodology, emphasizing domain specificity, structured output, safety constraints, and context preservation, enables practical application of large language models to verification workflows while maintaining appropriate boundaries between automation and human judgment.

Case studies from a flagship SoC project demonstrate feasibility, with expected savings of over 1,700 engineer-hours per regression cycle when deployed across the common verification domain. The conservative approach of automating information gathering while preserving human decision-making aligns with production safety requirements and provides a foundation for expanded automation as confidence in system reliability grows.

This work demonstrates that meaningful automation gains are achievable by focusing on systematic information collection rather than attempting complete error resolution. By recognizing the boundary between what can be safely automated and what requires human expertise, the approach delivers practical value while maintaining the oversight that production verification environments require.

---

## REFERENCES

[1] Y. Zhang et al., "AutoCodeRover: Autonomous Program Improvement," in Proc. ISSTA, 2024.

[2] C. Jimenez et al., "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?," in Proc. ICLR, 2024.

[3] M. Du et al., "DeepLog: Anomaly Detection and Diagnosis from System Logs through Deep Learning," in Proc. ACM CCS, 2017.

[4] S. Thakur et al., "Benchmarking and Improving Automated HDL Code Generation with LLMs," in Proc. DAC, 2024.

[5] M. Orenes-Vera et al., "AssertLLM: Generating and Evaluating Hardware Verification Assertions from Design Specifications via LLMs," arXiv:2402.00386, 2024.

[6] C. Qian et al., "ChatDev: Communicative Agents for Software Development," in Proc. ACL, 2024.

[7] S. Hong et al., "MetaGPT: Meta Programming for Multi-Agent Collaborative Framework," arXiv:2308.00352, 2023.

---

**END OF PAPER**
