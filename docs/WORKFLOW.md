# Workflow and Agent Lifecycle

## Table of Contents
- [Agent Lifecycle Overview](#agent-lifecycle-overview)
- [Message Flow](#message-flow)
- [Execution Patterns](#execution-patterns)
- [Tool Execution](#tool-execution)
- [State Transitions](#state-transitions)
- [Error Handling](#error-handling)
- [Common Workflows](#common-workflows)

## Agent Lifecycle Overview

### 1. Creation Phase

```
User Code
    ↓
create_react_agent(tools, config)
    ↓
1. Load model (Bedrock/OpenAI/Ollama)
2. Bind tools to model
3. Build state graph
4. Add nodes (call_model, tools, format_output)
5. Add edges (routing logic)
6. Compile graph
    ↓
Returns: Compiled Agent
```

**Step-by-step:**

```python
# Step 1: Import dependencies
from react_agent import create_react_agent
from valyu_tools import ValyuSearchTool

# Step 2: Create tools
tools = [ValyuSearchTool()]

# Step 3: Create agent
agent = create_react_agent(
    tools=tools,
    model_name='claude-3-5-sonnet'
)

# Agent is now ready to invoke
```

### 2. Invocation Phase

```
agent.invoke({"messages": [HumanMessage("query")]})
    ↓
Enters State Graph
    ↓
1. Start Node
2. call_model Node
3. Conditional Routing
4. (Optional) tools Node → back to call_model
5. (Optional) format_output Node
6. End Node
    ↓
Returns: {"messages": [...]}
```

### 3. Cleanup Phase

Agents are stateless by default - no cleanup needed unless using checkpointer.

## Message Flow

### Basic Flow (No Tools)

```
1. User Input
   ─────────────────────────────────────────────
   messages = [
     HumanMessage(content="What is a GPU?")
   ]

2. Agent Reasoning (call_model node)
   ─────────────────────────────────────────────
   → Model invoked with system prompt + messages
   → Model generates response (no tool calls)
   
   messages = [
     HumanMessage(content="What is a GPU?"),
     AIMessage(content="A GPU is a Graphics Processing Unit...")
   ]

3. Return to User
   ─────────────────────────────────────────────
   No tool calls → End state
   Return final messages
```

### Flow with Tools

```
1. User Input
   ─────────────────────────────────────────────
   messages = [
     HumanMessage(content="What are the latest npm updates?")
   ]

2. Agent Planning (call_model node)
   ─────────────────────────────────────────────
   → Model analyzes query
   → Decides to use search tool
   
   messages = [
     HumanMessage(content="What are the latest npm updates?"),
     AIMessage(
       content="I'll search for npm updates",
       tool_calls=[{
         "name": "valyu_deep_search",
         "args": {"query": "latest npm release notes"},
         "id": "call_123"
       }]
     )
   ]

3. Tool Execution (tools node)
   ─────────────────────────────────────────────
   → ValyuSearchTool.run(query="latest npm release notes")
   → Returns search results
   
   messages = [
     HumanMessage(content="What are the latest npm updates?"),
     AIMessage(..., tool_calls=[...]),
     ToolMessage(
       content='{"results": [...]}',
       tool_call_id="call_123"
     )
   ]

4. Agent Synthesis (call_model node - second time)
   ─────────────────────────────────────────────
   → Model sees tool results
   → Synthesizes final answer
   
   messages = [
     HumanMessage(content="What are the latest npm updates?"),
     AIMessage(..., tool_calls=[...]),
     ToolMessage(content='{"results": [...]}', ...),
     AIMessage(content="Based on the latest information, npm 11.5.2...")
   ]

5. Return to User
   ─────────────────────────────────────────────
   No more tool calls → End state
   Return final messages
```

### Multi-Step Tool Flow

```
1. User Input
   "Create a report on quantum computing"

2. Call Model → Tool Calls
   tool_calls = [search("quantum computing basics")]

3. Execute Tools → Results
   ToolMessage(content="search results...")

4. Call Model → More Tool Calls
   tool_calls = [search("quantum computing applications")]

5. Execute Tools → Results
   ToolMessage(content="more search results...")

6. Call Model → Final Response
   AIMessage(content="Here's a comprehensive report...")

7. End
```

## Execution Patterns

### 1. Single-Turn Pattern

```python
# Simple question-answer
result = agent.invoke({
    "messages": [HumanMessage(content="What is 2+2?")]
})

print(result["messages"][-1].content)
# Output: "4"
```

**Graph execution:**
```
START → call_model → END
```

### 2. Tool-Using Pattern

```python
# Question requiring external information
result = agent.invoke({
    "messages": [HumanMessage(content="What's the weather in London?")]
})

# Execution trace:
# 1. call_model: Decides to use weather tool
# 2. tools: Executes weather API call
# 3. call_model: Synthesizes response with data
# 4. END
```

**Graph execution:**
```
START → call_model → tools → call_model → END
```

### 3. Multi-Turn Pattern

```python
# Continued conversation (requires checkpointer)
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver()
)

# Turn 1
config = {"configurable": {"thread_id": "conversation_1"}}
result1 = agent.invoke({
    "messages": [HumanMessage(content="My name is Alice")]
}, config)

# Turn 2 - agent remembers Alice
result2 = agent.invoke({
    "messages": [HumanMessage(content="What's my name?")]
}, config)

print(result2["messages"][-1].content)
# Output: "Your name is Alice"
```

**State persistence:**
```
Turn 1: messages = [Human("My name is Alice"), AI("Nice to meet you, Alice")]
        ↓ (saved to checkpointer)
Turn 2: messages = [
          Human("My name is Alice"),
          AI("Nice to meet you, Alice"),
          Human("What's my name?"),
          AI("Your name is Alice")
        ]
```

### 4. Structured Output Pattern

```python
from pydantic import BaseModel

class Response(BaseModel):
    answer: str
    confidence: float

agent = create_react_agent(
    tools=[...],
    output_schema=Response
)

result = agent.invoke({
    "messages": [HumanMessage(content="What is AI?")]
})

# Result includes format_output step:
# 1. call_model: Generates text response
# 2. format_output: Converts to JSON matching Response schema
# 3. Validates with Pydantic
# 4. Returns structured data
```

**Graph execution:**
```
START → call_model → format_output → END
                  ↘ tools → call_model ↗
```

## Tool Execution

### Tool Call Lifecycle

```
1. Model Decision
   ─────────────────────────────────────────────
   Model analyzes task and decides tool is needed
   Returns: AIMessage with tool_calls field

2. Tool Node Receives Calls
   ─────────────────────────────────────────────
   For each tool_call in message:
     - Extract tool name
     - Extract arguments
     - Find matching tool

3. Tool Invocation
   ─────────────────────────────────────────────
   tool._run(**args)
     - Executes tool logic
     - May call external APIs
     - May process data
     - Returns result (string or dict)

4. Create Tool Messages
   ─────────────────────────────────────────────
   For each tool result:
     ToolMessage(
       content=result,
       tool_call_id=original_call_id
     )

5. Append to State
   ─────────────────────────────────────────────
   State updated with new ToolMessages
   Control returns to call_model node

6. Model Processes Results
   ─────────────────────────────────────────────
   Model sees ToolMessages in conversation
   Synthesizes information into response
```

### Tool Error Handling

```python
# If tool execution fails:
try:
    result = tool._run(**args)
except Exception as e:
    # Error captured as ToolMessage
    ToolMessage(
        content=f"Error: {str(e)}",
        tool_call_id=call_id,
        additional_kwargs={"error": True}
    )

# Model sees error and can:
# - Retry with different arguments
# - Try a different tool
# - Acknowledge limitation to user
```

### Parallel Tool Calls

```python
# Model can request multiple tools at once
AIMessage(
    content="I'll search for information",
    tool_calls=[
        {"name": "search", "args": {"query": "topic A"}, "id": "1"},
        {"name": "search", "args": {"query": "topic B"}, "id": "2"}
    ]
)

# Tools node executes in parallel
ToolNode([
    ToolMessage(content="results A", tool_call_id="1"),
    ToolMessage(content="results B", tool_call_id="2")
])

# Model receives all results together
```

## State Transitions

### State Structure Evolution

```python
# Initial State (Empty)
State(
    messages=[],
    is_last_step=False
)

# After User Input
State(
    messages=[
        HumanMessage(id="1", content="query")
    ],
    is_last_step=False
)

# After Model Planning
State(
    messages=[
        HumanMessage(id="1", content="query"),
        AIMessage(id="2", content="thinking", tool_calls=[...])
    ],
    is_last_step=False
)

# After Tool Execution
State(
    messages=[
        HumanMessage(id="1", content="query"),
        AIMessage(id="2", content="thinking", tool_calls=[...]),
        ToolMessage(id="3", content="results", tool_call_id="...")
    ],
    is_last_step=False
)

# After Final Response
State(
    messages=[
        HumanMessage(id="1", content="query"),
        AIMessage(id="2", content="thinking", tool_calls=[...]),
        ToolMessage(id="3", content="results", tool_call_id="..."),
        AIMessage(id="4", content="final answer")
    ],
    is_last_step=False
)
```

### State Updates with add_messages

The `add_messages` reducer handles message updates:

```python
# Existing state
messages = [
    HumanMessage(id="1", content="hello")
]

# New messages added
new_messages = [
    AIMessage(id="2", content="hi there")
]

# Reducer merges them
result = add_messages(messages, new_messages)
# Result: [
#     HumanMessage(id="1", content="hello"),
#     AIMessage(id="2", content="hi there")
# ]

# If same ID, message is updated
update_messages = [
    HumanMessage(id="1", content="hello updated")
]

result = add_messages(messages, update_messages)
# Result: [
#     HumanMessage(id="1", content="hello updated")
# ]
```

## Error Handling

### Loop Prevention

```python
# is_last_step prevents infinite loops
State(is_last_step=False)  # Normal operation

# After recursion_limit - 1 steps
State(is_last_step=True)   # Last step warning

# In call_model node:
if state.is_last_step and response.tool_calls:
    # Override tool calls, return error message
    return {
        "messages": [AIMessage(
            content="Sorry, could not complete in specified steps."
        )]
    }
```

### API Errors

```python
# In holistic_ai_bedrock.py
try:
    response = requests.post(api_endpoint, ...)
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    # Detailed error message with debugging info
    raise ValueError(
        f"Error calling Holistic AI Bedrock API: {e}\n"
        f"Response: {e.response.text if e.response else 'N/A'}"
    )
```

### Tool Errors

```python
# Tools can fail gracefully
class MyTool(BaseTool):
    def _run(self, param: str) -> str:
        try:
            result = do_something(param)
            return result
        except Exception as e:
            # Return error as string
            return f"Tool execution failed: {str(e)}"

# Agent receives error message and can adapt
```

## Common Workflows

### Workflow 1: Simple Q&A

```python
# No tools needed
agent = create_react_agent(tools=[], model_name='claude-3-5-sonnet')
result = agent.invoke({"messages": [HumanMessage("What is Python?")]})

# Execution:
# 1. call_model: Generates answer from knowledge
# 2. END
```

### Workflow 2: Research Query

```python
# Uses search tool
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)
result = agent.invoke({
    "messages": [HumanMessage("Latest AI breakthroughs in 2024")]
})

# Execution:
# 1. call_model: Decides to search
# 2. tools: Executes search
# 3. call_model: Synthesizes results
# 4. END
```

### Workflow 3: Multi-Step Task

```python
# Complex task requiring multiple tool calls
agent = create_react_agent(
    tools=[search_tool, calculator_tool, email_tool],
    model_name='claude-3-5-sonnet'
)
result = agent.invoke({
    "messages": [HumanMessage(
        "Research quantum computing, calculate market size, "
        "and email summary to team@company.com"
    )]
})

# Execution:
# 1. call_model: Plan subtasks
# 2. tools: search("quantum computing")
# 3. call_model: Analyze, plan next step
# 4. tools: calculator(market_size_calculation)
# 5. call_model: Draft email
# 6. tools: send_email(to="team@company.com", ...)
# 7. call_model: Confirm completion
# 8. END
```

### Workflow 4: Structured Data Extraction

```python
from pydantic import BaseModel
from typing import List

class Article(BaseModel):
    title: str
    author: str
    summary: str
    key_points: List[str]

agent = create_react_agent(
    tools=[web_scraper_tool],
    output_schema=Article,
    model_name='claude-3-5-sonnet'
)

result = agent.invoke({
    "messages": [HumanMessage(
        "Extract article info from https://example.com/article"
    )]
})

# Execution:
# 1. call_model: Decides to scrape
# 2. tools: Scrapes webpage
# 3. call_model: Extracts information
# 4. format_output: Converts to Article schema
# 5. END

# Result is validated Pydantic object
article = Article.parse_raw(result["messages"][-1].content)
```

### Workflow 5: Conversational Agent

```python
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver(),
    model_name='claude-3-5-sonnet'
)

config = {"configurable": {"thread_id": "user_123"}}

# Turn 1
agent.invoke({
    "messages": [HumanMessage("I need help with Python")]
}, config)

# Turn 2 - context maintained
agent.invoke({
    "messages": [HumanMessage("Show me an example")]
}, config)

# Turn 3 - full conversation history available
agent.invoke({
    "messages": [HumanMessage("Make it more advanced")]
}, config)

# Each invocation:
# 1. Loads previous messages from checkpointer
# 2. Appends new message
# 3. Processes with full context
# 4. Saves updated state
# 5. Returns response
```

## Execution Timing

### Typical Execution Times

```
Simple Q&A (no tools):
  - call_model: 1-3 seconds
  - Total: 1-3 seconds

Single Tool Call:
  - call_model (planning): 1-3 seconds
  - tools: 2-10 seconds (depends on tool)
  - call_model (synthesis): 1-3 seconds
  - Total: 4-16 seconds

Multi-Tool Workflow:
  - Each cycle adds 3-13 seconds
  - Total: 10-60 seconds (depends on complexity)

Structured Output:
  - Adds format_output step: 1-3 seconds
  - Total: +1-3 seconds on top of base
```

### Performance Optimization

```python
# 1. Choose appropriate model
agent = create_react_agent(
    tools=[...],
    model_name='nova-lite'  # Faster, cheaper for simple tasks
)

# 2. Disable checkpointing if not needed
agent = create_react_agent(
    tools=[...],
    checkpointer=None  # Default, fastest
)

# 3. Limit tool calls
context = Context(max_search_results=5)  # Fewer results = faster
agent = create_react_agent(tools=[...], context=context)

# 4. Use streaming for long responses
for chunk in agent.stream({"messages": [HumanMessage("...")]}):
    print(chunk)  # Process incrementally
```

## Next Steps

- See [ARCHITECTURE.md](ARCHITECTURE.md) for system design
- See [API_REFERENCE.md](API_REFERENCE.md) for API documentation
- See [GETTING_STARTED.md](GETTING_STARTED.md) for setup instructions
