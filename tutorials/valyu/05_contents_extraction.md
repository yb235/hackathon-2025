# Tutorial 5: Valyu Contents Extraction Tool

## Table of Contents
- [What is Contents Extraction?](#what-is-contents-extraction)
- [Basic Usage](#basic-usage)
- [Configuration Options](#configuration-options)
- [Common Use Cases](#common-use-cases)
- [Comparison with Search Tool](#comparison-with-search-tool)

## What is Contents Extraction?

The **ValyuContentsTool** extracts clean, readable content from specific URLs you provide. Unlike the search tool that finds content, this tool **extracts** content from known URLs.

### When to Use

- ✅ You already know the URLs
- ✅ Need clean text from web pages
- ✅ Want to extract from multiple pages at once
- ✅ Need optional summarization

### Search vs Contents

```
ValyuSearchTool:        Query → Find URLs → Extract content
ValyuContentsTool:      Known URLs → Extract content

Use Search when:        You need to FIND information
Use Contents when:      You need to EXTRACT from known sources
```

## Basic Usage

### Simple Extraction

```python
from langchain_valyu import ValyuContentsTool

tool = ValyuContentsTool()

# Extract from single URL
result = tool._run(urls=["https://example.com/article"])

# Extract from multiple URLs (max 10)
result = tool._run(urls=[
    "https://example.com/article1",
    "https://example.com/article2",
    "https://example.com/article3"
])
```

### Response Structure

```python
{
    "results": [
        {
            "url": "https://example.com/article",
            "title": "Article Title",
            "content": "Clean extracted text...",
            "status": "success",
            "price": 0.01,
            "length": 5000,
            "extraction_effort": "normal"
        }
    ]
}
```

### Accessing Results

```python
for result in result['results']:
    print(f"Title: {result['title']}")
    print(f"URL: {result['url']}")
    print(f"Length: {result['length']} characters")
    print(f"Content preview: {result['content'][:200]}...")
    print(f"Cost: ${result['price']:.4f}\n")
```

## Configuration Options

### Option 1: Summary

Generate summaries of extracted content.

```python
# Boolean flag
tool = ValyuContentsTool(summary=True)
result = tool._run(urls=["https://example.com/long-article"])

# Custom summary prompt
tool = ValyuContentsTool(summary="Summarize in 3 bullet points")
result = tool._run(urls=["https://example.com/article"])
```

### Option 2: Extract Effort

Control extraction quality vs speed.

```python
# Normal effort (default, balanced)
tool = ValyuContentsTool(extract_effort="normal")

# High effort (better quality, slower, more expensive)
tool = ValyuContentsTool(extract_effort="high")

# Auto effort (system decides based on page complexity)
tool = ValyuContentsTool(extract_effort="auto")
```

**When to use:**
- **normal**: Most cases, good balance
- **high**: Complex pages, important content
- **auto**: Let system optimize

### Option 3: Response Length

Control content length per URL.

```python
# Short content (~25k characters)
tool = ValyuContentsTool(response_length="short")

# Medium content (~50k characters)
tool = ValyuContentsTool(response_length="medium")

# Large content (~100k characters)
tool = ValyuContentsTool(response_length="large")

# Maximum available content
tool = ValyuContentsTool(response_length="max")

# Custom character limit
tool = ValyuContentsTool(response_length=10000)
```

### Combined Configuration

```python
tool = ValyuContentsTool(
    summary=True,
    extract_effort="high",
    response_length="medium"
)

result = tool._run(urls=[
    "https://arxiv.org/abs/2301.12345",
    "https://nature.com/articles/article123"
])
```

## Common Use Cases

### Use Case 1: Article Reading

Extract and summarize news articles.

```python
def read_articles(urls):
    """Read and summarize multiple articles."""
    tool = ValyuContentsTool(
        summary=True,
        response_length="short"
    )
    
    result = tool._run(urls=urls)
    
    articles = []
    for r in result['results']:
        if r['status'] == 'success':
            articles.append({
                'title': r['title'],
                'url': r['url'],
                'summary': r['content'],
                'length': r['length']
            })
    
    return articles

# Example
urls = [
    "https://techcrunch.com/article1",
    "https://wired.com/article2"
]

articles = read_articles(urls)
for article in articles:
    print(f"📰 {article['title']}")
    print(f"   {article['summary'][:150]}...")
    print()
```

### Use Case 2: Research Paper Extraction

Extract content from academic papers.

```python
def extract_papers(arxiv_urls):
    """Extract content from arXiv papers."""
    tool = ValyuContentsTool(
        extract_effort="high",
        response_length="large",
        summary="Extract: title, abstract, main findings, conclusion"
    )
    
    return tool._run(urls=arxiv_urls)

# Example
papers = [
    "https://arxiv.org/abs/2301.12345",
    "https://arxiv.org/abs/2302.54321"
]

results = extract_papers(papers)
```

### Use Case 3: Documentation Scraping

Extract documentation from multiple pages.

```python
def scrape_documentation(doc_urls):
    """Scrape and combine documentation."""
    tool = ValyuContentsTool(
        extract_effort="normal",
        response_length="medium"
    )
    
    result = tool._run(urls=doc_urls)
    
    # Combine into single document
    combined = []
    for r in result['results']:
        if r['status'] == 'success':
            combined.append(f"## {r['title']}\n\n{r['content']}")
    
    return "\n\n---\n\n".join(combined)

# Example
docs = [
    "https://docs.example.com/getting-started",
    "https://docs.example.com/api-reference",
    "https://docs.example.com/examples"
]

documentation = scrape_documentation(docs)
```

### Use Case 4: Competitive Analysis

Extract content from competitor websites.

```python
def analyze_competitors(companies):
    """Analyze competitor websites."""
    tool = ValyuContentsTool(
        extract_effort="high",
        response_length="medium",
        summary="Extract: products, features, pricing"
    )
    
    # Build URLs
    urls = [f"https://{company.lower()}.com/products" 
            for company in companies]
    
    result = tool._run(urls=urls)
    
    analysis = {}
    for r, company in zip(result['results'], companies):
        if r['status'] == 'success':
            analysis[company] = {
                'content': r['content'],
                'summary': r.get('summary', ''),
                'url': r['url']
            }
    
    return analysis

# Example
competitors = ["Competitor1", "Competitor2", "Competitor3"]
analysis = analyze_competitors(competitors)
```

### Use Case 5: Content Aggregation

Aggregate content from multiple sources on a topic.

```python
def aggregate_content(topic_urls):
    """Aggregate content from URLs and create overview."""
    tool = ValyuContentsTool(
        summary=True,
        response_length="short"
    )
    
    result = tool._run(urls=topic_urls)
    
    aggregation = {
        'topic': 'Aggregated Content',
        'sources': len(topic_urls),
        'successful': 0,
        'summaries': []
    }
    
    for r in result['results']:
        if r['status'] == 'success':
            aggregation['successful'] += 1
            aggregation['summaries'].append({
                'title': r['title'],
                'url': r['url'],
                'summary': r['content']
            })
    
    return aggregation

# Example
urls = [
    "https://source1.com/quantum-computing",
    "https://source2.com/quantum-computing",
    "https://source3.com/quantum-computing"
]

aggregated = aggregate_content(urls)
print(f"Aggregated {aggregated['successful']}/{aggregated['sources']} sources")
```

### Use Case 6: Link Following

Extract content and follow links.

```python
def extract_with_links(start_urls, max_depth=2):
    """Extract content and follow internal links."""
    import re
    
    tool = ValyuContentsTool(response_length="medium")
    
    extracted = {}
    to_process = list(start_urls)
    processed = set()
    
    while to_process and len(processed) < 10:  # Limit total pages
        current_urls = to_process[:10]  # Batch of 10
        to_process = to_process[10:]
        
        result = tool._run(urls=current_urls)
        
        for r in result['results']:
            if r['status'] == 'success':
                url = r['url']
                extracted[url] = r
                processed.add(url)
                
                # Find links in content (simplified)
                links = re.findall(r'https?://[^\s]+', r['content'])
                for link in links[:5]:  # Limit links per page
                    if link not in processed and link not in to_process:
                        to_process.append(link)
    
    return extracted

# Example (be careful with depth!)
content = extract_with_links(["https://example.com/start"], max_depth=1)
```

## Comparison with Search Tool

### When to Use Each

| Feature | ValyuSearchTool | ValyuContentsTool |
|---------|-----------------|-------------------|
| **Input** | Query string | List of URLs |
| **Purpose** | Find information | Extract from known URLs |
| **Discovery** | ✅ Yes | ❌ No |
| **URL Limit** | N/A | 10 URLs per request |
| **Relevance Scoring** | ✅ Yes | ❌ No (all URLs processed) |
| **Source Filtering** | ✅ Yes | ❌ No (you provide URLs) |
| **Time Filtering** | ✅ Yes | ❌ No |
| **Best For** | Research, discovery | Known sources, extraction |

### Workflow Combination

Often, you'll use both together:

```python
def research_and_extract(query, extract_top_n=3):
    """Search first, then extract from top results."""
    from langchain_valyu import ValyuSearchTool, ValyuContentsTool
    
    # Step 1: Search
    search_tool = ValyuSearchTool()
    search_results = search_tool._run(
        query=query,
        max_num_results=10,
        relevance_threshold=0.7
    )
    
    # Step 2: Get top URLs
    top_urls = [r['url'] for r in search_results['results'][:extract_top_n]]
    
    # Step 3: Extract detailed content
    extract_tool = ValyuContentsTool(
        extract_effort="high",
        response_length="large"
    )
    detailed_results = extract_tool._run(urls=top_urls)
    
    return {
        'search_results': search_results,
        'detailed_content': detailed_results
    }

# Example
results = research_and_extract("quantum computing applications", extract_top_n=3)
```

### In Agent Workflow

```python
from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool, ValyuContentsTool

# Create agent with both tools
agent = create_react_agent(
    tools=[
        ValyuSearchTool(),      # For finding information
        ValyuContentsTool()     # For extracting from specific URLs
    ],
    model_name='claude-3-5-sonnet'
)

# Agent can:
# 1. Search for relevant pages
# 2. Extract detailed content from specific URLs
# 3. Combine information from multiple sources

result = agent.invoke({
    'messages': [('user', 'Find information about quantum computing, '
                          'then extract detailed content from the top 2 sources')]
})
```

## Error Handling

### Handling Failed Extractions

```python
def safe_extract(urls):
    """Extract with error handling."""
    tool = ValyuContentsTool()
    
    result = tool._run(urls=urls)
    
    successes = []
    failures = []
    
    for r in result['results']:
        if r['status'] == 'success':
            successes.append(r)
        else:
            failures.append({
                'url': r['url'],
                'error': r.get('error', 'Unknown error')
            })
    
    return {
        'successes': successes,
        'failures': failures,
        'success_rate': len(successes) / len(urls) if urls else 0
    }

# Example
urls = [
    "https://valid-site.com/article",
    "https://invalid-url-12345.com",  # Will fail
    "https://another-valid.com/page"
]

result = safe_extract(urls)
print(f"Success rate: {result['success_rate']*100:.1f}%")
print(f"Failures: {len(result['failures'])}")
```

### Retry Logic

```python
import time

def extract_with_retry(urls, max_retries=3):
    """Extract with retry on failure."""
    tool = ValyuContentsTool()
    
    for attempt in range(max_retries):
        try:
            result = tool._run(urls=urls)
            return result
        except Exception as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt
                print(f"Attempt {attempt + 1} failed. Retrying in {wait_time}s...")
                time.sleep(wait_time)
            else:
                print(f"All retries failed: {e}")
                raise
```

## Summary

In this tutorial, you learned:

- ✅ ValyuContentsTool extracts content from known URLs
- ✅ Configure with summary, extract_effort, and response_length
- ✅ Use cases: article reading, research papers, documentation scraping
- ✅ Combine with ValyuSearchTool for powerful workflows
- ✅ Handle errors gracefully with proper error checking

**Next Steps:**

- **[Tutorial 6: Retrievers for RAG](./06_retrievers_for_rag.md)** - Build RAG applications
- **[Tutorial 7: Building Your First Agent](./07_building_your_first_agent.md)** - Complete agent with Valyu

Master content extraction for your AI applications! 📄
