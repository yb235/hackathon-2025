# Code Structure and Organization

## Table of Contents
- [Repository Overview](#repository-overview)
- [Directory Structure](#directory-structure)
- [Core Modules](#core-modules)
- [Track Directories](#track-directories)
- [Tutorials](#tutorials)
- [Configuration Files](#configuration-files)
- [Key Files Explained](#key-files-explained)

## Repository Overview

The repository is organized into several main sections:

```
hackathon-2025/
├── core/                    # Reusable agent framework code
├── tutorials/               # 8 Jupyter notebooks teaching agent development
├── track_a_iron_man/       # Track A: Reliability and performance
├── track_b_glass_box/      # Track B: Observability and explainability
├── track_c_dear_grandma/   # Track C: Security and red teaming
├── docs/                   # Documentation (this folder)
├── templates/              # Poster templates for submissions
├── assets/                 # Images and resources
└── [config files]          # requirements.txt, .env.example, etc.
```

## Directory Structure

### `/core` - Reusable Agent Framework

Optional starter code providing a foundation for building agents.

```
core/
├── react_agent/            # ReAct agent implementation
│   ├── __init__.py        # Module exports
│   ├── create_agent.py    # Agent factory function
│   ├── holistic_ai_bedrock.py  # AWS Bedrock integration
│   ├── state.py           # State management
│   ├── context.py         # Configuration
│   ├── prompts.py         # System prompts
│   ├── utils.py           # Helper functions
│   └── output_schema.py   # Structured output schemas
├── valyu_tools/           # Valyu search integration
│   ├── __init__.py
│   ├── tools.py           # ValyuSearchTool, ValyuContentsTool
│   └── retrievers.py      # Retriever implementations
├── README.md              # Core module documentation
└── requirements.txt       # Core dependencies
```

**Purpose:** Provides pre-built components that participants can use or reference. Not required - participants can build from scratch.

### `/tutorials` - Learning Path

8 self-contained Jupyter notebooks covering agent development:

```
tutorials/
├── 01_basic_agent.ipynb              # Introduction to ReAct agents
├── 02_custom_tools.ipynb             # Creating custom tools
├── 03_structured_output.ipynb        # Validated JSON responses
├── 04_model_monitoring.ipynb         # Tracking costs and emissions
├── 05_observability.ipynb            # Deep tracing with LangSmith
├── 06_benchmark_evaluation.ipynb     # Testing on benchmarks
├── 07_reinforcement_learning.ipynb   # RL training (advanced)
├── 08_attack_red_teaming.ipynb       # Security testing
└── README.md                         # Tutorial overview and setup
```

**Purpose:** Educational resources teaching agent development from basics to advanced topics.

### `/track_a_iron_man` - Performance & Reliability

Resources for building robust, efficient agents:

```
track_a_iron_man/
├── examples/
│   ├── react_agent/       # Simple ReAct agent example
│   └── deep_research/     # Complex multi-agent research system
├── Building Agents on AWS.pdf  # AWS deployment guide
└── README.md              # Track A overview and requirements
```

**Focus Areas:**
- Performance optimization
- Cost efficiency
- Robustness and error handling
- Carbon footprint reduction
- Production readiness

### `/track_b_glass_box` - Observability & Explainability

Resources for building transparent, observable agents:

```
track_b_glass_box/
├── examples/
│   ├── agent_graph_AAAI.pdf       # AgentGraph research paper
│   └── failure_attribution/       # Who_and_When dataset
├── traces/                         # Example execution traces
│   ├── trace-very-simple.json     # 13KB - Single turn
│   ├── trace-simple.json          # 65KB - Simple ReAct
│   ├── trace-normal.json          # 251KB - Normal complexity
│   └── trace-complex.json         # 15MB - Multi-agent system
└── README.md                       # Track B overview
```

**Focus Areas:**
- Complete traceability
- Human-interpretable reasoning
- Failure analysis
- Behavioral insights
- Visualization

### `/track_c_dear_grandma` - Security & Red Teaming

Resources for security testing and vulnerability assessment:

```
track_c_dear_grandma/
├── examples/
│   ├── red_teaming_datasets/      # Standardized test cases
│   │   ├── benign_test_cases.csv  # 101 benign queries
│   │   ├── harmful_test_cases.csv # 101 harmful queries
│   │   └── jailbreak_prompts.csv  # 100+ jailbreak attempts
│   ├── harmbench/                 # Automated red teaming framework
│   └── agentharm/                 # Agent harm benchmark
└── README.md                       # Track C overview + API endpoints
```

**Focus Areas:**
- Jailbreak attacks
- Prompt injection
- Tool misuse
- Attack Success Rate (ASR) measurement
- Systematic security assessment

### `/docs` - Documentation

Comprehensive documentation for the repository:

```
docs/
├── ARCHITECTURE.md         # System architecture overview
├── CODE_STRUCTURE.md       # This file - code organization
├── WORKFLOW.md             # Agent lifecycle and execution
├── API_REFERENCE.md        # API documentation
├── GETTING_STARTED.md      # Setup and first steps
├── TRACK_GUIDES.md         # Track-specific guidance
├── TOOLS_REFERENCE.md      # Tools and utilities
└── README.md               # Documentation index
```

### `/templates` - Submission Templates

Poster templates for project submissions.

### `/assets` - Resources

Images, PDFs, and other resources:
- `images/` - Logos and graphics
- `api-guide.pdf` - API documentation

## Core Modules

### 1. `react_agent` Module

**File: `create_agent.py`**
- Main function: `create_react_agent()`
- Creates a LangGraph-based ReAct agent
- Parameters:
  - `tools`: List of tool functions
  - `checkpointer`: Optional conversation persistence
  - `context`: Configuration object
  - `model_name`: Override default model
  - `output_schema`: Optional Pydantic schema for structured output
  - `system_prompt`: Custom system prompt

**File: `holistic_ai_bedrock.py`**
- Class: `HolisticAIBedrockChat` - LangChain-compatible chat model
- Class: `HolisticAIBedrockStructuredOutput` - Structured output wrapper
- Function: `get_chat_model()` - Factory for chat models
- Handles:
  - AWS Bedrock API integration
  - Message format conversion
  - Tool calling format
  - Structured output via response_format

**File: `state.py`**
- Class: `InputState` - Basic agent state with messages
- Class: `State` - Extended state with is_last_step
- Defines state structure for LangGraph

**File: `context.py`**
- Class: `Context` - Configuration dataclass
- Fields:
  - `system_prompt`: Agent behavior instructions
  - `model`: Model identifier
  - `max_search_results`: Tool configuration
  - `ollama_*`: Ollama-specific settings

**File: `prompts.py`**
- `SYSTEM_PROMPT`: Basic system prompt
- `EXPERIMENT_SYSTEM_PROMPT`: Detailed ReAct instructions

**File: `utils.py`**
- Function: `load_chat_model()` - Universal model loader
- Supports:
  - Holistic AI Bedrock models
  - OpenAI GPT-5 series
  - Ollama local models

**File: `output_schema.py`**
- Pydantic schemas for structured output
- Example: `AgentResponse` schema

### 2. `valyu_tools` Module

**File: `tools.py`**
- Class: `ValyuSearchTool` - Deep search across web and proprietary sources
  - Input: Query, search_type, max_num_results, relevance_threshold, etc.
  - Output: JSON with search results
  
- Class: `ValyuContentsTool` - Extract content from URLs
  - Input: List of URLs
  - Output: JSON with extracted content

**File: `retrievers.py`**
- Retriever implementations for document retrieval
- Integration with vector stores

## Track Directories

### Track A: Iron Man

**`examples/react_agent/`**
- Simple ReAct agent implementation
- Demonstrates basic agent structure
- Uses Tavily search tool

**`examples/deep_research/`**
- Complex multi-agent research system
- Spawns sub-agents for parallel research
- Generates markdown reports
- Demonstrates:
  - Multi-agent orchestration
  - State management across agents
  - Report generation
  - Error handling

### Track B: Glass Box

**`traces/`**
- Real LangSmith execution traces
- Demonstrates trace complexity progression
- Shows nested agent execution
- Useful for:
  - Understanding trace structure
  - Building visualization tools
  - Analyzing agent behavior
  - Learning observability patterns

**`examples/failure_attribution/`**
- Who_and_When dataset
- Annotated failure traces
- Demonstrates failure analysis

### Track C: Dear Grandma

**`examples/red_teaming_datasets/`**
- Standardized test cases
- Three CSV files:
  - Benign queries (baseline)
  - Harmful queries (safety testing)
  - Jailbreak prompts (attack testing)
- Used for systematic ASR measurement

**`examples/harmbench/`**
- Complete red teaming framework
- 18 attack methods
- Evaluation pipeline
- Used for advanced security testing

**`examples/agentharm/`**
- Agent harm benchmark
- 110 malicious tasks
- Multi-step attack scenarios
- Used for agent-specific vulnerability testing

## Tutorials

### Tutorial Sequence

1. **Basic Agent** - Foundation
   - Create first ReAct agent
   - Understand message flow
   - Use simple tools

2. **Custom Tools** - Extension
   - Build custom tools
   - Define tool schemas
   - Handle tool errors

3. **Structured Output** - Data Handling
   - Define Pydantic schemas
   - Validate responses
   - Parse JSON reliably

4. **Model Monitoring** - Performance
   - Track token usage
   - Measure costs
   - Monitor carbon emissions

5. **Observability** - Debugging
   - Set up LangSmith
   - Trace execution
   - Visualize agent behavior
   - Debug failures

6. **Benchmark Evaluation** - Testing
   - Load benchmark datasets
   - Evaluate agent performance
   - Use LLM-as-a-Judge
   - Calculate metrics

7. **Reinforcement Learning** - Optimization (Advanced)
   - RL concepts for agents
   - Training pipelines
   - Reward modeling
   - Optional/advanced topic

8. **Attack & Red Teaming** - Security
   - Jailbreak techniques
   - PAIR attacks
   - ASR measurement
   - Security testing

### Tutorial Dependencies

Each tutorial is self-contained but builds on previous concepts:

```
01 (Basic) → 02 (Tools) → 03 (Output) → 04 (Monitor)
                                     ↓
                                   05 (Observe)
                                     ↓
                              06 (Benchmark) → 08 (Security)
                                     ↓
                              07 (RL - Optional)
```

## Configuration Files

### `requirements.txt`
Main dependencies file:
- Core: LangGraph, LangChain
- Models: OpenAI integration
- Data: Pydantic, datasets
- Monitoring: LangSmith, CodeCarbon
- Development: Jupyter

### `.env.example`
Template for environment variables:
- Holistic AI Bedrock credentials
- Optional: OpenAI, Valyu, LangSmith keys
- Configuration options

### `.gitignore`
Excludes from version control:
- `.env` (secrets)
- Python cache files
- Jupyter checkpoints
- Build artifacts

### `README.md`
Main repository documentation:
- Quick start guide
- Track overviews
- Resource links
- Registration information

## Key Files Explained

### `core/react_agent/create_agent.py`

**What it does:** Creates a LangGraph agent with ReAct pattern.

**Key functions:**
```python
def create_react_agent(
    tools,              # List of tools
    checkpointer=None,  # Optional persistence
    context=None,       # Configuration
    model_name=None,    # Model override
    output_schema=None, # Structured output
    system_prompt=None  # Custom prompt
) -> CompiledGraph
```

**Internal nodes:**
- `call_model`: Invokes LLM
- `tools`: Executes tool calls
- `format_output`: (Optional) Structures response

**Graph structure:**
1. Start → call_model
2. call_model → (tools OR format_output OR end)
3. tools → call_model (loop)
4. format_output → end

### `core/react_agent/holistic_ai_bedrock.py`

**What it does:** Integrates AWS Bedrock via Holistic AI proxy.

**Key classes:**
- `HolisticAIBedrockChat`: Main chat model implementation
  - Inherits from `BaseChatModel`
  - Implements `_generate()` for API calls
  - Handles message format conversion
  - Supports tool calling
  
- `HolisticAIBedrockStructuredOutput`: Structured output wrapper
  - Uses response_format API feature
  - Validates with Pydantic schemas
  - Handles JSON parsing

**Message conversion:**
```python
LangChain Format          AWS Bedrock Format
-----------------        -------------------
SystemMessage      →     (injected in first message)
HumanMessage       →     {"role": "user", "content": "..."}
AIMessage          →     {"role": "assistant", "content": "..."}
  with tool_calls  →     {"role": "assistant", "content": [
                             {"type": "tool_use", ...}
                          ]}
ToolMessage        →     {"role": "user", "content": [
                             {"type": "tool_result", ...}
                          ]}
```

### `core/react_agent/state.py`

**What it does:** Defines agent state structure.

**State evolution:**
```python
Initial:
  messages = []

After user input:
  messages = [HumanMessage("user query")]

After model reasoning:
  messages = [
    HumanMessage("user query"),
    AIMessage("reasoning", tool_calls=[...])
  ]

After tool execution:
  messages = [
    HumanMessage("user query"),
    AIMessage("reasoning", tool_calls=[...]),
    ToolMessage("tool result 1"),
    ToolMessage("tool result 2")
  ]

After final response:
  messages = [
    HumanMessage("user query"),
    AIMessage("reasoning", tool_calls=[...]),
    ToolMessage("tool result 1"),
    ToolMessage("tool result 2"),
    AIMessage("final answer")
  ]
```

### `core/valyu_tools/tools.py`

**What it does:** Provides search and content extraction tools.

**ValyuSearchTool:**
- Deep search across web and proprietary sources
- Configurable search parameters
- Returns structured results with relevance scores

**ValyuContentsTool:**
- Extracts clean content from URLs
- Removes boilerplate
- Returns structured content

## File Relationships

```
User Script/Notebook
    ↓ imports
core/react_agent/__init__.py
    ↓ exports
core/react_agent/create_agent.py
    ↓ uses
core/react_agent/utils.py (load_chat_model)
    ↓ creates
core/react_agent/holistic_ai_bedrock.py (HolisticAIBedrockChat)
    ↓ returns model
create_agent.py
    ↓ configures with
core/react_agent/context.py (Context)
core/react_agent/prompts.py (EXPERIMENT_SYSTEM_PROMPT)
core/react_agent/state.py (State, InputState)
    ↓ builds graph
LangGraph StateGraph
    ↓ with tools
core/valyu_tools/tools.py (ValyuSearchTool, etc.)
    ↓ returns
Compiled Agent (ready to invoke)
```

## Import Patterns

### From tutorials:
```python
# Import core modules
import sys
sys.path.insert(0, '../core')
from react_agent import create_react_agent
from valyu_tools import ValyuSearchTool

# Or import from installed packages
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.memory import MemorySaver
```

### From core:
```python
# Within core modules
from .state import State, InputState
from .context import Context
from .utils import load_chat_model
```

## Next Steps

- See [ARCHITECTURE.md](ARCHITECTURE.md) for system architecture
- See [WORKFLOW.md](WORKFLOW.md) for agent execution flow
- See [API_REFERENCE.md](API_REFERENCE.md) for API details
