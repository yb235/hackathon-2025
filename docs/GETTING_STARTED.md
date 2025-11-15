# Getting Started Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Setup](#environment-setup)
- [First Agent](#first-agent)
- [Tutorial Path](#tutorial-path)
- [Choosing Your Track](#choosing-your-track)
- [Common Issues](#common-issues)

## Prerequisites

### System Requirements
- **Python**: 3.8 or higher
- **Operating System**: Linux, macOS, or Windows
- **RAM**: 4GB minimum, 8GB recommended
- **Disk Space**: 2GB for dependencies

### Account Setup

**For Hackathon Participants:**
1. **Form a team** (3-5 members) at UCL East - Marshgate, London
2. **Register your team**: [Team Registration Form](https://d3cayqk6b4dz0i.cloudfront.net/register.html?token=3crgPBFze8LnAaqDqc3KX8bq1Xu4qXhSnQ9eDycC8Tk)
3. **Receive credentials**: Team ID and API Token (AWS Bedrock access)
4. **For SageMaker**: DM Zekun Wu (`@zekunwu_73994`) on Discord

**For Practice/Learning:**
- Tutorial credentials provided in `.env.example`
- Optional: Get OpenAI API key at [platform.openai.com](https://platform.openai.com)

## Installation

### Step 1: Clone Repository

```bash
# Clone the repository
git clone https://github.com/holistic-ai/hackthon-2025.git

# Navigate to directory
cd hackthon-2025
```

### Step 2: Create Virtual Environment (Recommended)

```bash
# Create virtual environment
python -m venv venv

# Activate on Linux/macOS
source venv/bin/activate

# Activate on Windows
venv\Scripts\activate
```

### Step 3: Install Dependencies

```bash
# Install all dependencies
pip install -r requirements.txt

# Verify installation
python -c "import langgraph; print('LangGraph installed successfully!')"
```

**Dependencies installed:**
- LangGraph, LangChain Core, LangChain OpenAI
- Pydantic (data validation)
- Requests (HTTP client)
- Datasets, HuggingFace Hub
- LangSmith (observability)
- CodeCarbon (carbon tracking)
- Jupyter (notebooks)

## Environment Setup

### Step 1: Create .env File

```bash
# Copy example file
cp .env.example .env

# Edit with your preferred editor
nano .env  # or vim, code, etc.
```

### Step 2: Configure Credentials

**For Hackathon (After Registration):**

```bash
# Required: Holistic AI Bedrock credentials
HOLISTIC_AI_TEAM_ID=your-team-id-here
HOLISTIC_AI_API_TOKEN=your-team-api-token-here
```

**For Practice/Tutorial (Before Hackathon):**

```bash
# Use tutorial credentials
HOLISTIC_AI_TEAM_ID=tutorials_api
HOLISTIC_AI_API_TOKEN=SIcWmrU0745_QHALRull6gGpTPu3q268zCqGMrbQP4E
```

**Optional Configurations:**

```bash
# OpenAI (alternative model provider)
OPENAI_API_KEY=sk-your-openai-api-key

# Valyu (search tool - get free credits at platform.valyu.network)
VALYU_API_KEY=your-valyu-api-key

# LangSmith (observability - get free account at smith.langchain.com)
LANGSMITH_API_KEY=your-langsmith-api-key
LANGSMITH_PROJECT=hackathon-2025
LANGSMITH_TRACING=true
```

### Step 3: Verify Setup

```bash
# Test credentials
python -c "
import os
from dotenv import load_dotenv

load_dotenv()
team_id = os.getenv('HOLISTIC_AI_TEAM_ID')
api_token = os.getenv('HOLISTIC_AI_API_TOKEN')

if team_id and api_token:
    print('✅ Credentials loaded successfully!')
    print(f'Team ID: {team_id[:10]}...')
else:
    print('❌ Credentials not found. Check your .env file.')
"
```

## First Agent

### Quick Start (5 Minutes)

Create your first agent in a Python script or Jupyter notebook:

```python
# 1. Import dependencies
import sys
sys.path.insert(0, './core')
from react_agent import create_react_agent
from langchain_core.messages import HumanMessage

# 2. Create a simple agent (no tools)
agent = create_react_agent(
    tools=[],
    model_name='claude-3-5-sonnet'
)

# 3. Invoke the agent
result = agent.invoke({
    "messages": [HumanMessage(content="What is artificial intelligence?")]
})

# 4. Print the response
print(result["messages"][-1].content)
```

**Expected Output:**
```
✅ Using Holistic AI Bedrock Proxy: us.anthropic.claude-3-5-sonnet-20241022-v2:0
✅ Native tool calling enabled

Artificial intelligence (AI) refers to the simulation of human intelligence 
in machines that are programmed to think and learn like humans...
```

### Adding Tools (10 Minutes)

Add search capability to your agent:

```python
# 1. Import tools
from valyu_tools import ValyuSearchTool

# 2. Create agent with tools
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

# 3. Ask a question requiring current information
result = agent.invoke({
    "messages": [HumanMessage(content="What are the latest AI breakthroughs in 2024?")]
})

# 4. Print the response
print(result["messages"][-1].content)
```

**Expected Behavior:**
1. Agent decides it needs to search
2. Calls ValyuSearchTool with query
3. Tool returns search results
4. Agent synthesizes information into answer

### Conversation Memory (15 Minutes)

Enable multi-turn conversations:

```python
from langgraph.checkpoint.memory import MemorySaver

# 1. Create agent with checkpointer
agent = create_react_agent(
    tools=[],
    checkpointer=MemorySaver(),
    model_name='claude-3-5-sonnet'
)

# 2. Define conversation thread
config = {"configurable": {"thread_id": "conversation_1"}}

# 3. First turn
result1 = agent.invoke({
    "messages": [HumanMessage(content="My name is Alice")]
}, config)
print(result1["messages"][-1].content)

# 4. Second turn - agent remembers Alice
result2 = agent.invoke({
    "messages": [HumanMessage(content="What's my name?")]
}, config)
print(result2["messages"][-1].content)
# Output: "Your name is Alice"
```

### Structured Output (20 Minutes)

Get validated JSON responses:

```python
from pydantic import BaseModel
from typing import List

# 1. Define output schema
class ResearchSummary(BaseModel):
    topic: str
    key_points: List[str]
    conclusion: str
    confidence: float

# 2. Create agent with schema
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    output_schema=ResearchSummary,
    model_name='claude-3-5-sonnet'
)

# 3. Request structured output
result = agent.invoke({
    "messages": [HumanMessage(content="Research quantum computing")]
})

# 4. Parse structured response
import json
summary = ResearchSummary.parse_raw(result["messages"][-1].content)

print(f"Topic: {summary.topic}")
print(f"Key Points: {summary.key_points}")
print(f"Confidence: {summary.confidence}")
```

## Tutorial Path

### Recommended Learning Sequence

**Day 1 - Foundations (2-3 hours):**

1. **Tutorial 01: Basic Agent** (30 min)
   - Understand ReAct pattern
   - Create first agent
   - Learn message flow

2. **Tutorial 02: Custom Tools** (45 min)
   - Build custom tools
   - Define tool schemas
   - Handle tool errors

3. **Tutorial 03: Structured Output** (30 min)
   - Use Pydantic schemas
   - Validate responses
   - Parse JSON reliably

**Day 1 - Track Selection (1-2 hours):**

4. **Choose Your Track** (see below)
   - Read track requirements
   - Review resources
   - Start planning

**Day 2 - Deep Dive (3-4 hours):**

5. **Track-Specific Tutorials:**
   - **Track A**: Tutorial 04 (Monitoring), 06 (Evaluation)
   - **Track B**: Tutorial 05 (Observability)
   - **Track C**: Tutorial 08 (Red Teaming)

6. **Advanced Topics (Optional):**
   - Tutorial 07: Reinforcement Learning (advanced)

### Tutorial Access

**Jupyter Notebooks:**
```bash
# Start Jupyter
jupyter notebook

# Navigate to tutorials/
# Open desired notebook
```

**Google Colab:**
1. Upload notebook to Google Drive
2. Open with Google Colab
3. Install dependencies: `!pip install -r requirements.txt`

**Local Python Scripts:**
Convert notebook cells to a `.py` script and run.

## Choosing Your Track

### Track A: Agent Iron Man
**Focus:** Reliability, performance, cost efficiency

**Best For:**
- Building production-ready systems
- Optimizing performance
- Measuring impact

**Key Skills:**
- Error handling
- Performance monitoring
- Cost optimization
- Testing and benchmarking

**Start Here:**
1. Tutorial 04 (Model Monitoring)
2. Tutorial 06 (Benchmark Evaluation)
3. Review `track_a_iron_man/README.md`
4. Explore `track_a_iron_man/examples/`

### Track B: Agent Glass Box
**Focus:** Observability, explainability, transparency

**Best For:**
- Understanding agent behavior
- Building trust in AI
- Debugging complex systems

**Key Skills:**
- Tracing execution
- Visualizing reasoning
- Analyzing failures
- Building dashboards

**Start Here:**
1. Tutorial 05 (Observability)
2. Review `track_b_glass_box/README.md`
3. Analyze `track_b_glass_box/traces/`
4. Study AgentGraph and AgentSeer papers

### Track C: Dear Grandma
**Focus:** Security, red teaming, vulnerability assessment

**Best For:**
- Security testing
- Finding vulnerabilities
- Building robust defenses

**Key Skills:**
- Attack methodologies
- ASR measurement
- Systematic testing
- Security analysis

**Start Here:**
1. Tutorial 08 (Attack & Red Teaming)
2. Review `track_c_dear_grandma/README.md`
3. Test API endpoints with curl
4. Explore `track_c_dear_grandma/examples/red_teaming_datasets/`

### Can I Do Multiple Tracks?

**Yes!** You can submit to all three tracks. Recommended approach:

1. **Pick a primary track** based on your team's strengths
2. **Build your solution** with that track in mind
3. **Adapt for other tracks** by:
   - Track A: Add monitoring and benchmarks
   - Track B: Add observability and tracing
   - Track C: Add security testing

## Common Issues

### "Module not found" Error

**Problem:**
```
ModuleNotFoundError: No module named 'langgraph'
```

**Solution:**
```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Verify installation
pip list | grep langgraph
```

### "API key not found" Error

**Problem:**
```
ValueError: HOLISTIC_AI_TEAM_ID and HOLISTIC_AI_API_TOKEN not set
```

**Solution:**
1. Check `.env` file exists in project root
2. Verify credentials are correct (no extra spaces)
3. Reload environment:
   ```python
   from dotenv import load_dotenv
   load_dotenv(override=True)
   ```

### Jupyter Won't Start

**Problem:**
```
jupyter: command not found
```

**Solution:**
```bash
# Install Jupyter
pip install jupyter

# Start on different port if 8888 is busy
jupyter notebook --port=8889
```

### Model Connection Errors

**Problem:**
```
Error calling Holistic AI Bedrock API: Connection timeout
```

**Solutions:**
1. Check internet connection
2. Verify credentials are correct
3. Try alternative model:
   ```python
   agent = create_react_agent(
       tools=[...],
       model_name='claude-3-5-haiku'  # Faster alternative
   )
   ```
4. Check API status on Discord

### Import Path Issues

**Problem:**
```
ModuleNotFoundError: No module named 'react_agent'
```

**Solution:**
```python
# Add core to path
import sys
sys.path.insert(0, './core')  # From project root
# or
sys.path.insert(0, '../core')  # From tutorials/
```

### Tool Execution Timeouts

**Problem:**
Tool calls hang or timeout

**Solution:**
```python
# Increase timeout in context
from react_agent import Context

context = Context(ollama_timeout=120)  # 2 minutes
agent = create_react_agent(tools=[...], context=context)
```

## Getting Help

### During Hackathon

**Discord (Fastest):**
- Join: [discord.com/invite/QBTtWP2SU6](https://discord.com/invite/QBTtWP2SU6?referrer=luma)
- Channel: `#ask-for-help`
- Contact: Zekun Wu (`@zekunwu_73994`)

**GitHub Issues:**
- Report bugs: [github.com/holistic-ai/hackthon-2025/issues](https://github.com/holistic-ai/hackthon-2025/issues)
- See examples: [EXAMPLE_ISSUES.md](../docs/EXAMPLE_ISSUES.md)

**Email:**
- Direct contact: zekun.wu@holisticai.com

### Resources

- **Documentation**: `/docs` folder
- **Tutorials**: `/tutorials` folder
- **API Guide**: `/assets/api-guide.pdf`
- **Track Guides**: Each track's `README.md`

## Next Steps

1. **✅ Complete First Agent** - You should have a working agent now
2. **📚 Start Tutorials** - Begin with Tutorial 01
3. **🎯 Choose Track** - Review track requirements
4. **🔨 Build Project** - Start building your hackathon submission
5. **📊 Monitor Progress** - Track your agent's performance
6. **🚀 Submit** - Prepare poster and GitHub repository

**Ready to build?** Head to the [tutorials](../tutorials/) or [track guides](TRACK_GUIDES.md)!

---

## Quick Reference

### Essential Commands

```bash
# Start Jupyter
jupyter notebook

# Run Python script
python my_agent.py

# Install dependency
pip install package-name

# Check environment
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print(os.getenv('HOLISTIC_AI_TEAM_ID'))"
```

### Essential Imports

```python
# Agent creation
from react_agent import create_react_agent
from react_agent import Context

# Messages
from langchain_core.messages import HumanMessage, AIMessage

# Tools
from valyu_tools import ValyuSearchTool, ValyuContentsTool

# Memory
from langgraph.checkpoint.memory import MemorySaver

# Structured output
from pydantic import BaseModel
```

### Essential Patterns

```python
# Basic agent
agent = create_react_agent(tools=[], model_name='claude-3-5-sonnet')

# With tools
agent = create_react_agent(tools=[ValyuSearchTool()], model_name='claude-3-5-sonnet')

# With memory
agent = create_react_agent(tools=[], checkpointer=MemorySaver())

# With structured output
agent = create_react_agent(tools=[], output_schema=MySchema)

# Invoke
result = agent.invoke({"messages": [HumanMessage("query")]})

# Get response
response = result["messages"][-1].content
```

---

**Happy Building! 🚀**
