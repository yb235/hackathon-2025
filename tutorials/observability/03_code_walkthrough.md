# Tutorial 3: Code Walkthrough - Deep Dive

## 📖 Overview

**What You'll Learn:**
- Detailed code explanation of core observability components
- Line-by-line analysis of key files
- How observability is implemented in the codebase
- Understanding the ReAct agent implementation
- Tool system and state management internals

**Prerequisites:** 
- [Tutorial 1: What is Observability?](01_what_is_observability.md)
- [Tutorial 2: Architecture Deep Dive](02_architecture_deep_dive.md)
- Basic Python knowledge

**Time to Complete:** 45 minutes

**Difficulty:** ⭐⭐⭐ Advanced

---

## 📂 Repository Structure

```
hackathon-2025/
├── core/
│   ├── react_agent/          ← We'll focus here
│   │   ├── create_agent.py   ← Main agent factory
│   │   ├── state.py          ← State management
│   │   ├── context.py        ← Configuration
│   │   ├── prompts.py        ← System prompts
│   │   ├── utils.py          ← Model loading
│   │   └── holistic_ai_bedrock.py ← AWS integration
│   └── valyu_tools/          ← Tool implementations
├── tutorials/
│   └── 05_observability.ipynb ← Hands-on tutorial
└── docs/
    ├── ARCHITECTURE.md
    └── CODE_STRUCTURE.md
```

---

## 🔍 File 1: `create_agent.py` - The Heart of the Agent

**Location**: `core/react_agent/create_agent.py`

This file contains the `create_react_agent()` function that builds observable agents.

### Complete Function Signature

```python
def create_react_agent(
    tools: List[Callable[..., Any]],
    checkpointer: Optional[BaseCheckpointSaver] = None,
    context: Optional[Context] = None,
    model_name: Optional[str] = None,
    output_schema: Optional[Type[BaseModel]] = None,
    system_prompt: Optional[str] = None
):
    """Create ReAct agent with optional structured output"""
```

### Line-by-Line Breakdown

#### **Part 1: Setup and Configuration**

```python
# Setup context
if context is None:
    context = Context()
```
**What it does**: Creates default configuration if none provided.
**Observability**: Context includes model name, max_search_results, etc. that appear in trace metadata.

```python
if model_name is not None:
    context.model = model_name
```
**What it does**: Allows overriding model without creating new Context.
**Observability**: Model name is crucial metadata - appears in every trace.

```python
if system_prompt is not None:
    context.system_prompt = system_prompt
```
**What it does**: Custom system prompt override.
**Observability**: System prompt is logged in traces, helps understand agent behavior.

#### **Part 2: Structured Output Setup**

```python
use_structured_output = output_schema is not None
if use_structured_output:
    print(f"✓ Using structured output: {output_schema.__name__}")
```
**What it does**: Detects if structured output is needed.
**Observability**: This flag determines graph structure (adds format_output node).
**Trace impact**: Adds an extra span when enabled.

#### **Part 3: Model Loading**

```python
from .utils import load_chat_model
model = load_chat_model(context.model, context)
model = model.bind(stream=False)
```
**What it does**: 
- `load_chat_model()`: Factory function that creates appropriate model (Bedrock/OpenAI/Ollama)
- `bind(stream=False)`: Disables streaming for simplicity

**Observability**: Model instance includes hooks for LangSmith tracing.
- Every `model.invoke()` call automatically traced
- Token usage, latency, model name all captured

#### **Part 4: Tool Binding**

```python
if tools:
    # Check if model supports native tool calling
    gpt5_models = ['gpt-5-nano', 'gpt-5-mini', 'gpt-5']
    bedrock_models = ['claude', 'llama', 'nova', 'mistral']
    
    model_supports_tools = (
        context.model in gpt5_models or 
        context.model.startswith('gpt-oss') or 
        any(bedrock_model in context.model.lower() 
            for bedrock_model in bedrock_models)
    )
    
    if model_supports_tools:
        model = model.bind_tools(tools)
        print("✓ Native tool calling enabled")
    else:
        print("⚠ Fallback tool calling mode")
```

