# Tutorial 5: Tracing Basics - Understanding Traces and Spans

## 📖 Overview

**What You'll Learn:**
- What traces and spans are in detail
- How to read trace data structures
- Understanding execution paths
- Trace complexity levels
- Reading real trace examples from the repository

**Prerequisites:** 
- [Tutorial 1: What is Observability?](01_what_is_observability.md)
- [Tutorial 4: LangSmith Setup Guide](04_langsmith_setup_guide.md)

**Time to Complete:** 25 minutes

**Difficulty:** ⭐⭐ Medium

---

## 🎯 What is a Trace?

### The Simple Definition

A **trace** is a complete record of everything that happens when your agent processes a request, from start to finish.

Think of it like a **flight recorder** for your agent:
- Records every decision
- Captures every action
- Timestamps each step
- Saves all inputs and outputs

### Visual Example

```
User Query: "What are the latest npm updates?"

┌─────────────────────────────────────────────────────────────┐
│ TRACE: agent_execution                                      │
│ Start: 14:30:00.000                                        │
│ End:   14:30:09.600                                        │
│ Duration: 9.6 seconds                                       │
│ Status: ✅ Success                                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ├─ call_model_1 (2.3s)                                     │
│ │  └─ Decision: Use search tool                            │
│ │                                                           │
│ ├─ tools (5.2s)                                            │
│ │  └─ valyu_deep_search("latest npm release notes")        │
│ │                                                           │
│ └─ call_model_2 (2.1s)                                     │
│    └─ Synthesize: "npm 11.5.2 was released..."            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 What is a Span?

### The Simple Definition

A **span** is one unit of work within a trace. Every trace contains one or more spans.

### Analogy: Cooking a Meal

**Trace**: Making spaghetti dinner (30 minutes total)

**Spans**:
- Boil water (10 minutes)
- Cook pasta (8 minutes)
- Prepare sauce (12 minutes)
- Plate and serve (2 minutes)

Each span has:
- **Name**: "Boil water"
- **Duration**: 10 minutes
- **Start time**: 18:00:00
- **End time**: 18:10:00
- **Status**: Success/Failure

### In AI Agents

**Trace**: Research quantum computing (15 seconds)

**Spans**:
```
Root Span: agent_execution (15.0s)
├─ Span: call_model (2.0s)
│  └─ Span: model_inference (1.8s)
├─ Span: search_tool (8.0s)
│  ├─ Span: http_request (7.5s)
│  └─ Span: parse_results (0.5s)
└─ Span: call_model (5.0s)
   └─ Span: model_inference (4.8s)
```

---

## 📊 Trace Structure

### The Hierarchy

Traces are organized as **trees**:

```
Root Span (the whole request)
│
├─ Child Span 1
│  ├─ Grandchild Span 1.1
│  └─ Grandchild Span 1.2
│
├─ Child Span 2
│  └─ Grandchild Span 2.1
│     └─ Great-grandchild Span 2.1.1
│
└─ Child Span 3
```

### Real Example

```
agent_execution (10.5s)
│
├─ call_model (2.5s)
│  ├─ format_prompt (0.1s)
│  ├─ model_invoke (2.3s)
│  │  ├─ tokenize (0.2s)
│  │  ├─ inference (1.9s)
│  │  └─ parse_response (0.2s)
│  └─ extract_tool_calls (0.1s)
│
├─ tools (5.0s)
│  └─ valyu_search (5.0s)
│     ├─ format_query (0.1s)
│     ├─ api_call (4.5s)
│     └─ parse_results (0.4s)
│
└─ call_model (3.0s)
   └─ model_invoke (2.8s)
```

**Key insight**: Parent span duration = sum of children + overhead

---

## 🏗️ Span Attributes

Every span contains rich information:

### 1. **Basic Information**
```json
{
  "span_id": "span_abc123",
  "trace_id": "trace_xyz789",
  "parent_span_id": "span_parent456",
  "name": "call_model",
  "start_time": "2025-11-15T14:30:00.000Z",
  "end_time": "2025-11-15T14:30:02.300Z",
  "duration_ms": 2300,
  "status": "success"
}
```

### 2. **Inputs and Outputs**
```json
{
  "inputs": {
    "messages": [
      {"role": "user", "content": "What is AI?"}
    ]
  },
  "outputs": {
    "content": "AI stands for Artificial Intelligence...",
    "tokens": 1234
  }
}
```

### 3. **Metadata**
```json
{
  "metadata": {
    "model": "claude-3-5-sonnet",
    "temperature": 0.7,
    "max_tokens": 4096,
    "provider": "bedrock"
  }
}
```

### 4. **Metrics**
```json
{
  "metrics": {
    "input_tokens": 245,
    "output_tokens": 989,
    "total_tokens": 1234,
    "cost_usd": 0.0089,
    "latency_ms": 2300
  }
}
```

### 5. **Tags**
```json
{
  "tags": [
    "production",
    "v2.0",
    "high-priority"
  ]
}
```

---

## 📖 Reading Trace Examples

### Example 1: Simple Trace (No Tools)

**From**: `track_b_glass_box/traces/trace-very-simple.json` (13KB)

**Scenario**: Basic question-answer, no tools needed

**Structure**:
```
agent_execution (3.2s)
└─ call_model (3.1s)
   └─ model_invoke (2.9s)
