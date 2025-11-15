# Tutorial 2: Architecture Deep Dive

## 📖 Overview

**What You'll Learn:**
- Complete system architecture for observable agents
- How components interact and communicate
- Data flow from user query to final response
- State management and execution graphs
- Where observability data comes from

**Prerequisites:** 
- [Tutorial 1: What is Observability?](01_what_is_observability.md)

**Time to Complete:** 30 minutes

**Difficulty:** ⭐⭐ Medium

---

## 🏗️ System Architecture Overview

### The Big Picture

The hackathon repository uses a **layered architecture** that makes observability natural and automatic:

```
┌─────────────────────────────────────────────────────────────────┐
│                    👤 USER APPLICATION                          │
│              (Jupyter Notebooks / Python Scripts)               │
└──────────────────────────┬──────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  🤖 AGENT FRAMEWORK LAYER                       │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  LangGraph   │  │  LangChain   │  │   Custom     │        │
│  │  ReAct Agent │  │  Components  │  │   Tools      │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└──────────────────────────┬──────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  🧠 MODEL PROVIDER LAYER                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │ Holistic AI  │  │   OpenAI     │  │   Ollama     │        │
│  │   Bedrock    │  │   GPT-5      │  │  (Local)     │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└──────────────────────────┬──────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                📊 OBSERVABILITY LAYER                           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  LangSmith   │  │  CodeCarbon  │  │  CloudWatch  │        │
│  │   Tracing    │  │   Tracking   │  │  Monitoring  │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### Key Insight 🔑

**Observability is not an add-on** - it's built into every layer:
- LangGraph automatically captures state transitions
- LangChain emits events for every operation
- Models report token usage and latency
- LangSmith intercepts and records everything

This means **you get observability for free** when using this architecture!

---

## 🎯 Core Components Explained

### 1. ReAct Agent Framework

**Location**: `core/react_agent/`

**What it does**: Implements the ReAct (Reasoning + Acting) pattern

**ReAct Pattern**:
```
1. REASON: Analyze the problem
2. ACT: Take an action (use a tool or respond)
3. OBSERVE: See the result
4. REPEAT: Continue until done
```

**Why it matters for observability**:
- Each REASON step is captured as a trace span
- Each ACT creates a tool call span
- Each OBSERVE adds data to the trace
- The loop structure is visible in trace visualization

**Example Trace**:
```
User Query: "What's the weather in Tokyo?"

Trace captures:
├─ REASON: Agent analyzes query (span 1)
│  → "I need to use weather tool for Tokyo"
├─ ACT: Call weather_tool("Tokyo") (span 2)
├─ OBSERVE: Result = "22°C, sunny" (span 3)
├─ REASON: Synthesize response (span 4)
│  → "The weather in Tokyo is 22°C and sunny"
└─ ACT: Return final answer (span 5)
```

### 2. State Management System

**Location**: `core/react_agent/state.py`

**The State Structure**:
```python
@dataclass
class State:
    messages: List[Message]  # Conversation history
    is_last_step: bool       # Loop prevention
```

**How State Evolves**:
```
Initial State:
  messages = []

After User Input:
  messages = [
    HumanMessage("What's the weather?")
  ]

After Agent Reasoning:
  messages = [
    HumanMessage("What's the weather?"),
    AIMessage("I'll check the weather", tool_calls=[...])
  ]

After Tool Execution:
  messages = [
    HumanMessage("What's the weather?"),
    AIMessage("I'll check the weather", tool_calls=[...]),
    ToolMessage("Temperature: 22°C")
  ]

After Final Response:
  messages = [
    HumanMessage("What's the weather?"),
    AIMessage("I'll check the weather", tool_calls=[...]),
    ToolMessage("Temperature: 22°C"),
    AIMessage("It's 22°C and sunny!")
  ]
```

**Observability Insight**: Every state change is automatically traced! LangSmith captures each message addition, so you can replay the entire conversation.

### 3. Execution Graph (LangGraph)

**The Graph Structure**:
```
              ┌─────────┐
              │  START  │
              └────┬────┘
                   │
                   ▼
           ┌───────────────┐
           │  call_model   │ ← Main agent reasoning
           └───────┬───────┘
                   │
                   ▼
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
   ┌─────────┐          ┌──────────┐
   │  tools  │          │   END    │
   └────┬────┘          └──────────┘
        │
        │ (loop back)
        ▼
   ┌───────────────┐
   │  call_model   │
   └───────────────┘
