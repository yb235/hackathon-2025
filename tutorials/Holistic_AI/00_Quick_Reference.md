# Quick Reference - Holistic AI 2 Cheat Sheet

A quick reference guide for common tasks and patterns in Holistic AI 2.

## Table of Contents
- [Installation](#installation)
- [Basic Agent](#basic-agent)
- [Agent with Tools](#agent-with-tools)
- [Structured Output](#structured-output)
- [Custom Tools](#custom-tools)
- [Conversation Memory](#conversation-memory)
- [Common Patterns](#common-patterns)
- [Debugging](#debugging)

## Installation

```bash
# Clone repository
git clone https://github.com/holistic-ai/hackthon-2025.git
cd hackthon-2025

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your credentials
```

## Basic Agent

### Minimal Agent
```python
from react_agent import create_react_agent
from langchain_core.messages import HumanMessage

agent = create_react_agent(tools=[], model_name='claude-3-5-sonnet')
result = agent.invoke({"messages": [HumanMessage("Hello")]})
print(result["messages"][-1].content)
```

### With Custom System Prompt
```python
agent = create_react_agent(
    tools=[],
    model_name='claude-3-5-sonnet',
    system_prompt="You are a helpful coding assistant."
)
```

## Agent with Tools

### Using Built-in Search
```python
from valyu_tools import ValyuSearchTool

agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

result = agent.invoke({
    "messages": [HumanMessage("What are the latest AI trends?")]
})
```

### Multiple Tools
```python
from valyu_tools import ValyuSearchTool, ValyuContentsTool

agent = create_react_agent(
    tools=[ValyuSearchTool(), ValyuContentsTool()],
    model_name='claude-3-5-sonnet'
)
```

## Structured Output

### Define Schema
```python
from pydantic import BaseModel, Field
from typing import List

class Response(BaseModel):
    answer: str = Field(description="The answer")
    confidence: float = Field(description="Confidence 0-1")
    sources: List[str] = Field(description="Source URLs")

agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet',
    output_schema=Response
)
```

### Parse Response
```python
import json
result = agent.invoke({"messages": [HumanMessage("Question?")]})
data = json.loads(result["messages"][-1].content)
print(data['answer'])
```

## Custom Tools

### Basic Tool
```python
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

class CalculatorInput(BaseModel):
    expression: str = Field(description="Math expression")

class CalculatorTool(BaseTool):
    name: str = "calculator"
    description: str = "Evaluate math expressions"
    args_schema: Type[BaseModel] = CalculatorInput
    
    def _run(self, expression: str) -> str:
        try:
            result = eval(expression)
            return f"Result: {result}"
        except Exception as e:
            return f"Error: {str(e)}"

# Use it
agent = create_react_agent(
    tools=[CalculatorTool()],
    model_name='claude-3-5-sonnet'
)
```

## Conversation Memory

### Enable Memory
```python
from langgraph.checkpoint.memory import MemorySaver

agent = create_react_agent(
    tools=[...],
    checkpointer=MemorySaver(),
    model_name='claude-3-5-sonnet'
)

# Use with thread ID
config = {"configurable": {"thread_id": "user_123"}}

# Turn 1
agent.invoke({"messages": [HumanMessage("My name is Alice")]}, config)

# Turn 2 - remembers Alice
agent.invoke({"messages": [HumanMessage("What's my name?")]}, config)
```

## Common Patterns

### Research Pattern
```python
# Agent searches, reads, synthesizes
result = agent.invoke({
    "messages": [HumanMessage(
        "Research quantum computing and summarize the top 3 findings"
    )]
})
```

### Data Analysis Pattern
```python
# Agent queries data and provides insights
result = agent.invoke({
    "messages": [HumanMessage(
        "Analyze sales data for last quarter and identify trends"
    )]
})
```

### Multi-Step Workflow
```python
# Agent breaks down complex task
result = agent.invoke({
    "messages": [HumanMessage(
        "Find top AI companies, compare their products, send summary to team@company.com"
    )]
})
```

## Debugging

### Enable Logging
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Print Conversation
```python
result = agent.invoke({"messages": [HumanMessage("...")]})

for i, msg in enumerate(result["messages"]):
    print(f"{i}: {type(msg).__name__}: {msg.content[:100]}")
```

### Use LangSmith
```python
# In .env
LANGSMITH_API_KEY=your-key
LANGSMITH_PROJECT=my-project
LANGSMITH_TRACING=true

# View traces at smith.langchain.com
```

### Test Tool Independently
```python
tool = MyTool()
result = tool._run(param="test")
print(result)
```

## Model Selection

```python
# Claude (Best quality)
model_name='claude-3-5-sonnet'

# Claude (Fast)
model_name='claude-3-5-haiku'

# Llama (Open source)
model_name='llama3-2-90b'

# Amazon Nova
model_name='nova-lite'

# OpenAI (alternative)
from react_agent import get_chat_model
model = get_chat_model('gpt-5-mini', use_openai=True)
```

## Error Handling

### Try-Catch Pattern
```python
try:
    result = agent.invoke({"messages": [HumanMessage("...")]})
    print(result["messages"][-1].content)
except ValueError as e:
    print(f"Error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

### Tool Error Handling
```python
class MyTool(BaseTool):
    def _run(self, param: str) -> str:
        try:
            # Tool logic
            return result
        except Exception as e:
            return f"Error: {str(e)}"
```

## Environment Variables

```bash
# Required
HOLISTIC_AI_TEAM_ID=your-team-id
HOLISTIC_AI_API_TOKEN=your-token

# Optional
OPENAI_API_KEY=sk-your-key
VALYU_API_KEY=your-valyu-key
LANGSMITH_API_KEY=your-langsmith-key
LANGSMITH_PROJECT=my-project
LANGSMITH_TRACING=true
```

## Common Issues & Solutions

### "Module not found"
```bash
pip install -r requirements.txt --force-reinstall
```

### "API key not set"
```python
from dotenv import load_dotenv
load_dotenv()
```

### Agent too slow
```python
# Use faster model
model_name='claude-3-5-haiku'

# Reduce recursion limit
agent = graph.compile(recursion_limit=10)
```

### Tool not called
```python
# Improve tool description
description = "Use this when user asks about X. Returns Y."

# Make query more specific
"What's the weather?" → "What's the current weather in London?"
```

## File Structure

```
core/
├── react_agent/
│   ├── create_agent.py       # Agent factory
│   ├── holistic_ai_bedrock.py # Model wrapper
│   └── state.py              # State definitions
└── valyu_tools/
    └── tools.py              # Built-in tools
```

## Import Pattern

```python
import sys
sys.path.insert(0, './core')

from react_agent import create_react_agent
from valyu_tools import ValyuSearchTool
from langchain_core.messages import HumanMessage
```

## Useful Links

- **Repository**: [github.com/holistic-ai/hackthon-2025](https://github.com/holistic-ai/hackthon-2025)
- **Discord**: [discord.com/invite/QBTtWP2SU6](https://discord.com/invite/QBTtWP2SU6?referrer=luma)
- **Website**: [hackathon.holisticai.com](https://hackathon.holisticai.com/)
- **Tutorials**: [tutorials/Holistic_AI/](./README.md)
- **API Docs**: [docs/API_REFERENCE.md](../../docs/API_REFERENCE.md)

## Quick Tips

✅ **Do's:**
- Start simple, add complexity gradually
- Test tools independently first
- Use clear, specific queries
- Enable logging for debugging
- Check API credentials first

❌ **Don'ts:**
- Don't forget to load .env
- Don't use print() for debugging (use logging)
- Don't ignore error messages
- Don't create overly complex tools
- Don't forget timeouts in tools

## Next Steps

1. **New User?** → [01_Introduction](./01_Introduction_to_Holistic_AI.md)
2. **Want to Code?** → [02_Getting_Started](./02_Getting_Started.md)
3. **Need Examples?** → [11_Complete_Examples](./11_Complete_Examples.md)
4. **Having Issues?** → [12_Troubleshooting](./12_Troubleshooting_Guide.md)

---

**Print this page for quick reference!** 📄

*Last updated: November 2024*
