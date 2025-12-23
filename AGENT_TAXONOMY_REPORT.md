# OpenHands Agent Taxonomy Classification Report

## Executive Summary

This report provides a comprehensive multi-label classification of the OpenHands agent system according to the SWE Agent Taxonomy framework. OpenHands is a sophisticated AI-driven development platform that demonstrates advanced multi-agent coordination, iterative workflow patterns, dynamic planning, comprehensive tool integration, and sophisticated memory management capabilities.

---

## 1. AGENT ARCHITECTURE

### Labels
**MULTI-AGENT-HIERARCHICAL**, **MULTI-AGENT-COLLABORATIVE**

### Justification

OpenHands implements a **hierarchical multi-agent architecture** with manager-worker delegation patterns. The system features:

1. **Hierarchical Coordination**: The primary CodeActAgent acts as a coordinator that can delegate specialized tasks to worker agents (e.g., BrowsingAgent for web-based tasks). Evidence from `openhands/agenthub/README.md` (lines 96-143) explicitly describes multi-agent delegation: *"OpenHands is a multi-agentic system. Agents can delegate tasks to other agents...the CodeActAgent might delegate to the BrowsingAgent to answer questions that involve browsing the web."*

2. **Specialized Worker Agents**: The system includes multiple specialized agents:
   - **CodeActAgent**: Main agent for code execution and development tasks
   - **BrowsingAgent**: Specialized for web browsing and information retrieval
   - **LocAgent**: Graph-based code localization and search
   - **ReadonlyAgent**: Read-only repository analysis
   - **VisualBrowsingAgent**: Visual web interaction

3. **Delegation Mechanism**: The `AgentDelegateAction` (in `openhands/events/action/agent.py`, lines 77-86) enables task delegation with explicit agent selection and input specification. The `AgentController` (in `openhands/controller/agent_controller.py`) manages parent-child relationships through `parent` and `delegate` attributes (line 110-111).

4. **Collaborative Elements**: Different agents contribute complementary expertise - the CodeActAgent handles general development while delegating to specialized agents for domain-specific tasks, representing **collaborative** patterns where agents work together as a team with distinct roles.

---

## 2. AGENT COGNITION: Workflow Patterns

### Labels
**ITERATIVE**, **SELF-IMPROVING**

### Justification

OpenHands demonstrates **iterative workflow** with **self-improving** capabilities:

1. **Iterative Loop**: The core agent operates through continuous action-observation cycles. The `AgentController._step()` method implements the fundamental loop where:
   - The agent receives state information
   - Generates an action via `agent.step(state)`
   - Executes the action and receives observations
   - Uses feedback to inform the next action
   
   This matches the taxonomy's definition of ITERATIVE: *"The agent executes an action, immediately observes the result, and uses that feedback to plan its next small move, repeating the cycle (Thought → Action → Observation)."*

2. **Loop Detection and Recovery**: The `StuckDetector` class (in `openhands/controller/stuck.py`) monitors for repetitive patterns and enables self-correction. The system detects when agents are stuck in loops and can perform recovery actions (`LoopRecoveryAction`), demonstrating retrospective analysis and adjustment.

3. **Self-Improving Mechanisms**:
   - **Memory Condensation**: The `Condenser` component (referenced in `openhands/memory/README.md`) summarizes earlier events and learns from past interactions to improve future performance
   - **Conversation Memory**: The `ConversationMemory` class tracks interaction history and uses recorded memories to improve decision-making
   - **Reflective Learning**: The system prompt (in `openhands/agenthub/codeact_agent/prompts/system_prompt.j2`, lines 88-95) explicitly instructs the agent to reflect on failures: *"If you've made repeated attempts to solve a problem but tests still fail...Step back and reflect on 5-7 different possible sources of the problem"*

The combination of continuous feedback loops and explicit mechanisms for learning from mistakes aligns with both ITERATIVE and SELF-IMPROVING patterns.

---

## 3. AGENT COGNITION: Planning Strategies

### Labels
**HIERARCHICAL**, **REPLANNING**, **INTERACTIVE**, **RETRIEVAL-AUGMENTED-PLANNING**

### Justification

OpenHands employs multiple sophisticated planning strategies:

1. **Hierarchical Planning**: The `task_tracker` tool (in `openhands/agenthub/codeact_agent/tools/task_tracker.py`) enables hierarchical task decomposition. The tool description (lines 5-80) explicitly describes breaking high-level goals into increasingly detailed sub-goals: *"Multi-phase development work...Complex implementation tasks requiring systematic planning and coordination across multiple components."* Examples show decomposition like "build a house" → "foundation" → "digging" → "pouring".

