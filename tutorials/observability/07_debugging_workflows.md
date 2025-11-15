# Tutorial 7: Debugging Workflows with Observability

## 📖 Overview

**What You'll Learn:**
- Using observability to find bugs
- Common failure patterns in AI agents
- Root cause analysis techniques
- Debugging multi-step workflows
- Fixing issues systematically

**Prerequisites:** [Tutorial 6: Analyzing Traces](06_analyzing_traces.md)

**Time to Complete:** 35 minutes | **Difficulty:** ⭐⭐⭐ Advanced

---

## 🐛 Common Agent Failure Patterns

### Pattern 1: Tool Timeout
**Symptom:** Request fails after long wait
**Trace signature:**
```
agent_execution (FAILED after 30s)
└─ tools (FAILED)
   └─ search_tool (timeout after 30s)
```
**Fix:** Increase timeout or optimize tool

### Pattern 2: Invalid Tool Arguments
**Symptom:** Tool called with wrong parameters
**Trace signature:**
```
tools (FAILED)
└─ calculator_tool
   Input: {"equation": "what is 2 + 2"} ← Should be string, not dict
   Error: "TypeError: expected string"
```
**Fix:** Add input validation or improve prompt

### Pattern 3: Loop Exhaustion
**Symptom:** Agent hits recursion limit
**Trace signature:**
```
agent_execution (FAILED)
├─ call_model → tools → call_model → tools → ...
└─ [25 iterations] ← Hit limit!
```
**Fix:** Improve agent logic or increase limit

### Pattern 4: Context Overflow
**Symptom:** Token limit exceeded
**Trace signature:**
```
call_model (FAILED)
Input tokens: 128,000 ← Exceeds model limit!
Error: "Context length exceeded"
```
**Fix:** Summarize context or use larger model

---

## 🔬 Systematic Debugging Process

### Step 1: Reproduce the Issue
1. Get the trace ID from error report
2. Open trace in LangSmith
3. Note the exact error and where it occurred
4. Try to reproduce locally

### Step 2: Identify Root Cause
1. Find the failing span
2. Check inputs to that span
3. Check outputs (if any)
4. Trace backwards to understand why

### Step 3: Form Hypothesis
Based on analysis, hypothesize:
- What went wrong?
- Why did it happen?
- How can we fix it?

### Step 4: Test Fix
1. Implement fix
2. Run test cases
3. Check new traces
4. Verify issue is resolved

### Step 5: Prevent Recurrence
1. Add error handling
2. Add validation
3. Update tests
4. Document the fix

---

## 💡 Real Debugging Examples

### Example 1: Agent Ignores Tool Results

**Symptom:** Agent says "I don't know" despite getting search results

**Initial trace:**
```
agent_execution
├─ call_model: "I'll search for that"
├─ tools: search returns 10 results
└─ call_model: "I don't have information on that"  ← ???
```

**Analysis:**
- Search succeeded ✓
- Results were returned ✓
- Agent didn't use them ✗

**Hypothesis:** Agent not instructed to use results

**Fix:**
```python
system_prompt = """You are a helpful assistant.
IMPORTANT: When you use tools, ALWAYS incorporate 
the tool results into your response."""
```

**New trace:**
```
agent_execution
├─ call_model: "I'll search for that"
├─ tools: search returns 10 results
└─ call_model: "Based on the search results..." ✓
```

**Result:** Fixed! Agent now uses tool results.

---

## 🎯 Debugging Multi-Agent Systems

**Challenge:** Failures in nested agents are hard to trace

**Example trace:**
```
main_agent (FAILED)
├─ planner_agent (success)
├─ executor_agent (FAILED)
│  ├─ research_agent_1 (success)
│  ├─ research_agent_2 (FAILED) ← Problem here!
│  │  └─ search_tool (timeout)
│  └─ research_agent_3 (not started)
└─ [stopped]
```

**Debugging strategy:**
1. Find the deepest failed span (research_agent_2)
2. Understand why it was called
3. Check if it was necessary
4. Fix or add fallback

---

## 🎓 Key Takeaways

**Debugging with Traces:**
- Traces show exactly what happened
- Find failure point quickly
- Trace backwards to find cause
- Test fixes with new traces

**Common Fixes:**
- Add timeouts and retries
- Improve prompts and instructions
- Add input validation
- Implement fallbacks

**Next Steps:**
- [Tutorial 8: Monitoring Metrics](08_monitoring_metrics.md)
- [Tutorial 9: Advanced Patterns](09_advanced_patterns.md)

---

**🎉 Well done!** You can now debug agents systematically! Continue to [Tutorial 8: Monitoring Metrics](08_monitoring_metrics.md)!
