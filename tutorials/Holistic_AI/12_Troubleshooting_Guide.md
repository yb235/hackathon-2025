# Troubleshooting Guide - Solving Common Issues

This guide helps you solve common problems when working with Holistic AI. Find your issue, understand why it happens, and learn how to fix it.

## Table of Contents
- [Installation Issues](#installation-issues)
- [Environment and Configuration](#environment-and-configuration)
- [API and Authentication](#api-and-authentication)
- [Agent Execution Issues](#agent-execution-issues)
- [Tool Problems](#tool-problems)
- [Performance Issues](#performance-issues)
- [Common Error Messages](#common-error-messages)
- [Debugging Strategies](#debugging-strategies)
- [Getting Help](#getting-help)

## Installation Issues

### Problem: "Module not found" errors

**Error message**:
```
ModuleNotFoundError: No module named 'langgraph'
```

**Causes**:
1. Dependencies not installed
2. Wrong Python environment
3. Corrupted installation

**Solutions**:

```bash
# Solution 1: Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Solution 2: Check Python environment
python --version  # Should be 3.8+
pip --version      # Should match Python version

# Solution 3: Use virtual environment
python -m venv venv
source venv/bin/activate  # On Mac/Linux
# or
venv\Scripts\activate  # On Windows
pip install -r requirements.txt

# Solution 4: Install specific package
pip install langgraph langchain-core langchain-openai
```

### Problem: "Permission denied" during installation

**Error message**:
```
PermissionError: [Errno 13] Permission denied
```

**Solutions**:

```bash
# Solution 1: Install for current user only
pip install -r requirements.txt --user

# Solution 2: Use virtual environment (recommended)
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Solution 3: Run with sudo (NOT recommended)
# Only if absolutely necessary and you understand the risks
sudo pip install -r requirements.txt
```

### Problem: "pip not found" or "python not found"

**Solutions**:

```bash
# On Windows
py -m pip install -r requirements.txt

# On Mac/Linux with python3
python3 -m pip install -r requirements.txt

# Upgrade pip
python -m pip install --upgrade pip
```

## Environment and Configuration

### Problem: "HOLISTIC_AI_TEAM_ID not set"

**Error message**:
```
ValueError: HOLISTIC_AI_TEAM_ID and HOLISTIC_AI_API_TOKEN not set.
```

**Causes**:
1. `.env` file doesn't exist
2. `.env` file not in the correct location
3. Environment variables not loaded
4. Typo in variable names

**Solutions**:

```python
# Solution 1: Check .env file exists
import os
print(os.path.exists('.env'))  # Should print True

# Solution 2: Verify .env content
with open('.env', 'r') as f:
    print(f.read())
# Should see HOLISTIC_AI_TEAM_ID=...

# Solution 3: Load environment explicitly
from dotenv import load_dotenv
load_dotenv()
import os
print(os.getenv('HOLISTIC_AI_TEAM_ID'))  # Should print your team ID

# Solution 4: Set environment variables directly (temporary)
import os
os.environ['HOLISTIC_AI_TEAM_ID'] = 'your-team-id'
os.environ['HOLISTIC_AI_API_TOKEN'] = 'your-token'

# Solution 5: Check for extra spaces/quotes
# In .env file, use:
HOLISTIC_AI_TEAM_ID=team123
# NOT:
# HOLISTIC_AI_TEAM_ID="team123"  # Remove quotes
# HOLISTIC_AI_TEAM_ID = team123  # Remove spaces around =
```

### Problem: `.env` file not being loaded

**Causes**:
1. Wrong working directory
2. `python-dotenv` not installed
3. `.env` loaded after imports

**Solutions**:

```python
# Solution 1: Check working directory
import os
print("Current directory:", os.getcwd())
print(".env exists:", os.path.exists('.env'))

# Solution 2: Load from specific path
from dotenv import load_dotenv
load_dotenv('/full/path/to/.env')

# Solution 3: Load at the very start
from dotenv import load_dotenv
load_dotenv()  # FIRST LINE after imports

# Rest of your code...
import sys
sys.path.insert(0, './core')
from react_agent import create_react_agent

# Solution 4: Verify dotenv is installed
pip install python-dotenv
```

## API and Authentication

### Problem: "Error calling Holistic AI Bedrock API"

**Error message**:
```
ValueError: Error calling Holistic AI Bedrock API: 401 Unauthorized
```

**Causes**:
1. Invalid credentials
2. Expired credentials
3. Incorrect API endpoint

**Solutions**:

```python
# Solution 1: Verify credentials
import os
from dotenv import load_dotenv
load_dotenv()

team_id = os.getenv('HOLISTIC_AI_TEAM_ID')
api_token = os.getenv('HOLISTIC_AI_API_TOKEN')

print(f"Team ID: {team_id}")
print(f"Token length: {len(api_token) if api_token else 0}")

# Should print:
# Team ID: your-actual-team-id
# Token length: 40+ (typical token length)

# Solution 2: Test API connection
import requests

response = requests.post(
    "https://ctwa92wg1b.execute-api.us-east-1.amazonaws.com/prod/invoke",
    headers={
        "Content-Type": "application/json",
        "X-Team-ID": team_id,
        "X-API-Token": api_token
    },
    json={
        "team_id": team_id,
        "api_token": api_token,
        "model": "us.anthropic.claude-3-5-sonnet-20241022-v2:0",
        "messages": [{"role": "user", "content": "test"}],
        "max_tokens": 100
    }
)

print(f"Status: {response.status_code}")
print(f"Response: {response.text}")

# Solution 3: Contact organizers for new credentials
# During hackathon, DM @zekunwu_73994 on Discord
```

### Problem: "OpenAI API key not found"

**Solutions**:

```python
# Solution 1: Use Holistic AI Bedrock instead (recommended)
agent = create_react_agent(
    tools=[...],
    model_name='claude-3-5-sonnet'  # Uses Bedrock, not OpenAI
)

# Solution 2: Add OpenAI key to .env
# In .env file:
OPENAI_API_KEY=sk-your-actual-key-here

# Solution 3: Explicitly specify Bedrock
from react_agent import get_chat_model

model = get_chat_model('claude-3-5-sonnet', use_openai=False)
```

## Agent Execution Issues

### Problem: Agent is very slow

**Symptoms**:
- Takes 30+ seconds for simple queries
- Multiple tool calls for simple tasks
- Timeout errors

**Solutions**:

```python
# Solution 1: Use faster model
agent = create_react_agent(
    tools=[...],
    model_name='claude-3-5-haiku'  # Much faster than sonnet
)

# Solution 2: Reduce tool calls with better prompt
context = Context(
    system_prompt="""Answer directly when possible. 
    Only use tools when necessary for current information."""
)
agent = create_react_agent(tools=[...], context=context)

# Solution 3: Set lower recursion limit
from langgraph.graph import StateGraph
graph = StateGraph(State)
# ... build graph ...
agent = graph.compile(recursion_limit=10)  # Default is 25

# Solution 4: Use fast mode for Valyu search
search_tool = ValyuSearchTool()
result = search_tool._run(
    query="...",
    fast_mode=True  # Faster but shorter results
)

# Solution 5: Disable tools for simple queries
simple_agent = create_react_agent(
    tools=[],  # No tools = faster
    model_name='claude-3-5-haiku'
)
```

### Problem: Agent gives incomplete responses

**Symptoms**:
- Response ends mid-sentence
- "Could not complete in specified steps"
- Missing information

**Solutions**:

```python
# Solution 1: Increase recursion limit
from langgraph.graph import StateGraph
graph = StateGraph(State)
# ... build graph ...
agent = graph.compile(recursion_limit=50)  # Increased from 25

# Solution 2: Break down complex queries
# Instead of:
result = agent.invoke({
    "messages": [HumanMessage(
        "Research AI, quantum computing, and blockchain, then create a comprehensive report"
    )]
})

# Do:
topics = ["AI", "quantum computing", "blockchain"]
results = []
for topic in topics:
    result = agent.invoke({
        "messages": [HumanMessage(f"Research {topic}")]
    })
    results.append(result)

# Solution 3: Increase max_tokens
from pydantic import SecretStr
model = HolisticAIBedrockChat(
    team_id=...,
    api_token=SecretStr(...),
    max_tokens=4096  # Increased from 1024
)
```

### Problem: "Recursion limit reached"

**Error message**:
```
RecursionError: Maximum recursion limit of 25 reached
```

**Causes**:
1. Task is too complex
2. Agent stuck in loop
3. Tool keeps failing

**Solutions**:

```python
# Solution 1: Increase limit
agent = graph.compile(recursion_limit=50)

# Solution 2: Simplify task
# Break into smaller sub-tasks

# Solution 3: Check tool errors
# Enable detailed logging
import logging
logging.basicConfig(level=logging.DEBUG)

# Solution 4: Improve system prompt
context = Context(
    system_prompt="""Be efficient. Minimize tool calls.
    If a tool fails, try a different approach rather than retrying."""
)
```

## Tool Problems

### Problem: Tool not being called

**Symptoms**:
- Agent responds without using tools
- "I don't have access to..." responses

**Solutions**:

```python
# Solution 1: Verify tools are registered
agent = create_react_agent(
    tools=[search_tool, other_tool],  # Make sure tools are in list
    model_name='claude-3-5-sonnet'
)

# Solution 2: Check tool description
class MyTool(BaseTool):
    name = "my_tool"
    description = """
    Use this tool when the user asks about X, Y, or Z.
    This tool does A, B, and C.
    """  # Clear, detailed description

# Solution 3: Improve query
# Vague:
"Tell me about the weather"

# Clear:
"What's the current weather in London?"

# Solution 4: Check model supports tools
# These support tools:
# - claude-3-5-sonnet ✓
# - claude-3-5-haiku ✓
# - gpt-5-mini ✓
# - llama3-2-90b ✓

# Solution 5: Verify tool binding
print(" Native tool calling enabled" in agent output)
```

### Problem: Tool returns errors

**Error message from tool**:
```
Error: Invalid input / API error / etc.
```

**Solutions**:

```python
# Solution 1: Test tool independently
tool = MyTool()
result = tool._run(param="test")
print(result)  # Should work without agent

# Solution 2: Add better error handling
class MyTool(BaseTool):
    def _run(self, param: str) -> str:
        try:
            result = do_something(param)
            return result
        except ValueError as e:
            return f"Error: Invalid input - {str(e)}"
        except requests.Timeout:
            return "Error: Request timed out. Please try again."
        except Exception as e:
            return f"Error: {str(e)}"

# Solution 3: Validate inputs
class MyToolInput(BaseModel):
    param: str = Field(description="Parameter description")
    
    @validator('param')
    def validate_param(cls, v):
        if not v or not v.strip():
            raise ValueError("Parameter cannot be empty")
        return v

# Solution 4: Add timeouts
response = requests.get(url, timeout=30)  # Add timeout
```

### Problem: Valyu tools not working

**Solutions**:

```bash
# Solution 1: Install Valyu package
pip install valyu

# Solution 2: Set API key
# In .env:
VALYU_API_KEY=your-valyu-key-here

# Solution 3: Check quota/limits
# Contact Valyu support if you've exceeded limits

# Solution 4: Use alternative search
# If Valyu unavailable, implement backup:
from langchain_community.tools import DuckDuckGoSearchRun

class BackupSearchTool(BaseTool):
    name = "search"
    description = "Search the web"
    
    def _run(self, query: str) -> str:
        search = DuckDuckGoSearchRun()
        return search.run(query)
```

## Performance Issues

### Problem: High costs

**Symptoms**:
- Unexpectedly high API charges
- Many tokens used

**Solutions**:

```python
# Solution 1: Use cheaper model
agent = create_react_agent(
    tools=[...],
    model_name='claude-3-5-haiku'  # 5x cheaper than sonnet
)

# Solution 2: Monitor token usage
import tiktoken

def count_tokens(text: str) -> int:
    encoding = tiktoken.encoding_for_model("claude-3-5-sonnet")
    return len(encoding.encode(text))

tokens = count_tokens(result["messages"][-1].content)
print(f"Tokens used: {tokens}")

# Solution 3: Reduce max_tokens
model = HolisticAIBedrockChat(
    team_id=...,
    api_token=...,
    max_tokens=1024  # Limit response length
)

# Solution 4: Cache repeated queries
cache = {}

def cached_invoke(agent, query):
    if query in cache:
        return cache[query]
    result = agent.invoke({"messages": [HumanMessage(query)]})
    cache[query] = result
    return result

# Solution 5: Use search efficiently
search_tool._run(
    query="...",
    max_num_results=5,  # Reduce from default 10
    fast_mode=True  # Use fast mode
)
```

### Problem: Memory issues

**Symptoms**:
- "MemoryError"
- System slows down
- Process killed

**Solutions**:

```python
# Solution 1: Clear conversation history
# Don't use checkpointer for long conversations
agent = create_react_agent(
    tools=[...],
    checkpointer=None  # Stateless
)

# Solution 2: Limit conversation length
def truncate_messages(messages, max_messages=10):
    """Keep only recent messages"""
    if len(messages) > max_messages:
        # Keep first (system) and last N messages
        return [messages[0]] + messages[-max_messages:]
    return messages

# Solution 3: Use streaming
for chunk in agent.stream({"messages": [HumanMessage("...")]}):
    process_chunk(chunk)
    # Process incrementally instead of all at once

# Solution 4: Reduce max_tokens
# Smaller responses = less memory
```

## Common Error Messages

### "Tool 'X' not found"

**Solution**:
```python
# Make sure tool is in the tools list
agent = create_react_agent(
    tools=[tool1, tool2, tool3],  # Include all tools
    model_name='claude-3-5-sonnet'
)
```

### "Invalid message format"

**Solution**:
```python
# Use proper message types
from langchain_core.messages import HumanMessage, AIMessage

# Correct:
messages = [HumanMessage(content="Hello")]

# Wrong:
messages = ["Hello"]  # Plain strings don't work
messages = [{"content": "Hello"}]  # Dicts don't work
```

### "JSON decode error"

**Solution**:
```python
# When using structured output, ensure valid JSON
try:
    data = json.loads(result["messages"][-1].content)
except json.JSONDecodeError:
    # Handle parsing error
    print("Failed to parse response as JSON")
    # Try extracting JSON from text
    import re
    json_match = re.search(r'\{.*\}', content, re.DOTALL)
    if json_match:
        data = json.loads(json_match.group())
```

## Debugging Strategies

### Enable Debug Logging

```python
import logging

# Set up detailed logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# Now run your agent
result = agent.invoke(...)
# Will print detailed execution information
```

### Use LangSmith for Tracing

```python
# In .env file:
LANGSMITH_API_KEY=your-key
LANGSMITH_PROJECT=debugging
LANGSMITH_TRACING=true

# Run agent
result = agent.invoke(...)

# View trace at: https://smith.langchain.com
```

### Print Intermediate Steps

```python
# Print each message in conversation
result = agent.invoke({"messages": [HumanMessage("...")]})

print("\n=== CONVERSATION HISTORY ===")
for i, msg in enumerate(result["messages"]):
    print(f"\n--- Message {i+1}: {type(msg).__name__} ---")
    print(msg.content)
    if hasattr(msg, 'tool_calls') and msg.tool_calls:
        print(f"Tool calls: {msg.tool_calls}")
```

### Test Components Separately

```python
# Test 1: Model connection
from react_agent import get_chat_model
model = get_chat_model('claude-3-5-sonnet')
response = model.invoke([HumanMessage("test")])
print("Model works:", "test" not in str(response).lower())

# Test 2: Tool execution
tool = MyTool()
result = tool._run(param="test")
print("Tool works:", "error" not in result.lower())

# Test 3: Agent creation
agent = create_react_agent(tools=[tool])
print("Agent created:", agent is not None)

# Test 4: Simple invocation
result = agent.invoke({"messages": [HumanMessage("Hi")]})
print("Agent responds:", len(result["messages"]) > 1)
```

## Getting Help

### Before Asking for Help

1. **Check this guide** - Your issue might be covered
2. **Search existing issues** - Someone may have had the same problem
3. **Enable debug logging** - Get detailed error information
4. **Create minimal example** - Isolate the problem

### Where to Get Help

1. **Documentation**
   - [API Reference](../../docs/API_REFERENCE.md)
   - [FAQ](../../docs/FAQ.md)
   - [Architecture](../../docs/ARCHITECTURE.md)

2. **Discord Community**
   - [Join Discord](https://discord.com/invite/QBTtWP2SU6?referrer=luma)
   - #ask-for-help channel
   - DM @zekunwu_73994 for API issues

3. **GitHub Issues**
   - [Search issues](https://github.com/holistic-ai/hackthon-2025/issues)
   - [Create new issue](https://github.com/holistic-ai/hackthon-2025/issues/new)

4. **Email Support**
   - zekun.wu@holisticai.com

### When Asking for Help, Include:

1. **Error message** (full text)
2. **Code snippet** (minimal example)
3. **Environment info**:
   ```python
   import sys
   print(f"Python: {sys.version}")
   import langgraph
   print(f"LangGraph: {langgraph.__version__}")
   ```
4. **What you've tried** (solutions attempted)
5. **Expected vs actual** behavior

### Example Help Request

```
Title: Agent not using search tool

Environment:
- Python 3.10.5
- LangGraph 0.2.0
- Model: claude-3-5-sonnet

Problem:
My agent responds without using the search tool even though the 
question requires current information.

Code:
```python
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)
result = agent.invoke({
    "messages": [HumanMessage("What's the weather?")]
})
```

Expected: Agent uses search tool to get current weather
Actual: Agent says "I don't have access to current weather"

Tried:
1. Verified tool is in tools list
2. Checked that model supports tools (it does)
3. Tool works when tested independently

Debug logs attached: [...]
```

## Key Takeaways

✅ **Most issues have simple fixes** - Check this guide first
✅ **Test components independently** - Isolate the problem
✅ **Enable logging** - Get detailed error information
✅ **Ask for help** - Community is here to help
✅ **Document solutions** - Help others with similar issues

---

**Still stuck?** Don't hesitate to reach out on [Discord](https://discord.com/invite/QBTtWP2SU6?referrer=luma)! 💬