```

**How It Works**:

1. **START**: Entry point
2. **call_model**: Agent thinks and decides action
3. **Conditional routing**:
   - If tool_calls exist → go to **tools** node
   - If no tool_calls → go to **END**
4. **tools**: Execute tool calls
5. **Loop back** to call_model with results
6. **END**: Final state

**Observability Benefit**: Each node in the graph becomes a span in the trace!

**Visual Trace Example**:
```
Root Span: agent_execution
├─ Span: call_model (2.3s)
│  └─ Span: model_inference (2.1s)
├─ Span: tools (5.2s)
│  ├─ Span: search_tool (3.1s)
│  └─ Span: scrape_tool (2.1s)
├─ Span: call_model (1.8s)
│  └─ Span: model_inference (1.6s)
└─ Span: format_response (0.3s)

Total: 9.6 seconds
```

---

## 🔄 Data Flow: From Query to Response

### Step-by-Step Flow

Let's trace a complete request: **"What are the latest npm updates?"**

#### **Step 1: User Input**
```python
# User code
result = agent.invoke({
    "messages": [HumanMessage("What are the latest npm updates?")]
})
```

**What happens**:
- User creates initial state with one message
- LangSmith: Creates root trace span

#### **Step 2: Enter Graph**
```
Graph receives state:
  messages = [HumanMessage("What are the latest npm updates?")]
  is_last_step = False
```

**What happens**:
- State enters START node
- LangSmith: Records entry time and state

#### **Step 3: call_model Node (First Time)**
```python
# Inside call_model node
def call_model(state):
    # Add system prompt with timestamp
    messages_with_system = [
        SystemMessage("You are a helpful AI assistant. Current time: 2025-11-15 14:30")
    ] + state.messages
    
    # Call LLM
    response = model.invoke(messages_with_system)
    
    # Return updated state
    return {"messages": [response]}
```

**What happens**:
1. System prompt added with current timestamp
2. All messages sent to LLM
3. Model analyzes and decides to use search tool
4. Response includes tool_calls
5. LangSmith: Records model call span (2.3s)
   - Input tokens: 245
   - Output tokens: 89
   - Model: claude-3-5-sonnet
   - Cost: $0.0023

**State after**:
```
messages = [
  HumanMessage("What are the latest npm updates?"),
  AIMessage(
    content="I'll search for npm updates",
    tool_calls=[{
      "name": "valyu_deep_search",
      "args": {"query": "latest npm release notes"},
      "id": "call_xyz"
    }]
  )
]
```

#### **Step 4: Conditional Routing**
```python
# Graph checks if tools needed
if state.messages[-1].tool_calls:
    next_node = "tools"  # Yes, go to tools
else:
    next_node = END      # No, we're done
```

**What happens**:
- Graph sees tool_calls in last message
- Routes to tools node
- LangSmith: Records routing decision

#### **Step 5: tools Node**
```python
# Inside tools node
def execute_tools(state):
    tool_messages = []
    
    for tool_call in state.messages[-1].tool_calls:
        # Find the tool
        tool = find_tool(tool_call["name"])
        
        # Execute tool
        result = tool.run(**tool_call["args"])
        
        # Create tool message
        tool_messages.append(
            ToolMessage(
                content=result,
                tool_call_id=tool_call["id"]
            )
        )
    
    return {"messages": tool_messages}
```

**What happens**:
1. Extracts tool_calls from AI message
2. Finds ValyuSearchTool
3. Calls tool with query "latest npm release notes"
4. Tool executes HTTP request (5.1s)
5. Tool returns search results
6. Creates ToolMessage with results
7. LangSmith: Records tool execution span
   - Tool name: valyu_deep_search
   - Duration: 5.1s
   - Input: {"query": "latest npm release notes"}
   - Output: {search results JSON}

**State after**:
```
messages = [
  HumanMessage("What are the latest npm updates?"),
  AIMessage(..., tool_calls=[...]),
  ToolMessage(
    content='{"results": [...]}',
    tool_call_id="call_xyz"
  )
]
```

#### **Step 6: call_model Node (Second Time)**
```python
# Agent sees tool results and synthesizes
def call_model(state):
    # All messages including tool results
    messages_with_system = [
        SystemMessage("You are a helpful AI assistant...")
    ] + state.messages
    
    # Call LLM again
    response = model.invoke(messages_with_system)
    
    return {"messages": [response]}
