# Tutorial 3: Getting Started with Valyu Search Tool

## Table of Contents
- [Setup](#setup)
- [Your First Search](#your-first-search)
- [Understanding the Response](#understanding-the-response)
- [Basic Parameters](#basic-parameters)
- [Common Use Cases](#common-use-cases)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

## Setup

### Prerequisites

Before starting, ensure you have:

1. **Python 3.8 or higher**
2. **Valyu SDK installed**
3. **API key configured**

### Installation

```bash
# Install the official Valyu package
pip install valyu

# Or install the LangChain integration
pip install langchain-valyu
```

### Configure API Key

Add to your `.env` file:

```bash
VALYU_API_KEY=your-api-key-here
```

Or set in Python:

```python
import os
os.environ["VALYU_API_KEY"] = "your-api-key-here"
```

### Verify Setup

```python
from valyu import Valyu

# Test connection
client = Valyu()
response = client.search(query="test", max_num_results=1)
print("✅ Setup successful!" if response.results else "❌ Check your API key")
```

## Your First Search

Let's start with the simplest possible search:

### Example 1: Basic Search

```python
from langchain_valyu import ValyuSearchTool

# Create the tool
tool = ValyuSearchTool()

# Perform a search
result = tool._run(query="What is quantum computing?")

# Print results
print(result)
```

**Output Structure:**
```json
{
    "results": [
        {
            "title": "What is Quantum Computing?",
            "url": "https://example.com/quantum",
            "content": "Quantum computing is a type of computing...",
            "source": "example.com",
            "relevance_score": 0.92,
            "price": 0.02,
            "length": 2500,
            "data_type": "web"
        }
        // ... more results
    ],
    "query_metadata": {
        "total_results": 10,
        "search_type": "all",
        "query": "What is quantum computing?"
    }
}
```

### Example 2: Simple Agent Integration

```python
import sys
sys.path.insert(0, '../core')

from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool

# Create agent with Valyu search
agent = create_react_agent(
    tools=[ValyuSearchTool()],
    model_name='claude-3-5-sonnet'
)

# Ask a question
result = agent.invoke({
    'messages': [('user', 'What are the latest developments in quantum computing?')]
})

# Agent automatically uses the search tool and formulates an answer
print(result['messages'][-1].content)
```

**What happens:**
1. Agent receives your question
2. Decides to use `valyu_deep_search` tool
3. Searches for "latest developments in quantum computing"
4. Receives structured results
5. Formulates a comprehensive answer
6. Returns response with information from search results

## Understanding the Response

### Response Structure

Every search returns a structured response with two main parts:

#### 1. **Results Array**

Each result contains:

```python
{
    "title": str,              # Page/document title
    "url": str,                # Source URL
    "content": str,            # Extracted text content
    "source": str,             # Domain name
    "relevance_score": float,  # 0.0-1.0, higher is better
    "price": float,            # Cost in USD for this result
    "length": int,             # Content length in characters
    "data_type": str,          # "web", "proprietary", etc.
    "image_url": str,          # Optional thumbnail
}
```

#### 2. **Query Metadata**

Information about the search:

```python
{
    "total_results": int,      # Number of results returned
    "search_type": str,        # "all", "web", "proprietary"
    "query": str,              # Original query
    "total_price": float,      # Total cost of search
}
```

### Accessing Results

```python
result = tool._run(query="quantum computing")

# Get all results
all_results = result['results']

# Get first result
first_result = result['results'][0]
print(f"Title: {first_result['title']}")
print(f"URL: {first_result['url']}")
print(f"Relevance: {first_result['relevance_score']}")
print(f"Content: {first_result['content'][:200]}...")

# Get query metadata
metadata = result['query_metadata']
print(f"Total results: {metadata['total_results']}")
print(f"Total cost: ${metadata['total_price']:.4f}")
```

### Iterating Through Results

```python
for idx, result in enumerate(result['results'], 1):
    print(f"\n--- Result {idx} ---")
    print(f"Title: {result['title']}")
    print(f"Source: {result['source']}")
    print(f"Relevance: {result['relevance_score']:.2f}")
    print(f"Preview: {result['content'][:150]}...")
```

## Basic Parameters

Let's explore the most important parameters you'll use regularly.

### Parameter 1: `max_num_results`

Controls how many results to return (1-20).

```python
# Get just the top 3 results
result = tool._run(
    query="quantum computing",
    max_num_results=3
)

# Get maximum results for comprehensive research
result = tool._run(
    query="quantum computing",
    max_num_results=20
)
```

**When to use:**
- **3-5 results**: Quick answers, simple questions
- **10 results**: Default, balanced approach
- **15-20 results**: Deep research, comprehensive coverage

**Trade-offs:**
- More results = more information
- More results = higher cost
- More results = longer processing time

### Parameter 2: `relevance_threshold`

Filters results by minimum relevance (0.0-1.0).

```python
# Only highly relevant results
result = tool._run(
    query="quantum computing",
    relevance_threshold=0.8  # Only 80%+ relevance
)

# More lenient, broader results
result = tool._run(
    query="quantum computing",
    relevance_threshold=0.3  # 30%+ relevance
)
```

**Guidelines:**
- **0.8-1.0**: Very strict, only perfect matches
- **0.6-0.8**: Strict, high-quality results
- **0.4-0.6**: Moderate, balanced (default 0.5)
- **0.0-0.4**: Lenient, broader results

**Example: Quality vs Quantity**

```python
# High quality, fewer results
high_quality = tool._run(
    query="quantum entanglement applications",
    relevance_threshold=0.8,
    max_num_results=20
)
print(f"High quality: {len(high_quality['results'])} results")

# Lower threshold, more results
more_results = tool._run(
    query="quantum entanglement applications",
    relevance_threshold=0.5,
    max_num_results=20
)
print(f"More results: {len(more_results['results'])} results")
```

### Parameter 3: `search_type`

Specifies where to search: `"all"`, `"web"`, or `"proprietary"`.

```python
# Search everywhere (default)
result = tool._run(
    query="company policy on remote work",
    search_type="all"
)

# Search only public web
result = tool._run(
    query="quantum computing news",
    search_type="web"
)

# Search only proprietary/internal sources
result = tool._run(
    query="internal documentation",
    search_type="proprietary"
)
```

**When to use:**
- **"all"**: Default, comprehensive search
- **"web"**: Public information, news, research papers
- **"proprietary"**: Internal docs, company knowledge base

### Parameter 4: `max_price`

Sets budget limit in USD per search.

```python
# Budget-conscious search
result = tool._run(
    query="quantum computing",
    max_price=0.10  # Max $0.10
)

# Allow higher cost for comprehensive search
result = tool._run(
    query="quantum computing",
    max_price=1.00  # Max $1.00
)
```

**Pricing guidelines:**
- Typical search: $0.02 - $0.20
- High-quality content costs more
- More results = higher cost
- Longer content = higher cost

**Check actual cost:**
```python
result = tool._run(
    query="quantum computing",
    max_price=1.00
)

actual_cost = result['query_metadata']['total_price']
print(f"Actual cost: ${actual_cost:.4f}")
```

### Parameter 5: `fast_mode`

Trade speed for comprehensiveness.

```python
# Fast mode: quick results, shorter content
result = tool._run(
    query="What is quantum computing?",
    fast_mode=True
)

# Standard mode: comprehensive results (default)
result = tool._run(
    query="What is quantum computing?",
    fast_mode=False
)
```

**When to use fast mode:**
- ✅ Simple questions
- ✅ Time-sensitive applications
- ✅ High-volume searches
- ✅ Chatbot responses

**When to use standard mode:**
- ✅ Complex research
- ✅ Deep analysis needed
- ✅ Comprehensive answers
- ✅ Quality over speed

## Common Use Cases

### Use Case 1: Quick Answer

Get a fast answer to a simple question.

```python
def quick_answer(question):
    """Get a quick answer using Valyu search."""
    tool = ValyuSearchTool()
    
    result = tool._run(
        query=question,
        max_num_results=3,
        fast_mode=True,
        relevance_threshold=0.7
    )
    
    # Return top result's content
    if result['results']:
        return result['results'][0]['content']
    return "No results found."

# Example
answer = quick_answer("What is the capital of France?")
print(answer)
```

### Use Case 2: Research Assistant

Deep dive into a topic with multiple results.

```python
def research_topic(topic):
    """Comprehensive research on a topic."""
    tool = ValyuSearchTool()
    
    result = tool._run(
        query=topic,
        max_num_results=15,
        relevance_threshold=0.6,
        fast_mode=False
    )
    
    # Organize results
    research = {
        'topic': topic,
        'sources': [],
        'total_content_length': 0
    }
    
    for r in result['results']:
        research['sources'].append({
            'title': r['title'],
            'url': r['url'],
            'relevance': r['relevance_score'],
            'summary': r['content'][:200]
        })
        research['total_content_length'] += r['length']
    
    return research

# Example
research = research_topic("quantum computing applications")
print(f"Found {len(research['sources'])} sources")
print(f"Total content: {research['total_content_length']} characters")
```

### Use Case 3: News Monitoring

Monitor recent news on a topic.

```python
from datetime import datetime, timedelta

def get_recent_news(topic, days=7):
    """Get recent news articles."""
    tool = ValyuSearchTool()
    
    # Calculate date range
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days)
    
    result = tool._run(
        query=f"{topic} news",
        search_type="web",
        max_num_results=10,
        start_date=start_date.strftime("%Y-%m-%d"),
        end_date=end_date.strftime("%Y-%m-%d")
    )
    
    return result['results']

# Example
news = get_recent_news("artificial intelligence", days=7)
for article in news:
    print(f"📰 {article['title']}")
    print(f"   {article['url']}")
    print()
```

### Use Case 4: Competitive Intelligence

Monitor competitor information.

```python
def monitor_competitors(company_names):
    """Monitor competitor websites and news."""
    tool = ValyuSearchTool()
    
    results = {}
    
    for company in company_names:
        result = tool._run(
            query=f"{company} latest news products",
            search_type="web",
            max_num_results=5,
            included_sources=[f"{company.lower()}.com"]
        )
        
        results[company] = result['results']
    
    return results

# Example
competitors = ["competitor1", "competitor2"]
intel = monitor_competitors(competitors)

for company, articles in intel.items():
    print(f"\n{company}: {len(articles)} articles found")
```

### Use Case 5: Content Verification

Verify claims with authoritative sources.

```python
def verify_claim(claim):
    """Verify a claim using authoritative sources."""
    tool = ValyuSearchTool()
    
    result = tool._run(
        query=f"fact check: {claim}",
        max_num_results=5,
        relevance_threshold=0.8,  # High threshold for accuracy
        search_type="web"
    )
    
    # Look for verification keywords
    verification = {
        'claim': claim,
        'sources': [],
        'confidence': 'low'
    }
    
    for r in result['results']:
        content_lower = r['content'].lower()
        
        # Simple verification logic
        if 'true' in content_lower or 'verified' in content_lower:
            verification['confidence'] = 'high'
        elif 'false' in content_lower or 'debunked' in content_lower:
            verification['confidence'] = 'low'
        
        verification['sources'].append({
            'title': r['title'],
            'url': r['url'],
            'relevance': r['relevance_score']
        })
    
    return verification

# Example
result = verify_claim("The Earth is round")
print(f"Claim: {result['claim']}")
print(f"Confidence: {result['confidence']}")
print(f"Sources: {len(result['sources'])}")
```

## Error Handling

### Common Errors and Solutions

#### Error 1: Invalid API Key

```python
try:
    tool = ValyuSearchTool(valyu_api_key="invalid-key")
    result = tool._run(query="test")
except Exception as e:
    if "authentication" in str(e).lower():
        print("❌ Invalid API key")
        print("✅ Solution: Check your VALYU_API_KEY in .env")
```

#### Error 2: Empty Query

```python
try:
    result = tool._run(query="")
except Exception as e:
    print("❌ Empty query not allowed")
    print("✅ Solution: Provide a non-empty query string")
```

#### Error 3: Invalid Parameters

```python
try:
    result = tool._run(
        query="test",
        max_num_results=100  # Exceeds max of 20
    )
except Exception as e:
    print("❌ Invalid parameter value")
    print("✅ Solution: max_num_results must be 1-20")
```

#### Error 4: Rate Limit Exceeded

```python
import time

def search_with_retry(query, max_retries=3):
    """Search with automatic retry on rate limit."""
    tool = ValyuSearchTool()
    
    for attempt in range(max_retries):
        try:
            return tool._run(query=query)
        except Exception as e:
            if "rate limit" in str(e).lower():
                wait_time = 2 ** attempt  # Exponential backoff
                print(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise
    
    raise Exception("Max retries exceeded")
```

### Robust Search Function

```python
def robust_search(query, **kwargs):
    """
    Perform a search with comprehensive error handling.
    
    Args:
        query: Search query
        **kwargs: Additional search parameters
    
    Returns:
        dict: Search results or error information
    """
    tool = ValyuSearchTool()
    
    # Set defaults
    params = {
        'max_num_results': 5,
        'relevance_threshold': 0.5,
        'max_price': 1.0,
        'fast_mode': False
    }
    params.update(kwargs)
    
    try:
        # Validate query
        if not query or not query.strip():
            return {'error': 'Empty query', 'results': []}
        
        # Perform search
        result = tool._run(query=query, **params)
        
        # Check if results found
        if not result.get('results'):
            return {
                'error': 'No results found',
                'results': [],
                'suggestion': 'Try lowering relevance_threshold or increasing max_num_results'
            }
        
        return result
        
    except Exception as e:
        return {
            'error': str(e),
            'results': [],
            'query': query,
            'params': params
        }

# Usage
result = robust_search("quantum computing", max_num_results=10)

if 'error' in result:
    print(f"Error: {result['error']}")
    if 'suggestion' in result:
        print(f"Suggestion: {result['suggestion']}")
else:
    print(f"Success! Found {len(result['results'])} results")
```

## Best Practices

### 1. **Start Simple, Iterate**

```python
# Start with defaults
result = tool._run(query="quantum computing")

# If not satisfied, adjust
result = tool._run(
    query="quantum computing",
    max_num_results=15,  # More results
    relevance_threshold=0.7  # Higher quality
)
```

### 2. **Use Appropriate Result Counts**

```python
# Quick answers: 3-5 results
quick = tool._run(query="capital of France", max_num_results=3)

# Research: 10-15 results
research = tool._run(query="quantum computing", max_num_results=12)

# Comprehensive: 15-20 results
comprehensive = tool._run(query="AI history", max_num_results=20)
```

### 3. **Balance Cost and Quality**

```python
# Budget search
budget = tool._run(
    query="test",
    max_num_results=5,
    fast_mode=True,
    max_price=0.10
)

# Quality search
quality = tool._run(
    query="test",
    max_num_results=10,
    fast_mode=False,
    relevance_threshold=0.7,
    max_price=1.00
)
```

### 4. **Cache Results**

```python
import json
from pathlib import Path

def cached_search(query, cache_dir="cache"):
    """Search with file-based caching."""
    tool = ValyuSearchTool()
    
    # Create cache directory
    Path(cache_dir).mkdir(exist_ok=True)
    
    # Cache key from query
    cache_key = query.replace(" ", "_")[:50]
    cache_file = Path(cache_dir) / f"{cache_key}.json"
    
    # Check cache
    if cache_file.exists():
        print("📦 Loading from cache")
        with open(cache_file) as f:
            return json.load(f)
    
    # Perform search
    print("🔍 Searching...")
    result = tool._run(query=query)
    
    # Save to cache
    with open(cache_file, 'w') as f:
        json.dump(result, f)
    
    return result
```

### 5. **Log Usage**

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def logged_search(query, **kwargs):
    """Search with logging."""
    tool = ValyuSearchTool()
    
    logger.info(f"Searching for: {query}")
    logger.info(f"Parameters: {kwargs}")
    
    result = tool._run(query=query, **kwargs)
    
    logger.info(f"Found {len(result['results'])} results")
    logger.info(f"Cost: ${result['query_metadata']['total_price']:.4f}")
    
    return result
```

## Summary

In this tutorial, you learned:

- ✅ How to set up and verify Valyu Search Tool
- ✅ Perform your first search and understand the response structure
- ✅ Use basic parameters: max_num_results, relevance_threshold, search_type, max_price, fast_mode
- ✅ Implement common use cases: quick answers, research, news monitoring, competitive intelligence
- ✅ Handle errors gracefully with retry logic and validation
- ✅ Follow best practices: start simple, balance cost/quality, cache results, log usage

**Next Steps:**

- **[Tutorial 4: Advanced Search Parameters](./04_advanced_search.md)** - Master time filtering, source filtering, and more
- **[Tutorial 5: Valyu Contents Tool](./05_contents_extraction.md)** - Extract content from specific URLs
- **[Tutorial 7: Building Your First Agent](./07_building_your_first_agent.md)** - Create a complete Valyu-powered agent

You're now ready to build powerful search-enabled applications! 🚀
