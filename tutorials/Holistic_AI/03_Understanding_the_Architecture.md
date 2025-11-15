# Understanding the Architecture - How Holistic AI Works

Now that you've created your first agent, let's understand **how it actually works**. This tutorial explains the architecture and design of Holistic AI 2 in simple terms.

## Table of Contents
- [The Big Picture](#the-big-picture)
- [Core Components](#core-components)
- [How Components Work Together](#how-components-work-together)
- [The Graph-Based Execution Model](#the-graph-based-execution-model)
- [State Management](#state-management)
- [Message Types and Flow](#message-types-and-flow)
- [Tool Integration](#tool-integration)
- [Model Provider Layer](#model-provider-layer)
- [Real-World Analogy](#real-world-analogy)
- [Key Takeaways](#key-takeaways)

## The Big Picture

Holistic AI follows a **layered architecture**, where each layer has a specific responsibility:

```
┌─────────────────────────────────────────────────────────┐
│ Layer 5: YOUR APPLICATION                               │
│ What: Your Python script or Jupyter notebook            │
│ Role: Uses the agent to accomplish tasks                │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 4: AGENT FRAMEWORK (Holistic AI Core)            │
│ What: ReAct agent, tools, state management              │
│ Role: Orchestrates the agent's behavior                 │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 3: ORCHESTRATION (LangGraph)                      │
│ What: State graph, node execution, routing              │
│ Role: Controls the flow of execution                    │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 2: FOUNDATION (LangChain)                         │
│ What: Message abstractions, base classes                │
│ Role: Provides building blocks                          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 1: MODEL PROVIDER (AWS Bedrock / OpenAI)         │
│ What: Claude, GPT, Llama models                         │
│ Role: Provides AI intelligence                          │
└─────────────────────────────────────────────────────────┘
```

### Why This Architecture?

**Separation of Concerns**: Each layer focuses on one thing
- Makes code easier to understand
- Allows swapping components (e.g., different models)
- Simplifies debugging

**Modularity**: Components are independent
- Can test each piece separately
- Easy to extend with new features
- Reusable across projects

**Production-Ready**: Built for real-world use
- Error handling at each layer
- Performance optimization
- Monitoring and observability

## Core Components

Let's explore each major component:

### 1. ReAct Agent (`create_react_agent`)

**What it is**: The factory function that creates your agent.

**What it does**:
- Loads and configures the AI model
- Binds tools to the model
- Creates the state graph
- Compiles everything into a runnable agent

**Code location**: `core/react_agent/create_agent.py`

**Simple analogy**: Like a car factory that assembles all parts into a working vehicle.

```python
agent = create_react_agent(
    tools=[search_tool],      # What the agent can do
    model_name='claude-3-5-sonnet',  # Which AI to use
    checkpointer=None,         # Memory (optional)
    output_schema=None         # Structured output (optional)
)
```

### 2. State (`State` and `InputState`)

**What it is**: The agent's memory and context.

**What it contains**:
- All messages in the conversation
- Metadata about execution
- Flags like `is_last_step`

**Code location**: `core/react_agent/state.py`

**Simple analogy**: Like a notebook where the agent writes down everything that happens.

```python
@dataclass
class InputState:
    messages: List[Message]  # Conversation history

@dataclass
class State(InputState):
    is_last_step: bool = False  # Prevents infinite loops
```

### 3. Context (`Context`)

**What it is**: Configuration settings for the agent.

**What it contains**:
- System prompt (agent's personality/instructions)
- Model name
- Model parameters (temperature, timeout, etc.)

**Code location**: `core/react_agent/context.py`

**Simple analogy**: Like the settings menu in a game - controls how things behave.

```python
context = Context(
    model='claude-3-5-sonnet',
    system_prompt='You are a helpful assistant',
    max_search_results=10
)
```

### 4. Tools

**What they are**: Functions the agent can call to perform actions.

**Examples**:
- Search the web
- Extract content from URLs
- Send emails
- Query databases
- Perform calculations

**Code location**: `core/valyu_tools/tools.py` (built-in tools)

**Simple analogy**: Like apps on a smartphone - each does something specific.

```python
class MyTool(BaseTool):
    name = "my_tool"
    description = "What this tool does"
    
    def _run(self, input: str) -> str:
        # Tool logic here
        return result
```

### 5. Messages

**What they are**: The building blocks of conversation.

**Types**:
```python
# From the user
HumanMessage(content="What is AI?")

# From the agent
AIMessage(content="AI stands for...")

# From tools
ToolMessage(content="Search results: ...", tool_call_id="123")

# Instructions for the agent
SystemMessage(content="You are a helpful assistant")
```

**Simple analogy**: Like different types of messages in a chat app.

### 6. Model Wrapper (`HolisticAIBedrockChat`)

**What it is**: A bridge between Holistic AI and AWS Bedrock.

**What it does**:
- Converts messages to AWS format
- Handles authentication
- Manages tool calling
- Processes responses

**Code location**: `core/react_agent/holistic_ai_bedrock.py`

**Simple analogy**: Like a translator between two people who speak different languages.

## How Components Work Together

Let's trace what happens when you ask the agent a question:

### Step-by-Step Execution

```
1. YOU write code:
   agent.invoke({"messages": [HumanMessage("What's the weather?")]})
   
2. AGENT FRAMEWORK receives your message:
   - Creates State with your message
   - Starts graph execution
   
3. GRAPH enters "call_model" node:
   - Adds system prompt
   - Prepares messages
   
4. MODEL WRAPPER sends to AWS Bedrock:
   - Converts to Bedrock format
   - Includes authentication
   - Sends API request
   
5. CLAUDE (AI model) processes:
   - Understands the question
   - Decides if tools are needed
   - Generates response
   
6. MODEL WRAPPER receives response:
   - Converts back to LangChain format
   - Extracts tool calls (if any)
   
7. GRAPH makes routing decision:
   - Tool calls? → Go to "tools" node
   - No tools? → Go to END
   
8. IF TOOLS node:
   - Executes each tool
   - Creates ToolMessages with results
   - Returns to "call_model" node
   
9. AGENT FRAMEWORK returns result:
   - Full conversation in State
   - You extract the final message
```

### Visual Flow

```
Your Code
    ↓
┌───────────────────────┐
│ Agent Framework       │
│  ┌─────────────────┐ │
│  │  State Graph    │ │
│  │  ┌──────────┐   │ │
│  │  │call_model│   │ │
│  │  └────┬─────┘   │ │
│  │       ↓         │ │
│  │  ┌──────────┐   │ │
│  │  │  tools?  │   │ │
│  │  └─┬─────┬──┘   │ │
│  │    ↓     ↓      │ │
│  │  tools  END     │ │
│  └─────────────────┘ │
└───────┬───────────────┘
        ↓
┌───────────────────────┐
│ Model Wrapper         │
│  - Format conversion  │
│  - Authentication     │
│  - API calls          │
└───────┬───────────────┘
        ↓
┌───────────────────────┐
│ AWS Bedrock           │
│  - Claude Model       │
│  - Tool calling       │
│  - Response gen       │
└───────────────────────┘
```

## The Graph-Based Execution Model

Holistic AI uses **LangGraph** for execution control. Think of it as a flowchart:

### Basic Graph (No Tools)

```
START
  ↓
call_model
  ↓
END
```

### Graph with Tools

```
START
  ↓
call_model ←─────┐
  ↓              │
Decision         │
  ├→ tools ──────┘
  └→ END
```

### Graph with Structured Output

```
START
  ↓
call_model ←─────┐
  ↓              │
Decision         │
  ├→ tools ──────┘
  └→ format_output
       ↓
      END
```

### Understanding Nodes

**Nodes** are functions that process the state:

```python
def call_model(state: State) -> dict:
    """Node that calls the AI model"""
    # Add system prompt
    messages = [SystemMessage(...)] + state.messages
    
    # Call model
    response = model.invoke(messages)
    
    # Return update
    return {"messages": [response]}
```

**Key concept**: Nodes return dictionaries that update the state.

### Understanding Edges

**Edges** determine the flow:

**Unconditional edge**: Always goes to next node
```python
builder.add_edge("tools", "call_model")
# Always: tools → call_model
```

**Conditional edge**: Decision-based routing
```python
def route_decision(state: State):
    if state.messages[-1].tool_calls:
        return "tools"
    return "__end__"

builder.add_conditional_edges("call_model", route_decision)
# Decides: call_model → tools OR end
```

## State Management

The state is the **single source of truth** for the agent's execution.

### State Structure

```python
State = {
    "messages": [
        HumanMessage("Question"),
        AIMessage("Response")
    ],
    "is_last_step": False
}
```

### State Updates

State updates use the **reducer pattern**:

```python
# Current state
state = State(messages=[HumanMessage("Hi")])

# Node returns update
update = {"messages": [AIMessage("Hello!")]}

# Reducer merges them
# Result: messages = [HumanMessage("Hi"), AIMessage("Hello!")]
```

The `add_messages` reducer:
- Appends new messages
- Updates messages with the same ID
- Maintains order

### Why Immutable State?

Each state update creates a **new state object**:
- Enables time-travel debugging
- Makes execution predictable
- Allows checkpointing/replay

## Message Types and Flow

Understanding messages is crucial for working with Holistic AI.

### Message Flow in a Tool-Using Conversation

```
1. User Input
   HumanMessage(content="What's the weather in London?")
   
2. Agent Plans
   AIMessage(
       content="I'll check the weather",
       tool_calls=[{
           "name": "weather_tool",
           "args": {"city": "London"},
           "id": "call_123"
       }]
   )
   
3. Tool Executes
   ToolMessage(
       content='{"temp": 15, "conditions": "cloudy"}',
       tool_call_id="call_123"
   )
   
4. Agent Synthesizes
   AIMessage(
       content="The weather in London is 15°C and cloudy."
   )
```

### Message Attributes

All messages have:
- `content`: The text content
- `id`: Unique identifier
- `type`: Message type

Some messages have:
- `tool_calls`: List of tools to call (AIMessage)
- `tool_call_id`: Which tool call this responds to (ToolMessage)
- `additional_kwargs`: Extra metadata

## Tool Integration

Tools extend what the agent can do.

### Tool Definition

```python
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field

# Define input schema
class WeatherInput(BaseModel):
    city: str = Field(description="City name")

# Create tool
class WeatherTool(BaseTool):
    name = "get_weather"
    description = "Get current weather for a city"
    args_schema = WeatherInput
    
    def _run(self, city: str) -> str:
        # Call weather API
        result = call_weather_api(city)
        return f"Weather in {city}: {result}"
```

### Tool Binding

Tools are "bound" to the model:

```python
# Create model
model = get_chat_model('claude-3-5-sonnet')

# Bind tools
model_with_tools = model.bind_tools([weather_tool])

# Now the model knows about the tool
response = model_with_tools.invoke([HumanMessage("Weather in Paris?")])
# Response includes tool_calls
```

### Tool Execution

The `ToolNode` executes tools:

```python
# Agent has tool calls
tool_calls = [
    {"name": "get_weather", "args": {"city": "Paris"}, "id": "1"}
]

# ToolNode executes them
tool_node = ToolNode([weather_tool])
results = tool_node.invoke(state)

# Results are ToolMessages
# ToolMessage(content="Weather in Paris: 18°C, sunny", tool_call_id="1")
```

## Model Provider Layer

Holistic AI supports multiple AI models through a unified interface.

### Model Abstraction

All models implement the same interface:

```python
# Bedrock model
model = HolisticAIBedrockChat(
    team_id=...,
    api_token=...,
    model='claude-3-5-sonnet'
)

# OpenAI model
model = ChatOpenAI(model='gpt-5-mini')

# Both work the same way
response = model.invoke([HumanMessage("Hello")])
```

### Model Features

Different models have different capabilities:

| Feature | Claude | GPT-5 | Llama |
|---------|--------|-------|-------|
| Tool Calling | ✅ Native | ✅ Native | ✅ Native |
| Structured Output | ✅ Via API | ✅ Via API | ⚠️ Prompt-based |
| Streaming | ✅ Yes | ✅ Yes | ✅ Yes |
| Max Context | 200K tokens | 128K tokens | 128K tokens |

### Model Selection

Choose based on your needs:

**Claude 3.5 Sonnet**: 
- Best quality
- Strong reasoning
- Good with complex tasks

**Claude 3.5 Haiku**:
- Fast (1-2 sec)
- Cost-effective
- Good for simple tasks

**Llama 3.2 90B**:
- Open source
- No API lock-in
- Good performance

**Amazon Nova**:
- AWS native
- Low latency
- Competitive pricing

## Real-World Analogy

Think of Holistic AI like a **smart office assistant**:

**The Office (Your Application)**
- Where work gets done
- You give instructions to the assistant

**The Assistant (Agent Framework)**
- Understands your requests
- Knows which specialists to consult
- Coordinates everything

**The Specialists (Tools)**
- Internet researcher (search tool)
- Librarian (content extraction)
- Calculator (math tool)
- Secretary (email tool)

**The Brain (AI Model)**
- Claude, GPT, etc.
- Makes decisions
- Generates responses

**The Notebook (State)**
- Records everything
- Maintains context
- Tracks progress

**The Manager (LangGraph)**
- Decides who does what
- Controls the workflow
- Ensures tasks complete

## Key Takeaways

✅ **Layered Architecture**: Each layer has a specific role
✅ **Graph-Based Execution**: Flow is controlled by a state graph
✅ **State Management**: Immutable state updates via reducers
✅ **Message Flow**: Different message types for different purposes
✅ **Tool Integration**: Extends agent capabilities
✅ **Model Abstraction**: Unified interface for multiple AI models

## Understanding the Code Structure

Now let's see where everything lives:

```
core/
├── react_agent/
│   ├── create_agent.py      # Agent factory
│   ├── holistic_ai_bedrock.py  # Model wrapper
│   ├── state.py             # State definitions
│   ├── context.py           # Configuration
│   ├── prompts.py           # System prompts
│   └── utils.py             # Helper functions
│
└── valyu_tools/
    ├── tools.py             # Built-in tools
    └── retrievers.py        # Retrieval tools
```

## What's Next?

Now that you understand the architecture, you're ready to learn about:
- **ReAct Pattern**: How agents reason and act
- **Tool Development**: Creating custom tools
- **State Management**: Advanced state handling
- **Workflow Execution**: Detailed execution flow

**Continue to**: [04_ReAct_Agent_Explained.md](./04_ReAct_Agent_Explained.md)

## Additional Resources

- **Architecture Deep Dive**: [docs/ARCHITECTURE.md](../../docs/ARCHITECTURE.md)
- **Code Structure**: [docs/CODE_STRUCTURE.md](../../docs/CODE_STRUCTURE.md)
- **API Reference**: [docs/API_REFERENCE.md](../../docs/API_REFERENCE.md)

---

**Great progress!** 🎉 You now understand how Holistic AI is architected. This foundation will help you build more sophisticated agents in the upcoming tutorials!
