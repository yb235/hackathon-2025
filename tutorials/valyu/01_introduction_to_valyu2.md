# Tutorial 1: Introduction to Valyu 2 - What, Why, and How

## Table of Contents
- [What is Valyu 2?](#what-is-valyu-2)
- [Why Use Valyu 2?](#why-use-valyu-2)
- [Key Features](#key-features)
- [How Valyu 2 Works](#how-valyu-2-works)
- [Use Cases](#use-cases)
- [Prerequisites](#prerequisites)
- [Getting Your API Key](#getting-your-api-key)

## What is Valyu 2?

**Valyu 2** is an advanced AI-powered search and content extraction platform designed specifically for building intelligent AI agents. Unlike traditional search engines that simply return links, Valyu 2 provides **deep, contextual content** that AI models can directly use to answer questions and perform tasks.

### The Problem Valyu 2 Solves

When building AI agents, you often face these challenges:

1. **Limited Knowledge**: AI models are trained on data up to a certain date and lack real-time information
2. **Hallucinations**: Without access to factual data, models may generate incorrect information
3. **Context Limitations**: Models have token limits and can't process entire websites or documents
4. **Source Reliability**: Hard to verify where information comes from

### The Valyu 2 Solution

Valyu 2 acts as a **bridge between AI agents and real-world information**, providing:

- **Real-time search** across web and proprietary sources
- **Pre-processed, clean content** ready for AI consumption
- **Relevance scoring** to filter out noise
- **Source attribution** for verification
- **Cost-optimized retrieval** with pricing controls

Think of Valyu 2 as giving your AI agent a **research assistant** that can:
- Search the internet on demand
- Read and summarize web pages
- Extract key information
- Return structured, clean data

## Why Use Valyu 2?

### 1. **Built for AI Agents**

Valyu 2 is specifically designed for LLM consumption:

```
Traditional Search Engine:
Query → List of URLs → Agent must visit each URL → Parse HTML → Extract content

Valyu 2:
Query → Pre-processed, clean content → Ready for AI to use
```

### 2. **Better Results**

- **Relevance Filtering**: Only returns content above your threshold
- **Smart Ranking**: Uses AI to understand semantic relevance
- **Multi-Source**: Searches both public web and proprietary sources
- **Fresh Data**: Real-time access to latest information

### 3. **Cost Control**

AI operations have costs. Valyu 2 helps you manage them:

- **Max Price Limits**: Set budget caps per search
- **Result Limits**: Control how many results to retrieve
- **Fast Mode**: Get quicker, shorter results when appropriate
- **Response Length Control**: Choose how much content to return

### 4. **Production Ready**

- **Rate Limiting**: Built-in protection against abuse
- **Error Handling**: Graceful failures with meaningful messages
- **Monitoring**: Track usage and costs
- **Scalability**: Handles high-volume requests

## Key Features

### 1. **Deep Search** (`ValyuSearchTool`)

Searches across multiple sources and returns structured content:

```python
from langchain_valyu import ValyuSearchTool

tool = ValyuSearchTool()
result = tool._run(
    query="What is quantum computing?",
    max_num_results=5,
    relevance_threshold=0.7
)
```

**Parameters:**
- `query`: Your search question
- `search_type`: 'all', 'web', or 'proprietary'
- `max_num_results`: How many results (1-20)
- `relevance_threshold`: Minimum relevance score (0.0-1.0)
- `max_price`: Budget limit in dollars
- `fast_mode`: Quick results vs comprehensive
- `start_date`/`end_date`: Time filtering
- `country_code`: Geographic bias
- `included_sources`/`excluded_sources`: Source filtering

### 2. **Contents Extraction** (`ValyuContentsTool`)

Extract clean, readable content from specific URLs:

```python
from langchain_valyu import ValyuContentsTool

tool = ValyuContentsTool()
result = tool._run(
    urls=["https://example.com/article1", "https://example.com/article2"]
)
```

**Parameters:**
- `urls`: List of URLs (max 10)
- `summary`: Boolean or custom summary prompt
- `extract_effort`: 'normal', 'high', 'auto'
- `response_length`: 'short', 'medium', 'large', 'max', or character count

### 3. **Retrievers for RAG**

LangChain-compatible retrievers for Retrieval-Augmented Generation:

```python
from langchain_valyu import ValyuRetriever

retriever = ValyuRetriever(k=5, search_type="all")
docs = retriever.get_relevant_documents("quantum computing")
```

## How Valyu 2 Works

### Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    Your AI Agent                     │
│                                                      │
│  "What is quantum computing?"                       │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ Tool Call
                  ▼
┌─────────────────────────────────────────────────────┐
│               ValyuSearchTool                        │
│                                                      │
│  - Validates parameters                             │
│  - Prepares API request                             │
│  - Handles authentication                           │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ HTTP Request
                  ▼
┌─────────────────────────────────────────────────────┐
│               Valyu API Service                      │
│                                                      │
│  1. Query Understanding (AI-powered)                │
│  2. Multi-Source Search                             │
│     - Web crawling                                   │
│     - Proprietary databases                         │
│     - Real-time indexing                            │
│  3. Content Extraction                              │
│     - HTML parsing                                   │
│     - Text cleaning                                  │
│     - Structure preservation                        │
│  4. Relevance Ranking (AI-powered)                  │
│  5. Result Formatting                               │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ JSON Response
                  ▼
┌─────────────────────────────────────────────────────┐
│               ValyuSearchTool                        │
│                                                      │
│  - Parses response                                  │
│  - Validates data                                   │
│  - Returns structured result                        │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ Formatted Data
                  ▼
┌─────────────────────────────────────────────────────┐
│                    Your AI Agent                     │
│                                                      │
│  Uses the content to formulate answer:              │
│  "Quantum computing uses quantum mechanics..."      │
└─────────────────────────────────────────────────────┘
```

### Step-by-Step Process

1. **Query Submission**
   - Your agent calls ValyuSearchTool with a query
   - Parameters are validated (query length, result limits, etc.)
   - API key is authenticated

2. **Intelligent Search**
   - Valyu's AI understands the semantic meaning of your query
   - Multiple search strategies are employed:
     - Web search with proprietary crawlers
     - Access to curated databases
     - Real-time content indexing
   - Results are collected from various sources

3. **Content Processing**
   - Web pages are fetched and parsed
   - HTML is cleaned and converted to readable text
   - Important structures (headers, lists, etc.) are preserved
   - Images and irrelevant elements are filtered out

4. **Relevance Ranking**
   - Each result is scored for relevance to your query
   - AI-powered semantic matching
   - Results below your threshold are filtered out
   - Top results are ranked by relevance

5. **Response Formation**
   - Structured JSON response is created
   - Metadata is attached (source, URL, price, etc.)
   - Content is formatted for optimal LLM consumption
   - Response is returned to your agent

6. **Agent Utilization**
   - Your agent receives clean, structured content
   - Content is used to answer the user's question
   - Sources can be cited for transparency
   - Follow-up searches can refine information

## Use Cases

### 1. **Real-Time Information Agents**

Build agents that can answer questions about current events:

```python
# Agent that knows what's happening right now
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

result = agent.invoke({
    'messages': [('user', 'What are the latest developments in AI?')]
})
```

### 2. **Research Assistants**

Create agents that can deep-dive into specific topics:

```python
# Research agent with time filtering
retriever = ValyuRetriever(
    k=10,
    relevance_threshold=0.8,
    start_date="2024-01-01",  # Only recent papers
    search_type="all"
)
```

### 3. **Content Summarizers**

Build agents that extract and summarize web content:

```python
# Content extraction and summary
tool = ValyuContentsTool(
    summary=True,
    response_length="short"
)

result = tool._run(urls=["https://arxiv.org/abs/2401.12345"])
```

### 4. **Competitive Intelligence**

Monitor specific sources for business insights:

```python
# Track competitor websites
retriever = ValyuRetriever(
    included_sources=["competitor1.com", "competitor2.com"],
    k=20
)
```

### 5. **Customer Support Agents**

Agents that can search knowledge bases and documentation:

```python
# Support agent with proprietary data access
tool = ValyuSearchTool()
result = tool._run(
    query="How to reset password?",
    search_type="proprietary",  # Search internal docs
    max_num_results=3
)
```

## Prerequisites

Before you can use Valyu 2, you need:

### 1. **Python Environment**

```bash
python --version  # Should be 3.8 or higher
```

### 2. **Required Packages**

Install the official Valyu SDK:

```bash
pip install valyu
```

Or the LangChain integration:

```bash
pip install langchain-valyu
```

### 3. **API Key**

You need a Valyu API key to access the service.

## Getting Your API Key

### Option 1: Hackathon Participants

If you're participating in the Holistic AI x UCL Hackathon 2025:

1. **Check Your Environment**: Valyu credits may be included with your team credentials
2. **Look in `.env` file**: 
   ```bash
   cat .env | grep VALYU
   ```
3. **Contact Organizers**: Ask on Discord if you need Valyu access

### Option 2: General Users

1. **Visit Valyu Platform**: https://platform.valyu.network/
2. **Create Account**: Sign up for a free account
3. **Get API Key**: Navigate to API Keys section
4. **Free Credits**: New accounts receive free credits to start

### Setting Up Your API Key

Add to your `.env` file:

```bash
VALYU_API_KEY=your-api-key-here
```

Or set in your Python code:

```python
import os
os.environ["VALYU_API_KEY"] = "your-api-key-here"
```

### Verifying Your Setup

Test your API key:

```python
from valyu import Valyu

client = Valyu(api_key="your-api-key-here")

# Simple test search
response = client.search(
    query="test query",
    max_num_results=1
)

print("API Key is working!" if response.results else "Check your API key")
```

## Next Steps

Now that you understand what Valyu 2 is and why it's useful, continue to:

- **[Tutorial 2: Valyu Architecture Deep Dive](./02_architecture_deep_dive.md)** - Learn how Valyu 2 is built
- **[Tutorial 3: Getting Started with Valyu Search](./03_getting_started_search.md)** - Hands-on with ValyuSearchTool
- **[Tutorial 4: Advanced Search Parameters](./04_advanced_search.md)** - Master all search options

## Summary

In this tutorial, you learned:

- ✅ Valyu 2 is an AI-powered search platform designed for AI agents
- ✅ It provides clean, structured content ready for LLM consumption
- ✅ Key features include deep search, content extraction, and RAG retrievers
- ✅ Use cases range from real-time information to research to customer support
- ✅ You need Python 3.8+, the Valyu SDK, and an API key to get started
- ✅ API keys are available through the Valyu platform or hackathon organizers

Ready to dive deeper? Let's move on to understanding how Valyu 2 works under the hood! 🚀