2. **Replanning (Dynamic Adjustment)**: The system adapts plans when conditions change:
   - The `StuckDetector` identifies when execution deviates from expected behavior
   - The `LoopRecoveryAction` mechanism enables the agent to restart or adjust its approach when stuck
   - The agent can modify task states through `ModifyTaskAction` (referenced in `openhands/agenthub/README.md`)

3. **Interactive Planning**: The system supports human-in-the-loop decision making:
   - **Confirmation Mode**: The `confirmation_mode` parameter (in `AgentController.__init__`) allows the agent to request approval before executing actions
   - **User Messages**: The agent can send `MessageAction` to ask for clarification or guidance
   - **Traffic Control**: The system pauses for user input and can be resumed, as indicated by the `TRAFFIC_CONTROL_REMINDER` constant

4. **Retrieval-Augmented Planning**: OpenHands uses knowledge from past experiences:
   - **Microagents**: The system loads both global and user-specific microagents (in `openhands/memory/memory.py`, lines 80-84) that contain domain-specific knowledge templates
   - **RepoMicroagent**: Stores repository-specific conventions and patterns (e.g., from `.cursorrules`, `agents.md`) that guide planning
   - **KnowledgeMicroagent**: Provides general knowledge that informs planning decisions
   - The `RecallAction` (in `openhands/events/action/agent.py`, lines 89-104) retrieves workspace context and historical information to inform current planning

---

## 4. AGENT COGNITION: Reasoning Techniques

### Labels
**CHAIN-OF-THOUGHT**, **REACT**, **PROGRAM-AIDED**, **TOOL-COMPOSITION**, **TEST-DRIVEN**

### Justification

OpenHands implements multiple advanced reasoning techniques:

1. **Chain-of-Thought**: The agent explicitly uses step-by-step reasoning:
   - The `AgentThinkAction` (in `openhands/events/action/agent.py`, lines 46-59) logs intermediate reasoning steps
   - The `ThinkTool` (in `openhands/agenthub/codeact_agent/tools/think.py`) allows the agent to express its thinking process
   - Tool calls can include a `thought` parameter that captures the reasoning leading to each action

2. **ReAct (Reasoning + Acting)**: OpenHands perfectly exemplifies the ReAct pattern:
   - The agent interleaves **thought** (via `AgentThinkAction`), **action** (via various action types), and **observation** (via observation events)
   - The core loop in `AgentController` implements: Think → Act → Observe → Think again
   - This matches the taxonomy definition: *"The agent interleaves thought, action, and observation in a cycle"*

3. **Program-Aided Reasoning**: The agent generates and executes code to solve problems:
   - **IPython Tool**: Executes Python code via `IPythonRunCellAction` for calculations and data processing (in `openhands/agenthub/codeact_agent/tools/ipython.py`)
   - **Bash Tool**: Runs shell commands for file operations and system tasks
   - The CodeAct paradigm (described in `openhands/agenthub/codeact_agent/README.md`) consolidates actions into unified code execution, using *"code to do the hard work"*

4. **Tool-Composition**: OpenHands strategically chains multiple tools:
   - **Sequential Composition**: The agent uses output from one tool (e.g., `grep` to find files) as input to another (e.g., `str_replace_editor` to modify them)
   - **Parallel Capabilities**: Multiple tools can be configured simultaneously (bash, IPython, browser, file editor)
   - The function calling system (in `openhands/agenthub/codeact_agent/function_calling.py`) coordinates different capabilities to accomplish complex tasks

5. **Test-Driven Development**: The system emphasizes testing and verification:
   - The system prompt (lines 52-57) explicitly promotes TDD: *"For bug fixes: Create tests to verify issues before implementing fixes. For new features: Consider test-driven development when appropriate."*
   - The workflow includes a TESTING phase before IMPLEMENTATION
   - The agent verifies solutions meet required conditions before considering tasks complete

---

## 5. AGENT MEMORY

### Labels
**EPHEMERAL**, **EPISODIC**, **SEMANTIC**, **PROCEDURAL**, **MULTI-TIERED**, **AGENTIC**, **SHARED-MEMORY**, **MESSAGE-PASSING**

### Justification

OpenHands implements a sophisticated multi-faceted memory system:

1. **Ephemeral (Session-Based)**: The primary context is ephemeral within sessions:
   - The `State` object maintains current session context without long-term persistence by default
   - The `event_stream` provides real-time context during task execution
   - Memory is reset between independent sessions unless explicitly saved

2. **Episodic Memory**: The system remembers specific past events with temporal context:
   - The `State.history` (described in `openhands/agenthub/README.md`, lines 35-36) stores complete event sequences with timestamps
   - Events include `start_id` and `end_id` to track temporal ordering
   - The system can retrieve actions and observations from current and past sessions

