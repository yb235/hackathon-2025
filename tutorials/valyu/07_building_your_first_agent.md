# Tutorial 7: Building Your First Valyu-Powered Agent

## Table of Contents
- [Agent Architecture](#agent-architecture)
- [Setup](#setup)
- [Building the Agent](#building-the-agent)
- [Testing and Iteration](#testing-and-iteration)
- [Production Deployment](#production-deployment)

## Agent Architecture

### What We're Building

A complete AI agent that can:
1. Search the web for information using Valyu
2. Extract content from specific URLs
3. Answer questions with citations
4. Handle multi-step reasoning

### Architecture Diagram

```
User Question
     ↓
┌────────────────────┐
│   ReAct Agent      │
│                    │
│  1. Reasoning      │──→ Decides what to do
│  2. Action         │──→ Calls tools
│  3. Observation    │──→ Processes results
│  4. Repeat/Answer  │──→ Continues or responds
└────────────────────┘
     ↓
┌─────────────────────────────────┐
│         Available Tools          │
│                                  │
│  • ValyuSearchTool              │
│  • ValyuContentsTool            │
└─────────────────────────────────┘
```

## Setup

### Install Dependencies

```python
# Required packages
pip install langchain-valyu valyu langgraph

# For Holistic AI Bedrock
export HOLISTIC_AI_TEAM_ID=your-team-id
export HOLISTIC_AI_API_TOKEN=your-token

# For Valyu
export VALYU_API_KEY=your-valyu-key
```

### Import Required Modules

```python
import sys
sys.path.insert(0, '../core')

from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool, ValyuContentsTool
```

## Building the Agent

### Step 1: Create Basic Agent

```python
from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool

# Create agent with search capability
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

# Test it
result = agent.invoke({
    'messages': [('user', 'What is quantum computing?')]
})

print(result['messages'][-1].content)
```

### Step 2: Add Multiple Tools

```python
# Agent with search and content extraction
agent = create_react_agent(
    tools=[
        ValyuSearchTool(),
        ValyuContentsTool()
    ],
    model_name='claude-3-5-sonnet'
)

# Now agent can:
# - Search for information
# - Extract detailed content from URLs

result = agent.invoke({
    'messages': [('user', 'Find recent papers on quantum computing and summarize the top one')]
})
```

### Step 3: Configured Tools

```python
from langchain_valyu import ValyuSearchTool, ValyuContentsTool

# Optimized search tool
search_tool = ValyuSearchTool()

# Configured contents tool
contents_tool = ValyuContentsTool(
    summary=True,
    extract_effort="high",
    response_length="medium"
)

# Create agent
agent = create_react_agent(
    tools=[search_tool, contents_tool],
    model_name='claude-3-5-sonnet'
)
```

### Step 4: Custom System Prompt

```python
custom_prompt = """You are a research assistant AI. Your job is to:

1. Search for accurate, recent information
2. Extract detailed content when needed
3. Provide well-cited answers
4. Be honest when information is not found

When citing sources, always include:
- Title
- URL
- Relevance to the question

Current time: {system_time}
"""

agent = create_react_agent(
    tools=[ValyuSearchTool(), ValyuContentsTool()],
    model_name='claude-3-5-sonnet',
    system_prompt=custom_prompt
)
```

### Step 5: Conversational Agent

```python
from langgraph.checkpoint.memory import MemorySaver

# Create agent with memory
memory = MemorySaver()

agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet',
    checkpointer=memory
)

# Have a conversation
config = {"configurable": {"thread_id": "conversation-1"}}

# First question
result1 = agent.invoke({
    'messages': [('user', 'What is quantum computing?')]
}, config)

# Follow-up (maintains context)
result2 = agent.invoke({
    'messages': [('user', 'What are its main applications?')]
}, config)
```

## Complete Example: Research Agent

```python
import sys
sys.path.insert(0, '../core')

from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool, ValyuContentsTool
from langgraph.checkpoint.memory import MemorySaver

class ResearchAgent:
    """Complete research agent with Valyu tools."""
    
    def __init__(self, model_name='claude-3-5-sonnet'):
        """Initialize the research agent."""
        
        # Configure tools
        self.search_tool = ValyuSearchTool()
        self.contents_tool = ValyuContentsTool(
            summary=True,
            extract_effort="high",
            response_length="medium"
        )
        
        # Custom system prompt
        system_prompt = """You are an expert research assistant. 

Your capabilities:
- Search for information using valyu_deep_search
- Extract detailed content using valyu_contents_extract
- Synthesize information from multiple sources
- Provide citations for all claims

Guidelines:
- Always cite sources with [Title](URL)
- Verify information from multiple sources when possible
- Be clear when information is uncertain
- Use structured formatting for readability

Current time: {system_time}
"""
        
        # Create agent with memory
        self.agent = create_react_agent(
            tools=[self.search_tool, self.contents_tool],
            model_name=model_name,
            system_prompt=system_prompt,
            checkpointer=MemorySaver()
        )
        
        self.thread_id = "research-session-1"
    
    def research(self, question):
        """Conduct research on a question."""
        config = {"configurable": {"thread_id": self.thread_id}}
        
        result = self.agent.invoke({
            'messages': [('user', question)]
        }, config)
        
        return result['messages'][-1].content
    
    def follow_up(self, question):
        """Ask a follow-up question in the same context."""
        return self.research(question)


# Usage example
if __name__ == "__main__":
    # Create research agent
    agent = ResearchAgent()
    
    # Initial research
    print("Question 1: What is quantum computing?")
    answer1 = agent.research("What is quantum computing?")
    print(f"\nAnswer:\n{answer1}\n")
    print("-" * 80)
    
    # Follow-up question
    print("\nQuestion 2: What are its applications in cryptography?")
    answer2 = agent.follow_up("What are its applications in cryptography?")
    print(f"\nAnswer:\n{answer2}\n")
```

## Testing and Iteration

### Test Different Scenarios

```python
agent = ResearchAgent()

# Test cases
test_questions = [
    "What is quantum computing?",
    "Find recent news about AI regulations",
    "Explain the difference between machine learning and deep learning",
    "What are the latest developments in renewable energy?",
    "How does blockchain technology work?"
]

for question in test_questions:
    print(f"\n{'='*80}")
    print(f"Q: {question}")
    print(f"{'='*80}")
    
    answer = agent.research(question)
    print(f"A: {answer}\n")
```

### Monitor Tool Usage

```python
def research_with_monitoring(agent, question):
    """Research with tool usage monitoring."""
    print(f"Question: {question}\n")
    
    config = {"configurable": {"thread_id": "monitor-session"}}
    result = agent.agent.invoke({
        'messages': [('user', question)]
    }, config)
    
    # Analyze messages to see tool calls
    for msg in result['messages']:
        if hasattr(msg, 'tool_calls') and msg.tool_calls:
            print(f"🔧 Tool used: {msg.tool_calls[0]['name']}")
    
    return result['messages'][-1].content

# Example
agent = ResearchAgent()
answer = research_with_monitoring(agent, "What is quantum computing?")
```

### Performance Testing

```python
import time

def benchmark_agent(agent, questions):
    """Benchmark agent performance."""
    results = []
    
    for question in questions:
        start_time = time.time()
        
        answer = agent.research(question)
        
        elapsed = time.time() - start_time
        
        results.append({
            'question': question,
            'answer_length': len(answer),
            'time_seconds': elapsed
        })
        
        print(f"Q: {question[:50]}...")
        print(f"   Time: {elapsed:.2f}s, Length: {len(answer)} chars")
    
    return results

# Run benchmark
agent = ResearchAgent()
questions = [
    "What is quantum computing?",
    "Explain machine learning",
    "What is blockchain?"
]

benchmark_results = benchmark_agent(agent, questions)

# Summary
avg_time = sum(r['time_seconds'] for r in benchmark_results) / len(benchmark_results)
print(f"\nAverage response time: {avg_time:.2f}s")
```

## Production Deployment

### Error Handling

```python
class ProductionResearchAgent(ResearchAgent):
    """Production-ready research agent with error handling."""
    
    def research(self, question, max_retries=3):
        """Research with retry logic."""
        for attempt in range(max_retries):
            try:
                return super().research(question)
            except Exception as e:
                if attempt < max_retries - 1:
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(2 ** attempt)  # Exponential backoff
                else:
                    return f"Error after {max_retries} attempts: {str(e)}"
```

### Logging

```python
import logging

class LoggingResearchAgent(ResearchAgent):
    """Research agent with logging."""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        # Setup logging
        logging.basicConfig(level=logging.INFO)
        self.logger = logging.getLogger(__name__)
    
    def research(self, question):
        """Research with logging."""
        self.logger.info(f"Research query: {question}")
        
        try:
            answer = super().research(question)
            self.logger.info(f"Research completed: {len(answer)} chars")
            return answer
        except Exception as e:
            self.logger.error(f"Research failed: {e}")
            raise
```

### Cost Tracking

```python
class CostTrackingAgent(ResearchAgent):
    """Agent with cost tracking."""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.total_cost = 0.0
        self.query_count = 0
    
    def research(self, question):
        """Research with cost tracking."""
        self.query_count += 1
        
        # Estimate cost (simplified)
        estimated_cost = 0.05  # Per query
        self.total_cost += estimated_cost
        
        answer = super().research(question)
        
        print(f"💰 Query #{self.query_count} | Cost: ${self.total_cost:.4f}")
        
        return answer
```

### Rate Limiting

```python
import time
from datetime import datetime, timedelta

class RateLimitedAgent(ResearchAgent):
    """Agent with rate limiting."""
    
    def __init__(self, max_requests_per_minute=10, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.max_rpm = max_requests_per_minute
        self.request_times = []
    
    def research(self, question):
        """Research with rate limiting."""
        # Clean old requests
        cutoff = datetime.now() - timedelta(minutes=1)
        self.request_times = [t for t in self.request_times if t > cutoff]
        
        # Check rate limit
        if len(self.request_times) >= self.max_rpm:
            wait_time = 60 - (datetime.now() - self.request_times[0]).seconds
            print(f"⏳ Rate limit reached. Waiting {wait_time}s...")
            time.sleep(wait_time)
            self.request_times = []
        
        # Record request
        self.request_times.append(datetime.now())
        
        return super().research(question)
```

## Summary

In this tutorial, you learned:

- ✅ How to build a complete Valyu-powered agent
- ✅ Configure tools and system prompts
- ✅ Add conversational memory
- ✅ Test and monitor agent performance
- ✅ Deploy with error handling, logging, and rate limiting

**Next Steps:**

- **[Tutorial 8: Real-World Use Cases](./08_real_world_use_cases.md)** - Production patterns
- **[Tutorial 9: API Reference](./09_api_reference.md)** - Complete API documentation

You can now build production-ready AI agents! 🚀
