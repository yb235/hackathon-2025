# Architecture Overview

## Table of Contents
- [System Overview](#system-overview)
- [Core Components](#core-components)
- [Technology Stack](#technology-stack)
- [Agent Architecture](#agent-architecture)
- [Data Flow](#data-flow)
- [AWS Integration](#aws-integration)

## System Overview

The Hackathon 2025 repository provides a comprehensive framework for building production-grade AI agents using **LangGraph**, a library for building stateful, multi-actor applications with LLMs. The system is designed specifically for the Holistic AI x UCL Hackathon 2025, focusing on three key areas:

1. **Agent Iron Man (Track A)**: Building reliable, robust, and cost-efficient agents
2. **Agent Glass Box (Track B)**: Building observable and explainable agents
3. **Dear Grandma (Track C)**: Security testing and red teaming of agents

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       User Application                          │
│                    (Jupyter Notebooks/Scripts)                  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Agent Framework Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  LangGraph   │  │  LangChain   │  │  Custom      │         │
│  │  ReAct Agent │  │  Components  │  │  Tools       │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Model Provider Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Holistic AI  │  │   OpenAI     │  │   Ollama     │         │
│  │   Bedrock    │  │   GPT-5      │  │  (Local)     │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    External Services Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │    Valyu     │  │  LangSmith   │  │ CodeCarbon   │         │
│  │   Search     │  │   Tracing    │  │  Monitoring  │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. ReAct Agent Framework (`core/react_agent/`)

The heart of the system is a **ReAct (Reasoning + Acting)** agent implementation built on LangGraph. ReAct agents follow a loop:

1. **Reason**: Analyze the current state and decide what to do
2. **Act**: Execute an action (call a tool or respond to user)
3. **Observe**: Process the results
4. **Repeat**: Continue until the task is complete

**Key Files:**
- `create_agent.py`: Factory function for creating ReAct agents
- `state.py`: Defines the agent's state structure
- `context.py`: Configuration and context management
- `holistic_ai_bedrock.py`: AWS Bedrock API integration
- `prompts.py`: System prompts for agent behavior
- `utils.py`: Model loading and initialization utilities

### 2. State Management

The agent uses a **dataclass-based state** system:

```python
@dataclass
class InputState:
    messages: Annotated[Sequence[AnyMessage], add_messages]
    # Messages follow a pattern:
    # 1. HumanMessage - user input
    # 2. AIMessage with tool_calls - agent choosing tools
    # 3. ToolMessage(s) - tool execution results
    # 4. AIMessage without tool_calls - final response

@dataclass
class State(InputState):
    is_last_step: IsLastStep  # Prevents infinite loops
```

**State Flow:**
- Messages accumulate in append-only fashion
- Each message has a unique ID for updates
- Tool calls are tracked through message metadata
- State persists across the entire conversation

### 3. Graph-Based Execution

LangGraph uses a **state graph** to orchestrate agent execution:

```
[START] → [call_model] → [Decision]
                            ├→ [tools] → [call_model] (loop)
                            └→ [format_output] → [END]
```

**Nodes:**
- `call_model`: Invokes the LLM with current state
- `tools`: Executes tool calls from the model
- `format_output`: (Optional) Formats responses as structured JSON

**Edges:**
- Conditional routing based on model output
- Tool calls trigger the tools node
- No tool calls proceed to output formatting or end

### 4. Tool System

Tools are Python functions that extend agent capabilities:

**Built-in Tools:**
- `ValyuSearchTool`: Deep web search with proprietary sources
- `ValyuContentsTool`: Extract clean content from web pages

**Custom Tools:**
Users can create custom tools by:
1. Defining a Pydantic schema for inputs
2. Implementing a `_run()` method
3. Inheriting from `BaseTool`

Example:
```python
class MyTool(BaseTool):
    name: str = "my_tool"
    description: str = "What this tool does"
    args_schema: Type[BaseModel] = MyInputSchema
    
    def _run(self, param1: str) -> dict:
        # Tool implementation
        return {"result": "..."}
```

## Technology Stack

### Core Frameworks
- **LangGraph** (>=0.2.0): Agent orchestration and state management
- **LangChain Core** (>=0.3.0): Base abstractions for LLM applications
- **LangChain OpenAI** (>=0.3.0): OpenAI model integrations
- **Pydantic** (>=2.0.0): Data validation and structured output

### Model Providers
1. **Holistic AI Bedrock Proxy** (Recommended)
   - Managed AWS Bedrock access for hackathon participants
   - Models: Claude 3.5, Llama 3.2, Amazon Nova, Mistral
   - Native tool calling support

2. **OpenAI** (Optional)
   - GPT-5 series: gpt-5-nano, gpt-5-mini, gpt-5
   - Structured output support

3. **Ollama** (Local Development)
   - Open-weight models
   - Local deployment option

### Observability & Monitoring
- **LangSmith** (>=0.3.0): Execution tracing and debugging
- **CodeCarbon** (>=2.3.0): Carbon footprint tracking
- **TikToken** (>=0.5.0): Token counting

### Development Tools
- **Jupyter** (>=1.0.0): Interactive notebooks
- **Python-dotenv** (>=1.0.0): Environment configuration

## Agent Architecture

### 1. Message Flow

```
User Input (HumanMessage)
    ↓
[Agent Reasoning]
    ↓
Tool Calls (AIMessage with tool_calls)
    ↓
Tool Execution (ToolMessage)
    ↓
[Agent Synthesis]
    ↓
Final Response (AIMessage without tool_calls)
```

### 2. Model Binding

The system supports **native tool calling** for compatible models:

**Supported Models:**
- AWS Bedrock: Claude, Llama, Nova, Mistral
- OpenAI: GPT-5 series
- Ollama: gpt-oss, qwen3 variants

**Fallback Mode:**
Models without native tool calling use prompt-based tool selection.

### 3. Structured Output (Optional)

Agents can return validated JSON using Pydantic schemas:

```python
class MyResponse(BaseModel):
    answer: str
    confidence: float
    sources: List[str]

agent = create_react_agent(
    tools=[...],
    output_schema=MyResponse  # Enable structured output
)
```

When enabled, a `format_output` node is added to the graph to convert free-form responses into validated JSON.

### 4. Checkpointing (Optional)

Agents support **conversation persistence** through checkpointers:

```python
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver()  # Enable conversation history
)
```

Default behavior is **stateless** for performance. Enable checkpointing when conversation context is needed.

## Data Flow

### 1. Request Flow

```
1. User creates agent with tools and configuration
2. User invokes agent with initial message
3. Agent enters call_model node
   - System prompt injected with current timestamp
   - Messages converted to model format
   - Model invoked with tools bound
4. Model responds with tool calls or final answer
5. If tool calls:
   - Tools node executes each tool
   - Results added as ToolMessages
   - Loop back to call_model
6. If no tool calls:
   - (Optional) format_output node structures response
   - End state reached
7. Results returned to user
```

### 2. State Updates

State updates follow **reducer pattern**:
- `add_messages` reducer merges new messages
- Messages with same ID update existing entries
- State is immutable - each update creates new state object

### 3. Error Handling

The system includes built-in safeguards:
- `is_last_step`: Prevents infinite loops
- Tool timeout handling
- Model API error handling
- Graceful degradation on failures

## AWS Integration

### 1. Holistic AI Bedrock Proxy

Custom wrapper for AWS Bedrock API:

**Features:**
- Team-based authentication
- Model routing and configuration
- Tool calling format conversion (LangChain → Claude format)
- Structured output support via response_format
- Error handling and retry logic

**Authentication:**
```python
# Environment variables
HOLISTIC_AI_TEAM_ID=your-team-id
HOLISTIC_AI_API_TOKEN=your-token

# Automatic initialization
model = HolisticAIBedrockChat(
    team_id=os.getenv("HOLISTIC_AI_TEAM_ID"),
    api_token=SecretStr(os.getenv("HOLISTIC_AI_API_TOKEN")),
    model="us.anthropic.claude-3-5-sonnet-20241022-v2:0"
)
```

### 2. Model Format Conversion

The system automatically converts between LangChain and AWS formats:

**LangChain → AWS Bedrock:**
- `SystemMessage` → System prompt injection
- `HumanMessage` → User role
- `AIMessage` with tool_calls → Assistant role with tool_use blocks
- `ToolMessage` → User role with tool_result blocks

**AWS Bedrock → LangChain:**
- Text content → AIMessage content
- Tool_use blocks → AIMessage tool_calls
- Stop reason → Response metadata

### 3. AWS Service Integration

Additional AWS services available:

**AWS Strands Agents SDK:**
- Multi-agent workflows
- MCP server integration
- Bedrock AgentCore deployment

**AWS Observability:**
- CloudWatch: Logs and metrics
- X-Ray: Distributed tracing
- Bedrock Monitoring: Token usage and costs

### 4. Production Deployment

For production use:
- Deploy to AWS Lambda or ECS
- Use Bedrock AgentCore for scaling
- Integrate CloudWatch for monitoring
- Use X-Ray for distributed tracing

---

## Architecture Benefits

1. **Modularity**: Components are loosely coupled and replaceable
2. **Extensibility**: Easy to add new tools, models, and capabilities
3. **Observability**: Built-in tracing and monitoring
4. **Scalability**: Designed for production deployment
5. **Flexibility**: Multiple model providers and deployment options
6. **Safety**: Error handling and loop prevention
7. **Performance**: Stateless by default, optional persistence

## Next Steps

- See [CODE_STRUCTURE.md](CODE_STRUCTURE.md) for detailed code organization
- See [WORKFLOW.md](WORKFLOW.md) for agent lifecycle and execution flow
- See [API_REFERENCE.md](API_REFERENCE.md) for API documentation
