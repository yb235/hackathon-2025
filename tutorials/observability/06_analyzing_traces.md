# Tutorial 6: Analyzing Traces - Deep Dive into Debugging

## 📖 Overview

**What You'll Learn:**
- Step-by-step trace analysis techniques
- Identifying performance bottlenecks
- Understanding decision points
- Analyzing tool calls and responses
- Performance optimization strategies

**Prerequisites:** 
- [Tutorial 5: Tracing Basics](05_tracing_basics.md)

**Time to Complete:** 30 minutes | **Difficulty:** ⭐⭐ Medium

---

## 🔍 Systematic Trace Analysis

### Step 1: Get the Big Picture

**Questions to ask:**
- What was the user trying to do?
- Did the request succeed or fail?
- How long did it take overall?
- Were tools used?

**Where to look:**
```
Root Span:
├─ Name: What operation? (e.g., "agent_execution")
├─ Duration: Total time
├─ Status: Success/Failure/Error
├─ Inputs: User query
└─ Outputs: Final response
```

### Step 2: Analyze the Flow

**Trace the execution path:**
```
Start → call_model → tools → call_model → End
```

**Questions:**
- Does the flow make sense?
- Are there unexpected loops?
- Were tools used appropriately?
- Any redundant operations?

### Step 3: Identify Bottlenecks

**Calculate time distribution:**
```
Total: 15.0s
├─ Model calls: 4.5s (30%)
├─ Tool execution: 9.0s (60%) ← BOTTLENECK
└─ Overhead: 1.5s (10%)
```

**Optimization opportunity:** Focus on the 60% (tool execution)

### Step 4: Examine Decision Points

**Look at each call_model span:**
- What did the model decide to do?
- Why did it choose that action?
- Was it the right decision?

**Example:**
```
call_model_1:
  Input: "Find npm updates"
  Decision: Use search tool with query "npm latest release"
  Reasoning: ✓ Correct - needs current information
```

---

## 📊 Performance Analysis Patterns

### Pattern 1: High Latency

**Symptoms:**
- Total execution time > expected
- User complaint: "It's too slow"

**Analysis:**
```
agent_execution (45.2s) ← TOO LONG!
├─ call_model (2.1s) - OK
├─ tools (40.8s) ← 90% OF TIME!
│  └─ search_tool (40.5s) ← PROBLEM!
└─ call_model (2.3s) - OK
```

**Root causes:**
- Tool timeout or slow API
- Network issues
- Too many tool calls
- Inefficient tool implementation

**Solutions:**
- Add timeout limits
- Parallelize tool calls
- Use faster tools
- Cache frequent queries

### Pattern 2: High Cost

**Symptoms:**
- Token usage > expected
- Cost per request too high

**Analysis:**
```
Total tokens: 25,000 ($0.18) ← EXPENSIVE!
├─ call_model_1: 2,000 tokens ($0.01)
├─ call_model_2: 20,000 tokens ($0.14) ← 80% OF COST!
│  Input: 15,000 tokens ← Why so many?
│  Output: 5,000 tokens ← Response too verbose?
└─ call_model_3: 3,000 tokens ($0.02)
```

**Root causes:**
- Large context windows
- Verbose responses
- Unnecessary information
- Inefficient prompting

**Solutions:**
- Summarize long contexts
- Limit response length
- Remove unnecessary context
- Use cheaper models for simple tasks

### Pattern 3: Failures

**Symptoms:**
- Request fails
- Error in trace

**Analysis:**
```
agent_execution (FAILED)
├─ call_model (success)
├─ tools (FAILED) ← ERROR!
│  └─ search_tool
│     Error: "HTTPTimeout: Request timeout after 30s"
└─ [not executed]
```

**Root causes:**
- Tool timeout
- API errors
- Invalid inputs
- Rate limits

**Solutions:**
- Add retry logic
- Implement fallbacks
- Validate inputs before calling
- Handle errors gracefully

---

## 🎯 Real-World Analysis Examples

### Example 1: Research Agent Too Slow

**Problem:** Research queries take 60+ seconds

