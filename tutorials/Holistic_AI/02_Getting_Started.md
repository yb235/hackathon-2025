# Getting Started with Holistic AI 2 - Installation and First Agent

Welcome! In this tutorial, you'll install everything you need and create your first working AI agent. By the end, you'll have a functioning agent that can answer questions and use tools.

## Table of Contents
- [System Requirements](#system-requirements)
- [Installation Steps](#installation-steps)
- [Environment Configuration](#environment-configuration)
- [Creating Your First Agent](#creating-your-first-agent)
- [Understanding What Happened](#understanding-what-happened)
- [Testing Your Agent](#testing-your-agent)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

## System Requirements

Before starting, ensure you have:

### Required ✅
- **Python 3.8 or higher**
  ```bash
  python --version  # Should show 3.8+
  ```
- **pip** (Python package manager)
  ```bash
  pip --version
  ```
- **Internet connection** (for downloading packages and API calls)
- **4GB+ RAM** (8GB recommended)
- **Text editor or IDE** (VS Code, PyCharm, or even Notepad++)

### Optional but Recommended 📦
- **Jupyter Notebook** (for interactive development)
- **Git** (for version control)
- **Virtual environment tool** (venv, conda, or virtualenv)

## Installation Steps

### Step 1: Clone the Repository

First, get a copy of the Holistic AI code:

```bash
# Clone the repository
git clone https://github.com/holistic-ai/hackthon-2025.git

# Navigate to the directory
cd hackthon-2025
```

**Don't have Git?** Download the ZIP file from GitHub and extract it.

### Step 2: Create a Virtual Environment (Recommended)

Creating a virtual environment keeps your project dependencies isolated:

```bash
# Create virtual environment
python -m venv venv

# Activate it
# On Windows:
venv\Scripts\activate

# On Mac/Linux:
source venv/bin/activate
```

You'll see `(venv)` in your terminal prompt when activated.

**Why use a virtual environment?**
- Keeps project dependencies separate
- Avoids conflicts with other Python projects
- Makes it easy to replicate your setup

### Step 3: Install Dependencies

Now install all required packages:

```bash
# Install from requirements file
pip install -r requirements.txt
```

This installs:
- **LangGraph**: Agent orchestration framework
- **LangChain**: LLM application framework
- **Pydantic**: Data validation
- **Requests**: HTTP library for API calls
- **Python-dotenv**: Environment variable management
- **And more...**

**Installation takes 2-5 minutes** depending on your internet speed.

### Step 4: Verify Installation

Check that key packages installed correctly:

```bash
python -c "import langgraph; print('LangGraph version:', langgraph.__version__)"
python -c "import langchain; print('LangChain installed ✓')"
python -c "import pydantic; print('Pydantic installed ✓')"
```

All commands should complete without errors.

## Environment Configuration

### Step 1: Create .env File

Copy the example environment file:

```bash
# Copy the template
cp .env.example .env
```

**On Windows**, you might need to use:
```bash
copy .env.example .env
```

### Step 2: Get API Credentials

You need credentials to use AI models. During the hackathon, you'll receive:

**Holistic AI Bedrock Credentials** (Provided at Event):
- Team ID
- API Token

These give you access to Claude, Llama, and other models via AWS Bedrock.

**Alternative - OpenAI** (Optional):
If you want to use GPT models, get an API key from [OpenAI](https://platform.openai.com/).

### Step 3: Configure Your .env File

Open `.env` in your text editor and add your credentials:

```bash
# Required - Holistic AI Bedrock (provided at hackathon)
HOLISTIC_AI_TEAM_ID=your-team-id-here
HOLISTIC_AI_API_TOKEN=your-api-token-here

# Optional - OpenAI (if you want to use GPT models)
OPENAI_API_KEY=sk-your-key-here

# Optional - Valyu Search (for advanced search features)
VALYU_API_KEY=your-valyu-key-here

# Optional - LangSmith (for tracing and debugging)
LANGSMITH_API_KEY=your-langsmith-key-here
LANGSMITH_PROJECT=my-hackathon-project
LANGSMITH_TRACING=true
```

**Important**: Replace `your-team-id-here` and `your-api-token-here` with your actual credentials!

**Security Note**: Never commit the `.env` file to Git. It's already in `.gitignore` to prevent accidental exposure of secrets.

### Step 4: Verify Configuration

Test that your credentials work:

```bash
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print('Team ID:', os.getenv('HOLISTIC_AI_TEAM_ID'))"
```

You should see your Team ID printed (not 'None').

## Creating Your First Agent

Now for the exciting part - let's create an AI agent!

### Step 1: Create a Python Script

Create a new file called `my_first_agent.py`:

```python
"""My First Holistic AI Agent"""

import sys
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Add core modules to path
sys.path.insert(0, './core')

# Import Holistic AI components
from react_agent import create_react_agent
from langchain_core.messages import HumanMessage

# Step 1: Create the agent
print("🤖 Creating your first AI agent...")
agent = create_react_agent(
    tools=[],  # No tools for now - we'll add them later
    model_name='claude-3-5-sonnet'  # Using Claude 3.5 Sonnet
)
print("✅ Agent created successfully!")

# Step 2: Ask the agent a question
print("\n💭 Asking the agent a question...")
question = "What is an AI agent and how does it work?"

result = agent.invoke({
    "messages": [HumanMessage(content=question)]
})

# Step 3: Display the response
print("\n🤖 Agent's Response:")
print("-" * 60)
print(result["messages"][-1].content)
print("-" * 60)

print("\n✨ Success! Your first agent is working!")
```

### Step 2: Run Your Agent

Execute the script:

```bash
python my_first_agent.py
```

You should see output like:

```
🤖 Creating your first AI agent...
 Native tool calling enabled
✅ Agent created successfully!

💭 Asking the agent a question...

🤖 Agent's Response:
------------------------------------------------------------
An AI agent is an intelligent system that combines large language 
models with the ability to reason, make decisions, and take actions 
to accomplish tasks autonomously...

[Full response continues...]
------------------------------------------------------------

✨ Success! Your first agent is working!
```

**Congratulations!** 🎉 You've just created and run your first AI agent!

### What Just Happened?

Let's break down the code step by step:

```python
# 1. Import required modules
from react_agent import create_react_agent
from langchain_core.messages import HumanMessage
```
We import:
- `create_react_agent`: Factory function to build agents
- `HumanMessage`: Represents a message from the user

```python
# 2. Create the agent
agent = create_react_agent(
    tools=[],  # No tools yet
    model_name='claude-3-5-sonnet'
)
```
This:
- Creates a ReAct agent using Claude 3.5 Sonnet model
- Doesn't add any tools yet (we'll do that in the next tutorial)
- Returns a compiled agent ready to use

```python
# 3. Invoke the agent
result = agent.invoke({
    "messages": [HumanMessage(content=question)]
})
```
This:
- Sends your question to the agent
- The agent processes it and generates a response
- Returns a result dictionary with all messages

```python
# 4. Extract the response
print(result["messages"][-1].content)
```
This:
- Gets the last message from the result (the agent's response)
- Prints the content

## Understanding What Happened

Behind the scenes, here's what occurred:

```
┌─────────────────────────────────────────────────────┐
│ 1. You invoke agent with question                   │
└────────────────────┬────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────┐
│ 2. Agent enters "call_model" node                   │
│    - Adds system prompt                             │
│    - Sends messages to Claude model                 │
└────────────────────┬────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────┐
│ 3. Claude processes the question                    │
│    - Understands what you're asking                 │
│    - Generates a thoughtful response                │
└────────────────────┬────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────┐
│ 4. Agent receives Claude's response                 │
│    - No tool calls needed                           │
│    - Routes to END                                  │
└────────────────────┬────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────┐
│ 5. Result returned to you                           │
│    - Contains full conversation                     │
│    - You print the final response                   │
└─────────────────────────────────────────────────────┘
```

## Testing Your Agent

Let's test the agent with different questions to see how it responds:

### Example 1: Simple Factual Question

```python
result = agent.invoke({
    "messages": [HumanMessage(content="What is Python?")]
})
print(result["messages"][-1].content)
```

The agent will explain what Python is using its training knowledge.

### Example 2: Explanation Request

```python
result = agent.invoke({
    "messages": [HumanMessage(content="Explain how neural networks work in simple terms")]
})
print(result["messages"][-1].content)
```

The agent breaks down complex concepts into understandable explanations.

### Example 3: Creative Task

```python
result = agent.invoke({
    "messages": [HumanMessage(content="Write a haiku about AI")]
})
print(result["messages"][-1].content)
```

The agent can handle creative requests too!

### Create an Interactive Loop

Want to have a conversation? Try this:

```python
def chat_with_agent():
    """Interactive chat with the agent"""
    print("💬 Chat with your AI agent (type 'exit' to quit)\n")
    
    while True:
        # Get user input
        question = input("You: ")
        
        # Check for exit
        if question.lower() in ['exit', 'quit', 'bye']:
            print("👋 Goodbye!")
            break
        
        # Ask the agent
        result = agent.invoke({
            "messages": [HumanMessage(content=question)]
        })
        
        # Print response
        print(f"\n🤖 Agent: {result['messages'][-1].content}\n")

# Run the chat
chat_with_agent()
```

**Note**: This simple version doesn't maintain conversation history. We'll add that feature in Tutorial 06 on State Management.

## Exploring Different Models

Holistic AI supports multiple models. Try changing the model:

### Using Claude 3.5 Haiku (Faster, Cheaper)

```python
agent = create_react_agent(
    tools=[],
    model_name='claude-3-5-haiku'  # Faster model
)
```

### Using Llama 3.2 90B

```python
agent = create_react_agent(
    tools=[],
    model_name='llama3-2-90b'  # Meta's Llama model
)
```

### Using Amazon Nova

```python
agent = create_react_agent(
    tools=[],
    model_name='nova-lite'  # Amazon's model
)
```

Each model has different characteristics:
- **Claude 3.5 Sonnet**: Best quality, most capable
- **Claude 3.5 Haiku**: Fast, cost-effective
- **Llama 3.2**: Open source, good performance
- **Nova**: AWS native, good balance

## Troubleshooting

### Problem: "Module not found" errors

**Solution**:
```bash
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall

# Verify path
python -c "import sys; print(sys.path)"
```

### Problem: "HOLISTIC_AI_TEAM_ID not set"

**Solution**:
1. Check that `.env` file exists
2. Verify credentials are correct (no extra spaces)
3. Try loading manually:
   ```python
   from dotenv import load_dotenv
   load_dotenv()
   import os
   print(os.getenv('HOLISTIC_AI_TEAM_ID'))
   ```

### Problem: "Error calling Holistic AI Bedrock API"

**Solution**:
1. Verify your Team ID and API Token are correct
2. Check internet connection
3. Confirm credentials are active (contact organizers if during hackathon)

### Problem: Agent is slow

**Explanation**: 
- First call takes 5-10 seconds (loading model)
- Subsequent calls are faster (1-3 seconds)
- Try a faster model like `claude-3-5-haiku`

### Problem: Import errors with core modules

**Solution**:
```python
import sys
import os

# Add core to path
sys.path.insert(0, os.path.join(os.getcwd(), 'core'))
```

### Still Having Issues?

1. Check [FAQ](../../docs/FAQ.md)
2. Search [GitHub Issues](https://github.com/holistic-ai/hackthon-2025/issues)
3. Ask on [Discord](https://discord.com/invite/QBTtWP2SU6?referrer=luma)

## Practice Exercises

Try these to reinforce your learning:

### Exercise 1: Ask Different Questions
Create a script that asks your agent 5 different questions and prints all responses.

### Exercise 2: Model Comparison
Create the same agent with different models and compare response times and quality.

### Exercise 3: Error Handling
Add try-except blocks to handle potential errors gracefully.

**Solution hints** available in `examples/` folder.

## Key Takeaways

✅ You now know how to:
1. Install Holistic AI and dependencies
2. Configure environment variables
3. Create a basic agent
4. Invoke the agent with questions
5. Switch between different models
6. Debug common issues

## What's Next?

In the next tutorial, you'll learn about the **architecture** of Holistic AI:
- How components fit together
- The ReAct pattern in detail
- State management
- Message flow

**Continue to**: [03_Understanding_the_Architecture.md](./03_Understanding_the_Architecture.md)

## Additional Resources

- **Example Scripts**: Check `tutorials/01_basic_agent.ipynb` for Jupyter notebook version
- **API Reference**: [docs/API_REFERENCE.md](../../docs/API_REFERENCE.md)
- **Model Documentation**: [assets/api-guide.pdf](../../assets/api-guide.pdf)

---

**Great job!** 🎉 You've successfully set up Holistic AI and created your first agent. Keep going - the next tutorials will show you how to build even more powerful agents!
