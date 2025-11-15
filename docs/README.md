# Documentation Index

Welcome to the comprehensive documentation for the Holistic AI x UCL Hackathon 2025 repository! This documentation is designed to help first-time users understand the codebase, architecture, and workflow.

## 📚 Documentation Overview

This documentation provides a complete guide to building production-grade AI agents for the hackathon. Whether you're new to AI agents or an experienced developer, you'll find everything you need here.

## 🗂️ Documentation Files

### 1. [ARCHITECTURE.md](ARCHITECTURE.md)
**Understanding the System Design**

Learn about the high-level architecture, core components, and technology stack:
- System overview and component diagram
- ReAct agent framework architecture
- State management and graph-based execution
- Tool system design
- Technology stack (LangGraph, LangChain, AWS Bedrock)
- AWS integration and deployment patterns

**Read this to understand:** How the entire system is structured and how components work together.

---

### 2. [CODE_STRUCTURE.md](CODE_STRUCTURE.md)
**Navigating the Codebase**

Detailed breakdown of the repository organization and file structure:
- Directory structure and organization
- Core modules (`react_agent`, `valyu_tools`)
- Track directories (A, B, C) and their contents
- Tutorial sequence and dependencies
- Configuration files
- File relationships and import patterns

**Read this to understand:** Where everything is located and how to navigate the codebase.

---

### 3. [WORKFLOW.md](WORKFLOW.md)
**Understanding Agent Execution**

Deep dive into the agent lifecycle and execution patterns:
- Agent creation, invocation, and cleanup phases
- Message flow through the system
- Execution patterns (single-turn, tool-using, multi-turn, structured output)
- Tool execution lifecycle
- State transitions and updates
- Error handling mechanisms
- Common workflows and timing

**Read this to understand:** How agents execute tasks and process information.

---

### 4. [API_REFERENCE.md](API_REFERENCE.md)
**API Documentation**

Complete API reference for all core functions and classes:
- `create_react_agent()` - Agent factory function
- `HolisticAIBedrockChat` - AWS Bedrock integration
- `ValyuSearchTool`, `ValyuContentsTool` - Built-in tools
- State and context APIs
- Environment variables
- Error handling

**Read this to understand:** How to use each API and what parameters are available.

---

### 5. [GETTING_STARTED.md](GETTING_STARTED.md)
**Setup and First Steps**

Step-by-step guide to get up and running:
- Prerequisites and system requirements
- Installation instructions
- Environment setup (.env configuration)
- First agent examples (5, 10, 15, 20 minutes)
- Tutorial learning path
- Common issues and troubleshooting
- Getting help

**Read this first if:** You're setting up the repository for the first time.

---

### 6. [TRACK_GUIDES.md](TRACK_GUIDES.md)
**Competition Track Information**

Detailed guide for each hackathon track:
- **Track A (Iron Man)**: Reliability and performance
- **Track B (Glass Box)**: Observability and explainability  
- **Track C (Dear Grandma)**: Security and red teaming
- Evaluation criteria for each track
- Required deliverables
- Cross-track strategies
- Submission requirements

**Read this to understand:** What each track requires and how to succeed.

---

### 7. [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md)
**Tools and Utilities Guide**

Comprehensive reference for all tools and utilities:
- Built-in tools (ValyuSearchTool, ValyuContentsTool)
- Monitoring tools (CodeCarbon, TikToken)
- Development utilities (model loaders, context)
- Testing tools (benchmarks, LLM-as-a-Judge, ASR)
- Observability tools (LangSmith, OpenTelemetry)
- Creating custom tools with examples

**Read this to understand:** Available tools and how to use or create them.

---

## 🚀 Quick Start Guides

### For Complete Beginners
1. **Start here:** [GETTING_STARTED.md](GETTING_STARTED.md)
2. **Then read:** [ARCHITECTURE.md](ARCHITECTURE.md) - System overview
3. **Then read:** [CODE_STRUCTURE.md](CODE_STRUCTURE.md) - Navigation
4. **Then explore:** Tutorials in `/tutorials` folder
5. **Finally read:** [TRACK_GUIDES.md](TRACK_GUIDES.md) - Choose your track

### For Experienced Developers
1. **Skim:** [ARCHITECTURE.md](ARCHITECTURE.md) - Get the big picture
2. **Reference:** [API_REFERENCE.md](API_REFERENCE.md) - As needed
3. **Deep dive:** [WORKFLOW.md](WORKFLOW.md) - Understand execution
4. **Choose track:** [TRACK_GUIDES.md](TRACK_GUIDES.md)
5. **Start building:** Use [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md) as reference

### For Track-Specific Development