```

**What happens**:
1. Messages include tool results
2. Model sees search results
3. Model synthesizes final answer
4. Response has NO tool_calls (we're done)
5. LangSmith: Records second model call span (2.5s)
   - Input tokens: 1,234 (more context now!)
   - Output tokens: 156
   - Cost: $0.0089

**State after**:
```
messages = [
  HumanMessage("What are the latest npm updates?"),
  AIMessage(..., tool_calls=[...]),
  ToolMessage(...),
  AIMessage("Based on the latest information, npm 11.5.2 was released...")
]
```

#### **Step 7: Conditional Routing (Again)**
```python
if state.messages[-1].tool_calls:
    next_node = "tools"  # No
else:
    next_node = END      # Yes, go to END
```

**What happens**:
- No tool_calls in last message
- Routes to END node
- LangSmith: Records completion

#### **Step 8: END Node**
```
Final state returned to user:
  messages = [... all 4 messages ...]
```

**What happens**:
- Graph completes
- Final state returned
- LangSmith: Closes root span, calculates total time

### Complete Trace Visualization

```
Root: agent_execution (9.9s)
│
├─ call_model_1 (2.3s)
│  ├─ add_system_prompt (0.01s)
│  └─ model_invoke (2.29s)
│     ├─ tokenize_input (0.1s)
│     ├─ inference (2.0s)
│     └─ parse_response (0.19s)
│
├─ tools (5.2s)
│  └─ valyu_deep_search (5.1s)
│     ├─ format_query (0.05s)
│     ├─ http_request (4.8s)
│     └─ parse_results (0.25s)
│
└─ call_model_2 (2.5s)
   ├─ add_system_prompt (0.01s)
   └─ model_invoke (2.49s)
      ├─ tokenize_input (0.15s)
      ├─ inference (2.2s)
      └─ parse_response (0.14s)
```

---

## 💾 Message Format and Tool Calling

### Message Types

The system uses different message types for different purposes:

#### **1. HumanMessage** (User Input)
```python
HumanMessage(
    content="What's the weather?",
    id="msg_1"
)
```

#### **2. AIMessage** (Agent Response/Reasoning)
```python
# Without tools
AIMessage(
    content="The weather is sunny!",
    id="msg_2"
)

# With tool calls
AIMessage(
    content="I'll check the weather",
    tool_calls=[{
        "name": "weather_tool",
        "args": {"location": "Tokyo"},
        "id": "call_1"
    }],
    id="msg_3"
)
```

#### **3. ToolMessage** (Tool Results)
```python
ToolMessage(
    content='{"temperature": 22, "condition": "sunny"}',
    tool_call_id="call_1",
    id="msg_4"
)
```

#### **4. SystemMessage** (Instructions)
```python
SystemMessage(
    content="You are a helpful assistant. Current time: 2025-11-15"
)
```

### Tool Calling Flow

**How Tools Are Invoked**:

```
1. Agent decides tool is needed
   ↓
2. AIMessage with tool_calls created
   ↓
3. Graph routes to tools node
   ↓
4. tools node extracts tool_calls
   ↓
5. For each tool_call:
   - Find tool by name
   - Extract arguments
   - Execute tool._run(**args)
   - Create ToolMessage with result
   ↓
6. ToolMessages added to state
   ↓
7. Graph loops back to call_model
```

**Observability Capture**:
```
Trace shows:
├─ AIMessage creation (what agent decided)
├─ Tool selection logic
├─ Tool execution time
├─ Tool input/output
├─ ToolMessage creation
└─ How agent uses results
```

---

## 🔌 Model Provider Integration

### Holistic AI Bedrock Integration

**Location**: `core/react_agent/holistic_ai_bedrock.py`

**What it does**:
1. Wraps AWS Bedrock API
2. Converts LangChain messages to Claude format
3. Handles tool calling format
4. Manages authentication
5. Emits observability data

**Message Conversion Example**:

```python
# LangChain format
messages = [
    HumanMessage(content="Hello"),
    AIMessage(content="Hi!", tool_calls=[...])
]