```

**What happened**:
1. User asked: "What is a GPU?"
2. Agent analyzed question
3. Agent responded from knowledge (no tools needed)
4. Done!

**Key characteristics**:
- Only 1 model call
- No tool calls
- Fast (3.2 seconds)
- Simple linear flow

### Example 2: Simple Trace with Tools

**From**: `track_b_glass_box/traces/trace-simple.json` (65KB)

**Scenario**: Question requiring search tool

**Structure**:
```
agent_execution (9.8s)
├─ call_model_1 (2.1s)
│  └─ Decision: Use search
├─ tools (5.2s)
│  └─ search_tool (5.0s)
└─ call_model_2 (2.5s)
   └─ Synthesize results
```

**What happened**:
1. User asked: "What are the latest npm updates?"
2. Agent decided to search
3. Search tool executed
4. Agent synthesized results
5. Done!

**Key characteristics**:
- 2 model calls (plan + synthesize)
- 1 tool call (search)
- Medium duration (9.8 seconds)
- Loop: model → tool → model

### Example 3: Normal Complexity

**From**: `track_b_glass_box/traces/trace-normal.json` (251KB)

**Scenario**: Research task with multiple tool calls

**Structure**:
```
agent_execution (28.5s)
├─ call_model_1 (2.3s)
├─ tools (8.1s)
│  ├─ search_tool (5.2s)
│  └─ scrape_tool (2.9s)
├─ call_model_2 (2.1s)
├─ tools (10.3s)
│  ├─ search_tool (6.1s)
│  └─ search_tool (4.2s)
└─ call_model_3 (5.7s)
```

**What happened**:
1. User requested research on a topic
2. Agent searched initially
3. Agent scraped relevant pages
4. Agent did follow-up searches
5. Agent wrote comprehensive report
6. Done!

**Key characteristics**:
- 3 model calls (plan, refine, finalize)
- 4 tool calls (multiple searches + scrape)
- Longer duration (28.5 seconds)
- Complex workflow with iterations

### Example 4: Complex Multi-Agent

**From**: `track_b_glass_box/traces/trace-complex.json` (15MB!)

**Scenario**: Deep research with sub-agents

**Structure** (simplified):
```
agent_execution (120s)
├─ research_planner (5s)
│  └─ Generate research plan
├─ research_executor (90s)
│  ├─ sub_agent_1: Topic A (30s)
│  │  ├─ search (8s)
│  │  ├─ scrape (12s)
│  │  └─ synthesize (10s)
│  ├─ sub_agent_2: Topic B (30s)
│  │  └─ [similar structure]
│  └─ sub_agent_3: Topic C (30s)
│     └─ [similar structure]
└─ report_generator (25s)
   ├─ write_sections (15s)
   └─ format_report (10s)
```

**What happened**:
1. User requested deep research report
2. Planner agent created research strategy
3. Executor spawned 3 sub-agents in parallel
4. Each sub-agent researched its topic
5. Report generator compiled results
6. Done!

**Key characteristics**:
- Multi-agent orchestration
- Parallel execution
- Nested hierarchies
- Very long duration (2 minutes)
- Large trace (15MB of data)

---

## 🎨 Trace Visualization Patterns

### Pattern 1: Linear Flow
```
[Start] → [Step 1] → [Step 2] → [Step 3] → [End]
```
**Example**: Simple Q&A, no tools
**Characteristics**: Fast, predictable, easy to debug

### Pattern 2: Loop Pattern
```
[Start] → [Model] → [Tool] → [Model] → [End]
                 ↑           ↓
                 └───────────┘
```
**Example**: Agent using tools
**Characteristics**: Most common pattern, iterative

### Pattern 3: Multi-Loop Pattern
```
[Start] → [Model] → [Tool1] → [Model] → [Tool2] → [Model] → [End]
                              ↑                  ↓
                              └──────────────────┘
```
**Example**: Complex research with multiple tool calls
**Characteristics**: Longer duration, more complex

### Pattern 4: Parallel Pattern
```
[Start] → [Orchestrator] → [Sub-agent 1] → [Synthesizer] → [End]
                        ├→ [Sub-agent 2] ─┤
                        └→ [Sub-agent 3] ─┘
