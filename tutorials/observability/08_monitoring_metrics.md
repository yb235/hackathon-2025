# Tutorial 8: Monitoring Metrics - Performance and Cost Tracking

## 📖 Overview

**What You'll Learn:**
- Token usage and cost tracking
- Latency and performance metrics
- Carbon footprint monitoring with CodeCarbon
- Setting up alerts and dashboards
- Optimization strategies

**Prerequisites:** [Tutorial 5: Tracing Basics](05_tracing_basics.md)

**Time to Complete:** 25 minutes | **Difficulty:** ⭐⭐ Medium

---

## 📊 Key Metrics for AI Agents

### 1. Token Usage
**Why track:** Directly impacts cost
**Metrics:**
- Input tokens per request
- Output tokens per request
- Total tokens per request
- Tokens per user/session

**In traces:**
```json
{
  "metrics": {
    "input_tokens": 1,245,
    "output_tokens": 856,
    "total_tokens": 2,101
  }
}
```

### 2. Latency
**Why track:** User experience
**Metrics:**
- Total request time
- Model inference time
- Tool execution time
- Time to first token (TTFT)

**Example:**
```
Total: 9.6s
├─ Model: 4.5s (47%)
├─ Tools: 4.2s (44%)
└─ Overhead: 0.9s (9%)
```

### 3. Cost
**Why track:** Budget management
**Calculation:**
```python
# GPT-5 pricing (example)
input_cost = (input_tokens / 1000) * $0.01
output_cost = (output_tokens / 1000) * $0.03
total_cost = input_cost + output_cost
```

### 4. Success Rate
**Why track:** Reliability
**Metrics:**
- Successful requests / Total requests
- Error rate by type
- Timeout rate

---

## 💰 Cost Tracking Example

**Using TikToken for token counting:**
```python
import tiktoken

def count_tokens(text, model="gpt-5"):
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# Example
text = "What is observability in AI agents?"
tokens = count_tokens(text)
print(f"Tokens: {tokens}")  # Output: Tokens: 8

# Estimate cost
cost = (tokens / 1000) * 0.01
print(f"Estimated cost: ${cost:.4f}")
```

---

## 🌱 Carbon Footprint Tracking

**Using CodeCarbon:**
```python
from codecarbon import EmissionsTracker

# Start tracking
tracker = EmissionsTracker()
tracker.start()

# Run your agent
result = agent.invoke({"messages": [...]})

# Stop and get emissions
emissions = tracker.stop()
print(f"CO2 emissions: {emissions:.4f} kg")
```

**Example output:**
```
CO2 emissions: 0.0023 kg (2.3 grams)
Energy consumed: 0.0045 kWh
```

---

## 📈 Setting Up Monitoring

### LangSmith Dashboards

1. **Go to "Monitoring" tab**
2. **Create dashboard widgets:**
   - Average latency over time
   - Token usage per day
   - Success rate trend
   - Cost per request

### Example Queries

**High latency requests:**
```
Duration > 10s AND Status = success
```

**Expensive requests:**
```
Tokens > 5000 ORDER BY tokens DESC
```

**Recent failures:**
```
Status = error AND Time = last_24h
```

---

## 🚨 Alerts and Thresholds

### Recommended Alerts

**Latency alert:**
```
Alert when: avg(latency) > 5s
Period: Last 10 minutes
Action: Email team@company.com
```

**Cost alert:**
```
Alert when: sum(tokens) > 100,000
Period: Last hour
Action: Slack #cost-alerts
```

**Error rate alert:**
```
Alert when: error_rate > 5%
Period: Last 15 minutes
Action: PagerDuty high-priority
```

---

## 🎯 Optimization Strategies

### Reduce Token Usage
- Summarize long contexts
- Limit response length
- Remove unnecessary information
- Use smaller models for simple tasks

### Reduce Latency
- Parallelize tool calls
- Use faster models
- Add caching
- Optimize tool implementations

### Reduce Costs
- Use cheaper models when possible
- Implement token budgets
- Cache frequent queries
- Batch similar requests

---

## 🎓 Key Takeaways

**Metrics to Track:**
- Token usage (cost)
- Latency (user experience)
- Success rate (reliability)
- Carbon footprint (sustainability)

**Tools:**
- LangSmith (tracing + metrics)
- CodeCarbon (emissions)
- TikToken (token counting)

**Next Steps:**
- [Tutorial 9: Advanced Patterns](09_advanced_patterns.md)
- [Tutorial 10: AWS Observability](10_aws_observability.md)

---

**🎉 Great work!** You now know how to monitor agent performance! Continue to [Tutorial 9: Advanced Patterns](09_advanced_patterns.md)!
