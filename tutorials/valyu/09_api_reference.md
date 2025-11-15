# Tutorial 9: API Reference and Troubleshooting

## Table of Contents
- [ValyuSearchTool API](#valyusearchtool-api)
- [ValyuContentsTool API](#valucontentstool-api)
- [ValyuRetriever API](#valuretriever-api)
- [ValyuContentsRetriever API](#valucontentsretriever-api)
- [Common Errors](#common-errors)
- [Troubleshooting Guide](#troubleshooting-guide)

## ValyuSearchTool API

### Class Definition

```python
from langchain_valyu import ValyuSearchTool

class ValyuSearchTool(BaseTool):
    """Valyu deep search tool for searching proprietary and web sources."""
    
    name: str = "valyu_deep_search"
    description: str = "Search for relevant content from proprietary and web sources"
    valyu_api_key: Optional[str] = None
```

### Constructor

```python
tool = ValyuSearchTool(valyu_api_key: Optional[str] = None)
```

**Parameters:**
- `valyu_api_key` (str, optional): Valyu API key. If not provided, reads from `VALYU_API_KEY` environment variable.

### _run() Method

```python
result = tool._run(
    query: str,
    search_type: str = "all",
    max_num_results: int = 10,
    relevance_threshold: float = 0.5,
    max_price: float = 50.0,
    is_tool_call: bool = True,
    start_date: Optional[str] = None,
    end_date: Optional[str] = None,
    included_sources: Optional[List[str]] = None,
    excluded_sources: Optional[List[str]] = None,
    response_length: Optional[Union[int, str]] = None,
    country_code: Optional[str] = None,
    fast_mode: bool = False
) -> dict
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | str | Required | Search query string |
| `search_type` | str | "all" | Search type: "all", "web", "proprietary" |
| `max_num_results` | int | 10 | Maximum results to return (1-20) |
| `relevance_threshold` | float | 0.5 | Minimum relevance score (0.0-1.0) |
| `max_price` | float | 50.0 | Maximum cost in USD |
| `is_tool_call` | bool | True | Optimize for LLM consumption |
| `start_date` | str | None | Start date (YYYY-MM-DD) |
| `end_date` | str | None | End date (YYYY-MM-DD) |
| `included_sources` | List[str] | None | Include only these sources/domains |
| `excluded_sources` | List[str] | None | Exclude these sources/domains |
| `response_length` | int or str | None | "short", "medium", "large", "max", or character count |
| `country_code` | str | None | 2-letter ISO country code (e.g., "US", "GB") |
| `fast_mode` | bool | False | Enable fast mode for quicker results |

**Returns:**
```python
{
    "results": [
        {
            "title": str,              # Result title
            "url": str,                # Source URL
            "content": str,            # Extracted content
            "source": str,             # Domain name
            "relevance_score": float,  # 0.0-1.0
            "price": float,            # Cost in USD
            "length": int,             # Content length
            "data_type": str,          # "web", "proprietary"
            "image_url": str           # Optional thumbnail
        }
    ],
    "query_metadata": {
        "total_results": int,      # Number of results
        "search_type": str,        # Search type used
        "query": str,              # Original query
        "total_price": float       # Total cost
    }
}
```

### Examples

```python
# Basic search
result = tool._run(query="quantum computing")

# Advanced search
result = tool._run(
    query="AI news",
    search_type="web",
    max_num_results=15,
    relevance_threshold=0.7,
    start_date="2024-01-01",
    included_sources=["reuters.com", "bbc.com"],
    country_code="US",
    fast_mode=True
)
```

## ValyuContentsTool API

### Class Definition

```python
from langchain_valyu import ValyuContentsTool

class ValyuContentsTool(BaseTool):
    """Valyu contents extraction tool for extracting clean content from web pages."""
    
    name: str = "valyu_contents_extract"
    description: str = "Extract clean content from web pages"
    valyu_api_key: Optional[str] = None
    summary: Optional[Union[bool, str]] = None
    extract_effort: Literal["normal", "high", "auto"] = "normal"
    response_length: Union[int, str] = "short"
```

### Constructor

```python
tool = ValyuContentsTool(
    valyu_api_key: Optional[str] = None,
    summary: Optional[Union[bool, str]] = None,
    extract_effort: Literal["normal", "high", "auto"] = "normal",
    response_length: Union[int, str] = "short"
)
```

**Parameters:**
- `valyu_api_key`: Valyu API key (optional, reads from env)
- `summary`: True for auto-summary, string for custom summary prompt, None for no summary
- `extract_effort`: Extraction quality - "normal", "high", or "auto"
- `response_length`: "short" (~25k), "medium" (~50k), "large" (~100k), "max", or character count

### _run() Method

```python
result = tool._run(urls: List[str]) -> dict
```

**Parameters:**
- `urls` (List[str]): List of URLs to extract (max 10)

**Returns:**
```python
{
    "results": [
        {
            "url": str,                # Source URL
            "title": str,              # Page title
            "content": str,            # Extracted content
            "status": str,             # "success" or "error"
            "price": float,            # Cost in USD
            "length": int,             # Content length
            "extraction_effort": str,  # Effort level used
            "error": str               # Error message if failed
        }
    ]
}
```

### Examples

```python
# Basic extraction
tool = ValyuContentsTool()
result = tool._run(urls=["https://example.com/article"])

# With summarization
tool = ValyuContentsTool(summary=True, response_length="medium")
result = tool._run(urls=["https://example.com/long-article"])

# Custom summary
tool = ValyuContentsTool(summary="Extract: title, main points, conclusion")
result = tool._run(urls=["https://example.com/paper"])
```

## ValyuRetriever API

### Class Definition

```python
from langchain_valyu import ValyuRetriever

class ValyuRetriever(BaseRetriever):
    """Retriever for Valyu deep search API."""
    
    k: int = 10
    search_type: str = "all"
    relevance_threshold: float = 0.5
    max_price: float = 50.0
    # ... all search parameters
```

### Constructor

```python
retriever = ValyuRetriever(
    k: int = 10,
    search_type: str = "all",
    relevance_threshold: float = 0.5,
    max_price: float = 50.0,
    is_tool_call: bool = True,
    start_date: Optional[str] = None,
    end_date: Optional[str] = None,
    included_sources: Optional[List[str]] = None,
    excluded_sources: Optional[List[str]] = None,
    response_length: Optional[Union[int, str]] = None,
    country_code: Optional[str] = None,
    fast_mode: bool = False,
    valyu_api_key: Optional[str] = None
)
```

**Parameters:** Same as ValyuSearchTool, except:
- `k` instead of `max_num_results`: Number of documents to retrieve

### get_relevant_documents() Method

```python
docs = retriever.get_relevant_documents(query: str) -> List[Document]
```

**Parameters:**
- `query` (str): Search query

**Returns:** List of LangChain `Document` objects with:
```python
Document(
    page_content=str,      # The content
    metadata={
        "title": str,
        "url": str,
        "source": str,
        "relevance_score": float,
        "price": float,
        "length": int,
        "data_type": str
    }
)
```

### Examples

```python
# Basic retrieval
retriever = ValyuRetriever(k=5)
docs = retriever.get_relevant_documents("quantum computing")

# In RAG chain
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI

qa = RetrievalQA.from_chain_type(
    llm=ChatOpenAI(model="gpt-4"),
    retriever=retriever
)
```

## ValyuContentsRetriever API

### Class Definition

```python
from langchain_valyu import ValyuContentsRetriever

class ValyuContentsRetriever(BaseRetriever):
    """Retriever for Valyu contents extraction API."""
    
    urls: List[str] = Field(default_factory=list)
    summary: Optional[Union[bool, str]] = None
    extract_effort: Literal["normal", "high", "auto"] = "normal"
    response_length: Union[int, str] = "short"
```

### Constructor

```python
retriever = ValyuContentsRetriever(
    urls: List[str] = [],
    summary: Optional[Union[bool, str]] = None,
    extract_effort: Literal["normal", "high", "auto"] = "normal",
    response_length: Union[int, str] = "short",
    valyu_api_key: Optional[str] = None
)
```

### get_relevant_documents() Method

```python
docs = retriever.get_relevant_documents(query: str) -> List[Document]
```

**Note:** Query is not used in filtering; all configured URLs are retrieved.

### Examples

```python
# Basic contents retrieval
retriever = ValyuContentsRetriever(
    urls=["https://docs.example.com/page1", "https://docs.example.com/page2"]
)
docs = retriever.get_relevant_documents("")  # Query ignored
```

## Common Errors

### Error 1: Authentication Failed

```python
# Error
AuthenticationError: Invalid API key

# Solution
import os
os.environ["VALYU_API_KEY"] = "your-valid-key"
# Or pass directly
tool = ValyuSearchTool(valyu_api_key="your-valid-key")
```

### Error 2: Rate Limit Exceeded

```python
# Error
RateLimitError: Too many requests

# Solution
import time
for query in queries:
    result = tool._run(query=query)
    time.sleep(1)  # Add delay between requests
```

### Error 3: Invalid Parameter

```python
# Error
ValidationError: max_num_results must be between 1 and 20

# Solution
result = tool._run(
    query="test",
    max_num_results=10  # Valid range: 1-20
)
```

### Error 4: Empty Query

```python
# Error
ValueError: Query cannot be empty

# Solution
query = query.strip()
if not query:
    raise ValueError("Please provide a query")
result = tool._run(query=query)
```

### Error 5: URL Extraction Failed

```python
# Error in ValyuContentsTool
ExtractionError: Failed to extract content from URL

# Solution - Check status in results
result = tool._run(urls=["https://example.com"])
for r in result['results']:
    if r['status'] == 'success':
        content = r['content']
    else:
        error = r.get('error', 'Unknown error')
        print(f"Failed: {error}")
```

## Troubleshooting Guide

### Issue: No Results Returned

**Symptoms:**
```python
result = tool._run(query="test")
len(result['results'])  # Returns 0
```

**Possible Causes & Solutions:**

1. **Relevance threshold too high**
   ```python
   # Lower the threshold
   result = tool._run(query="test", relevance_threshold=0.3)
   ```

2. **Too few results requested**
   ```python
   # Increase max results
   result = tool._run(query="test", max_num_results=20)
   ```

3. **Query too specific**
   ```python
   # Broaden the query
   result = tool._run(query="quantum computing")  # Instead of "quantum computing superconducting qubits with error correction"
   ```

### Issue: High Costs

**Symptoms:**
```python
result['query_metadata']['total_price']  # Returns high value
```

**Solutions:**

1. **Set price limits**
   ```python
   result = tool._run(query="test", max_price=0.50)
   ```

2. **Use fast mode**
   ```python
   result = tool._run(query="test", fast_mode=True)
   ```

3. **Reduce result count**
   ```python
   result = tool._run(query="test", max_num_results=5)
   ```

4. **Limit response length**
   ```python
   result = tool._run(query="test", response_length="short")
   ```

### Issue: Slow Response Times

**Solutions:**

1. **Enable fast mode**
   ```python
   result = tool._run(query="test", fast_mode=True)
   ```

2. **Reduce result count**
   ```python
   result = tool._run(query="test", max_num_results=3)
   ```

3. **Use shorter response length**
   ```python
   result = tool._run(query="test", response_length="short")
   ```

### Issue: Irrelevant Results

**Solutions:**

1. **Increase relevance threshold**
   ```python
   result = tool._run(query="test", relevance_threshold=0.8)
   ```

2. **Use more specific query**
   ```python
   # Instead of: "AI"
   result = tool._run(query="artificial intelligence machine learning deep learning")
   ```

3. **Filter by sources**
   ```python
   result = tool._run(
       query="test",
       included_sources=["arxiv.org", "nature.com"]
   )
   ```

4. **Add time constraints**
   ```python
   result = tool._run(
       query="test",
       start_date="2024-01-01"
   )
   ```

### Issue: Agent Not Using Tools

**Symptoms:**
Agent doesn't call Valyu tools when expected.

**Solutions:**

1. **Check tool names**
   ```python
   # Tool names should be clear
   tools = [ValyuSearchTool(), ValyuContentsTool()]
   print([t.name for t in tools])  # Check names
   ```

2. **Improve system prompt**
   ```python
   system_prompt = """You have access to search tools. 
   Use valyu_deep_search to find information.
   Use valyu_contents_extract to get content from specific URLs."""
   ```

3. **Test with explicit instruction**
   ```python
   result = agent.invoke({
       'messages': [('user', 'Use the search tool to find information about quantum computing')]
   })
   ```

### Issue: Content Extraction Fails

**Symptoms:**
ValyuContentsTool returns errors for URLs.

**Solutions:**

1. **Check URL validity**
   ```python
   import requests
   response = requests.head(url)
   if response.status_code != 200:
       print(f"URL not accessible: {url}")
   ```

2. **Use higher extraction effort**
   ```python
   tool = ValyuContentsTool(extract_effort="high")
   ```

3. **Handle errors gracefully**
   ```python
   result = tool._run(urls=[url])
   for r in result['results']:
       if r['status'] != 'success':
           print(f"Failed: {r.get('error')}")
   ```

### Debugging Tips

1. **Enable logging**
   ```python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   ```

2. **Check API key**
   ```python
   import os
   key = os.environ.get('VALYU_API_KEY')
   print(f"API key loaded: {key[:10]}..." if key else "No API key")
   ```

3. **Test with simple queries**
   ```python
   # Start simple
   result = tool._run(query="test")
   
   # Then add complexity
   result = tool._run(query="test", max_num_results=5)
   ```

4. **Inspect tool response**
   ```python
   import json
   print(json.dumps(result, indent=2))
   ```

## Summary

In this tutorial, you learned:

- ✅ Complete API reference for all Valyu tools and retrievers
- ✅ Parameter types, defaults, and valid ranges
- ✅ Common errors and how to fix them
- ✅ Troubleshooting guide for various issues
- ✅ Debugging tips and best practices

**Reference Links:**

- **[Valyu Platform](https://platform.valyu.network/)** - Get API keys
- **[Tutorial 1: Introduction](./01_introduction_to_valyu2.md)** - Start here
- **[README](./README.md)** - Quick reference

Complete API documentation for Valyu 2! 📚