**What it does**: 
- Checks if model supports native tool calling
- Binds tools to model if supported
- Falls back to prompt-based tool selection otherwise

**Observability Impact**:
- Native tool calling: Tool calls appear as structured data in traces
- Fallback mode: Tool selection visible in model reasoning text

**Why it matters**: Native tool calling provides better trace structure and debugging.

#### **Part 5: Graph Construction**

```python
builder = StateGraph(State, input_schema=InputState, context_schema=Context)
```

**What it does**: Creates state graph builder.

**Parameters**:
- `State`: Full state structure (messages + is_last_step)
- `InputState`: Input-only structure (just messages)
- `Context`: Configuration schema

**Observability**: LangGraph automatically traces graph execution:
- Every node entry/exit
- Every edge traversal
- State at each step

#### **Part 6: call_model Node**

This is the core reasoning node. Let's break it down in detail:

```python
def call_model(state: State, *, runtime=None):
    """Call the model with tools."""
    from datetime import UTC, datetime
    from langchain_core.messages import AIMessage, SystemMessage
    
    # Get context (runtime or default)
    context_to_use = runtime.context if (runtime and runtime.context) else context
    
    # Create system message with timestamp
    system_message = context_to_use.system_prompt.format(
        system_time=datetime.now(tz=UTC).isoformat()
    )
```

**What it does**:
1. Gets context (runtime or default)
2. Formats system prompt with current timestamp

**Why timestamp**: Helps agent understand "now" for time-sensitive queries.

**Observability**: System message appears in trace, shows prompt used.

```python
    # Prepare messages
    messages = [SystemMessage(content=system_message)] + state.messages
    
    # Call model
    response = model.invoke(messages)
```

**What it does**: 
- Adds system message to conversation
- Invokes model with full message history

**Observability**: 
- **Automatic tracing**: This `invoke()` creates a trace span
- **Captured data**: 
  - Input messages (all of them)
  - Output response
  - Token counts (input + output)
  - Latency
  - Model parameters

**Example trace span**:
```json
{
  "name": "model_invoke",
  "duration_ms": 2300,
  "inputs": {
    "messages": [
      {"role": "system", "content": "You are..."},
      {"role": "user", "content": "What is AI?"}
    ]
  },
  "outputs": {
    "content": "AI is...",
    "tokens_used": 1234
  }
}
```

```python
    # Handle is_last_step (loop prevention)
    if state.is_last_step and response.tool_calls:
        # Override tool calls, return warning
        return {
            "messages": [AIMessage(
                content="Sorry, I could not complete the task in time.",
                id=response.id
            )]
        }
    
    return {"messages": [response]}
```

**What it does**: 
- Checks if we're at recursion limit
- If yes and model wants more tools, override with error message
- Otherwise return response normally

**Observability**: 
- Loop prevention events visible in trace
- Shows why agent stopped if hit limit

**Why it matters**: Prevents infinite loops in production.

#### **Part 7: Tool Execution Node**

```python
tool_node = ToolNode(tools)
```

**What it does**: Creates tool execution node.

**ToolNode internals**:
```python
# Simplified view of what ToolNode does
class ToolNode:
    def __call__(self, state):
        tool_messages = []
        
        for tool_call in state.messages[-1].tool_calls:
            # Find tool by name
            tool = self.find_tool(tool_call["name"])
            
            # Execute tool
            try:
                result = tool._run(**tool_call["args"])
                status = "success"
            except Exception as e:
                result = f"Error: {str(e)}"
                status = "error"
            
            # Create tool message
            tool_messages.append(
                ToolMessage(
                    content=result,
                    tool_call_id=tool_call["id"],
                    status=status
                )
            )
        
        return {"messages": tool_messages}
```

**Observability**: 
- Each tool execution creates a span
- Tool inputs, outputs, errors all captured
- Execution time tracked
- Tool name tagged in trace

**Example tool trace**:
```json
{
  "name": "tool_execution",
  "duration_ms": 5100,
  "tool_name": "valyu_deep_search",
  "inputs": {"query": "latest npm updates"},
  "outputs": {"results": [...]},
  "status": "success"
}
```

#### **Part 8: Graph Assembly**

