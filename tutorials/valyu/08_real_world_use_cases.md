# Tutorial 8: Real-World Use Cases and Best Practices

## Table of Contents
- [Customer Support Bot](#customer-support-bot)
- [Research Assistant](#research-assistant)
- [News Monitoring System](#news-monitoring-system)
- [Competitive Intelligence](#competitive-intelligence)
- [Content Recommendation Engine](#content-recommendation-engine)
- [Best Practices Summary](#best-practices-summary)

## Customer Support Bot

### Architecture

```python
import sys
sys.path.insert(0, '../core')

from react_agent import create_react_agent
from langchain_valyu import ValyuSearchTool, ValyuContentsRetriever
from langgraph.checkpoint.memory import MemorySaver

class CustomerSupportBot:
    """AI-powered customer support with Valyu."""
    
    def __init__(self, company_docs_urls, faq_urls):
        """Initialize support bot with company documentation."""
        
        # Tool for general web search
        self.search_tool = ValyuSearchTool()
        
        # Retriever for company docs
        self.docs_retriever = ValyuContentsRetriever(
            urls=company_docs_urls + faq_urls,
            response_length="medium"
        )
        
        system_prompt = """You are a helpful customer support assistant.

Priority:
1. First check company documentation
2. If not found, search the web
3. Always be polite and professional
4. Escalate complex issues to human support

When answering:
- Be concise but complete
- Provide step-by-step instructions
- Include relevant links
- Offer to help further

Current time: {system_time}
"""
        
        self.agent = create_react_agent(
            tools=[self.search_tool],
            model_name='claude-3-5-sonnet',
            system_prompt=system_prompt,
            checkpointer=MemorySaver()
        )
    
    def handle_query(self, customer_id, query):
        """Handle customer support query."""
        config = {"configurable": {"thread_id": f"customer-{customer_id}"}}
        
        # First, search company docs
        docs = self.docs_retriever.get_relevant_documents(query)
        
        # Add docs to context
        context = "\n\n".join([
            f"[{doc.metadata['title']}]\n{doc.page_content[:500]}"
            for doc in docs[:3]
        ])
        
        enhanced_query = f"""Customer question: {query}

Relevant company documentation:
{context}

Please answer the customer's question."""
        
        result = self.agent.invoke({
            'messages': [('user', enhanced_query)]
        }, config)
        
        return result['messages'][-1].content


# Example usage
company_docs = [
    "https://company.com/docs/getting-started",
    "https://company.com/docs/troubleshooting",
    "https://company.com/docs/api-reference"
]

faq_urls = [
    "https://company.com/faq",
    "https://company.com/support/common-issues"
]

bot = CustomerSupportBot(company_docs, faq_urls)

# Handle customer queries
response = bot.handle_query(
    customer_id="12345",
    query="How do I reset my password?"
)
print(response)
```

### Conversation Flow

```python
# Customer conversation
customer_id = "user123"

queries = [
    "How do I reset my password?",
    "I didn't receive the reset email",
    "What's your email address for support?"
]

for query in queries:
    print(f"\nCustomer: {query}")
    response = bot.handle_query(customer_id, query)
    print(f"Bot: {response}")
```

## Research Assistant

### Academic Research System

```python
from langchain_valyu import ValyuRetriever
from datetime import datetime, timedelta

class AcademicResearchAssistant:
    """Research assistant for academic work."""
    
    def __init__(self):
        # Academic sources only
        self.retriever = ValyuRetriever(
            k=10,
            relevance_threshold=0.8,
            included_sources=[
                "arxiv.org",
                "scholar.google.com",
                "ieee.org",
                "acm.org",
                "nature.com",
                "sciencedirect.com"
            ],
            response_length="large"
        )
        
        system_prompt = """You are an academic research assistant.

Provide:
- Comprehensive literature reviews
- Citation information
- Research methodology insights
- Critical analysis

Format citations as: [Authors, Year, Title, Journal/Conference]

Current time: {system_time}
"""
        
        self.agent = create_react_agent(
            tools=[ValyuSearchTool()],
            model_name='claude-3-5-sonnet',
            system_prompt=system_prompt
        )
    
    def literature_review(self, topic, recent_years=5):
        """Conduct literature review on topic."""
        start_date = (datetime.now() - timedelta(days=365*recent_years)).strftime("%Y-%m-%d")
        
        # Search for papers
        docs = self.retriever.get_relevant_documents(topic)
        
        # Filter by date if possible
        recent_docs = [
            doc for doc in docs
            if 'arxiv' in doc.metadata.get('url', '')  # Simplified date filter
        ]
        
        # Organize by relevance
        papers = []
        for doc in recent_docs[:10]:
            papers.append({
                'title': doc.metadata.get('title', 'Unknown'),
                'url': doc.metadata.get('url', ''),
                'relevance': doc.metadata.get('relevance_score', 0),
                'content': doc.page_content[:500]
            })
        
        return papers
    
    def find_related_work(self, paper_title):
        """Find papers related to a specific paper."""
        query = f"papers related to '{paper_title}' OR citing '{paper_title}'"
        
        docs = self.retriever.get_relevant_documents(query)
        
        return [
            {
                'title': doc.metadata.get('title'),
                'url': doc.metadata.get('url'),
                'relevance': doc.metadata.get('relevance_score')
            }
            for doc in docs[:5]
        ]


# Usage
assistant = AcademicResearchAssistant()

# Literature review
papers = assistant.literature_review("quantum machine learning", recent_years=3)
print(f"Found {len(papers)} relevant papers")

for paper in papers:
    print(f"\n📄 {paper['title']}")
    print(f"   Relevance: {paper['relevance']:.2f}")
    print(f"   URL: {paper['url']}")
```

## News Monitoring System

### Real-Time News Aggregator

```python
from datetime import datetime, timedelta
import time

class NewsMonitor:
    """Monitor news on specific topics."""
    
    def __init__(self, topics, news_sources=None):
        self.topics = topics
        self.news_sources = news_sources or [
            "reuters.com",
            "bbc.com",
            "bloomberg.com",
            "ft.com",
            "techcrunch.com"
        ]
        
        self.tool = ValyuSearchTool()
        self.last_check = datetime.now() - timedelta(days=1)
        self.seen_urls = set()
    
    def check_updates(self):
        """Check for new articles."""
        updates = {}
        
        for topic in self.topics:
            # Search recent articles
            result = self.tool._run(
                query=f"{topic} news",
                search_type="web",
                max_num_results=20,
                relevance_threshold=0.6,
                included_sources=self.news_sources,
                start_date=self.last_check.strftime("%Y-%m-%d"),
                fast_mode=True
            )
            
            # Filter new articles
            new_articles = [
                r for r in result['results']
                if r['url'] not in self.seen_urls
            ]
            
            # Update seen URLs
            self.seen_urls.update(r['url'] for r in new_articles)
            
            updates[topic] = {
                'count': len(new_articles),
                'articles': new_articles
            }
        
        self.last_check = datetime.now()
        return updates
    
    def generate_summary(self, updates):
        """Generate summary of updates."""
        summary = []
        
        for topic, data in updates.items():
            if data['count'] > 0:
                summary.append(f"\n## {topic} ({data['count']} new articles)")
                
                for article in data['articles'][:5]:
                    summary.append(f"- [{article['title']}]({article['url']})")
        
        return "\n".join(summary)
    
    def run_continuous(self, check_interval_minutes=30):
        """Run continuous monitoring."""
        print(f"Starting news monitor for: {', '.join(self.topics)}")
        print(f"Checking every {check_interval_minutes} minutes\n")
        
        while True:
            print(f"[{datetime.now().strftime('%Y-%m-%d %H:%M')}] Checking for updates...")
            
            updates = self.check_updates()
            summary = self.generate_summary(updates)
            
            if summary:
                print(summary)
            else:
                print("No new articles found.")
            
            print(f"\nNext check in {check_interval_minutes} minutes...")
            time.sleep(check_interval_minutes * 60)


# Usage
monitor = NewsMonitor(
    topics=['artificial intelligence', 'quantum computing', 'climate change'],
    news_sources=['reuters.com', 'bbc.com', 'nature.com']
)

# Single check
updates = monitor.check_updates()
print(monitor.generate_summary(updates))

# Continuous monitoring (uncomment to run)
# monitor.run_continuous(check_interval_minutes=30)
```

## Competitive Intelligence

### Competitor Monitoring

```python
class CompetitiveIntelligence:
    """Monitor competitors and market trends."""
    
    def __init__(self, company_name, competitors):
        self.company = company_name
        self.competitors = competitors
        self.tool = ValyuSearchTool()
    
    def analyze_competitor(self, competitor, days_back=90):
        """Comprehensive competitor analysis."""
        start_date = (datetime.now() - timedelta(days=days_back)).strftime("%Y-%m-%d")
        
        analysis = {
            'company': competitor,
            'products': [],
            'news': [],
            'strategy': []
        }
        
        # Product information
        products = self.tool._run(
            query=f"{competitor} new products features",
            max_num_results=5,
            start_date=start_date
        )
        analysis['products'] = products['results']
        
        # Recent news
        news = self.tool._run(
            query=f"{competitor} news announcements",
            max_num_results=10,
            start_date=start_date,
            search_type="web"
        )
        analysis['news'] = news['results']
        
        # Strategy insights
        strategy = self.tool._run(
            query=f"{competitor} strategy market position",
            max_num_results=5,
            relevance_threshold=0.7
        )
        analysis['strategy'] = strategy['results']
        
        return analysis
    
    def comparative_analysis(self):
        """Compare company with all competitors."""
        report = {
            'company': self.company,
            'competitors': {}
        }
        
        for competitor in self.competitors:
            print(f"Analyzing {competitor}...")
            analysis = self.analyze_competitor(competitor)
            report['competitors'][competitor] = analysis
            time.sleep(1)  # Rate limiting
        
        return report
    
    def generate_report(self, analysis):
        """Generate markdown report."""
        report = [f"# Competitive Intelligence Report"]
        report.append(f"\n## Company: {analysis['company']}")
        report.append(f"Date: {datetime.now().strftime('%Y-%m-%d')}\n")
        
        for competitor, data in analysis['competitors'].items():
            report.append(f"\n### {competitor}")
            report.append(f"\n#### Products ({len(data['products'])} items)")
            for p in data['products'][:3]:
                report.append(f"- {p['title']}")
            
            report.append(f"\n#### Recent News ({len(data['news'])} items)")
            for n in data['news'][:5]:
                report.append(f"- [{n['title']}]({n['url']})")
        
        return "\n".join(report)


# Usage
intel = CompetitiveIntelligence(
    company_name="Our Company",
    competitors=["Competitor A", "Competitor B", "Competitor C"]
)

# Run analysis
analysis = intel.comparative_analysis()

# Generate report
report = intel.generate_report(analysis)
print(report)

# Save report
with open('competitive_intel_report.md', 'w') as f:
    f.write(report)
```

## Content Recommendation Engine

### Personalized Content Recommendations

```python
class ContentRecommender:
    """Recommend content based on user interests."""
    
    def __init__(self):
        self.tool = ValyuSearchTool()
        self.user_profiles = {}
    
    def update_user_profile(self, user_id, interests):
        """Update user interest profile."""
        self.user_profiles[user_id] = {
            'interests': interests,
            'history': []
        }
    
    def get_recommendations(self, user_id, count=10):
        """Get personalized recommendations."""
        profile = self.user_profiles.get(user_id)
        if not profile:
            return []
        
        recommendations = []
        
        for interest in profile['interests']:
            result = self.tool._run(
                query=f"{interest} latest articles tutorials",
                max_num_results=count // len(profile['interests']),
                relevance_threshold=0.7,
                start_date=(datetime.now() - timedelta(days=7)).strftime("%Y-%m-%d"),
                fast_mode=True
            )
            
            for r in result['results']:
                if r['url'] not in [h['url'] for h in profile['history']]:
                    recommendations.append({
                        'title': r['title'],
                        'url': r['url'],
                        'interest': interest,
                        'relevance': r['relevance_score'],
                        'summary': r['content'][:200]
                    })
        
        # Sort by relevance
        recommendations.sort(key=lambda x: x['relevance'], reverse=True)
        
        return recommendations[:count]
    
    def record_interaction(self, user_id, article_url, interaction_type='view'):
        """Record user interaction with content."""
        if user_id in self.user_profiles:
            self.user_profiles[user_id]['history'].append({
                'url': article_url,
                'type': interaction_type,
                'timestamp': datetime.now()
            })


# Usage
recommender = ContentRecommender()

# Setup user
recommender.update_user_profile(
    user_id="user123",
    interests=['machine learning', 'quantum computing', 'robotics']
)

# Get recommendations
recommendations = recommender.get_recommendations("user123", count=10)

print("Recommended for you:\n")
for idx, rec in enumerate(recommendations, 1):
    print(f"{idx}. {rec['title']}")
    print(f"   Topic: {rec['interest']}")
    print(f"   Relevance: {rec['relevance']:.2f}")
    print(f"   URL: {rec['url']}\n")

# Record interaction
recommender.record_interaction("user123", recommendations[0]['url'], 'view')
```

## Best Practices Summary

### 1. API Key Management

```python
# ✅ Good: Use environment variables
import os
api_key = os.environ.get("VALYU_API_KEY")

# ❌ Bad: Hardcode in code
api_key = "abc123..."  # Don't do this
```

### 2. Error Handling

```python
# ✅ Good: Handle errors gracefully
try:
    result = tool._run(query="test")
except Exception as e:
    logger.error(f"Search failed: {e}")
    return default_response

# ❌ Bad: No error handling
result = tool._run(query="test")  # May crash
```

### 3. Rate Limiting

```python
# ✅ Good: Implement rate limiting
import time
for query in queries:
    result = tool._run(query=query)
    time.sleep(1)  # Rate limit

# ❌ Bad: Rapid-fire requests
for query in queries:
    result = tool._run(query=query)  # May hit rate limit
```

### 4. Caching

```python
# ✅ Good: Cache results
cache = {}
if query in cache:
    return cache[query]
result = tool._run(query=query)
cache[query] = result

# ❌ Bad: Repeat same queries
result1 = tool._run(query="test")
result2 = tool._run(query="test")  # Duplicate search
```

### 5. Cost Optimization

```python
# ✅ Good: Set budget limits
result = tool._run(
    query="test",
    max_price=0.50,
    fast_mode=True,
    max_num_results=5
)

# ❌ Bad: No cost controls
result = tool._run(
    query="test",
    max_num_results=20,
    response_length="max"  # Expensive
)
```

### 6. Logging

```python
# ✅ Good: Log important events
import logging
logger = logging.getLogger(__name__)
logger.info(f"Search query: {query}")

# ❌ Bad: No logging
# Can't debug issues
```

### 7. Testing

```python
# ✅ Good: Test edge cases
test_cases = [
    "",  # Empty query
    "x" * 1000,  # Very long query
    "special!@#$%characters",  # Special chars
]

# ❌ Bad: Only test happy path
query = "normal query"
```

## Summary

In this tutorial, you learned:

- ✅ Real-world use case implementations
- ✅ Customer support bots
- ✅ Research assistants
- ✅ News monitoring systems
- ✅ Competitive intelligence
- ✅ Content recommendation engines
- ✅ Production best practices

**Next Steps:**

- **[Tutorial 9: API Reference](./09_api_reference.md)** - Complete API documentation
- **[README](./README.md)** - Overview and quick start

Build production-grade applications with Valyu! 🏗️