```
**Example**: Multi-agent systems
**Characteristics**: Fastest for complex tasks, hardest to debug

---

## 🔬 Analyzing Span Data

### Duration Analysis

**Question**: Why did this take so long?

**Look at span durations**:
```
agent_execution (15.2s)  ← Total time
├─ call_model (2.1s)     ← 14% of total
├─ tools (10.8s)         ← 71% of total  ← BOTTLENECK!
│  └─ search_tool (10.5s)
└─ call_model (2.3s)     ← 15% of total
```

**Insight**: 71% of time spent in search tool. Optimize this!

### Token Analysis

**Question**: Why is this so expensive?

**Look at token usage**:
```
Total tokens: 15,234
├─ call_model_1: 1,234 tokens ($0.0089)
├─ call_model_2: 8,900 tokens ($0.0641)  ← EXPENSIVE!
└─ call_model_3: 5,100 tokens ($0.0367)
```

**Insight**: call_model_2 used 58% of tokens. Why so many?
- Check input: Is context too large?
- Check output: Is response too verbose?

### Error Analysis

**Question**: Where did it fail?

**Look at status codes**:
```
agent_execution (failed)
├─ call_model (success)
├─ tools (failed)  ← ERROR HERE!
│  └─ search_tool (error: "API timeout")
└─ [not executed]
```

**Insight**: Search tool timed out. Root cause found!

---

## 🎯 Trace Complexity Levels

### Level 1: Trivial (< 10KB)
- Single model call
- No tools
- < 5 seconds
- < 2,000 tokens

**Example**: "What is 2+2?"

### Level 2: Simple (10-100KB)
- 2-3 model calls
- 1-2 tool calls
- 5-15 seconds
- 2,000-5,000 tokens

**Example**: "Search for npm updates"

### Level 3: Moderate (100KB-1MB)
- 3-10 model calls
- 3-10 tool calls
- 15-60 seconds
- 5,000-20,000 tokens

**Example**: "Research quantum computing"

### Level 4: Complex (1-10MB)
- 10+ model calls
- 10+ tool calls
- 1-5 minutes
- 20,000-100,000 tokens

**Example**: "Write comprehensive report on AI"

### Level 5: Massive (> 10MB)
- Multi-agent systems
- Parallel execution
- 5+ minutes
- 100,000+ tokens

**Example**: "Deep research with sub-agents"

---

## 📝 Reading Trace JSON

### Basic Structure

```json
{
  "id": "trace_xyz789",
  "name": "agent_execution",
  "run_type": "chain",
  "start_time": "2025-11-15T14:30:00.000Z",
  "end_time": "2025-11-15T14:30:09.600Z",
  "inputs": {
    "messages": [...]
  },
  "outputs": {
    "messages": [...]
  },
  "child_runs": [
    {
      "id": "span_abc123",
      "name": "call_model",
      "run_type": "chain",
      "start_time": "2025-11-15T14:30:00.100Z",
      "end_time": "2025-11-15T14:30:02.400Z",
      "inputs": {...},
      "outputs": {...},
      "child_runs": [...]
    }
  ]
}
```

### Key Fields

**id**: Unique identifier for this span
**name**: Human-readable name
**run_type**: Type of operation (chain, llm, tool, etc.)
**start_time**: When span started
**end_time**: When span ended
**inputs**: Data passed into span
**outputs**: Data returned from span
**child_runs**: Nested spans (children)

---

## 🎓 Key Takeaways

### Understanding Traces:

1. **Trace** = Complete request journey
2. **Span** = One unit of work
3. **Hierarchy** = Tree structure of spans
4. **Duration** = How long each step took
5. **Data** = Inputs, outputs, metadata

### Reading Traces:

- Look at **structure** to understand flow
- Check **durations** to find bottlenecks
- Examine **inputs/outputs** to debug
- Review **errors** to find failures
- Analyze **tokens** to optimize costs

### Trace Complexity:

- **Simple traces**: Linear, fast, easy to understand
- **Complex traces**: Multi-level, slower, require careful analysis
- **Tools add complexity**: Each tool call adds a span
- **Multi-agent systems**: Most complex, hardest to debug

### Next Steps:

✅ You understand what traces and spans are!

**Continue to**:
- **[Tutorial 6: Analyzing Traces](06_analyzing_traces.md)** - Learn analysis techniques
- **[Tutorial 7: Debugging Workflows](07_debugging_workflows.md)** - Debug with traces
- Look at real traces in `track_b_glass_box/traces/`

---

## ❓ Practice Exercise

**Try This**: Open `track_b_glass_box/traces/trace-simple.json` and answer:

1. How long did the entire execution take?
2. How many model calls were made?
3. Which tool was used?
4. How long did the tool call take?
5. What was the user's query?

**Answers**:
1. Check root span duration
2. Count call_model spans
3. Look in tools node
4. Check tool span duration
5. Look at inputs.messages[0].content

---

**🎉 Great work!** You now understand traces and spans! Ready to analyze them in depth? Continue to [Tutorial 6: Analyzing Traces](06_analyzing_traces.md)!
