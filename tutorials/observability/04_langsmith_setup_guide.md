# Tutorial 4: LangSmith Setup Guide

## 📖 Overview

**What You'll Learn:**
- How to create a LangSmith account
- Configuration and API keys
- Connecting LangSmith to your agents
- Viewing your first trace
- Understanding the LangSmith UI

**Prerequisites:** 
- [Tutorial 1: What is Observability?](01_what_is_observability.md)
- Internet connection
- Email address (for account creation)

**Time to Complete:** 15 minutes

**Difficulty:** ⭐ Easy

---

## 🚀 Quick Start

### Step 1: Create LangSmith Account

1. **Go to LangSmith**: https://smith.langchain.com/

2. **Click "Sign Up"** or "Get Started"

3. **Choose sign-up method**:
   - GitHub (recommended - fastest)
   - Google
   - Email

4. **Complete registration**

5. **You're in!** You should see the LangSmith dashboard.

### Step 2: Get Your API Key

1. **Click on your profile** (bottom left corner)

2. **Select "Settings"**

3. **Go to "API Keys" tab**

4. **Click "Create API Key"**

5. **Name your key**: e.g., "Hackathon 2025"

6. **Copy the key** immediately!
   - ⚠️ **Warning**: You can only see it once!
   - Store it safely (we'll add it to `.env` next)

**Your API key looks like**: `lsv2_pt_abc123...xyz789`

### Step 3: Configure Environment Variables

**Method 1: Using .env File (Recommended)**

1. **Navigate to repository root**:
```bash
cd /path/to/hackathon-2025
```

2. **Create or edit `.env` file**:
```bash
# If .env doesn't exist, copy from example
cp .env.example .env

# Then edit it
nano .env  # or use your favorite editor
```

3. **Add LangSmith configuration**:
```bash
# LangSmith Configuration (Required for observability)
LANGSMITH_API_KEY=lsv2_pt_your_key_here
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=hackathon-2025

# Optional: Custom endpoint (default is fine)
# LANGSMITH_ENDPOINT=https://api.smith.langchain.com
```

4. **Save the file**

**Method 2: Direct in Notebook (Quick Testing)**

If you're using Jupyter notebooks, you can set variables directly:

```python
import os

# Set LangSmith credentials
os.environ["LANGSMITH_API_KEY"] = "lsv2_pt_your_key_here"
os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_PROJECT"] = "hackathon-2025"

print("✅ LangSmith configured!")
```

**Method 3: Shell Export (Temporary)**

```bash
export LANGSMITH_API_KEY="lsv2_pt_your_key_here"
export LANGSMITH_TRACING="true"
export LANGSMITH_PROJECT="hackathon-2025"
```

⚠️ **Note**: This only lasts for current terminal session.

### Step 4: Verify Configuration

**Test your setup with this Python code**:

```python
import os
from pathlib import Path
from dotenv import load_dotenv

# Load .env file
env_path = Path('.env')
if env_path.exists():
    load_dotenv(env_path)
    print("✅ .env file loaded")
else:
    print("⚠️  No .env file found")

# Check if LangSmith is configured
if os.getenv('LANGSMITH_API_KEY'):
    print("✅ LangSmith API key found")
    print(f"   Key starts with: {os.getenv('LANGSMITH_API_KEY')[:15]}...")
else:
    print("❌ LangSmith API key not found")

if os.getenv('LANGSMITH_TRACING') == 'true':
    print("✅ LangSmith tracing enabled")
else:
    print("⚠️  LangSmith tracing not enabled")

project = os.getenv('LANGSMITH_PROJECT', 'default')
print(f"✅ LangSmith project: {project}")
```

**Expected output**:
```
✅ .env file loaded
✅ LangSmith API key found
   Key starts with: lsv2_pt_abc1234...
✅ LangSmith tracing enabled
✅ LangSmith project: hackathon-2025
```

---

## 🧪 Create Your First Trace

Now let's create your first trace to make sure everything works!

### Simple Example: Basic Agent

```python
import os
from pathlib import Path
from dotenv import load_dotenv

# Load environment
load_dotenv(Path('../.env'))

# Import required packages
import sys
sys.path.insert(0, '../core')

from react_agent import create_react_agent
from langchain_core.messages import HumanMessage

# Create a simple agent (no tools)
print("Creating agent...")
agent = create_react_agent(tools=[])

print("\nRunning agent (this will create a trace)...")
result = agent.invoke({
    "messages": [HumanMessage("What is observability?")]
})

print("\n" + "="*70)
print("Response:")
print(result['messages'][-1].content)
print("="*70)
print("\n✅ Done! Now check LangSmith for your trace!")
print("   Go to: https://smith.langchain.com")
print(f"   Project: {os.getenv('LANGSMITH_PROJECT', 'default')}")
```

**Run this code and you should see**:
```
Creating agent...
✓ Native tool calling enabled

Running agent (this will create a trace)...

======================================================================
Response:
Observability is the ability to understand what's happening inside 
a system by examining its outputs. It involves collecting and 
analyzing traces, metrics, and logs to gain visibility into system 
behavior, performance, and issues.
======================================================================

✅ Done! Now check LangSmith for your trace!
   Go to: https://smith.langchain.com
   Project: hackathon-2025
```

### View Your Trace in LangSmith

1. **Open LangSmith**: https://smith.langchain.com

2. **Select your project**: "hackathon-2025" (from top dropdown)

3. **You should see your trace!**
   - Look for recent activity
   - The query should be "What is observability?"
   - Status should be "success" (green)

4. **Click on the trace** to view details

---

## 🖥️ Understanding the LangSmith UI

### Main Dashboard

When you open LangSmith, you'll see:

```
┌─────────────────────────────────────────────────────────────────┐
│  LangSmith                                    [Your Project ▼]  │
├─────────────────────────────────────────────────────────────────┤
│  📊 Overview  |  🔍 Traces  |  📈 Monitoring  |  ⚙️ Settings   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Recent Traces:                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ ✅ What is observability?          3.2s    1,234 tokens  │  │
│  │ ✅ What are the latest npm updates? 9.6s   2,456 tokens  │  │
│  │ ❌ Complex research query          ERROR   Token limit   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Performance Stats:                                            │
│  - Average latency: 4.2s                                      │
│  - Success rate: 95%                                          │
│  - Total tokens: 45,000                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Trace Detail View

Click on a trace to see detailed view:

```
┌─────────────────────────────────────────────────────────────────┐
│  Trace: What is observability?                    3.2s  ✅      │
├─────────────────────────────────────────────────────────────────┤
│  📋 Timeline  |  📊 Inputs/Outputs  |  💰 Tokens  |  🏷️ Metadata │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Timeline View:                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  agent_execution (3.2s)                                  │  │
│  │  ├─ call_model (2.3s)                                    │  │
│  │  │  └─ model_invoke (2.1s)                               │  │
│  │  │     - Model: claude-3-5-sonnet                        │  │
│  │  │     - Tokens: 1,234                                   │  │
│  │  │     - Cost: $0.0089                                   │  │
│  │  └─ format_output (0.9s)                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  Inputs:                                                        │
│  {                                                              │
│    "messages": [                                                │
│      {"type": "human", "content": "What is observability?"}    │
│    ]                                                            │
│  }                                                              │
│                                                                 │
│  Outputs:                                                       │
│  "Observability is the ability to understand..."               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key UI Elements

#### 1. **Timeline/Tree View**
- Shows execution flow as a tree
- Each node is a span
- Hover for details
- Click to expand

#### 2. **Inputs/Outputs Tab**
- Shows exact data passed between components
- Useful for debugging data issues
- Can copy/paste for testing

#### 3. **Tokens Tab**
- Token usage breakdown
- Cost calculation
- Input vs output tokens
- Per-model breakdown if multiple models used

#### 4. **Metadata Tab**
- Custom tags and metadata
- Model parameters
- Environment info
- User/session IDs

---

## 🎨 Projects and Organization

### What are Projects?

Projects help you organize traces:

**Example project structure**:
```
hackathon-2025/
├── development/        ← Dev testing
├── staging/           ← Pre-production testing
├── production/        ← Live traffic
└── experiments/       ← A/B tests
```

### Creating Projects

1. **Click project dropdown** (top of page)
2. **Click "+ New Project"**
3. **Name your project**
4. **Select project type**: Application / Dataset / Prompt

### Switching Projects

```python
# In your code
os.environ["LANGSMITH_PROJECT"] = "production"

# Or per agent invocation
agent.invoke(
    {"messages": [...]},
    {"tags": ["production"], "metadata": {"project": "production"}}
)
```

---

## 🏷️ Tags and Metadata

### Adding Tags

Tags help filter and search traces:

```python
# Add tags during invocation
agent.invoke(
    {"messages": [HumanMessage("query")]},
    {
        "tags": ["v2.0", "user-feedback", "high-priority"]
    }
)
```

**Common tags**:
- Version: `v1.0`, `v2.0`, `v3.0`
- Environment: `dev`, `staging`, `prod`
- Feature: `search`, `summarize`, `extract`
- User segment: `free-tier`, `premium`, `enterprise`

### Adding Metadata

Metadata provides context:

```python
agent.invoke(
    {"messages": [HumanMessage("query")]},
    {
        "metadata": {
            "user_id": "user_123",
            "session_id": "session_456",
            "experiment": "improved_prompts",
            "version": "v2.1.3"
        }
    }
)
```

**View in LangSmith**: Metadata appears in "Metadata" tab of trace detail.

---

## 🔍 Searching and Filtering

### Basic Search

**In LangSmith UI**:
1. Go to "Traces" tab
2. Use search bar at top
3. Search by:
   - Input text: "What is AI?"
   - Output text: "artificial intelligence"
   - Metadata values: "user_123"

### Advanced Filters

**Filter by**:
- **Status**: Success, Error, Timeout
- **Duration**: > 5s, < 1s, etc.
- **Tokens**: > 1000, < 500, etc.
- **Tags**: Select one or more tags
- **Date range**: Last hour, day, week, custom

**Example filters**:
```
Status: Error
Duration: > 10s
Tag: production
Date: Last 24 hours
```

This shows all production errors that took more than 10 seconds in the last day.

---

## 📊 Monitoring and Alerts

### Setting Up Monitors

1. **Go to "Monitoring" tab**
2. **Click "New Monitor"**
3. **Configure**:
   - Name: "High Latency Alert"
   - Condition: "Average latency > 5s"
   - Period: "Last 5 minutes"
   - Alert: Email/Slack

### Common Monitors

**Latency Monitor**:
```
Alert when: Average latency > 5s
Over: Last 10 minutes
Notify: email@company.com
```

**Error Rate Monitor**:
```
Alert when: Error rate > 5%
Over: Last 15 minutes
Notify: #alerts Slack channel
```

**Token Usage Monitor**:
```
Alert when: Total tokens > 100,000
Over: Last hour
Notify: cost-alerts@company.com
```

---

## 🛠️ Troubleshooting

### Issue: "No traces appearing in LangSmith"

**Checklist**:
1. ✓ Is `LANGSMITH_API_KEY` set correctly?
2. ✓ Is `LANGSMITH_TRACING=true`?
3. ✓ Is your API key valid? (not expired)
4. ✓ Are you looking at the correct project?
5. ✓ Wait 10-30 seconds for traces to appear

**Debug script**:
```python
import os

# Check configuration
print("LANGSMITH_API_KEY:", "✓" if os.getenv('LANGSMITH_API_KEY') else "✗")
print("LANGSMITH_TRACING:", os.getenv('LANGSMITH_TRACING'))
print("LANGSMITH_PROJECT:", os.getenv('LANGSMITH_PROJECT'))

# Test connection
from langsmith import Client
try:
    client = Client()
    print("✓ LangSmith connection successful!")
except Exception as e:
    print(f"✗ LangSmith connection failed: {e}")
```

### Issue: "Permission denied" or "Invalid API key"

**Solutions**:
1. **Regenerate API key** in LangSmith settings
2. **Check for extra spaces** in `.env` file
3. **Ensure key is quoted** if it has special characters
4. **Verify account is active** (check email for activation)

### Issue: "Traces are delayed"

**Normal behavior**: Traces can take 10-30 seconds to appear.

**If longer than 1 minute**:
1. Check internet connection
2. Check LangSmith service status: https://status.langchain.com
3. Try refreshing browser

### Issue: "Can't see tool calls in trace"

**Solutions**:
1. Verify your model supports native tool calling
2. Check that tools are bound correctly:
```python
# Should see this message
print("✓ Native tool calling enabled")
```
3. Look in "Inputs/Outputs" tab of trace for tool_calls field

---

## 🎓 Key Takeaways

### Setup Steps:
1. Create LangSmith account (free)
2. Get API key from settings
3. Add to `.env` file:
   - `LANGSMITH_API_KEY`
   - `LANGSMITH_TRACING=true`
   - `LANGSMITH_PROJECT=your-project`
4. Run agent code
5. View traces in LangSmith UI

### UI Navigation:
- **Dashboard**: Overview and recent traces
- **Traces tab**: Search and filter traces
- **Trace detail**: Timeline, inputs/outputs, tokens, metadata
- **Monitoring tab**: Set up alerts

### Best Practices:
- Use descriptive project names
- Add relevant tags for filtering
- Include metadata (user_id, version, etc.)
- Set up monitors for production
- Regularly review traces for optimization

### Next Steps:

✅ You have LangSmith set up and working!

**Continue to**:
- **[Tutorial 5: Tracing Basics](05_tracing_basics.md)** - Understand trace structure
- **[Tutorial 6: Analyzing Traces](06_analyzing_traces.md)** - Learn to analyze traces
- **[../05_observability.ipynb](../05_observability.ipynb)** - Interactive tutorial

---

## 📚 Additional Resources

- **LangSmith Docs**: https://docs.smith.langchain.com/
- **API Reference**: https://api.smith.langchain.com/docs
- **Community**: https://discord.gg/langchain
- **Status Page**: https://status.langchain.com

---

**🎉 Congratulations!** You're now set up with LangSmith and ready to explore observability. Continue to [Tutorial 5: Tracing Basics](05_tracing_basics.md) to learn how to read and understand traces!
