# API Reference

## Table of Contents
- [Core APIs](#core-apis)
- [Model APIs](#model-apis)
- [Tool APIs](#tool-apis)
- [State APIs](#state-apis)
- [Configuration APIs](#configuration-apis)

## Core APIs

### `create_react_agent()`

Create a LangGraph-based ReAct agent.

**Location:** `core/react_agent/create_agent.py`

**Signature:**
```python
def create_react_agent(
    tools: List[Callable[..., Any]],
    checkpointer: Optional[BaseCheckpointSaver] = None,
    context: Optional[Context] = None,
    model_name: Optional[str] = None,
    output_schema: Optional[Type[BaseModel]] = None,
    system_prompt: Optional[str] = None
) -> CompiledGraph
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `tools` | `List[Callable]` | Required | List of tool functions to provide to the agent |
| `checkpointer` | `BaseCheckpointSaver` | `None` | Optional checkpoint saver for conversation history. Pass `MemorySaver()` to enable conversation context |
| `context` | `Context` | `Context()` | Configuration object controlling agent behavior |
| `model_name` | `str` | `None` | Model identifier. Overrides `context.model` if provided |
| `output_schema` | `Type[BaseModel]` | `None` | Optional Pydantic schema for structured output. If None, returns natural text responses |
| `system_prompt` | `str` | `None` | Custom system prompt. Overrides default if provided |

**Returns:**
- `CompiledGraph`: Compiled LangGraph agent ready to invoke

**Examples:**

```python
# Basic agent
from react_agent import create_react_agent

agent = create_react_agent(tools=[])
result = agent.invoke({"messages": [("user", "Hello")]})

# With tools
from valyu_tools import ValyuSearchTool

agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

# With conversation memory
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver()
)

# With structured output
from pydantic import BaseModel

class Response(BaseModel):
    answer: str
    confidence: float

agent = create_react_agent(
    tools=[...],
    output_schema=Response
)
```

**Raises:**
- `ValueError`: If model credentials are missing or invalid
- `ImportError`: If required dependencies are not installed

---

## Model APIs

### `HolisticAIBedrockChat`

LangChain-compatible chat model for AWS Bedrock via Holistic AI proxy.

**Location:** `core/react_agent/holistic_ai_bedrock.py`

**Signature:**
```python
class HolisticAIBedrockChat(BaseChatModel):
    def __init__(
        self,
        team_id: str,
        api_token: SecretStr,
        model: str = "us.anthropic.claude-3-5-sonnet-20241022-v2:0",
        max_tokens: int = 1024,
        temperature: float = 0.7,
        timeout: int = 60
    )
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `team_id` | `str` | Required | Team ID for Holistic AI authentication |
| `api_token` | `SecretStr` | Required | API token for authentication |
| `model` | `str` | `"us.anthropic.claude-3-5-sonnet-20241022-v2:0"` | Bedrock model identifier |
| `max_tokens` | `int` | `1024` | Maximum tokens to generate |
| `temperature` | `float` | `0.7` | Sampling temperature (0.0-1.0) |
| `timeout` | `int` | `60` | Request timeout in seconds |

**Methods:**

#### `bind_tools(tools: List[Any]) -> HolisticAIBedrockChat`
Bind tools to the model for native tool calling.

**Parameters:**
- `tools`: List of tool objects

**Returns:** New model instance with tools bound

**Example:**
```python
from pydantic import SecretStr

model = HolisticAIBedrockChat(
    team_id="your-team-id",
    api_token=SecretStr("your-token"),
    model="us.anthropic.claude-3-5-sonnet-20241022-v2:0"
)

model_with_tools = model.bind_tools([search_tool, calculator_tool])
```

#### `with_structured_output(schema: Type[BaseModel]) -> HolisticAIBedrockStructuredOutput`
Create a model that returns structured output matching the schema.

**Parameters:**
- `schema`: Pydantic model class defining output structure

**Returns:** Structured output wrapper

**Example:**
```python
from pydantic import BaseModel

class Answer(BaseModel):
    response: str
    confidence: float

structured_model = model.with_structured_output(Answer)
result = structured_model.invoke("What is AI?")
# result is an Answer instance
```

### `get_chat_model()`

Factory function for creating chat models.

**Location:** `core/react_agent/holistic_ai_bedrock.py`

**Signature:**
```python
def get_chat_model(
    model_name: str = "claude-3-5-sonnet",
    use_openai: bool = False,
    **kwargs
) -> BaseChatModel
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model_name` | `str` | `"claude-3-5-sonnet"` | Model identifier (short name or full Bedrock ID) |
| `use_openai` | `bool` | `False` | If True, use OpenAI instead of Bedrock |
| `**kwargs` | `Any` | - | Additional model parameters |

**Supported Models:**

**Bedrock Models (default):**
- `claude-3-5-sonnet` → `us.anthropic.claude-3-5-sonnet-20241022-v2:0`
- `claude-3-5-haiku` → `us.anthropic.claude-3-5-haiku-20241022-v1:0`
- `llama3-2-90b` → `us.meta.llama3-2-90b-instruct-v1:0`
- `nova-pro` → `us.amazon.nova-pro-v1:0`
- `nova-lite` → `us.amazon.nova-lite-v1:0`

**OpenAI Models (use_openai=True):**
- `gpt-5-nano`, `gpt-5-mini`, `gpt-5`

**Examples:**
```python
# Bedrock (default)
model = get_chat_model("claude-3-5-sonnet")

# OpenAI (alternative)
model = get_chat_model("gpt-5-mini", use_openai=True)

# With custom parameters
model = get_chat_model(
    "claude-3-5-haiku",
    temperature=0.5,
    max_tokens=2048
)
```

**Raises:**
- `ValueError`: If credentials are not set or model is invalid

---

## Tool APIs

### `ValyuSearchTool`

Deep search tool for web and proprietary sources.

**Location:** `core/valyu_tools/tools.py`

**Signature:**
```python
class ValyuSearchTool(BaseTool):
    name: str = "valyu_deep_search"
    description: str = "Search proprietary and web sources"
    args_schema: Type[BaseModel] = ValyuToolInput
    valyu_api_key: Optional[str] = None
```

**Input Schema:**

```python
class ValyuToolInput(BaseModel):
    query: str                              # Search query (required)
    search_type: str = "all"                # 'all', 'proprietary', or 'web'
    max_num_results: int = 10               # Maximum results (1-20)
    relevance_threshold: float = 0.5        # Min relevance score (0.0-1.0)
    max_price: float = 50.0                 # Max cost in dollars
    is_tool_call: bool = True               # Optimize for LLM consumption
    start_date: Optional[str] = None        # YYYY-MM-DD format
    end_date: Optional[str] = None          # YYYY-MM-DD format
    included_sources: Optional[List[str]] = None  # URLs/domains to include
    excluded_sources: Optional[List[str]] = None  # URLs/domains to exclude
    response_length: Optional[Union[int, str]] = None  # 'short', 'medium', 'large', 'max'
    country_code: Optional[str] = None      # 2-letter ISO code (e.g., 'GB')
    fast_mode: bool = False                 # Faster but shorter results
```

**Output:**
```python
{
    "results": [
        {
            "title": str,
            "url": str,
            "content": str,
            "relevance_score": float,
            "source_type": str
        },
        ...
    ],
    "metadata": {
        "total_results": int,
        "search_time": float,
        "cost": float
    }
}
```

**Examples:**
```python
# Basic usage
tool = ValyuSearchTool()
result = tool._run(query="quantum computing")

# Advanced usage
result = tool._run(
    query="AI breakthroughs 2024",
    search_type="web",
    max_num_results=5,
    start_date="2024-01-01",
    end_date="2024-12-31",
    country_code="US",
    fast_mode=True
)
```

### `ValyuContentsTool`

Extract clean content from web pages.

**Location:** `core/valyu_tools/tools.py`

**Signature:**
```python
class ValyuContentsTool(BaseTool):
    name: str = "valyu_contents_extract"
    description: str = "Extract clean content from web pages"
    args_schema: Type[BaseModel] = ValyuContentsToolInput
    valyu_api_key: Optional[str] = None
    
    # Configuration
    summary: Optional[Union[bool, str]] = None
    extract_effort: Optional[Literal["normal", "high", "auto"]] = "normal"
    response_length: Optional[Union[int, str]] = "short"
```

**Input Schema:**
```python
class ValyuContentsToolInput(BaseModel):
    urls: List[str]  # List of URLs (max 10 per request)
```

**Output:**
```python
{
    "contents": [
        {
            "url": str,
            "title": str,
            "content": str,
            "author": Optional[str],
            "publish_date": Optional[str],
            "summary": Optional[str]
        },
        ...
    ]
}
```

**Examples:**
```python
tool = ValyuContentsTool(
    summary=True,
    extract_effort="high",
    response_length="medium"
)

result = tool._run(urls=[
    "https://example.com/article1",
    "https://example.com/article2"
])
```

### Custom Tool Creation

**Example:**
```python
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

class CalculatorInput(BaseModel):
    expression: str = Field(description="Mathematical expression to evaluate")

class CalculatorTool(BaseTool):
    name: str = "calculator"
    description: str = "Evaluate mathematical expressions"
    args_schema: Type[BaseModel] = CalculatorInput
    
    def _run(self, expression: str) -> str:
        try:
            result = eval(expression)  # Use safely in production!
            return f"Result: {result}"
        except Exception as e:
            return f"Error: {str(e)}"

# Use in agent
agent = create_react_agent(tools=[CalculatorTool()])
```

---

## State APIs

### `InputState`

Basic agent state with messages.

**Location:** `core/react_agent/state.py`

**Definition:**
```python
@dataclass
class InputState:
    messages: Annotated[Sequence[AnyMessage], add_messages] = field(
        default_factory=list
    )
```

**Fields:**
- `messages`: Sequence of conversation messages with `add_messages` reducer

**Message Types:**
- `HumanMessage`: User input
- `AIMessage`: Agent response
- `ToolMessage`: Tool execution result
- `SystemMessage`: System instructions

### `State`

Extended agent state with step tracking.

**Location:** `core/react_agent/state.py`

**Definition:**
```python
@dataclass
class State(InputState):
    is_last_step: IsLastStep = field(default=False)
```

**Fields:**
- Inherits `messages` from `InputState`
- `is_last_step`: Boolean indicating if this is the final step before recursion limit

**Usage:**
```python
# State is managed automatically by LangGraph
# Access in custom nodes:
def my_node(state: State):
    messages = state.messages
    is_last = state.is_last_step
    # ... node logic
    return {"messages": [new_message]}
```

---

## Configuration APIs

### `Context`

Configuration dataclass for agent behavior.

**Location:** `core/react_agent/context.py`

**Definition:**
```python
@dataclass(kw_only=True)
class Context:
    system_prompt: str = EXPERIMENT_SYSTEM_PROMPT
    model: str = "claude-3-5-sonnet"
    max_search_results: int = 10
    ollama_temperature: float = 0.1
    ollama_timeout: int = 60
    ollama_num_predict: int = 256
```

**Fields:**

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `system_prompt` | `str` | `EXPERIMENT_SYSTEM_PROMPT` | Agent's system prompt defining behavior |
| `model` | `str` | `"claude-3-5-sonnet"` | Model identifier (short name or full ID) |
| `max_search_results` | `int` | `10` | Maximum search results per query |
| `ollama_temperature` | `float` | `0.1` | Temperature for Ollama models |
| `ollama_timeout` | `int` | `60` | Timeout for Ollama requests (seconds) |
| `ollama_num_predict` | `int` | `256` | Maximum tokens for Ollama models |

**Examples:**
```python
from react_agent import Context

# Custom configuration
context = Context(
    model="claude-3-5-haiku",
    max_search_results=5,
    system_prompt="You are a helpful research assistant."
)

agent = create_react_agent(
    tools=[...],
    context=context
)
```

### System Prompts

**Location:** `core/react_agent/prompts.py`

**Available Prompts:**

#### `SYSTEM_PROMPT`
```python
SYSTEM_PROMPT = """You are a helpful AI assistant.

System time: {system_time}"""
```

#### `EXPERIMENT_SYSTEM_PROMPT`
```python
EXPERIMENT_SYSTEM_PROMPT = """You are a general ReAct agent that can solve multi-step tasks by planning, using tools, and producing clear results. You have access to multiple registered tools; their names, descriptions, and argument schemas are provided to you.

Instructions:
1. Clarify or decompose the task if needed; plan minimal steps.
2. Use tools when they materially improve correctness or efficiency.
3. Ground factual claims in retrieved information and avoid hallucinations.
4. Only call send_email if the user explicitly asks to send an email and a valid recipient is provided.
5. Otherwise, produce a final answer with clear, actionable steps."""
```

**Usage:**
```python
from react_agent import Context
from react_agent.prompts import EXPERIMENT_SYSTEM_PROMPT

context = Context(system_prompt=EXPERIMENT_SYSTEM_PROMPT)
```

---

## Invocation API

### Agent Invocation

**Methods:**

#### `invoke(input: dict, config: Optional[dict] = None) -> dict`
Execute agent synchronously.

**Parameters:**
- `input`: Dictionary with `"messages"` key containing list of messages
- `config`: Optional configuration for checkpointer (thread_id, etc.)

**Returns:** Dictionary with `"messages"` key containing full conversation

**Example:**
```python
result = agent.invoke({
    "messages": [HumanMessage(content="Hello")]
})

print(result["messages"][-1].content)
```

#### `stream(input: dict, config: Optional[dict] = None) -> Iterator[dict]`
Execute agent with streaming updates.

**Parameters:**
- `input`: Dictionary with `"messages"` key
- `config`: Optional configuration

**Returns:** Iterator yielding state updates

**Example:**
```python
for chunk in agent.stream({"messages": [HumanMessage("Hello")]}):
    print(chunk)
```

#### `astream(input: dict, config: Optional[dict] = None) -> AsyncIterator[dict]`
Async version of stream.

**Example:**
```python
async for chunk in agent.astream({"messages": [HumanMessage("Hello")]}):
    print(chunk)
```

### Checkpointer Configuration

**With Memory:**
```python
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver()
)

# Invoke with thread ID
config = {"configurable": {"thread_id": "conversation_1"}}
result = agent.invoke({"messages": [...]}, config)
```

---

## Environment Variables

**Required for Holistic AI Bedrock:**
```bash
HOLISTIC_AI_TEAM_ID=your-team-id
HOLISTIC_AI_API_TOKEN=your-token
```

**Optional:**
```bash
# OpenAI (alternative)
OPENAI_API_KEY=sk-your-key

# Valyu Search
VALYU_API_KEY=your-valyu-key

# LangSmith Observability
LANGSMITH_API_KEY=your-langsmith-key
LANGSMITH_PROJECT=hackathon-2025
LANGSMITH_TRACING=true

# Ollama (local models)
OLLAMA_BASE_URL=http://localhost:11434
```

---

## Error Handling

**Common Exceptions:**

```python
# Missing credentials
ValueError: "HOLISTIC_AI_TEAM_ID and HOLISTIC_AI_API_TOKEN not set"

# API errors
ValueError: "Error calling Holistic AI Bedrock API: [error details]"

# Model not found
ValueError: "Model 'invalid-model' not found"

# Tool errors
# Tools return error messages as strings
ToolMessage(content="Error: [tool error details]")

# Recursion limit
# When is_last_step=True and tool_calls present:
AIMessage(content="Sorry, could not complete in specified steps.")
```

---

## Next Steps

- See [ARCHITECTURE.md](ARCHITECTURE.md) for system design
- See [WORKFLOW.md](WORKFLOW.md) for execution flow
- See [GETTING_STARTED.md](GETTING_STARTED.md) for setup guide