```python
# Add nodes
builder.add_node("call_model", call_model, metadata={"target": "call_model"})
builder.add_node("tools", tool_node)

# Add edges
builder.add_edge("__start__", "call_model")

def should_continue(state: State):
    """Route based on tool calls"""
    last_message = state.messages[-1]
    if last_message.tool_calls:
        return "tools"  # Has tool calls, go to tools
    else:
        return "__end__"  # No tool calls, we're done

builder.add_conditional_edges("call_model", should_continue)
builder.add_edge("tools", "call_model")
```

**What it does**:
1. Adds call_model node
2. Adds tools node
3. Start → call_model
4. call_model → tools (if tool_calls) OR end (if no tool_calls)
5. tools → call_model (loop back)

**Observability**: 
- Each edge traversal logged
- Routing decisions visible
- Can see why agent took certain path

**Routing trace example**:
```
Routing decision at call_model:
  Condition: should_continue
  State: messages[-1] has tool_calls
  Decision: Route to 'tools'
```

#### **Part 9: Structured Output (Optional)**

```python
if use_structured_output:
    # Add format_output node before end
    def format_output(state: State, *, runtime=None):
        # ... formatting logic ...
        pass
    
    builder.add_node("format_output", format_output)
    # Modify routing to include format_output
```

**What it does**: Adds a node to convert free-text to structured JSON.

**Observability**: 
- Adds extra span for formatting
- Shows input text and output JSON
- Validation errors visible if JSON doesn't match schema

#### **Part 10: Compilation**

```python
agent = builder.compile(checkpointer=checkpointer)
return agent
```

**What it does**: Compiles graph into executable agent.

**Observability**: 
- Compiled agent includes all instrumentation
- Ready to produce traces on invoke()

---

## 🗂️ File 2: `state.py` - State Management

**Location**: `core/react_agent/state.py`

### Complete Code with Explanations

```python
from dataclasses import dataclass
from typing import Annotated, Sequence
from langchain_core.messages import AnyMessage
from langgraph.graph import add_messages

@dataclass
class InputState:
    """Input state schema for the agent"""
    messages: Annotated[Sequence[AnyMessage], add_messages]
```

**What it does**:
- Defines input-only state structure
- `messages`: List of conversation messages
- `Annotated[..., add_messages]`: Special type that tells LangGraph how to merge messages

**add_messages reducer**:
```python
# Conceptual implementation
def add_messages(existing, new):
    """Merge message lists"""
    result = list(existing)
    for msg in new:
        # If message with same ID exists, update it
        # Otherwise append new message
        if msg.id in [m.id for m in result]:
            # Update existing
            result = [msg if m.id == msg.id else m for m in result]
        else:
            # Append new
            result.append(msg)
    return result
```

**Observability**: 
- Every message addition is a state change
- LangSmith logs each state transition
- Can replay entire conversation

```python
@dataclass
class State(InputState):
    """Full state schema including internal tracking"""
    is_last_step: IsLastStep = False
```

**What it does**:
- Extends InputState with internal fields
- `is_last_step`: Boolean flag for loop prevention

**IsLastStep**:
```python
# From LangGraph
class IsLastStep(bool):
    """Special bool that's set by graph executor"""
    pass
```

**Observability**: 
- `is_last_step` changes visible in trace
- Shows when agent hits recursion limit

---

## 🔧 File 3: `holistic_ai_bedrock.py` - AWS Integration

**Location**: `core/react_agent/holistic_ai_bedrock.py`

This file is complex - let's focus on observability-relevant parts.

### Key Class: HolisticAIBedrockChat

```python
class HolisticAIBedrockChat(BaseChatModel):
    """LangChain-compatible chat model for Holistic AI Bedrock"""
    
    team_id: str
    api_token: SecretStr
    model: str
    temperature: float = 0.7
    max_tokens: int = 4096
    # ... more fields ...
```

**What it does**: Wraps AWS Bedrock API to look like a LangChain model.

**Observability benefit**: Being a BaseChatModel means automatic LangSmith integration.

### _generate Method (Core Logic)