3. **Semantic (Structured Knowledge)**: OpenHands stores factual knowledge and relationships:
   - **Microagents**: Store structured domain knowledge about repositories, coding conventions, and best practices
   - **KnowledgeMicroagent**: Contains factual information about technologies and patterns
   - **RepoMicroagent**: Stores repository-specific semantic knowledge (dependencies, architecture patterns)
   - The `metadata` field in microagents includes structured information like version, author, and tags

4. **Procedural (Skills and How-To)**: The system maintains executable operational knowledge:
   - **Agent Skills**: The `AgentSkillsRequirement` plugin (in `openhands/runtime/plugins`) provides reusable procedural knowledge
   - **Microagents**: Contain executable procedures and workflows (e.g., how to handle npm installations, GitHub operations)
   - The agent skills include file operations, repository operations, and utility functions that represent encoded expertise

5. **Multi-Tiered (Hierarchical Storage)**: Memory is organized in layers:
   - **Working Memory**: Active `ConversationMemory` for current context (fast access)
   - **Condensed Memory**: The `Condenser` component summarizes older events into compressed summaries (described in `openhands/memory/README.md`, lines 12-18)
   - **Long-Term Storage**: Microagents and persistent state stored in file system
   - The condenser progressively summarizes earlier events first, moving less-used information to compressed storage

6. **Agentic (Self-Organizing)**: Memory autonomously reorganizes:
   - The `Condenser` automatically determines when and what to condense based on context window constraints
   - The memory system dynamically selects which microagents to load based on workspace context
   - Condensation happens autonomously without explicit external instruction

7. **Shared-Memory (Multi-Agent Access)**: Multiple agents access common knowledge:
   - All agents can read from the shared `event_stream`
   - Microagents are accessible to all agents in the system
   - The `State` object is shared across agent delegation boundaries

8. **Message-Passing (Agent-to-Agent Communication)**: Direct agent communication:
   - The `AgentDelegateAction` passes information to worker agents with explicit `inputs` parameter
   - The `AgentDelegateObservation` returns results from delegated agents
   - Parent and child agents communicate through the event stream while maintaining separate internal states

---

## 6. AGENT TOOL: External Capabilities and Environmental Interactions

### Labels
**FILE-MANAGEMENT**, **CODE-EDITING**, **STRUCTURAL-RETRIEVAL**, **VERSION-CONTROL**, **PYTHON-TOOLS**, **TESTING-TOOLS**, **LINTING-VALIDATION**, **DATABASE-TOOLS**, **SYSTEM-UTILITIES**, **TEXT-PROCESSING**, **SHELL-SCRIPTING**, **PACKAGE-MANAGEMENT**, **BUILD-TOOLS**, **CONTAINER-TOOLS**, **WEB-TOOLS**, **NAVIGATION-TOOLS**, **ENVIRONMENT-VARIABLES**, **TERMINAL-CONTROL**

### Justification

OpenHands provides comprehensive tool capabilities spanning nearly all taxonomy categories:

1. **File-Management**: Complete file system operations:
   - Navigation: `pwd`, `ls`, `cd`, `find`, `tree` (via bash commands)
   - Manipulation: `cp`, `mv`, `rm`, `mkdir`, `touch`
   - Inspection: `cat`, `head`, `tail`, `less`, `wc`
   - Permissions: `chmod`, `ln`

2. **Code-Editing**: Specialized code modification tools:
   - **str_replace_editor**: String-based file editing with exact matching (`openhands/agenthub/codeact_agent/tools/str_replace_editor.py`)
   - **LLM-based editor**: AI-powered file editing with line ranges (`openhands/agenthub/codeact_agent/tools/llm_based_edit.py`)
   - File reading via `FileReadAction`
   - File writing via `FileWriteAction`
   - `vi`, `nano`, `edit` commands through bash

3. **Structural-Retrieval**: Advanced code comprehension:
   - **LocAgent Tools**: Graph-based structural analysis with `search_code_snippets`, `get_entity_contents`, `explore_tree_structure` (described in `openhands/agenthub/loc_agent/README.md`)
   - **Pattern Matching**: `grep`, `egrep` for code search
   - **AST-based Analysis**: The LocAgent parses codebases into directed heterogeneous graphs for structural understanding

4. **Version-Control**: Comprehensive Git integration:
   - Full `git` command suite available through bash
   - Git credential management (system prompt lines 35-36)
   - Commit guidelines and `.gitignore` management
   - Pull request operations (system prompt lines 42-47)