**Trace analysis:**
```
agent_execution (67.3s)
├─ call_model: Plan research (2.1s)
├─ search_tool: Query 1 (15.2s)
├─ call_model: Analyze results (2.3s)
├─ search_tool: Query 2 (14.8s)
├─ call_model: More analysis (2.1s)
├─ search_tool: Query 3 (16.1s)  ← Sequential searches!
└─ call_model: Write report (14.7s)
```

**Insight:** Searches are sequential (15s each × 3 = 45s)

**Solution:** Parallelize searches
```python
# Before (sequential): 45s
results1 = search("query 1")  # 15s
results2 = search("query 2")  # 15s
results3 = search("query 3")  # 15s

# After (parallel): 15s
results = await asyncio.gather(
    search("query 1"),
    search("query 2"),
    search("query 3")
)
```

**Result:** 67.3s → 37.3s (45% faster!)

### Example 2: Agent Makes Wrong Decisions

**Problem:** Agent searches but doesn't use results

**Trace analysis:**
```
agent_execution
├─ call_model_1: Decides to search
├─ tools: search_tool returns 10 results
├─ call_model_2: 
│  Input includes: search results (5,000 tokens)
│  Output: "I don't have enough information"  ← IGNORING RESULTS!
└─ End (unhelpful response)
```

**Insight:** Model not using search results

**Possible reasons:**
- Results not in correct format
- Too many results (overwhelming)
- Model not instructed to use them
- Results not relevant

**Solution:** Improve prompt
```python
# Before
system_prompt = "You are a helpful assistant."

# After
system_prompt = """You are a helpful assistant.
When you use search tools, ALWAYS:
1. Read the search results carefully
2. Use information from the results in your response
3. Cite sources when possible"""
```

**Result:** Agent now uses search results effectively

---

## 💡 Advanced Analysis Techniques

### Token Distribution Analysis

**Track where tokens go:**
```
Request breakdown:
├─ System prompt: 500 tokens (constant)
├─ User query: 50 tokens (varies)
├─ Tool results: 3,000 tokens (varies) ← Can we reduce this?
├─ Conversation history: 1,000 tokens (grows over time)
└─ Response: 1,500 tokens
```

**Optimization ideas:**
- Summarize tool results (3,000 → 1,000)
- Limit conversation history (last 5 messages only)
- Compress system prompt

### Temporal Analysis

**Look at time progression:**
```
Time distribution by phase:
├─ Planning phase: 2.1s (14%)
├─ Execution phase: 10.2s (68%) ← Most time here
└─ Synthesis phase: 2.7s (18%)
```

**Questions:**
- Is this the right balance?
- Can we plan faster?
- Can we execute in parallel?

---

## 🛠️ Debugging Workflows

### Workflow 1: Find Why Agent Failed

1. **Locate the error span**
```
Look for red/failed status in trace
```

2. **Read the error message**
```
Error: "Tool timeout after 30s"
```

3. **Check inputs to that span**
```
What was passed to the tool?
Was it valid?
```

4. **Check parent spans**
```
Why was this tool called?
Was it necessary?
```

5. **Formulate hypothesis**
```
Tool is timing out because query is too complex
```

6. **Test fix**
```
Simplify query or increase timeout
```

### Workflow 2: Optimize Performance

1. **Calculate time per component**
2. **Find the slowest component** (> 50% of time)
3. **Analyze why it's slow**
4. **Test optimization**
5. **Measure improvement**

---

## 🎓 Key Takeaways

**Analysis Process:**
1. Understand the big picture
2. Trace the execution flow
3. Identify bottlenecks
4. Examine decision points
5. Formulate optimizations

**Common Issues:**
- **Slow tool calls** → Parallelize or optimize
- **High token usage** → Reduce context or responses
- **Wrong decisions** → Improve prompts
- **Failures** → Add error handling

**Next Steps:**
- [Tutorial 7: Debugging Workflows](07_debugging_workflows.md)
- [Tutorial 8: Monitoring Metrics](08_monitoring_metrics.md)

---

**🎉 Excellent!** You can now analyze traces like a pro! Continue to [Tutorial 7: Debugging Workflows](07_debugging_workflows.md) to learn systematic debugging!