```python
def _generate(self, messages, stop=None, run_manager=None, **kwargs):
    """Generate response from messages"""
    
    # Convert messages to Bedrock format
    bedrock_messages = self._convert_messages_to_bedrock(messages)
    
    # Prepare request
    payload = {
        "anthropic_version": "bedrock-2023-05-31",
        "messages": bedrock_messages,
        "max_tokens": self.max_tokens,
        "temperature": self.temperature,
    }
    
    # Add tools if present
    if self.tools:
        payload["tools"] = self._format_tools_for_bedrock(self.tools)
    
    # Call API
    start_time = time.time()
    response = requests.post(
        f"{self.api_base_url}/v1/chat/completions",
        json=payload,
        headers={"Authorization": f"Bearer {self.api_token.get_secret_value()}"}
    )
    latency = time.time() - start_time
    
    # Parse response
    response_data = response.json()
    
    # Extract usage info
    usage = response_data.get("usage", {})
    input_tokens = usage.get("input_tokens", 0)
    output_tokens = usage.get("output_tokens", 0)
    
    # Create LangChain response
    message = self._convert_bedrock_to_message(response_data)
    
    # Create generation with metadata
    generation = ChatGeneration(
        message=message,
        generation_info={
            "model": self.model,
            "usage": usage,
            "latency": latency
        }
    )
    
    return ChatResult(generations=[generation])
```

**Observability Highlights**:

1. **Timing**: 
```python
start_time = time.time()
# ... API call ...
latency = time.time() - start_time
```
Captures exact API latency.

2. **Token Usage**:
```python
usage = response_data.get("usage", {})
input_tokens = usage.get("input_tokens", 0)
output_tokens = usage.get("output_tokens", 0)
```
Extracts token counts for cost tracking.

3. **Metadata Attachment**:
```python
generation_info={
    "model": self.model,
    "usage": usage,
    "latency": latency
}
```
All this metadata automatically appears in LangSmith traces!

### Message Conversion

```python
def _convert_messages_to_bedrock(self, messages):
    """Convert LangChain messages to Bedrock format"""
    bedrock_messages = []
    
    for msg in messages:
        if isinstance(msg, SystemMessage):
            # System messages handled separately in Bedrock
            continue
        elif isinstance(msg, HumanMessage):
            bedrock_messages.append({
                "role": "user",
                "content": [{"type": "text", "text": msg.content}]
            })
        elif isinstance(msg, AIMessage):
            content = []
            if msg.content:
                content.append({"type": "text", "text": msg.content})
            if msg.tool_calls:
                for tool_call in msg.tool_calls:
                    content.append({
                        "type": "tool_use",
                        "id": tool_call["id"],
                        "name": tool_call["name"],
                        "input": tool_call["args"]
                    })
            bedrock_messages.append({
                "role": "assistant",
                "content": content
            })
        elif isinstance(msg, ToolMessage):
            bedrock_messages.append({
                "role": "user",
                "content": [{
                    "type": "tool_result",
                    "tool_use_id": msg.tool_call_id,
                    "content": msg.content
                }]
            })
    
    return bedrock_messages
```

**Observability**: 
- Conversion preserves all message data
- Tool calls visible in traces
- Can see exact format sent to API

---

## 🛠️ File 4: `utils.py` - Model Loading

**Location**: `core/react_agent/utils.py`

### load_chat_model Function

```python
def load_chat_model(model_name: str, context: Context):
    """Load chat model by name"""
    
    # Try Holistic AI Bedrock first (recommended)
    if os.getenv("HOLISTIC_AI_TEAM_ID") and os.getenv("HOLISTIC_AI_API_TOKEN"):
        from .holistic_ai_bedrock import get_chat_model
        try:
            return get_chat_model(model_name)
        except Exception as e:
            print(f"⚠ Bedrock failed, trying alternatives: {e}")
    
    # Try OpenAI
    if os.getenv("OPENAI_API_KEY"):
        from langchain_openai import ChatOpenAI
        return ChatOpenAI(model=model_name)
    
    # Try Ollama (local)
    try:
        from langchain_ollama import ChatOllama
        return ChatOllama(
            model=model_name,
            base_url=context.ollama_base_url
        )
    except Exception:
        pass
    
    raise ValueError("No model provider configured!")
```