# Converted to AWS Bedrock format
bedrock_messages = [
    {
        "role": "user",
        "content": [{"type": "text", "text": "Hello"}]
    },
    {
        "role": "assistant",
        "content": [
            {"type": "text", "text": "Hi!"},
            {"type": "tool_use", "id": "...", "name": "...", "input": {...}}
        ]
    }
]
```

**Observability Integration**:
```python
class HolisticAIBedrockChat(BaseChatModel):
    def _generate(self, messages, **kwargs):
        # Start timing
        start_time = time.time()
        
        # Convert messages
        bedrock_msgs = self._convert_messages(messages)
        
        # Call API
        response = requests.post(API_URL, json={
            "messages": bedrock_msgs,
            ...
        })
        
        # Calculate metrics
        latency = time.time() - start_time
        tokens_used = response.json()["usage"]
        
        # Metrics automatically captured by LangSmith!
        return response
```

---

## 📊 Where Observability Data Comes From

### Automatic Data Collection Points

#### 1. **LangGraph State Changes**
```python
# Every state update is captured
state = State(messages=[])
# → LangSmith records initial state

state.messages.append(HumanMessage("query"))
# → LangSmith records state change

# You get full state history for free!
```

#### 2. **Model Invocations**
```python
# Every LLM call is traced
response = model.invoke(messages)
# → LangSmith captures:
#    - Input messages
#    - Output response
#    - Tokens used
#    - Latency
#    - Model name
#    - Temperature, etc.
```

#### 3. **Tool Executions**
```python
# Every tool call is traced
result = tool._run(query="search term")
# → LangSmith captures:
#    - Tool name
#    - Input arguments
#    - Output result
#    - Execution time
#    - Success/failure
```

#### 4. **Graph Node Transitions**
```python
# Every node transition is traced
graph.invoke(state)
# → LangSmith captures:
#    - Node entry/exit
#    - Routing decisions
#    - Time in each node
#    - State at each step
```

### The Magic: Automatic Instrumentation

**You don't need to add logging code!** The frameworks do it automatically:

```python
# This simple code...
agent = create_react_agent(tools=[search_tool])
result = agent.invoke({"messages": [HumanMessage("query")]})

# ...automatically produces:
# - Complete trace with all spans
# - Token usage metrics
# - Tool call details
# - State transitions
# - Timing information
# - Error tracking

# IF LangSmith is configured!
```

**Configuration**:
```bash
# .env file
LANGSMITH_API_KEY=your_key
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=my_project
```

That's it! No code changes needed.

---

## 🎯 Key Architecture Benefits for Observability

### 1. **Structured State**
- State is a dataclass with clear structure
- Every change is tracked
- State can be serialized and replayed

### 2. **Graph-Based Execution**
- Nodes map directly to trace spans
- Edges show decision flow
- Easy to visualize

### 3. **Message-Based Communication**
- All data flows through messages
- Messages are immutable
- Full history preserved

### 4. **Automatic Instrumentation**
- No manual logging needed
- Consistent trace format
- Works across all agents

### 5. **Layered Architecture**
- Each layer emits events
- Events roll up into complete trace
- Easy to debug any layer

---

## 🎓 Key Takeaways

### Architecture Principles:

1. **ReAct Pattern**: Reason → Act → Observe loop
2. **State Graph**: Nodes and edges define execution
3. **Message Flow**: Immutable messages preserve history
4. **Automatic Tracing**: Built-in observability
5. **Layered Design**: Clear separation of concerns

### Observability Integration:

- LangGraph captures state transitions
- LangChain emits operation events
- Models report usage metrics
- LangSmith collects everything
- **Zero-code observability!**

### Next Steps:

✅ You now understand how the system works!

**Continue to**:
- **[Tutorial 3: Code Walkthrough](03_code_walkthrough.md)** - See the code in detail
- **[Tutorial 4: LangSmith Setup](04_langsmith_setup_guide.md)** - Start tracing
- **[Tutorial 5: Tracing Basics](05_tracing_basics.md)** - Understand traces

---

## ❓ Questions

### Q: Why use a graph instead of simple function calls?
**A**: Graphs provide:
- Clear execution flow visualization
- Easy conditional routing
- Built-in loop prevention
- Automatic observability
- Easier debugging

### Q: What if I don't use LangSmith?
**A**: You'll still get basic logging, but you'll miss:
- Visual trace exploration
- Token usage tracking
- Performance analytics
- Team collaboration features

### Q: Can I customize the observability?
**A**: Yes! You can:
- Add custom metadata
- Implement custom tracers
- Add your own logging
- Integrate other tools

---

**🎉 Great job!** You understand the architecture. Ready to see the actual code? Continue to [Tutorial 3: Code Walkthrough](03_code_walkthrough.md)!