5. **Python-Tools**: Full Python ecosystem support:
   - **IPython**: Interactive Python execution via `IPythonRunCellAction` with kernel gateway
   - **pip**: Package management (`pip`, `pip3`)
   - Multiple Python versions support
   - Environment variables: `PYTHONPATH`, `PYTHONIOENCODING`, `PYTHONWARNINGS`

6. **Testing-Tools**: Test execution and validation:
   - Framework support: `pytest`, `nose2`, `nosetests`
   - Custom test scripts via bash execution
   - Test-driven development workflow (system prompt lines 52-57)

7. **Linting-Validation**: Code quality analysis:
   - `flake8` available in runtime dependencies
   - `pylint` support with `PYLINTHOME` configuration
   - Code style enforcement capabilities

8. **Database-Tools**: Database interaction (via bash):
   - `mysql`, `psql`, `sqlite3` clients
   - Database administration tools
   - Credential management via environment variables

9. **System-Utilities**: Operating system operations:
   - Process control: `ps`, `kill`, `pkill`, `nohup`, `sleep`, `timeout`
   - Environment management: `env`, `export`, `unset`, `source`, `alias`
   - System information: `hostname`, `locale`, `which`
   - Administrative tools: `sudo`, `systemctl`, `service`

10. **Text-Processing**: Text transformation tools:
    - Stream editors: `sed`, `awk`
    - Encoding converters: `iconv`
    - Binary viewers: `hexdump`, `xxd`

11. **Shell-Scripting**: Bash programming support:
    - Full bash interpreter access via `execute_bash` tool
    - Control structures: `if`, `for`, loops
    - Flow control: `break`, `continue`
    - Script variables and functions

12. **Package-Management**: Dependency management:
    - System packages: `apt-get`, `apt-cache` (via bash)
    - Environment managers: `conda`, `tox` (available in dependencies)
    - npm microagent for Node.js packages (referenced in README)

13. **Build-Tools**: Compilation and build systems:
    - `make` available through bash
    - `java` compiler support
    - Document compilation tools

14. **Container-Tools**: Docker integration:
    - The system itself runs in Docker containers
    - `docker` Python library in dependencies (`pyproject.toml`)
    - Container-based runtime isolation

15. **Web-Tools**: Network and web capabilities:
    - **Browser Tool**: Full web interaction via `BrowserTool` (`openhands/agenthub/codeact_agent/tools/browser.py`)
    - **BrowsingAgent**: Dedicated agent for web browsing tasks
    - HTTP clients: `curl`, `wget` via bash
    - Web reading: `web_read` function for markdown conversion

16. **Navigation-Tools**: Workspace exploration:
    - Directory operations through file management tools
    - Efficient codebase traversal using `find`, `grep`, `tree`
    - Workspace context awareness

17. **Environment-Variables**: Configuration management:
    - Locale settings: `LANG`, `LC_ALL`
    - Display configurations: `MPLBACKEND`
    - Custom environment variable support via bash

18. **Terminal-Control**: Interactive terminal capabilities:
    - The bash tool supports interactive processes with STDIN input
    - Process interruption capabilities
    - Timeout handling for long-running commands

**Not Identified**:
- **Embedding-Retrieval**: No evidence of vector database or dense embedding-based semantic search. The LocAgent uses graph-based structural retrieval, not vector embeddings.
- **Knowledge-Graph**: While LocAgent uses graph representation, it's specifically for code structure, not a general knowledge graph with ontologies for semantic reasoning.
- **Django-Tools**: No Django-specific utilities found.
- **Sphinx-Documentation**: No Sphinx documentation tools identified.
- **Debugging-Tools**: No explicit debugging tools like `strace` or debuggers found.
- **Media-Tools**: No image/video processing tools identified.
- **Archive-Tools**: No compression/archiving utilities identified.

---

## Novel Features

No novel features requiring new taxonomy labels were identified. All observed capabilities map cleanly to existing taxonomy categories.

---

## Summary

OpenHands is a sophisticated multi-agent software engineering system that demonstrates:

- **Advanced multi-agent architecture** with hierarchical coordination and specialized worker agents
- **Iterative and self-improving workflows** with loop detection and recovery mechanisms
- **Comprehensive planning** using hierarchical decomposition, dynamic replanning, interactive collaboration, and knowledge retrieval
- **Multiple reasoning techniques** including chain-of-thought, ReAct patterns, program-aided reasoning, tool composition, and test-driven development
- **Sophisticated memory systems** spanning ephemeral to persistent storage with episodic, semantic, and procedural knowledge
- **Extensive tool integration** covering 18 of 28 tool categories, with particular strength in code editing, structural retrieval, version control, and web interaction

The system represents a state-of-the-art implementation of agentic AI for software engineering tasks, combining multiple advanced capabilities in a cohesive, production-ready platform.
