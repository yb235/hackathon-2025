# Tools and Utilities Reference

## Table of Contents
- [Built-in Tools](#built-in-tools)
- [Monitoring Tools](#monitoring-tools)
- [Development Utilities](#development-utilities)
- [Testing Tools](#testing-tools)
- [Observability Tools](#observability-tools)
- [Creating Custom Tools](#creating-custom-tools)

## Built-in Tools

### ValyuSearchTool

Deep search across web and proprietary sources powered by Valyu AI.

**Location:** `core/valyu_tools/tools.py`

**Purpose:** Search for information across multiple sources with relevance ranking.

**Configuration:**
```python
from valyu_tools import ValyuSearchTool

tool = ValyuSearchTool(valyu_api_key="your-key")  # or use VALYU_API_KEY env var
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | str | Required | Search query |
| `search_type` | str | "all" | 'all', 'proprietary', or 'web' |
| `max_num_results` | int | 10 | Max results (1-20) |
| `relevance_threshold` | float | 0.5 | Min relevance (0.0-1.0) |
| `max_price` | float | 50.0 | Max cost in dollars |
| `fast_mode` | bool | False | Faster but shorter results |
| `start_date` | str | None | YYYY-MM-DD format |
| `end_date` | str | None | YYYY-MM-DD format |
| `country_code` | str | None | ISO 2-letter code |

**Example:**
```python
# In agent
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

# Direct usage
tool = ValyuSearchTool()
results = tool._run(
    query="quantum computing breakthroughs",
    search_type="web",
    max_num_results=5,
    start_date="2024-01-01"
)
```

**Output Format:**
```json
{
    "results": [
        {
            "title": "Article Title",
            "url": "https://example.com/article",
            "content": "Article excerpt...",
            "relevance_score": 0.95,
            "source_type": "web"
        }
    ],
    "metadata": {
        "total_results": 5,
        "search_time": 1.23,
        "cost": 0.05
    }
}
```

### ValyuContentsTool

Extract clean content from web pages.

**Location:** `core/valyu_tools/tools.py`

**Purpose:** Get structured content from URLs with boilerplate removal.

**Configuration:**
```python
from valyu_tools import ValyuContentsTool

tool = ValyuContentsTool(
    valyu_api_key="your-key",
    summary=True,                      # Include summary
    extract_effort="high",              # 'normal', 'high', or 'auto'
    response_length="medium"            # 'short', 'medium', 'large', 'max', or int
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `urls` | List[str] | URLs to extract (max 10) |

**Example:**
```python
# In agent
agent = create_react_agent(
    tools=[ValyuContentsTool(summary=True)],
    model_name='claude-3-5-sonnet'
)

# Direct usage
tool = ValyuContentsTool()
results = tool._run(urls=[
    "https://arxiv.org/abs/2024.12345",
    "https://blog.example.com/article"
])
```

**Output Format:**
```json
{
    "contents": [
        {
            "url": "https://example.com",
            "title": "Article Title",
            "content": "Main content...",
            "author": "Author Name",
            "publish_date": "2024-11-15",
            "summary": "Brief summary..."
        }
    ]
}
```

---

## Monitoring Tools

### CodeCarbon

Track carbon emissions from model usage.

**Installation:**
```bash
pip install codecarbon
```

**Basic Usage:**
```python
from codecarbon import EmissionsTracker

# Wrap agent invocation
tracker = EmissionsTracker()
tracker.start()

result = agent.invoke({"messages": [...]})

emissions = tracker.stop()
print(f"CO2 Emissions: {emissions} kg")
```

**Advanced Usage (Tutorial 04):**
```python
from codecarbon import OfflineEmissionsTracker

tracker = OfflineEmissionsTracker(
    country_iso_code="GBR",
    project_name="hackathon-agent",
    output_dir="./emissions"
)

tracker.start()
# Run multiple agent invocations
for query in queries:
    agent.invoke({"messages": [query]})
tracker.stop()

# Emissions saved to ./emissions/emissions.csv
```

**Output:**
- Emissions in kg CO2
- Energy consumed in kWh
- Duration
- Location-based factors

### TikToken

Count tokens for cost estimation.

**Installation:**
```bash
pip install tiktoken
```

**Usage:**
```python
import tiktoken

# Get encoding for model
encoding = tiktoken.encoding_for_model("gpt-4")

# Count tokens
text = "Your prompt or response here"
tokens = encoding.encode(text)
token_count = len(tokens)

print(f"Tokens: {token_count}")

# Estimate cost (example rates)
input_cost = token_count * 0.00003  # $0.03 per 1K tokens
output_cost = token_count * 0.00006  # $0.06 per 1K tokens
```

**With Agent (Tutorial 04):**
```python
def count_agent_tokens(messages):
    """Count tokens in agent conversation"""
    encoding = tiktoken.encoding_for_model("gpt-4")
    total = 0
    
    for msg in messages:
        if hasattr(msg, 'content'):
            total += len(encoding.encode(msg.content))
    
    return total

result = agent.invoke({"messages": [...]})
tokens = count_agent_tokens(result["messages"])
print(f"Total tokens: {tokens}")
```

### Custom Performance Tracker

**Example (Tutorial 04):**
```python
import time
from dataclasses import dataclass

@dataclass
class PerformanceMetrics:
    latency: float
    tokens: int
    cost: float
    carbon: float

def track_performance(agent, query):
    """Track comprehensive performance metrics"""
    
    # Start tracking
    start_time = time.time()
    carbon_tracker.start()
    
    # Invoke agent
    result = agent.invoke({"messages": [query]})
    
    # Stop tracking
    latency = time.time() - start_time
    carbon = carbon_tracker.stop()
    
    # Calculate metrics
    tokens = count_tokens(result["messages"])
    cost = calculate_cost(tokens, model="claude-3-5-sonnet")
    
    return PerformanceMetrics(
        latency=latency,
        tokens=tokens,
        cost=cost,
        carbon=carbon
    )
```

---

## Development Utilities

### load_chat_model()

Universal model loader supporting multiple providers.

**Location:** `core/react_agent/utils.py`

**Signature:**
```python
def load_chat_model(model_name: str, context=None) -> BaseChatModel
```

**Supported Models:**

**Holistic AI Bedrock:**
- `claude-3-5-sonnet`, `claude-3-5-haiku`, `claude-3-opus`
- `llama3-2-90b`, `llama3-2-11b`, `llama3-2-3b`
- `nova-pro`, `nova-lite`

**OpenAI:**
- `gpt-5-nano`, `gpt-5-mini`, `gpt-5`

**Ollama:**
- `gpt-oss`, `qwen3` (OpenAI-compatible)
- `llama3.1:8b`, `qwen2.5:7b` (standard)

**Example:**
```python
from react_agent.utils import load_chat_model

# Bedrock (requires credentials)
model = load_chat_model("claude-3-5-sonnet")

# OpenAI (requires OPENAI_API_KEY)
model = load_chat_model("gpt-5-mini")

# Ollama (requires local Ollama server)
model = load_chat_model("llama3.1:8b")
```

### Context Configuration

**Location:** `core/react_agent/context.py`

**Usage:**
```python
from react_agent import Context

# Custom configuration
context = Context(
    model="claude-3-5-haiku",
    max_search_results=5,
    ollama_temperature=0.3,
    ollama_timeout=120,
    system_prompt="Custom instructions here"
)

agent = create_react_agent(tools=[...], context=context)
```

**Overriding Settings:**
```python
# Override model
agent = create_react_agent(
    tools=[...],
    context=context,
    model_name="claude-3-5-sonnet"  # Overrides context.model
)

# Override system prompt
agent = create_react_agent(
    tools=[...],
    system_prompt="Different instructions"  # Overrides context.system_prompt
)
```

---

## Testing Tools

### Benchmark Evaluation

**Example (Tutorial 06):**
```python
import pandas as pd

# Load benchmark
benchmark = pd.read_csv('benchmark_questions.csv')

# Evaluate agent
results = []
for idx, row in benchmark.iterrows():
    result = agent.invoke({
        "messages": [HumanMessage(content=row['question'])]
    })
    
    response = result["messages"][-1].content
    results.append({
        'question': row['question'],
        'expected': row['expected_answer'],
        'actual': response,
        'correct': evaluate_answer(response, row['expected_answer'])
    })

# Calculate accuracy
accuracy = sum(r['correct'] for r in results) / len(results)
print(f"Accuracy: {accuracy * 100}%")
```

### LLM-as-a-Judge

**Example (Tutorial 06, Track C):**
```python
def llm_judge(question, answer, expected):
    """Use LLM to evaluate answer quality"""
    
    judge_prompt = f"""
    Question: {question}
    Expected Answer: {expected}
    Actual Answer: {answer}
    
    Rate the actual answer on a scale of 0-10 for:
    1. Correctness
    2. Completeness
    3. Clarity
    
    Return only a JSON object with scores.
    """
    
    judge_model = get_chat_model("gpt-5-mini")
    result = judge_model.invoke([HumanMessage(content=judge_prompt)])
    
    # Parse scores
    scores = json.loads(result.content)
    return scores
```

### Attack Success Rate (ASR) Measurement

**Example (Tutorial 08, Track C):**
```python
def calculate_asr(agent, test_cases):
    """Calculate Attack Success Rate"""
    
    successful_attacks = 0
    total_attempts = len(test_cases)
    
    for test in test_cases:
        response = agent.invoke({
            "messages": [HumanMessage(content=test['attack_prompt'])]
        })
        
        # Check if attack succeeded
        if is_attack_successful(response, test['expected_refusal']):
            successful_attacks += 1
    
    asr = successful_attacks / total_attempts
    return asr

# Usage
benign_tests = load_test_cases('benign_test_cases.csv')
harmful_tests = load_test_cases('harmful_test_cases.csv')

benign_asr = calculate_asr(agent, benign_tests)
harmful_asr = calculate_asr(agent, harmful_tests)

print(f"Benign ASR: {benign_asr * 100}%")  # Should be low
print(f"Harmful ASR: {harmful_asr * 100}%")  # Indicates vulnerability
```

---

## Observability Tools

### LangSmith

Automatic execution tracing and debugging.

**Setup:**
```bash
# In .env
LANGSMITH_API_KEY=your-key
LANGSMITH_PROJECT=hackathon-2025
LANGSMITH_TRACING=true
```

**Usage:**
```python
from dotenv import load_dotenv
load_dotenv()

# Tracing is automatic!
agent = create_react_agent(tools=[...])
result = agent.invoke({"messages": [...]})

# View traces at: https://smith.langchain.com
```

**Export Traces:**
```python
from langsmith import Client

client = Client()

# Get project runs
runs = client.list_runs(project_name="hackathon-2025")

# Export specific run
run = client.read_run(run_id="...")
trace = run.to_dict()

# Save to file
import json
with open('trace.json', 'w') as f:
    json.dump(trace, f, indent=2)
```

### OpenTelemetry

Vendor-neutral observability.

**Installation:**
```bash
pip install opentelemetry-api opentelemetry-sdk
```

**Basic Usage:**
```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor, ConsoleSpanExporter

# Setup
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Add processor
trace.get_tracer_provider().add_span_processor(
    SimpleSpanProcessor(ConsoleSpanExporter())
)

# Trace agent execution
with tracer.start_as_current_span("agent_invocation"):
    result = agent.invoke({"messages": [...]})
```

### Custom Tracing

**Example:**
```python
import json
from datetime import datetime

class AgentTracer:
    def __init__(self):
        self.traces = []
    
    def trace_invocation(self, agent, input_msg):
        """Trace single invocation"""
        trace = {
            'timestamp': datetime.now().isoformat(),
            'input': input_msg,
            'steps': []
        }
        
        # Hook into agent execution
        # (implementation depends on access to agent internals)
        
        result = agent.invoke({"messages": [input_msg]})
        
        trace['output'] = result["messages"][-1].content
        trace['total_steps'] = len(result["messages"])
        
        self.traces.append(trace)
        return result
    
    def export_traces(self, filename):
        """Export to JSON"""
        with open(filename, 'w') as f:
            json.dump(self.traces, f, indent=2)

# Usage
tracer = AgentTracer()
result = tracer.trace_invocation(agent, HumanMessage("query"))
tracer.export_traces('agent_traces.json')
```

---

## Creating Custom Tools

### Basic Tool Template

```python
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type, Optional

class MyToolInput(BaseModel):
    """Input schema for MyTool"""
    param1: str = Field(description="Description of param1")
    param2: int = Field(default=10, description="Description of param2")

class MyTool(BaseTool):
    """Description of what this tool does"""
    
    name: str = "my_tool"
    description: str = "Detailed description for the LLM to understand when to use this tool"
    args_schema: Type[BaseModel] = MyToolInput
    
    def _run(self, param1: str, param2: int = 10) -> str:
        """
        Execute the tool.
        
        Args:
            param1: First parameter
            param2: Second parameter
            
        Returns:
            Result as string (or dict/list)
        """
        try:
            # Tool logic here
            result = do_something(param1, param2)
            return f"Success: {result}"
        
        except Exception as e:
            # Return errors as strings
            return f"Error: {str(e)}"
    
    async def _arun(self, param1: str, param2: int = 10) -> str:
        """Async version (optional)"""
        # Async implementation
        return await async_do_something(param1, param2)
```

### Tool with External API

```python
import requests
from typing import Dict

class WeatherToolInput(BaseModel):
    location: str = Field(description="City name or coordinates")

class WeatherTool(BaseTool):
    name: str = "get_weather"
    description: str = "Get current weather for a location"
    args_schema: Type[BaseModel] = WeatherToolInput
    
    api_key: str = Field(default="")
    
    def _run(self, location: str) -> Dict:
        """Fetch weather data"""
        try:
            url = f"https://api.weather.com/v1/current"
            params = {
                'location': location,
                'api_key': self.api_key
            }
            
            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()
            
            data = response.json()
            return {
                'temperature': data['temp'],
                'conditions': data['conditions'],
                'humidity': data['humidity']
            }
        
        except requests.RequestException as e:
            return {'error': f"API error: {str(e)}"}

# Usage
tool = WeatherTool(api_key="your-key")
agent = create_react_agent(tools=[tool])
```

### Tool with State

```python
class DatabaseTool(BaseTool):
    name: str = "query_database"
    description: str = "Query internal database"
    args_schema: Type[BaseModel] = DatabaseQueryInput
    
    # Stateful connection
    _connection: Optional[any] = None
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._connection = self._connect_to_db()
    
    def _connect_to_db(self):
        """Establish database connection"""
        # Connection logic
        return connection
    
    def _run(self, query: str) -> str:
        """Execute database query"""
        try:
            cursor = self._connection.execute(query)
            results = cursor.fetchall()
            return json.dumps(results)
        
        except Exception as e:
            return f"Database error: {str(e)}"
    
    def __del__(self):
        """Cleanup connection"""
        if self._connection:
            self._connection.close()
```

### Tool Best Practices

1. **Clear Descriptions**: LLM needs to understand when to use the tool
2. **Strong Types**: Use Pydantic for input validation
3. **Error Handling**: Return errors as strings, don't raise exceptions
4. **Documentation**: Document parameters and return values
5. **Timeout Handling**: Set timeouts for external calls
6. **Rate Limiting**: Respect API limits
7. **Caching**: Cache results when appropriate

### Tool Testing

```python
def test_my_tool():
    """Test tool independently"""
    tool = MyTool()
    
    # Test normal case
    result = tool._run(param1="test", param2=5)
    assert "Success" in result
    
    # Test error case
    result = tool._run(param1="invalid", param2=-1)
    assert "Error" in result
    
    print("Tool tests passed!")

test_my_tool()
```

---

## Utility Scripts

### Batch Testing

```python
def batch_test_agent(agent, test_cases_file):
    """Test agent on multiple cases"""
    import pandas as pd
    
    tests = pd.read_csv(test_cases_file)
    results = []
    
    for idx, test in tests.iterrows():
        print(f"Testing {idx + 1}/{len(tests)}...")
        
        result = agent.invoke({
            "messages": [HumanMessage(content=test['query'])]
        })
        
        results.append({
            'query': test['query'],
            'response': result["messages"][-1].content,
            'expected': test['expected'],
            'passed': evaluate(result, test['expected'])
        })
    
    # Save results
    df = pd.DataFrame(results)
    df.to_csv('test_results.csv', index=False)
    
    print(f"Pass rate: {df['passed'].mean() * 100}%")
```

### Trace Analysis

```python
def analyze_trace_file(trace_file):
    """Analyze exported trace"""
    with open(trace_file, 'r') as f:
        trace = json.load(f)
    
    stats = {
        'total_steps': count_steps(trace),
        'tool_calls': count_tool_calls(trace),
        'tokens': estimate_tokens(trace),
        'latency': calculate_latency(trace),
        'errors': find_errors(trace)
    }
    
    return stats
```

---

## Next Steps

- See [API_REFERENCE.md](API_REFERENCE.md) for detailed API documentation
- See [GETTING_STARTED.md](GETTING_STARTED.md) for setup instructions
- See tutorials for practical examples of each tool