**What it does**: 
- Tries providers in order: Bedrock → OpenAI → Ollama
- Falls through until one works
- Raises error if none available

**Observability**: 
- Provider choice affects trace metadata
- Different providers have different trace formats
- But all work with LangSmith!

---

## 🎯 How Observability Works: The Complete Picture

### When You Call agent.invoke()

```python
agent = create_react_agent(tools=[search_tool])
result = agent.invoke({"messages": [HumanMessage("query")]})
```

**What happens behind the scenes**:

1. **LangGraph intercepts invoke()**:
```python
# Inside LangGraph
def invoke(self, input, config=None):
    # Create root trace span
    with tracer.start_span("agent_execution") as root_span:
        # Record input
        root_span.log_input(input)
        
        # Execute graph
        result = self._execute_graph(input, config)
        
        # Record output
        root_span.log_output(result)
        
        return result
```

2. **Each node execution traced**:
```python
# For each node in graph
with tracer.start_span(f"node:{node_name}") as node_span:
    # Record state before
    node_span.log_state(state)
    
    # Execute node
    result = node_function(state)
    
    # Record state after
    node_span.log_state(result)
```

3. **Model calls traced**:
```python
# Inside model.invoke()
with tracer.start_span("model_invoke") as model_span:
    # Log input
    model_span.log_input(messages)
    
    # Call API
    response = self._api_call(messages)
    
    # Log output and metadata
    model_span.log_output(response)
    model_span.log_metadata({
        "tokens": response.usage,
        "latency": response.latency,
        "model": self.model_name
    })
```

4. **Tool calls traced**:
```python
# Inside tool._run()
with tracer.start_span(f"tool:{tool.name}") as tool_span:
    # Log input
    tool_span.log_input(args)
    
    # Execute tool
    result = self._execute(**args)
    
    # Log output
    tool_span.log_output(result)
```

5. **Complete trace tree**:
```
agent_execution (10s)
├─ node:call_model (2.3s)
│  └─ model_invoke (2.1s)
├─ node:tools (5.2s)
│  └─ tool:search (5.0s)
└─ node:call_model (2.5s)
   └─ model_invoke (2.3s)
```

**All automatic! No code changes needed!**

---

## 🎓 Key Takeaways

### Code Structure:

1. **create_agent.py**: Factory function that builds agents
2. **state.py**: State management with automatic merging
3. **holistic_ai_bedrock.py**: AWS API wrapper with built-in metrics
4. **utils.py**: Model loading with fallback logic

### Observability Integration:

1. **LangGraph**: Automatic node and edge tracing
2. **BaseChatModel**: Automatic model call tracing
3. **ToolNode**: Automatic tool execution tracing
4. **add_messages**: Automatic state change tracking

### Key Insights:

- Observability is **built into the framework**
- You don't need to add logging code
- Just configure LangSmith environment variables
- Traces are **automatically rich and detailed**

### Next Steps:

✅ You understand how the code implements observability!

**Continue to**:
- **[Tutorial 4: LangSmith Setup](04_langsmith_setup_guide.md)** - Start using it
- **[Tutorial 5: Tracing Basics](05_tracing_basics.md)** - Understand traces
- **[Tutorial 6: Analyzing Traces](06_analyzing_traces.md)** - Debug with traces

---

## ❓ Questions

### Q: Do I need to understand all this code to use observability?
**A**: No! You can use observability without understanding the internals. But understanding helps with:
- Custom tool development
- Debugging complex issues
- Contributing to the framework

### Q: Can I add my own observability code?
**A**: Yes! You can:
- Add custom log statements
- Add custom metadata to traces
- Implement custom tracers
- Integrate other monitoring tools

### Q: What if I want to use a different model provider?
**A**: The framework supports plugging in new providers. Just implement `BaseChatModel` interface and you get observability for free!

---

**🎉 Excellent work!** You've seen how observability is implemented at the code level. Ready to start tracing your own agents? Continue to [Tutorial 4: LangSmith Setup Guide](04_langsmith_setup_guide.md)!
