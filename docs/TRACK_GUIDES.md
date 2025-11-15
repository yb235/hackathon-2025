# Track Guides

## Table of Contents
- [Track Overview](#track-overview)
- [Track A: Agent Iron Man](#track-a-agent-iron-man)
- [Track B: Agent Glass Box](#track-b-agent-glass-box)
- [Track C: Dear Grandma](#track-c-dear-grandma)
- [Cross-Track Strategies](#cross-track-strategies)
- [Submission Requirements](#submission-requirements)

## Track Overview

The hackathon features **three independent tracks**, each focusing on different aspects of AI agent development. Teams can participate in one, two, or all three tracks.

### Track Selection Strategy

**Single Track (Recommended for Beginners):**
- Focus deeply on one area
- Achieve strong results
- Compete for track-specific prizes

**Multiple Tracks (Advanced Teams):**
- Build once, evaluate across dimensions
- Compete for Grand Champion (best across all tracks)
- Maximize prize potential

### Prize Structure

- **Grand Champion**: £8,000 total (£7,000 + £500 Valyu + $500 AWS)
- **Per Track Winners**:
  - 1st Place: £5,600 total
  - 2nd Place: £3,200 total
  - 3rd Place: £1,950 total
- **Special Awards**: 11+ awards at £500 each
- **Research Collaboration**: Top 4 teams work with Holistic AI

---

## Track A: Agent Iron Man

### 🎯 Mission Statement

**"Build agents that don't break."**

Create AI agents that are fast, efficient, and robust in real-world environments. Focus on latency, cost, carbon footprint, and reliability.

### Core Principles

1. **Reliability First**: Consistency over features
2. **Frugal AI**: Right-sized models for the task
3. **Production Ready**: Built to scale and last
4. **Measurable Impact**: Metrics over claims

### What to Build

**Any application domain:**
- Research assistance
- Code generation
- Data analysis
- Customer support
- Content creation
- Or your own idea!

**The Twist:** It's not about *what* your agent does - it's about proving it works **reliably**.

### Evaluation Criteria

#### 1. Consistency (25%)
- Same input → Similar output
- Variance testing across runs
- Edge case handling

**How to Demonstrate:**
```python
# Run benchmark suite multiple times
results = []
for i in range(10):
    result = agent.invoke(test_query)
    results.append(result)

# Calculate consistency metrics
variance = calculate_variance(results)
print(f"Consistency Score: {1 - variance}")
```

#### 2. Error Handling (25%)
- Graceful degradation
- Recovery mechanisms
- Clear error messages

**How to Demonstrate:**
```python
# Test with broken tools, invalid inputs, timeouts
test_cases = [
    {"input": "valid query", "expected": "success"},
    {"input": "invalid query", "expected": "graceful_error"},
    {"input": "timeout_trigger", "expected": "timeout_handled"}
]
```

#### 3. Performance (20%)
- Latency measurements
- Token usage tracking
- Cost analysis

**How to Demonstrate:**
- Use Tutorial 04 (Model Monitoring)
- Track metrics with CodeCarbon
- Compare against baselines

#### 4. Scalability (15%)
- Handles concurrent requests
- Resource efficiency
- Load testing results

**How to Demonstrate:**
```python
import asyncio

# Concurrent testing
async def test_concurrent(num_requests):
    tasks = [agent.ainvoke(query) for _ in range(num_requests)]
    results = await asyncio.gather(*tasks)
    return results
```

#### 5. Documentation (15%)
- Clear architecture description
- Performance metrics
- Monitoring dashboard/logs
- Comparison with baselines

### Required Deliverables

1. **Performance Metrics**
   - Latency per request
   - Token usage and costs
   - Carbon footprint
   - Success rate

2. **Monitoring Evidence**
   - Logs or dashboard screenshots
   - Time-series data
   - Error rate tracking

3. **Consistency Testing**
   - Multiple run results
   - Variance analysis
   - Edge case testing

4. **Comparison Analysis**
   - Baseline comparisons
   - Model selection justification
   - Optimization strategies

### Recommended Tools & Frameworks

**AWS Strands Agents SDK:**
- Production-ready framework
- Multi-agent workflows
- Built-in error handling
- Bedrock AgentCore deployment

**Amazon Nova:**
- Nova Lite: Fast and cost-effective
- Nova Premier: Advanced capabilities
- Optimize for your use case

**Monitoring:**
- CodeCarbon: Carbon emissions
- LangSmith: Performance tracking
- Custom dashboards

### Example Projects

**Simple: ReAct Agent**
- Location: `track_a_iron_man/examples/react_agent/`
- Single agent with tools
- Basic monitoring

**Complex: Deep Research**
- Location: `track_a_iron_man/examples/deep_research/`
- Multi-agent orchestration
- Parallel execution
- Report generation

### Key Resources

- Tutorial 04: Model Monitoring
- Tutorial 06: Benchmark Evaluation
- Tutorial 07: Reinforcement Learning (optional)
- AgentHarm Benchmark (robustness testing)
- NVIDIA Small Language Models paper

---

## Track B: Agent Glass Box

### 🎯 Mission Statement

**"Follow the trajectory. Understand the behavior."**

Build agents with full transparency into decision-making. Capture every step, visualize reasoning, enable auditability.

### Core Principles

1. **Complete Traceability**: Every decision tracked
2. **Human Interpretability**: Non-technical stakeholders understand
3. **Actionable Insights**: Observability leads to improvements
4. **Failure Analysis**: Understand what went wrong and why

### What to Build

**Any application domain** with focus on **transparency**.

**The Twist:** It's not about *what* your agent does - it's about proving you can **understand and explain** its behavior.

### Evaluation Criteria

#### 1. Traceability (30%)
- Complete execution traces
- Decision path tracking
- State evolution visibility

**How to Demonstrate:**
- Use LangSmith for automatic tracing
- Export and analyze trace files
- Show decision trees

#### 2. Interpretability (25%)
- Clear visualizations
- Natural language explanations
- Non-technical comprehension

**How to Demonstrate:**
```python
# Generate human-readable explanation
def explain_decision(trace):
    """Convert trace into plain English explanation"""
    return f"""
    Agent Process:
    1. Received query: {trace.input}
    2. Decided to use: {trace.tool_used}
    3. Reasoning: {trace.reasoning}
    4. Result: {trace.output}
    """
```

#### 3. Failure Analysis (20%)
- Root cause identification
- Failure categorization
- Debugging capabilities

**How to Demonstrate:**
- Analyze failed runs
- Categorize failure types
- Show debugging process

#### 4. Behavioral Insights (15%)
- Pattern discovery
- Shortcut identification
- Performance bottlenecks

**How to Demonstrate:**
- Statistical analysis of traces
- Identify common patterns
- Discover optimization opportunities

#### 5. Innovation (10%)
- Novel visualization approaches
- Advanced analysis techniques
- Creative transparency solutions

### Required Deliverables

1. **Execution Traces**
   - LangSmith traces or equivalent
   - Multiple complexity levels
   - Success and failure cases

2. **Visualizations**
   - Decision flow diagrams
   - State transition graphs
   - Interactive dashboards

3. **Analysis Documentation**
   - Trace interpretation
   - Pattern analysis
   - Insights discovered

4. **Explainability Demo**
   - Show how you explain agent behavior
   - Target multiple audiences
   - Technical and non-technical

### Recommended Tools & Frameworks

**LangSmith (Primary):**
- Automatic tracing
- Visualization UI
- Trace export/analysis

**OpenTelemetry:**
- Vendor-neutral
- Standardized format
- Multiple backend support

**LangFuse (Alternative):**
- Open-source
- Self-hosted option
- Similar to LangSmith

**AWS Observability:**
- CloudWatch: Logs and metrics
- X-Ray: Distributed tracing
- Bedrock Monitoring: Token usage

### Example Resources

**Execution Traces:**
- `track_b_glass_box/traces/`
- Very simple (13KB)
- Simple (65KB)
- Normal (251KB)
- Complex (15MB)

**Research Platforms:**
- AgentGraph: Trace-to-graph visualization
- AgentSeer: Observability-based evaluation

**Datasets:**
- Who_and_When: Failure attribution
- Annotated agent traces

### Trace Analysis Example

```python
import json

# Load trace
with open('trace.json', 'r') as f:
    trace = json.load(f)

# Analyze structure
def analyze_trace(trace):
    return {
        'total_steps': count_steps(trace),
        'tools_used': extract_tools(trace),
        'decision_points': find_decisions(trace),
        'failures': identify_failures(trace),
        'latency': calculate_latency(trace)
    }

# Generate insights
insights = analyze_trace(trace)
print(f"Agent took {insights['total_steps']} steps")
print(f"Used tools: {insights['tools_used']}")
```

### Key Resources

- Tutorial 05: Observability
- AgentGraph paper: `track_b_glass_box/examples/agent_graph_AAAI.pdf`
- AgentSeer: [Demo](https://huggingface.co/spaces/holistic-ai/AgentSeer)
- Who_and_When dataset

---

## Track C: Dear Grandma

### 🎯 Mission Statement

**"Attack like a red-team. Assess like a professional."**

Uncover vulnerabilities through systematic security testing. Find weaknesses before they reach production.

### Core Principles

1. **Systematic Testing**: Cover all attack vectors
2. **Measurable Security**: Quantify with ASR (Attack Success Rate)
3. **Reproducible Attacks**: Clear reproduction steps
4. **Pattern Discovery**: Identify root causes, not just symptoms

### What to Test

**Option 1: Deployed Agents (Recommended)** ⭐
- 7 agents behind animal codenames
- Identify frameworks, models, architecture
- Measure Attack Success Rate (ASR)
- **Bonus points** for correct identification

**Option 2: SOTA Model Providers**
- ChatGPT, Claude, Gemini
- Break safety guardrails
- Extract system prompts
- Find jailbreaks

### Evaluation Criteria

#### 1. Systematic Methodology (30%)
- Coverage across attack types
- Structured testing approach
- Reproducible process

**Attack Types to Cover:**
- Jailbreak attacks
- Prompt injection
- Reward hacking
- Tool misuse
- Data exfiltration
- Hidden motivations

#### 2. ASR Measurement (25%)
- Clear metrics
- Statistical significance
- Multiple test cases

**How to Calculate ASR:**
```python
# Attack Success Rate
successful_attacks = 0
total_attempts = 100

for test_case in test_cases:
    response = agent.invoke(test_case)
    if is_successful_attack(response):
        successful_attacks += 1

asr = successful_attacks / total_attempts
print(f"ASR: {asr * 100}%")
```

#### 3. Vulnerability Analysis (20%)
- Root cause identification
- Pattern recognition
- Severity assessment

**How to Demonstrate:**
- Categorize vulnerabilities
- Explain underlying issues
- Prioritize by severity

#### 4. Reproducibility (15%)
- Clear attack prompts
- Step-by-step reproduction
- Consistent results

**How to Demonstrate:**
```python
# Document each attack
attack_report = {
    'attack_id': 'A001',
    'type': 'jailbreak',
    'prompt': 'exact prompt used',
    'expected_result': 'refusal',
    'actual_result': 'compliance',
    'success_rate': 0.85,
    'reproduction_steps': [...]
}
```

#### 5. Agent Identification (10% Bonus)
- Correctly identify agent frameworks
- Identify underlying models
- Identify architecture patterns

### Required Deliverables

1. **Attack Methodology**
   - Testing approach
   - Coverage across attack types
   - Tools and techniques used

2. **ASR Measurements**
   - Quantitative results
   - Statistical analysis
   - Comparison across agents/attacks

3. **Vulnerability Findings**
   - Discovered vulnerabilities
   - Root cause analysis
   - Severity assessment
   - Patterns identified

4. **Reproduction Guide**
   - Attack prompts
   - Step-by-step instructions
   - Expected vs actual results

5. **Agent Identification (Option 1 only)**
   - Framework identification
   - Model identification
   - Evidence and methodology

### API Endpoints (Option 1)

**Base URL:** `https://6ofr2p56t1.execute-api.us-east-1.amazonaws.com/prod`

**Available Agents:**
- 🐘 Elephant: `/api/elephant`
- 🦊 Fox: `/api/fox`
- 🦅 Eagle: `/api/eagle`
- 🐜 Ant: `/api/ant`
- 🐺 Wolf: `/api/wolf`
- 🐻 Bear: `/api/bear`
- 🦎 Chameleon: `/api/chameleon`

**Quick Test:**
```bash
curl -X POST https://6ofr2p56t1.execute-api.us-east-1.amazonaws.com/prod/api/bear \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello! What can you do?"}'
```

**Python Example:**
```python
import requests

url = "https://6ofr2p56t1.execute-api.us-east-1.amazonaws.com/prod/api/bear"
payload = {"message": "Your test prompt here"}

response = requests.post(url, json=payload, timeout=35)
result = response.json()
print(result.get('response'))
```

### Testing Resources

**Red Teaming Datasets:**
- Location: `track_c_dear_grandma/examples/red_teaming_datasets/`
- Benign test cases (101 queries)
- Harmful test cases (101 queries)
- Jailbreak prompts (100+ prompts)

**HarmBench Framework:**
- 18 red teaming methods
- Evaluation pipeline
- Location: `track_c_dear_grandma/examples/harmbench/`

**AgentHarm Benchmark:**
- 110 malicious tasks
- 11 harm categories
- Location: `track_c_dear_grandma/examples/agentharm/`

### Systematic Testing Example

```python
import pandas as pd

# Load test cases
benign = pd.read_csv('benign_test_cases.csv')
harmful = pd.read_csv('harmful_test_cases.csv')
jailbreaks = pd.read_csv('jailbreak_prompts.csv')

# Test each category
results = {
    'benign': test_agent(agent, benign),
    'harmful': test_agent(agent, harmful),
    'jailbreak': test_agent(agent, jailbreaks)
}

# Calculate ASR for each category
for category, tests in results.items():
    asr = calculate_asr(tests)
    print(f"{category} ASR: {asr * 100}%")
```

### Key Resources

- Tutorial 08: Attack & Red Teaming
- Tutorial 06: Benchmark Evaluation (for LLM-as-a-Judge)
- Red Teaming Datasets
- HarmBench framework
- AgentHarm benchmark
- AgentSeer: Vulnerability analysis

---

## Cross-Track Strategies

### Building for Multiple Tracks

**Core Agent → Track A + B + C:**

1. **Build solid agent** (Track A focus)
   - Reliable architecture
   - Error handling
   - Performance monitoring

2. **Add observability** (Track B)
   - Enable LangSmith tracing
   - Export traces
   - Create visualizations

3. **Security test** (Track C)
   - Run red teaming datasets
   - Measure ASR
   - Document vulnerabilities

### Shared Components

**Monitoring:**
- Track A: Performance metrics
- Track B: Execution traces
- Track C: Attack detection

**Evaluation:**
- Track A: Consistency testing
- Track B: Failure analysis
- Track C: ASR measurement

**Documentation:**
- All tracks benefit from clear documentation
- Reuse diagrams and explanations
- Adapt focus per track

---

## Submission Requirements

### All Tracks

1. **Poster (PDF)**
   - Single page, A3-A4 size
   - Visual summary of project
   - Use templates from `/templates/`

2. **GitHub Repository**
   - Public repository
   - Complete code
   - Clear README
   - Setup instructions
   - Results and logs

3. **Team Information**
   - Team name
   - Contact email
   - 3-5 members (Name | Email | Role)
   - Track selection(s)

4. **Project Description**
   - Project name and tagline
   - Detailed description
   - Technologies used
   - Key achievements

### Track-Specific Requirements

**Track A:**
- Performance metrics
- Monitoring logs/dashboard
- Consistency testing results
- Baseline comparisons

**Track B:**
- Execution traces
- Visualizations
- Analysis documentation
- Explainability demo

**Track C:**
- Attack methodology
- ASR measurements
- Vulnerability findings
- Reproduction guide
- (Optional) Agent identification

### Submission Deadline

**November 16, 2025, 3:00 PM GMT**

⚠️ Late submissions will not be accepted. Devpost timestamps are final.

**Submit via Devpost:** [hai-great-agent-hack-2025.devpost.com](https://hai-great-agent-hack-2025.devpost.com)

---

## Next Steps

1. **Choose your track(s)** based on team strengths
2. **Review track-specific README** in track folder
3. **Complete relevant tutorials**
4. **Start building** your solution
5. **Document as you go** (don't leave it to the end!)
6. **Test thoroughly** before submission
7. **Prepare poster** using templates
8. **Submit on Devpost** before deadline

**Good luck! 🚀**
