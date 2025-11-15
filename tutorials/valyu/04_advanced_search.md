# Tutorial 4: Advanced Valyu Search - Parameters and Filtering

## Table of Contents
- [Time-Based Filtering](#time-based-filtering)
- [Source Filtering](#source-filtering)
- [Response Length Control](#response-length-control)
- [Geographic Targeting](#geographic-targeting)
- [Advanced Query Techniques](#advanced-query-techniques)
- [Parameter Combinations](#parameter-combinations)
- [Performance Optimization](#performance-optimization)

## Time-Based Filtering

Time-based filtering is crucial for finding recent information or historical data.

### Basic Date Filtering

```python
from langchain_valyu import ValyuSearchTool
from datetime import datetime, timedelta

tool = ValyuSearchTool()

# Last 7 days
week_ago = (datetime.now() - timedelta(days=7)).strftime("%Y-%m-%d")
today = datetime.now().strftime("%Y-%m-%d")

result = tool._run(
    query="artificial intelligence news",
    start_date=week_ago,
    end_date=today
)
```

### Common Time Ranges

#### Last 24 Hours (Breaking News)

```python
def get_latest_news(topic):
    """Get news from last 24 hours."""
    tool = ValyuSearchTool()
    
    yesterday = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d")
    today = datetime.now().strftime("%Y-%m-%d")
    
    return tool._run(
        query=f"{topic} news",
        start_date=yesterday,
        end_date=today,
        max_num_results=10,
        search_type="web"
    )

# Example
news = get_latest_news("quantum computing")
print(f"Found {len(news['results'])} articles from last 24 hours")
```

#### Last Week (Recent Developments)

```python
def get_weekly_updates(topic):
    """Get updates from last week."""
    tool = ValyuSearchTool()
    
    last_week = (datetime.now() - timedelta(days=7)).strftime("%Y-%m-%d")
    today = datetime.now().strftime("%Y-%m-%d")
    
    return tool._run(
        query=f"{topic} latest updates",
        start_date=last_week,
        end_date=today,
        relevance_threshold=0.6
    )
```

#### Last Month (Trend Analysis)

```python
def get_monthly_trends(topic):
    """Get trends from last month."""
    tool = ValyuSearchTool()
    
    last_month = (datetime.now() - timedelta(days=30)).strftime("%Y-%m-%d")
    today = datetime.now().strftime("%Y-%m-%d")
    
    return tool._run(
        query=f"{topic} trends analysis",
        start_date=last_month,
        end_date=today,
        max_num_results=15
    )
```

#### Specific Date Range (Historical Research)

```python
def get_historical_data(topic, start_year, end_year):
    """Get historical information from specific years."""
    tool = ValyuSearchTool()
    
    start_date = f"{start_year}-01-01"
    end_date = f"{end_year}-12-31"
    
    return tool._run(
        query=f"{topic} history",
        start_date=start_date,
        end_date=end_date,
        max_num_results=20
    )

# Example: Get AI history from 2020-2023
historical = get_historical_data("artificial intelligence", 2020, 2023)
```

### Time-Based Use Cases

#### Monitoring System

```python
class NewsMonitor:
    """Monitor news updates on specific topics."""
    
    def __init__(self, topics):
        self.topics = topics
        self.tool = ValyuSearchTool()
        self.last_check = datetime.now()
    
    def check_updates(self):
        """Check for new articles since last check."""
        updates = {}
        
        for topic in self.topics:
            result = self.tool._run(
                query=f"{topic} news",
                start_date=self.last_check.strftime("%Y-%m-%d"),
                end_date=datetime.now().strftime("%Y-%m-%d"),
                max_num_results=5
            )
            
            updates[topic] = {
                'count': len(result['results']),
                'articles': result['results']
            }
        
        self.last_check = datetime.now()
        return updates

# Usage
monitor = NewsMonitor(['AI', 'quantum computing', 'blockchain'])
updates = monitor.check_updates()

for topic, data in updates.items():
    print(f"{topic}: {data['count']} new articles")
```

## Source Filtering

Control which sources are searched to focus on specific domains or exclude unreliable sources.

### Including Specific Sources

```python
tool = ValyuSearchTool()

# Search only from specific domains
result = tool._run(
    query="machine learning",
    included_sources=[
        "arxiv.org",
        "nature.com",
        "science.org"
    ],
    max_num_results=10
)
```

### Excluding Sources

```python
# Exclude specific domains
result = tool._run(
    query="AI news",
    excluded_sources=[
        "socialmedia.com",
        "unreliable-source.com"
    ],
    max_num_results=10
)
```

### Academic Research Filter

```python
def academic_search(topic):
    """Search only academic sources."""
    tool = ValyuSearchTool()
    
    academic_sources = [
        "arxiv.org",
        "scholar.google.com",
        "ieee.org",
        "acm.org",
        "nature.com",
        "sciencedirect.com",
        "jstor.org",
        "pubmed.gov"
    ]
    
    return tool._run(
        query=f"{topic} research",
        included_sources=academic_sources,
        max_num_results=15,
        relevance_threshold=0.7
    )

# Example
papers = academic_search("quantum machine learning")
print(f"Found {len(papers['results'])} academic papers")
```

### News Source Filter

```python
def news_search(topic):
    """Search only major news outlets."""
    tool = ValyuSearchTool()
    
    news_sources = [
        "reuters.com",
        "bbc.com",
        "nytimes.com",
        "theguardian.com",
        "wsj.com",
        "ft.com",
        "bloomberg.com"
    ]
    
    return tool._run(
        query=f"{topic} news",
        included_sources=news_sources,
        max_num_results=10,
        search_type="web"
    )
```

### Tech Blog Filter

```python
def tech_blog_search(topic):
    """Search technology blogs."""
    tool = ValyuSearchTool()
    
    tech_blogs = [
        "techcrunch.com",
        "wired.com",
        "arstechnica.com",
        "theverge.com",
        "medium.com",
        "towardsdatascience.com"
    ]
    
    return tool._run(
        query=f"{topic} tutorial guide",
        included_sources=tech_blogs,
        max_num_results=12
    )
```

### Company-Specific Research

```python
def company_research(companies):
    """Research specific companies."""
    tool = ValyuSearchTool()
    
    results = {}
    
    for company in companies:
        company_domain = f"{company.lower().replace(' ', '')}.com"
        
        result = tool._run(
            query=f"{company} products services",
            included_sources=[company_domain],
            max_num_results=5
        )
        
        results[company] = result
    
    return results

# Example
research = company_research(["Google", "Microsoft", "Amazon"])
```

## Response Length Control

Control how much content is returned per result to balance detail and cost.

### Predefined Lengths

```python
tool = ValyuSearchTool()

# Short summaries (~25k characters)
result = tool._run(
    query="quantum computing",
    response_length="short"
)

# Medium content (~50k characters)
result = tool._run(
    query="quantum computing",
    response_length="medium"
)

# Large content (~100k characters)
result = tool._run(
    query="quantum computing",
    response_length="large"
)

# Maximum available content
result = tool._run(
    query="quantum computing",
    response_length="max"
)
```

### Custom Character Limits

```python
# Specify exact character count
result = tool._run(
    query="quantum computing",
    response_length=10000  # 10k characters
)
```

### Use Case-Based Length Selection

#### Quick Answers (Short)

```python
def quick_answer(question):
    """Quick answer with short content."""
    tool = ValyuSearchTool()
    
    return tool._run(
        query=question,
        max_num_results=3,
        response_length="short",
        fast_mode=True
    )
```

#### Detailed Analysis (Medium)

```python
def detailed_analysis(topic):
    """Detailed analysis with medium content."""
    tool = ValyuSearchTool()
    
    return tool._run(
        query=topic,
        max_num_results=10,
        response_length="medium",
        relevance_threshold=0.6
    )
```

#### Comprehensive Research (Large/Max)

```python
def comprehensive_research(topic):
    """Comprehensive research with maximum content."""
    tool = ValyuSearchTool()
    
    return tool._run(
        query=topic,
        max_num_results=15,
        response_length="large",
        relevance_threshold=0.7
    )
```

### Cost vs Detail Trade-off

```python
def cost_conscious_search(query, budget=0.50):
    """Search within budget by adjusting content length."""
    tool = ValyuSearchTool()
    
    # Try with short content first
    result = tool._run(
        query=query,
        response_length="short",
        max_num_results=10,
        max_price=budget
    )
    
    actual_cost = result['query_metadata']['total_price']
    
    # If under budget, could have used medium
    if actual_cost < budget * 0.5:
        print(f"💡 Tip: Budget allows for 'medium' response_length")
    
    return result
```

## Geographic Targeting

Target search results to specific countries or regions.

### Country-Specific Search

```python
tool = ValyuSearchTool()

# United Kingdom
result = tool._run(
    query="COVID-19 statistics",
    country_code="GB",
    max_num_results=10
)

# United States
result = tool._run(
    query="election news",
    country_code="US",
    max_num_results=10
)

# France
result = tool._run(
    query="actualités technologie",
    country_code="FR",
    max_num_results=10
)
```

### Common Country Codes

```python
COUNTRY_CODES = {
    "United States": "US",
    "United Kingdom": "GB",
    "Canada": "CA",
    "Australia": "AU",
    "Germany": "DE",
    "France": "FR",
    "Spain": "ES",
    "Italy": "IT",
    "Japan": "JP",
    "China": "CN",
    "India": "IN",
    "Brazil": "BR"
}

def regional_search(query, country_name):
    """Search with country-specific bias."""
    tool = ValyuSearchTool()
    
    country_code = COUNTRY_CODES.get(country_name)
    
    if not country_code:
        raise ValueError(f"Unknown country: {country_name}")
    
    return tool._run(
        query=query,
        country_code=country_code,
        max_num_results=10
    )

# Example
uk_results = regional_search("tech startups", "United Kingdom")
us_results = regional_search("tech startups", "United States")
```

### Multi-Region Comparison

```python
def compare_regions(query, countries):
    """Compare search results across different regions."""
    tool = ValyuSearchTool()
    
    results = {}
    
    for country_name, country_code in countries.items():
        result = tool._run(
            query=query,
            country_code=country_code,
            max_num_results=5
        )
        
        results[country_name] = {
            'count': len(result['results']),
            'top_sources': [r['source'] for r in result['results'][:3]]
        }
    
    return results

# Example
countries = {
    "US": "US",
    "UK": "GB",
    "Germany": "DE"
}

comparison = compare_regions("renewable energy", countries)

for country, data in comparison.items():
    print(f"{country}: {data['count']} results")
    print(f"  Top sources: {', '.join(data['top_sources'])}")
```

## Advanced Query Techniques

### Boolean-Style Queries

```python
tool = ValyuSearchTool()

# AND logic (implicit)
result = tool._run(
    query="quantum computing AND machine learning"
)

# OR logic
result = tool._run(
    query="artificial intelligence OR machine learning"
)

# NOT logic (use excluded keywords in query)
result = tool._run(
    query="AI -crypto -blockchain"  # Exclude crypto and blockchain
)
```

### Phrase Matching

```python
# Exact phrase search
result = tool._run(
    query='"quantum entanglement" applications'
)

# Multiple phrases
result = tool._run(
    query='"machine learning" "deep learning" comparison'
)
```

### Question Formulation

```python
# Natural questions often work best
queries = [
    "What is quantum computing?",
    "How does machine learning work?",
    "Why is AI important?",
    "When was the first computer invented?",
    "Where is quantum computing used?"
]

for q in queries:
    result = tool._run(query=q, max_num_results=3)
    print(f"Query: {q}")
    print(f"Results: {len(result['results'])}\n")
```

### Specific vs Broad Queries

```python
# Broad query - more results, varied relevance
broad = tool._run(
    query="AI",
    max_num_results=20,
    relevance_threshold=0.4
)

# Specific query - fewer results, higher relevance
specific = tool._run(
    query="transformer architecture in natural language processing",
    max_num_results=20,
    relevance_threshold=0.7
)

print(f"Broad: {len(broad['results'])} results")
print(f"Specific: {len(specific['results'])} results")
```

## Parameter Combinations

Combine multiple parameters for powerful, targeted searches.

### Academic Research Pattern

```python
def academic_research_search(topic, recent_only=True):
    """Comprehensive academic research search."""
    tool = ValyuSearchTool()
    
    params = {
        'query': f"{topic} research paper",
        'search_type': "all",
        'max_num_results': 15,
        'relevance_threshold': 0.7,
        'response_length': "large",
        'included_sources': [
            "arxiv.org",
            "scholar.google.com",
            "ieee.org"
        ],
        'max_price': 2.0
    }
    
    # Add time filter if recent_only
    if recent_only:
        one_year_ago = (datetime.now() - timedelta(days=365)).strftime("%Y-%m-%d")
        params['start_date'] = one_year_ago
        params['end_date'] = datetime.now().strftime("%Y-%m-%d")
    
    return tool._run(**params)

# Example
papers = academic_research_search("quantum machine learning", recent_only=True)
```

### News Aggregation Pattern

```python
def aggregate_news(topic, hours=24):
    """Aggregate news from multiple sources."""
    tool = ValyuSearchTool()
    
    time_ago = (datetime.now() - timedelta(hours=hours)).strftime("%Y-%m-%d")
    today = datetime.now().strftime("%Y-%m-%d")
    
    return tool._run(
        query=f"{topic} breaking news",
        search_type="web",
        max_num_results=20,
        relevance_threshold=0.5,
        fast_mode=True,
        start_date=time_ago,
        end_date=today,
        included_sources=[
            "reuters.com",
            "bbc.com",
            "bloomberg.com",
            "ft.com"
        ],
        response_length="short"
    )

# Example
news = aggregate_news("technology", hours=24)
```

### Competitive Intelligence Pattern

```python
def competitive_intelligence(company, competitors):
    """Gather competitive intelligence."""
    tool = ValyuSearchTool()
    
    intel = {
        'company': company,
        'competitors': {}
    }
    
    # Search about target company
    company_info = tool._run(
        query=f"{company} products news strategy",
        max_num_results=10,
        relevance_threshold=0.6,
        start_date=(datetime.now() - timedelta(days=90)).strftime("%Y-%m-%d"),
        included_sources=[f"{company.lower().replace(' ', '')}.com"]
    )
    intel['company_info'] = company_info
    
    # Search about each competitor
    for competitor in competitors:
        comp_info = tool._run(
            query=f"{competitor} latest products announcements",
            max_num_results=5,
            relevance_threshold=0.6,
            start_date=(datetime.now() - timedelta(days=90)).strftime("%Y-%m-%d")
        )
        intel['competitors'][competitor] = comp_info
    
    return intel

# Example
intel = competitive_intelligence(
    company="OpenAI",
    competitors=["Anthropic", "Google AI", "Microsoft AI"]
)
```

### Market Research Pattern

```python
def market_research(industry, region="US"):
    """Comprehensive market research."""
    tool = ValyuSearchTool()
    
    research = {}
    
    # Industry trends
    research['trends'] = tool._run(
        query=f"{industry} market trends 2024",
        max_num_results=10,
        relevance_threshold=0.7,
        country_code=region,
        response_length="medium"
    )
    
    # Key players
    research['players'] = tool._run(
        query=f"{industry} leading companies market share",
        max_num_results=8,
        relevance_threshold=0.7,
        country_code=region
    )
    
    # Recent news
    research['news'] = tool._run(
        query=f"{industry} news",
        max_num_results=15,
        start_date=(datetime.now() - timedelta(days=30)).strftime("%Y-%m-%d"),
        country_code=region,
        fast_mode=True
    )
    
    return research

# Example
research = market_research("electric vehicles", region="US")
```

## Performance Optimization

### Caching Strategy

```python
import json
import hashlib
from pathlib import Path

class CachedValyuSearch:
    """Valyu search with intelligent caching."""
    
    def __init__(self, cache_dir="valyu_cache", cache_ttl_hours=24):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.cache_ttl = timedelta(hours=cache_ttl_hours)
        self.tool = ValyuSearchTool()
    
    def _get_cache_key(self, query, **params):
        """Generate cache key from query and parameters."""
        cache_data = json.dumps({
            'query': query,
            'params': params
        }, sort_keys=True)
        return hashlib.md5(cache_data.encode()).hexdigest()
    
    def _is_cache_valid(self, cache_file):
        """Check if cache file is still valid."""
        if not cache_file.exists():
            return False
        
        file_time = datetime.fromtimestamp(cache_file.stat().st_mtime)
        return datetime.now() - file_time < self.cache_ttl
    
    def search(self, query, **params):
        """Search with caching."""
        cache_key = self._get_cache_key(query, **params)
        cache_file = self.cache_dir / f"{cache_key}.json"
        
        # Check cache
        if self._is_cache_valid(cache_file):
            print("📦 Cache hit")
            with open(cache_file) as f:
                return json.load(f)
        
        # Perform search
        print("🔍 Cache miss - searching...")
        result = self.tool._run(query=query, **params)
        
        # Save to cache
        with open(cache_file, 'w') as f:
            json.dump(result, f)
        
        return result

# Usage
searcher = CachedValyuSearch(cache_ttl_hours=24)
result1 = searcher.search("quantum computing", max_num_results=5)
result2 = searcher.search("quantum computing", max_num_results=5)  # Uses cache
```

### Batch Processing

```python
def batch_search(queries, delay_seconds=1):
    """Process multiple queries with rate limiting."""
    import time
    
    tool = ValyuSearchTool()
    results = {}
    
    for idx, query in enumerate(queries, 1):
        print(f"Processing {idx}/{len(queries)}: {query}")
        
        try:
            result = tool._run(
                query=query,
                max_num_results=5,
                fast_mode=True
            )
            results[query] = result
            
        except Exception as e:
            print(f"Error: {e}")
            results[query] = {'error': str(e)}
        
        # Rate limiting
        if idx < len(queries):
            time.sleep(delay_seconds)
    
    return results

# Example
queries = [
    "quantum computing",
    "machine learning",
    "blockchain technology"
]

results = batch_search(queries, delay_seconds=1)
```

### Progressive Enhancement

```python
def progressive_search(query):
    """Start fast, enhance if needed."""
    tool = ValyuSearchTool()
    
    # Phase 1: Fast initial search
    print("Phase 1: Quick search...")
    result = tool._run(
        query=query,
        max_num_results=3,
        fast_mode=True,
        response_length="short"
    )
    
    # Check quality
    if result['results'] and result['results'][0]['relevance_score'] > 0.8:
        print("✅ High quality results found!")
        return result
    
    # Phase 2: Enhanced search if needed
    print("Phase 2: Enhanced search...")
    result = tool._run(
        query=query,
        max_num_results=10,
        fast_mode=False,
        response_length="medium",
        relevance_threshold=0.6
    )
    
    return result

# Example
result = progressive_search("quantum entanglement applications")
```

## Summary

In this tutorial, you mastered:

- ✅ Time-based filtering with start_date and end_date
- ✅ Source filtering with included_sources and excluded_sources
- ✅ Response length control for cost optimization
- ✅ Geographic targeting with country codes
- ✅ Advanced query techniques and boolean logic
- ✅ Powerful parameter combinations for specific use cases
- ✅ Performance optimization with caching and batch processing

**Next Steps:**

- **[Tutorial 5: Contents Extraction Tool](./05_contents_extraction.md)** - Extract content from specific URLs
- **[Tutorial 6: Retrievers for RAG](./06_retrievers_for_rag.md)** - Build RAG applications
- **[Tutorial 7: Building Your First Agent](./07_building_your_first_agent.md)** - Complete agent implementation

You now have expert-level knowledge of Valyu search parameters! 🎯