**Track A (Iron Man):**
1. [GETTING_STARTED.md](GETTING_STARTED.md) - Setup
2. [TRACK_GUIDES.md](TRACK_GUIDES.md#track-a-agent-iron-man) - Track A details
3. [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#monitoring-tools) - Monitoring
4. `/track_a_iron_man/README.md` - Track-specific resources

**Track B (Glass Box):**
1. [GETTING_STARTED.md](GETTING_STARTED.md) - Setup
2. [TRACK_GUIDES.md](TRACK_GUIDES.md#track-b-agent-glass-box) - Track B details
3. [WORKFLOW.md](WORKFLOW.md) - Understand execution flow
4. [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#observability-tools) - Observability
5. `/track_b_glass_box/README.md` - Track-specific resources

**Track C (Dear Grandma):**
1. [GETTING_STARTED.md](GETTING_STARTED.md) - Setup
2. [TRACK_GUIDES.md](TRACK_GUIDES.md#track-c-dear-grandma) - Track C details
3. [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#testing-tools) - Testing & ASR
4. `/track_c_dear_grandma/README.md` - Track-specific resources

---

## 📖 Additional Documentation

### Repository Root
- **`README.md`**: Main repository overview, quick start, tracks, resources
- **`.env.example`**: Environment variable template

### Track Folders
- **`track_a_iron_man/README.md`**: Performance and reliability focus
- **`track_b_glass_box/README.md`**: Observability and explainability focus
- **`track_c_dear_grandma/README.md`**: Security and red teaming focus

### Tutorials
- **`tutorials/README.md`**: Tutorial overview and setup
- **8 Jupyter notebooks**: Hands-on learning from basics to advanced

### Core Module
- **`core/README.md`**: Optional starter code documentation

### Event Documentation
- **`docs/EVENT_SCHEDULE.md`**: Hackathon schedule
- **`docs/HACKATHON_RULES.md`**: Official rules
- **`docs/FAQ.md`**: Frequently asked questions
- **`docs/JUDGES.md`**: Judge information
- **`docs/CODE_OF_CONDUCT.md`**: Code of conduct

---

## 🎯 Key Concepts Explained

### What is a ReAct Agent?
A **ReAct (Reasoning + Acting)** agent follows a loop:
1. **Reason**: Analyze the situation
2. **Act**: Execute an action (use a tool or respond)
3. **Observe**: Process the results
4. **Repeat**: Continue until task is complete

See: [ARCHITECTURE.md](ARCHITECTURE.md#1-react-agent-framework-corereact_agent)

### What is LangGraph?
**LangGraph** is a library for building stateful, multi-actor applications with LLMs. It uses a graph-based approach to orchestrate agent execution.

See: [ARCHITECTURE.md](ARCHITECTURE.md#3-graph-based-execution)

### What is State Management?
**State** tracks the conversation history and agent status. Messages accumulate in an append-only fashion, with each step adding new information.

See: [WORKFLOW.md](WORKFLOW.md#state-transitions)

### What are Tools?
**Tools** are functions that extend agent capabilities (search, calculate, API calls, etc.). The agent decides when to use tools based on the task.

See: [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md)

---

## 🔍 Finding Information

### By Topic

**Setting up the environment:**
→ [GETTING_STARTED.md](GETTING_STARTED.md#environment-setup)

**Creating your first agent:**
→ [GETTING_STARTED.md](GETTING_STARTED.md#first-agent)

**Understanding agent execution:**
→ [WORKFLOW.md](WORKFLOW.md#message-flow)

**Using tools:**
→ [API_REFERENCE.md](API_REFERENCE.md#tool-apis)
→ [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#built-in-tools)

**Creating custom tools:**
→ [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#creating-custom-tools)

**Monitoring performance:**
→ [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#monitoring-tools)

**Tracking observability:**
→ [TOOLS_REFERENCE.md](TOOLS_REFERENCE.md#observability-tools)

**Security testing:**
→ [TRACK_GUIDES.md](TRACK_GUIDES.md#track-c-dear-grandma)

**AWS Bedrock integration:**
→ [ARCHITECTURE.md](ARCHITECTURE.md#aws-integration)

**Error handling:**
→ [WORKFLOW.md](WORKFLOW.md#error-handling)

---

## 💡 Common Questions

### Q: I'm new to AI agents. Where do I start?
**A:** Start with [GETTING_STARTED.md](GETTING_STARTED.md), then work through Tutorial 01 (`tutorials/01_basic_agent.ipynb`).

### Q: How do I choose between models?
**A:** See [API_REFERENCE.md](API_REFERENCE.md#get_chat_model) for model options. For hackathon, use Holistic AI Bedrock (recommended) with Claude 3.5 Sonnet (balanced) or Haiku (faster).

### Q: What's the difference between the tracks?
**A:** 
- **Track A**: Build reliable, efficient agents
- **Track B**: Build observable, explainable agents
- **Track C**: Test security and find vulnerabilities

See [TRACK_GUIDES.md](TRACK_GUIDES.md) for details.

### Q: Can I use my own tools or frameworks?
**A:** Yes! The `core/` folder is optional. You can build from scratch using any framework.

### Q: How do I get help during the hackathon?
**A:** Join Discord ([link](https://discord.com/invite/QBTtWP2SU6?referrer=luma)), post in `#ask-for-help`, or contact Zekun Wu (`@zekunwu_73994`).

### Q: What should I submit?
**A:** Poster (PDF), GitHub repository, team info, and track-specific deliverables. See [TRACK_GUIDES.md](TRACK_GUIDES.md#submission-requirements).

---

## 🛠️ Tools and Resources

### Essential Tools
- **LangGraph**: Agent orchestration
- **LangChain**: LLM abstractions
- **AWS Bedrock**: Model access (via Holistic AI)
- **Valyu**: Search tool
- **LangSmith**: Observability
- **CodeCarbon**: Carbon tracking

### Optional Tools
- **OpenAI**: Alternative model provider
- **Ollama**: Local models
- **HarmBench**: Advanced red teaming
- **AgentGraph**: Trace visualization

---

## 📞 Support

### During Hackathon
- **Discord** (fastest): [#ask-for-help channel](https://discord.com/invite/QBTtWP2SU6?referrer=luma)
- **GitHub Issues**: [Report bugs](https://github.com/holistic-ai/hackthon-2025/issues)
- **Email**: zekun.wu@holisticai.com

### Resources
- **Event Website**: [hackathon.holisticai.com](https://hackathon.holisticai.com/)
- **Devpost**: [hai-great-agent-hack-2025.devpost.com](https://hai-great-agent-hack-2025.devpost.com)
- **API Guide**: `/assets/api-guide.pdf`

---

## 🎓 Learning Path

### Beginner Path (6-8 hours)
1. Read [GETTING_STARTED.md](GETTING_STARTED.md) (1 hour)
2. Complete Tutorial 01: Basic Agent (30 min)
3. Complete Tutorial 02: Custom Tools (45 min)
4. Complete Tutorial 03: Structured Output (30 min)
5. Choose track and read [TRACK_GUIDES.md](TRACK_GUIDES.md) (1 hour)
6. Build your project (3-4 hours)

### Intermediate Path (4-6 hours)
1. Skim [ARCHITECTURE.md](ARCHITECTURE.md) (30 min)
2. Read track guide in [TRACK_GUIDES.md](TRACK_GUIDES.md) (30 min)
3. Complete track-specific tutorials (1-2 hours)
4. Review examples in track folders (30 min)
5. Build your project (2-3 hours)

### Advanced Path (2-4 hours)
1. Quick review of [API_REFERENCE.md](API_REFERENCE.md) (20 min)
2. Review [WORKFLOW.md](WORKFLOW.md) for execution patterns (30 min)
3. Choose track and plan approach (30 min)
4. Build and iterate (1-2.5 hours)

---

## 🏆 Success Criteria

### For Track A (Iron Man)
- ✅ Performance metrics documented
- ✅ Error handling demonstrated
- ✅ Consistency testing completed
- ✅ Monitoring in place

### For Track B (Glass Box)
- ✅ Execution traces captured
- ✅ Visualizations created
- ✅ Analysis documented
- ✅ Explainability demonstrated

### For Track C (Dear Grandma)
- ✅ Systematic testing methodology
- ✅ ASR measurements calculated
- ✅ Vulnerabilities documented
- ✅ Attacks reproducible

---

## 🚀 Ready to Build?

1. **✅ Set up environment**: [GETTING_STARTED.md](GETTING_STARTED.md)
2. **📚 Learn the basics**: Tutorials 01-03
3. **🎯 Choose your track**: [TRACK_GUIDES.md](TRACK_GUIDES.md)
4. **🔨 Start building**: Use docs as reference
5. **📊 Track progress**: Monitor and document
6. **🎉 Submit**: Prepare poster and repository

**Good luck, and happy building! 🎊**

---

## 📝 Documentation Feedback

Found an issue or have a suggestion? 
- Open an issue: [GitHub Issues](https://github.com/holistic-ai/hackthon-2025/issues)
- Contact: zekun.wu@holisticai.com

---

**Last Updated:** November 15, 2025
**Repository:** [github.com/holistic-ai/hackthon-2025](https://github.com/holistic-ai/hackthon-2025)
**Event:** [hackathon.holisticai.com](https://hackathon.holisticai.com/)
