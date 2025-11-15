# Tutorial 12: Hands-On Exercises and Practice

## 📖 Overview

**What You'll Learn:**
- Practical exercises with solutions
- Real-world scenarios
- Challenge problems
- Self-assessment

**Prerequisites:** Tutorials 1-11

**Time to Complete:** 60+ minutes | **Difficulty:** ⭐⭐⭐ Advanced

---

## 🎯 Exercise 1: Analyze a Trace

**Task:** Open `track_b_glass_box/traces/trace-simple.json`

**Questions:**
1. What was the user query?
2. How many model calls were made?
3. Which tool was used and how long did it take?
4. What was the total token count?
5. What optimization would you suggest?

**Solution Guide:**
```
1. Look at root span inputs.messages[0].content
2. Count spans with name="call_model"
3. Find span with run_type="tool"
4. Sum token counts from all model calls
5. Check if tool calls could be parallelized
```

---

## 🎯 Exercise 2: Debug a Failure

**Scenario:** Agent times out on complex queries

**Your task:**
1. Create an agent that searches for information
2. Make it timeout (use very low timeout)
3. Analyze the trace to find the problem
4. Fix it and verify in new trace

**Solution:**
```python
# Intentionally low timeout
agent = create_react_agent(
    tools=[search_tool],
    timeout=1  # Too low!
)

# After finding problem in trace, increase timeout
agent = create_react_agent(
    tools=[search_tool],
    timeout=30  # Better
)
```

---

## 🎯 Exercise 3: Optimize Costs

**Scenario:** Agent costs too much per request

**Your task:**
1. Analyze token usage in traces
2. Find the most expensive component
3. Implement optimization
4. Measure cost reduction

**Common optimizations:**
- Reduce system prompt length
- Summarize tool results
- Limit response length
- Use cheaper model for simple tasks

---

## 🎯 Exercise 4: Set Up Monitoring

**Task:**
1. Create a LangSmith project
2. Run 10 test queries
3. Create a dashboard showing:
   - Average latency
   - Token usage
   - Success rate
4. Set up an alert for high latency

---

## 🎯 Challenge: Build an Observable Agent

**Final challenge:**

Build a research agent that:
- Uses at least 2 tools
- Has complete observability
- Tracks custom metrics
- Has error handling
- Costs < $0.05 per query
- Responds in < 10 seconds

**Requirements:**
- LangSmith tracing enabled
- Custom metadata for each request
- Cost tracking
- Performance monitoring
- Error logging

---

## �� Self-Assessment

**Check your understanding:**

✅ I can read and understand traces  
✅ I can identify performance bottlenecks  
✅ I can debug agent failures  
✅ I can track costs and optimize  
✅ I can set up production monitoring  
✅ I understand AWS observability tools  
✅ I can implement best practices  

**If you checked all boxes, congratulations! You've mastered observability for AI agents!**

---

## 📚 What's Next?

**Apply your knowledge:**
- Build agents for Track B: Glass Box
- Implement observability in your own projects
- Share learnings with the community
- Continue learning about advanced patterns

**Resources:**
- LangSmith Documentation
- Track B materials
- Community Discord
- Research papers in `track_b_glass_box/examples/`

---

**🎉 Congratulations!** You've completed the entire observability tutorial series! You're now equipped to build production-grade observable AI agents!

**Thank you for learning with us!** 🚀
