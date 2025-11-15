# ReAct Agent Explained - Reasoning + Acting Pattern

This tutorial explains the **ReAct (Reasoning + Acting) pattern**, which is the core concept behind how Holistic AI agents work. You'll understand why ReAct is powerful and how to leverage it effectively.

## Table of Contents
- [What is ReAct?](#what-is-react)
- [Why ReAct Matters](#why-react-matters)
- [The ReAct Loop](#the-react-loop)
- [ReAct in Action](#react-in-action)
- [Comparing Different Approaches](#comparing-different-approaches)
- [ReAct Implementation in Holistic AI](#react-implementation-in-holistic-ai)
- [Understanding Agent Reasoning](#understanding-agent-reasoning)
- [Handling Multiple Tool Calls](#handling-multiple-tool-calls)
- [Loop Prevention](#loop-prevention)
- [Best Practices](#best-practices)
- [Key Takeaways](#key-takeaways)

## What is ReAct?

**ReAct** stands for **Reasoning + Acting**. It's a pattern for building AI agents that:
1. **Reason** about what to do
2. **Act** by using tools or providing answers
3. **Observe** the results
4. **Repeat** until the task is complete

### The Core Idea

Traditional LLMs (like ChatGPT) just generate text based on their training.

ReAct agents go further:
- They can **think** about problems
- They can **use tools** to get information
- They can **learn from results** and adjust
- They can **chain multiple actions** together

### Visual Representation

```
Traditional LLM:
User Question → LLM → Response
(Limited to training data)

ReAct Agent:
User Question → Reason → Act → Observe → Reason → Act → ... → Response
                  ↓        ↓       ↓
              "Need info" Tool  Results
```

## Why ReAct Matters

### Problem: LLMs Have Limitations

1. **Outdated Information**: Training data has a cutoff date
2. **No External Access**: Can't search, query databases, or use APIs
3. **Can't Verify Facts**: Just generates plausible text
4. **Limited Reasoning**: Single-pass generation

### Solution: ReAct Pattern

ReAct solves these problems by:

**1. Tool Use**: Access current information
```python
# Without ReAct
"What's the weather?" → "I don't know, my training ended in 2023"

# With ReAct
"What's the weather?" → [Search web] → "It's 18°C and sunny"
```

**2. Multi-Step Reasoning**: Break down complex tasks
```python
# Without ReAct
"Research quantum computing and create a report" → Simple text

# With ReAct
1. Search "quantum computing basics"
2. Search "quantum computing applications"
3. Search "quantum computing future trends"
4. Synthesize all results
5. Create structured report
```

**3. Verification**: Check and validate
```python
# Without ReAct
"What's the capital of Australia?" → May hallucinate

# With ReAct
"What's the capital of Australia?" → [Search to verify] → "Canberra"
```

## The ReAct Loop

The ReAct pattern follows a cycle:

### Step 1: Reason (Think)

The agent analyzes the situation and decides what to do next.

```
User: "What are the latest AI breakthroughs?"

Agent thinks:
- This requires current information
- My training data is outdated
- I should use a search tool
- Search query: "latest AI breakthroughs 2024"
```

### Step 2: Act (Do)

The agent executes an action (call a tool or provide an answer).

```
Agent acts:
- Calls search_tool("latest AI breakthroughs 2024")
- Waits for results
```

### Step 3: Observe (Learn)

The agent receives and processes the results.

```
Agent observes:
- Receives search results
- Reads articles about GPT-4, Claude 3, etc.
- Notes key findings
```

### Step 4: Repeat or Respond

The agent decides if more actions are needed or if it can respond.

```
Agent thinks:
- I have good information now
- No more tools needed
- I can formulate a comprehensive answer

Agent responds:
"Recent AI breakthroughs in 2024 include..."
```

### The Complete Loop

```
┌─────────────────────────────────────┐
│ 1. REASON                           │
│    "What should I do next?"         │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 2. ACT                              │
│    Execute tool or respond          │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 3. OBSERVE                          │
│    Process results                  │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│ 4. DECIDE                           │
│    Need more? → REASON              │
│    Done? → RESPOND                  │
└─────────────────────────────────────┘
```

## ReAct in Action

Let's see a real example with detailed reasoning:

### Example: "Find and summarize a recent article about machine learning"

```python
# Create agent with search tool
from react_agent import create_react_agent
from valyu_tools import ValyuSearchTool, ValyuContentsTool
from langchain_core.messages import HumanMessage

agent = create_react_agent(
    tools=[ValyuSearchTool(), ValyuContentsTool()],
    model_name='claude-3-5-sonnet'
)

# Ask question
result = agent.invoke({
    "messages": [HumanMessage(
        content="Find and summarize a recent article about machine learning"
    )]
})
```

### What Happens Inside:

**Turn 1 - Initial Reasoning**
```
Agent thinks:
"I need to:
1. Search for recent ML articles
2. Get the content
3. Summarize it

Let me start by searching."

Agent acts:
- Calls: valyu_deep_search(query="recent machine learning articles")
```

**Turn 2 - Process Search Results**
```
Agent observes:
- Receives list of articles
- Selects most relevant one
- URL: https://example.com/ml-article

Agent thinks:
"I found a good article. Now I need to get its full content."

Agent acts:
- Calls: valyu_contents_extract(urls=["https://example.com/ml-article"])
```

**Turn 3 - Synthesize and Respond**
```
Agent observes:
- Receives full article content
- Reads and understands it

Agent thinks:
"I now have all information needed. Time to summarize."

Agent acts:
- Generates summary
- Provides response to user
```

### The Message Flow

```python
# Message history after execution:
[
    HumanMessage("Find and summarize a recent article..."),
    
    AIMessage(
        "I'll search for recent ML articles",
        tool_calls=[{
            "name": "valyu_deep_search",
            "args": {"query": "recent machine learning articles"}
        }]
    ),
    
    ToolMessage(
        content='{"results": [...articles...]}',
        tool_call_id="call_1"
    ),
    
    AIMessage(
        "Found an article, getting full content",
        tool_calls=[{
            "name": "valyu_contents_extract",
            "args": {"urls": ["https://example.com/ml-article"]}
        }]
    ),
    
    ToolMessage(
        content='{"content": "Article text..."}',
        tool_call_id="call_2"
    ),
    
    AIMessage(
        "Here's a summary of the article:\n\n[Summary text]"
    )
]
```

## Comparing Different Approaches

### Approach 1: Simple LLM (No ReAct)

```python
# Just ask GPT directly
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-5-mini",
    messages=[{"role": "user", "content": "What's the weather?"}]
)

# Response: "I don't have access to current weather data..."
```

**Limitations:**
- No access to current data
- Can't verify information
- Limited to training knowledge

### Approach 2: Manual Tool Calling

```python
# You manually orchestrate tools
query = "What's the weather?"

# Step 1: You decide to search
search_results = search_tool.run("current weather")

# Step 2: You extract the answer
weather = extract_weather(search_results)

# Step 3: You format response
response = f"The weather is {weather}"
```

**Limitations:**
- You do all the thinking
- Hard to handle complex scenarios
- Not scalable

### Approach 3: ReAct Agent (Holistic AI)

```python
# Agent figures everything out
agent = create_react_agent(tools=[search_tool])

result = agent.invoke({
    "messages": [HumanMessage("What's the weather?")]
})

# Agent automatically:
# 1. Decides to search
# 2. Calls search tool
# 3. Processes results
# 4. Formats response
```

**Benefits:**
- Autonomous decision-making
- Handles complexity automatically
- Scalable to any task

## ReAct Implementation in Holistic AI

Let's look at how Holistic AI implements the ReAct pattern:

### The Graph Structure

```python
# Simplified version of what happens in create_react_agent

# Define the nodes
def call_model(state):
    """REASON: What should I do next?"""
    response = model.invoke(state.messages)
    return {"messages": [response]}

def tools(state):
    """ACT: Execute the tools"""
    results = execute_tools(state.messages[-1].tool_calls)
    return {"messages": results}

# Build the graph
graph = StateGraph(State)
graph.add_node("call_model", call_model)  # Reasoning node
graph.add_node("tools", tools)            # Acting node

# Add routing logic
def route(state):
    """DECIDE: Need tools or done?"""
    if state.messages[-1].tool_calls:
        return "tools"
    return "__end__"

graph.add_conditional_edges("call_model", route)
graph.add_edge("tools", "call_model")  # Loop back to reason

# OBSERVE happens automatically when tools return to call_model
```

### The Execution Flow

```
User Input
    ↓
┌─────────────────┐
│  call_model     │ ← REASON: Analyze situation
│  (reasoning)    │
└────────┬────────┘
         ↓
    Tool calls?
    ┌────┴────┐
    Yes      No
    ↓         ↓
┌─────────────────┐  END
│     tools       │
│   (acting)      │
└────────┬────────┘
         ↓
    OBSERVE: Got results
         ↓
┌─────────────────┐
│  call_model     │ ← REASON: What next?
│  (reasoning)    │
└─────────────────┘
    (loop continues)
```

## Understanding Agent Reasoning

How does the agent "think"? Let's examine the thought process:

### System Prompt

The agent is guided by a system prompt that defines its behavior:

```python
# From core/react_agent/prompts.py
EXPERIMENT_SYSTEM_PROMPT = """
You are a general ReAct agent that can solve multi-step tasks 
by planning, using tools, and producing clear results.

Instructions:
1. Clarify or decompose the task if needed; plan minimal steps.
2. Use tools when they materially improve correctness or efficiency.
3. Ground factual claims in retrieved information and avoid hallucinations.
4. Produce a final answer with clear, actionable steps.
"""
```

This prompt teaches the agent to:
- Break down complex tasks
- Use tools strategically
- Be factual and accurate
- Provide clear outputs

### Model's Reasoning Process

When the model receives messages, it:

1. **Reads conversation history**
   - What has been said
   - What tools were used
   - What results came back

2. **Analyzes the current state**
   - What's the goal?
   - What information is missing?
   - What tools are available?

3. **Decides next action**
   - Call a tool?
   - Need more information?
   - Ready to respond?

4. **Generates output**
   - Tool call with arguments
   - Or final text response

### Example: Model's Internal "Thoughts"

```python
# User asks: "What's the latest Python version and its features?"

# Model's reasoning (not visible to us, but happens):
"""
Analysis:
- Need current information (latest Python version)
- This requires web search
- I have a search tool available

Plan:
1. Search for "latest Python version 2024"
2. Get the results
3. Summarize key features

Action: Call search tool
"""

# Model's output:
AIMessage(
    content="I'll search for the latest Python information",
    tool_calls=[{
        "name": "valyu_deep_search",
        "args": {"query": "latest Python version 2024 features"}
    }]
)
```

## Handling Multiple Tool Calls

Agents can call multiple tools in sequence or parallel:

### Sequential Tool Calls

```python
# Task: "Find a Python tutorial and summarize it"

# Turn 1: Search
tool_calls = [{"name": "search", "args": {"query": "Python tutorial"}}]

# Turn 2: Get content
tool_calls = [{"name": "extract", "args": {"url": "tutorial-url"}}]

# Turn 3: Respond with summary
```

### Parallel Tool Calls

Some models can call multiple tools at once:

```python
# Task: "Compare weather in London and Paris"

# Single turn with multiple tools
tool_calls = [
    {"name": "weather", "args": {"city": "London"}},
    {"name": "weather", "args": {"city": "Paris"}}
]

# Both execute simultaneously
# Results come back together
# Agent compares them
```

### Managing Complex Workflows

For complex tasks, the agent orchestrates many steps:

```python
# Task: "Research AI companies, analyze their products, 
#        and create a comparison report"

# Turn 1: Search for companies
# Turn 2: Search for company A details
# Turn 3: Search for company B details
# Turn 4: Search for company C details
# Turn 5: Synthesize comparison
# Turn 6: Format as structured report

# Each turn: Reason → Act → Observe → Decide
```

## Loop Prevention

ReAct agents could potentially loop forever. Holistic AI prevents this:

### Recursion Limit

```python
# Built into LangGraph
graph.compile(
    recursion_limit=25  # Maximum 25 steps
)
```

### is_last_step Flag

```python
# In state.py
@dataclass
class State:
    messages: List[Message]
    is_last_step: bool = False  # True on step 24/25

# In create_agent.py
def call_model(state):
    response = model.invoke(state.messages)
    
    # If last step and still has tool calls, abort
    if state.is_last_step and response.tool_calls:
        return {
            "messages": [AIMessage(
                content="Sorry, could not complete in specified steps."
            )]
        }
    
    return {"messages": [response]}
```

### Why This Matters

Without loop prevention:
```python
# Bad scenario
User: "Tell me about everything"

Agent: [Search "everything"]
Result: [Millions of results]
Agent: [Search more about topic 1]
Result: [More results]
Agent: [Search more about topic 2]
Result: [More results]
# ... infinite loop ...
```

With loop prevention:
```python
# Safe scenario
User: "Tell me about everything"

Agent: [Search] [Search] [Search] ... (24 times)
Agent: "I couldn't cover everything in 25 steps. 
        Here's what I found..." (Final response)
```

## Best Practices

### 1. Clear Task Instructions

Good:
```python
"Search for the latest Python version, list its top 3 features"
```

Bad:
```python
"Tell me about Python"  # Too vague, agent might loop
```

### 2. Provide Context

Good:
```python
"I'm learning web development. Find beginner-friendly React tutorials"
```

Bad:
```python
"Find tutorials"  # Missing context
```

### 3. Break Down Complex Tasks

Good:
```python
# Task 1
"Search for top 5 AI companies"

# Task 2
"For each company, find their main product"

# Task 3
"Compare the products and create a table"
```

Bad:
```python
"Research everything about AI companies and make a complete analysis"
# Too broad for single invocation
```

### 4. Use Appropriate Tools

Provide tools that match the task:
```python
# For research tasks
agent = create_react_agent(
    tools=[ValyuSearchTool(), ValyuContentsTool()]
)

# For data tasks
agent = create_react_agent(
    tools=[database_tool, calculator_tool]
)
```

### 5. Monitor and Debug

Use LangSmith to visualize reasoning:
```python
# Enable tracing
import os
os.environ["LANGSMITH_TRACING"] = "true"

# Run agent
result = agent.invoke(...)

# View trace at smith.langchain.com
```

## Key Takeaways

✅ **ReAct = Reasoning + Acting**: Agents think, act, observe, and repeat
✅ **More Powerful than LLMs**: Can access external data and tools
✅ **Autonomous Decision-Making**: Agents decide when to use tools
✅ **Multi-Step Reasoning**: Can break down complex tasks
✅ **Loop Prevention**: Built-in safety mechanisms
✅ **Observable Process**: Can trace every decision

## Practice Exercises

### Exercise 1: Trace the ReAct Loop
Run an agent with a complex query and identify each:
- Reasoning step
- Acting step
- Observation step

### Exercise 2: Compare Approaches
Try the same query with:
- Direct LLM call
- ReAct agent with tools
Compare the results

### Exercise 3: Optimize Tool Use
Create an agent and test different system prompts to minimize tool calls while maintaining accuracy

## What's Next?

Now that you understand ReAct, let's learn about:
- **Creating Custom Tools**: Extend agent capabilities
- **Tool Best Practices**: Design effective tools
- **Multi-Tool Orchestration**: Complex workflows

**Continue to**: [05_Working_with_Tools.md](./05_Working_with_Tools.md)

## Additional Resources

- **Original ReAct Paper**: [arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
- **LangGraph Documentation**: [langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)
- **Workflow Details**: [docs/WORKFLOW.md](../../docs/WORKFLOW.md)

---

**Excellent!** 🎉 You now understand the ReAct pattern that powers Holistic AI agents. This is the foundation for building intelligent, autonomous systems!
