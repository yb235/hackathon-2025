# Complete Examples - Real-World Holistic AI Applications

This tutorial provides complete, working examples that demonstrate how to build real-world applications with Holistic AI. Each example includes full code, explanations, and best practices.

## Table of Contents
- [Example 1: Research Assistant](#example-1-research-assistant)
- [Example 2: Data Analyst Agent](#example-2-data-analyst-agent)
- [Example 3: Customer Support Bot](#example-3-customer-support-bot)
- [Example 4: Content Summarizer](#example-4-content-summarizer)
- [Example 5: Multi-Step Workflow Agent](#example-5-multi-step-workflow-agent)
- [Example 6: Structured Data Extractor](#example-6-structured-data-extractor)

## Example 1: Research Assistant

A complete research assistant that can search, read, and synthesize information.

### Full Code

```python
"""
Research Assistant Agent
Searches for information, reads articles, and creates summaries
"""

import sys
import os
from dotenv import load_dotenv
from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field
from typing import List

# Setup
load_dotenv()
sys.path.insert(0, './core')

from react_agent import create_react_agent
from valyu_tools import ValyuSearchTool, ValyuContentsTool

# Define structured output
class ResearchReport(BaseModel):
    """Structure for research reports"""
    topic: str = Field(description="Research topic")
    key_findings: List[str] = Field(description="List of key findings")
    summary: str = Field(description="Overall summary")
    sources: List[str] = Field(description="List of source URLs")

# Create research assistant
print("🔬 Creating Research Assistant...")
research_assistant = create_react_agent(
    tools=[
        ValyuSearchTool(),
        ValyuContentsTool()
    ],
    model_name='claude-3-5-sonnet',
    output_schema=ResearchReport,  # Get structured output
    system_prompt="""You are a thorough research assistant. 
    When researching a topic:
    1. Search for relevant, recent information
    2. Read the most promising sources
    3. Extract key findings
    4. Synthesize information into a clear summary
    5. Always cite your sources"""
)

print("✅ Research Assistant ready!\n")

# Research topic
topic = "Latest developments in quantum computing"

print(f"📚 Researching: {topic}\n")
print("⏳ This may take 30-60 seconds...\n")

# Run research
result = research_assistant.invoke({
    "messages": [HumanMessage(content=f"Research: {topic}")]
})

# Extract structured report
import json
final_message = result["messages"][-1]

# Parse JSON output
if hasattr(final_message, 'additional_kwargs') and 'structured_output' in final_message.additional_kwargs:
    report = final_message.additional_kwargs['structured_output']
else:
    report = json.loads(final_message.content)

# Display report
print("=" * 60)
print("📊 RESEARCH REPORT")
print("=" * 60)
print(f"\n🎯 Topic: {report['topic']}\n")
print("🔍 Key Findings:")
for i, finding in enumerate(report['key_findings'], 1):
    print(f"  {i}. {finding}")
print(f"\n📝 Summary:\n{report['summary']}\n")
print("🔗 Sources:")
for source in report['sources']:
    print(f"  - {source}")
print("\n" + "=" * 60)
```

### How It Works

1. **Setup**: Creates agent with search and content extraction tools
2. **Structured Output**: Uses Pydantic schema for consistent reports
3. **System Prompt**: Guides the agent's research methodology
4. **Execution**: Agent autonomously searches, reads, and synthesizes
5. **Output**: Returns validated JSON with findings and sources

### Usage Examples

```python
# Different research topics
topics = [
    "Recent advances in renewable energy",
    "Impact of AI on healthcare",
    "Latest cybersecurity threats 2024"
]

for topic in topics:
    result = research_assistant.invoke({
        "messages": [HumanMessage(f"Research: {topic}")]
    })
    # Process result...
```

## Example 2: Data Analyst Agent

An agent that analyzes data and answers questions using SQL.

### Full Code

```python
"""
Data Analyst Agent
Queries databases and provides insights
"""

import sqlite3
from langchain_core.tools import BaseTool
from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field
from typing import Type

# Custom SQL query tool
class SQLQueryInput(BaseModel):
    query: str = Field(description="SQL SELECT query to execute")

class SQLAnalysisTool(BaseTool):
    name: str = "analyze_data"
    description: str = """
    Query the sales database to analyze business data.
    Available tables:
    - sales(id, product, amount, date, region)
    - customers(id, name, email, country)
    - products(id, name, category, price)
    
    Example queries:
    - SELECT SUM(amount) FROM sales WHERE date >= '2024-01-01'
    - SELECT product, COUNT(*) as count FROM sales GROUP BY product
    """
    args_schema: Type[BaseModel] = SQLQueryInput
    
    def _run(self, query: str) -> str:
        try:
            # Validate query
            if not query.strip().upper().startswith("SELECT"):
                return "Error: Only SELECT queries allowed"
            
            # Execute query (example with in-memory DB)
            conn = sqlite3.connect(':memory:')
            cursor = conn.cursor()
            
            # Create sample data
            cursor.execute('''
                CREATE TABLE sales (
                    id INTEGER, product TEXT, amount REAL,
                    date TEXT, region TEXT
                )
            ''')
            cursor.execute('''
                INSERT INTO sales VALUES
                (1, 'Widget', 100.00, '2024-01-15', 'North'),
                (2, 'Gadget', 150.00, '2024-01-16', 'South'),
                (3, 'Widget', 120.00, '2024-01-17', 'North')
            ''')
            
            # Execute user query
            cursor.execute(query)
            results = cursor.fetchall()
            columns = [desc[0] for desc in cursor.description]
            
            # Format results
            if not results:
                return "No results found"
            
            output = " | ".join(columns) + "\n"
            output += "-" * len(output) + "\n"
            for row in results:
                output += " | ".join(str(v) for v in row) + "\n"
            
            return output
            
        except sqlite3.Error as e:
            return f"SQL Error: {str(e)}"
        finally:
            conn.close()

# Create analyst agent
analyst = create_react_agent(
    tools=[SQLAnalysisTool()],
    model_name='claude-3-5-sonnet',
    system_prompt="""You are a data analyst. 
    When asked about data:
    1. Understand what information is needed
    2. Write appropriate SQL queries
    3. Interpret the results
    4. Provide clear insights"""
)

# Example analysis
result = analyst.invoke({
    "messages": [HumanMessage(
        "What are the total sales by region?"
    )]
})

print(result["messages"][-1].content)
```

### Advanced Usage

```python
# Interactive data analysis
def analyze_data():
    """Interactive data analysis session"""
    print("📊 Data Analyst Agent")
    print("Ask questions about the sales data!\n")
    
    while True:
        question = input("\n🤔 Question (or 'exit'): ")
        if question.lower() in ['exit', 'quit']:
            break
        
        result = analyst.invoke({
            "messages": [HumanMessage(question)]
        })
        
        print(f"\n📈 Analysis:\n{result['messages'][-1].content}\n")

analyze_data()
```

## Example 3: Customer Support Bot

An intelligent customer support agent with knowledge base access.

### Full Code

```python
"""
Customer Support Bot
Handles customer inquiries using a knowledge base
"""

from langchain_core.tools import BaseTool
from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field
from typing import Type
from langgraph.checkpoint.memory import MemorySaver

# Knowledge base tool
class KBInput(BaseModel):
    topic: str = Field(description="Topic to search in knowledge base")

class KnowledgeBaseTool(BaseTool):
    name: str = "search_kb"
    description: str = """
    Search the company knowledge base for information about:
    - Product features and specifications
    - Pricing and plans
    - Common issues and solutions
    - Company policies
    """
    args_schema: Type[BaseModel] = KBInput
    
    def _run(self, topic: str) -> str:
        # Simulate knowledge base (in real app, query actual KB)
        kb = {
            "pricing": """
            Our pricing plans:
            - Basic: $10/month - 100 requests/day
            - Pro: $50/month - 1000 requests/day
            - Enterprise: Custom pricing - Unlimited requests
            """,
            "features": """
            Key features:
            - Real-time search across multiple sources
            - Content extraction and summarization
            - API access with full documentation
            - 99.9% uptime SLA
            """,
            "refund": """
            Refund policy:
            - 30-day money-back guarantee
            - No questions asked
            - Full refund processed within 5-7 business days
            """,
            "support": """
            Support options:
            - Email: support@company.com (24h response)
            - Chat: Available 9AM-5PM EST
            - Phone: Available for Enterprise customers
            """
        }
        
        # Simple keyword matching
        topic_lower = topic.lower()
        for key, content in kb.items():
            if key in topic_lower:
                return content
        
        return "I don't have specific information about that topic. Let me search our general documentation..."

# Create support bot with memory
support_bot = create_react_agent(
    tools=[KnowledgeBaseTool()],
    model_name='claude-3-5-sonnet',
    checkpointer=MemorySaver(),  # Remember conversation
    system_prompt="""You are a friendly customer support agent.
    
    Guidelines:
    1. Greet customers warmly
    2. Search the knowledge base for relevant information
    3. Provide clear, helpful answers
    4. Offer to help with follow-up questions
    5. Be empathetic and patient"""
)

# Conversation loop
def customer_support_chat():
    """Interactive customer support chat"""
    print("👋 Welcome to Customer Support!")
    print("How can I help you today?\n")
    
    # Create conversation thread
    thread_id = "customer_001"
    config = {"configurable": {"thread_id": thread_id}}
    
    while True:
        message = input("You: ")
        if message.lower() in ['exit', 'quit', 'bye']:
            print("\n👋 Thank you for contacting us! Have a great day!")
            break
        
        result = support_bot.invoke(
            {"messages": [HumanMessage(message)]},
            config=config
        )
        
        response = result["messages"][-1].content
        print(f"\n🤖 Support: {response}\n")

# Run the chat
customer_support_chat()
```

### Example Conversation

```
👋 Welcome to Customer Support!
How can I help you today?

You: What are your pricing plans?

🤖 Support: Let me check our pricing information for you!
[Uses search_kb tool]

We offer three pricing tiers:
1. Basic ($10/month): Perfect for individuals, includes 100 requests per day
2. Pro ($50/month): For teams, includes 1000 requests per day
3. Enterprise (Custom): Unlimited requests with dedicated support

Which plan would best fit your needs?

You: What's your refund policy?

🤖 Support: Great question! We have a customer-friendly refund policy:
- 30-day money-back guarantee
- No questions asked
- Full refund within 5-7 business days

You're fully protected if our service isn't right for you!

You: exit

👋 Thank you for contacting us! Have a great day!
```

## Example 4: Content Summarizer

Automatically summarizes articles, documents, or web pages.

### Full Code

```python
"""
Content Summarizer
Fetches and summarizes web content
"""

from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field
from typing import List
from valyu_tools import ValyuContentsTool

# Structured output for summaries
class ContentSummary(BaseModel):
    title: str = Field(description="Title or main topic")
    key_points: List[str] = Field(description="3-5 key points")
    summary: str = Field(description="Brief summary (2-3 sentences)")
    category: str = Field(description="Content category (tech, business, science, etc.)")
    sentiment: str = Field(description="Overall sentiment (positive, negative, neutral)")

# Create summarizer
summarizer = create_react_agent(
    tools=[ValyuContentsTool()],
    model_name='claude-3-5-haiku',  # Fast model for quick summaries
    output_schema=ContentSummary,
    system_prompt="""You are an expert content summarizer.
    Extract the key information and create concise summaries."""
)

# Batch summarize multiple URLs
def summarize_articles(urls: List[str]):
    """Summarize multiple articles"""
    print(f"📰 Summarizing {len(urls)} articles...\n")
    
    summaries = []
    for i, url in enumerate(urls, 1):
        print(f"Processing {i}/{len(urls)}: {url}")
        
        result = summarizer.invoke({
            "messages": [HumanMessage(f"Summarize this article: {url}")]
        })
        
        import json
        summary = json.loads(result["messages"][-1].content)
        summaries.append(summary)
        
        # Display summary
        print(f"\n✨ {summary['title']}")
        print(f"Category: {summary['category']} | Sentiment: {summary['sentiment']}")
        print("\nKey Points:")
        for point in summary['key_points']:
            print(f"  • {point}")
        print(f"\nSummary: {summary['summary']}\n")
        print("-" * 60 + "\n")
    
    return summaries

# Example usage
articles = [
    "https://example.com/tech-article",
    "https://example.com/business-news",
    "https://example.com/science-discovery"
]

summaries = summarize_articles(articles)

# Generate aggregate report
print("📊 AGGREGATE REPORT")
print(f"Total Articles: {len(summaries)}")
print(f"Categories: {', '.join(set(s['category'] for s in summaries))}")
print(f"Sentiment Distribution:")
sentiments = {}
for s in summaries:
    sentiments[s['sentiment']] = sentiments.get(s['sentiment'], 0) + 1
for sentiment, count in sentiments.items():
    print(f"  {sentiment}: {count}")
```

## Example 5: Multi-Step Workflow Agent

A complex agent that executes multi-step workflows.

### Full Code

```python
"""
Multi-Step Workflow Agent
Handles complex tasks requiring multiple coordinated actions
"""

from langchain_core.messages import HumanMessage
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type, Dict, Any
from valyu_tools import ValyuSearchTool

# Email tool (simulated)
class EmailInput(BaseModel):
    to: str = Field(description="Recipient email")
    subject: str = Field(description="Email subject")
    body: str = Field(description="Email body")

class EmailTool(BaseTool):
    name: str = "send_email"
    description: str = "Send an email. Only use when explicitly asked to send email."
    args_schema: Type[BaseModel] = EmailInput
    
    def _run(self, to: str, subject: str, body: str) -> str:
        # Simulate sending email
        print(f"\n📧 Sending email to: {to}")
        print(f"Subject: {subject}")
        print(f"Body: {body[:100]}...")
        return f"Email sent successfully to {to}"

# Workflow agent
workflow_agent = create_react_agent(
    tools=[
        ValyuSearchTool(),
        EmailTool()
    ],
    model_name='claude-3-5-sonnet',
    system_prompt="""You are a workflow automation agent.
    Break down complex tasks into steps and execute them systematically.
    Only send emails when explicitly requested."""
)

# Complex workflow example
task = """
Research the top 3 AI companies in 2024, find their main products,
and send a summary email to team@company.com
"""

print("🔄 Executing Multi-Step Workflow")
print(f"Task: {task}\n")
print("⏳ This will take 1-2 minutes...\n")

result = workflow_agent.invoke({
    "messages": [HumanMessage(task)]
})

print("\n✅ Workflow Complete!")
print(f"\nFinal Output:\n{result['messages'][-1].content}")
```

### Expected Workflow

The agent will:
1. **Search** for "top AI companies 2024"
2. **Search** for each company's main products
3. **Synthesize** the information
4. **Draft** email content
5. **Send** email to team@company.com

## Example 6: Structured Data Extractor

Extract structured data from unstructured text.

### Full Code

```python
"""
Structured Data Extractor
Extracts structured information from text/documents
"""

from langchain_core.messages import HumanMessage
from pydantic import BaseModel, Field
from typing import List, Optional

# Define extraction schema
class Person(BaseModel):
    name: str = Field(description="Person's full name")
    title: str = Field(description="Job title")
    email: Optional[str] = Field(description="Email address if mentioned")
    phone: Optional[str] = Field(description="Phone number if mentioned")

class Company(BaseModel):
    name: str = Field(description="Company name")
    industry: str = Field(description="Industry/sector")
    employees: List[Person] = Field(description="List of people mentioned")

class Document(BaseModel):
    company: Company = Field(description="Company information")
    key_topics: List[str] = Field(description="Main topics discussed")
    action_items: List[str] = Field(description="Action items or to-dos")

# Create extractor
extractor = create_react_agent(
    tools=[],  # No tools needed for text extraction
    model_name='claude-3-5-sonnet',
    output_schema=Document,
    system_prompt="""Extract structured data from text.
    Be thorough and accurate. If information is not present, use null."""
)

# Example document
document_text = """
Meeting Notes - TechCorp Partnership Discussion

TechCorp is a leading AI solutions provider in the enterprise software industry.

Attendees:
- John Smith, CEO of TechCorp (john@techcorp.com, 555-0123)
- Sarah Johnson, CTO (sarah@techcorp.com)
- Mike Williams, VP of Sales

Topics Discussed:
- AI integration roadmap for Q1 2024
- Pricing and licensing options
- Technical requirements and API specifications

Action Items:
1. Schedule technical demo for next week
2. Prepare proposal document by Friday
3. Set up integration testing environment
"""

print("📄 Extracting Structured Data...\n")

result = extractor.invoke({
    "messages": [HumanMessage(f"Extract data from:\n\n{document_text}")]
})

# Parse and display
import json
data = json.loads(result["messages"][-1].content)

print("=" * 60)
print("EXTRACTED DATA")
print("=" * 60)

print(f"\n🏢 Company: {data['company']['name']}")
print(f"Industry: {data['company']['industry']}")

print("\n👥 Employees:")
for person in data['company']['employees']:
    print(f"  • {person['name']} - {person['title']}")
    if person.get('email'):
        print(f"    Email: {person['email']}")
    if person.get('phone'):
        print(f"    Phone: {person['phone']}")

print("\n📋 Key Topics:")
for topic in data['key_topics']:
    print(f"  • {topic}")

print("\n✅ Action Items:")
for i, item in enumerate(data['action_items'], 1):
    print(f"  {i}. {item}")

print("\n" + "=" * 60)
```

## Key Takeaways

✅ **Real-World Applications**: These examples demonstrate practical use cases
✅ **Structured Outputs**: Pydantic schemas ensure reliable data extraction
✅ **Custom Tools**: Extend agents for specific domains
✅ **Multi-Step Workflows**: Agents can coordinate complex tasks
✅ **Error Handling**: Production-ready code with proper error handling
✅ **Conversation Memory**: Use checkpointers for stateful interactions

## Running the Examples

### Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Set up environment
cp .env.example .env
# Edit .env with your credentials

# Run examples
python research_assistant.py
python data_analyst.py
python support_bot.py
# etc.
```

### Customization Tips

1. **Adjust Models**: Try different models for speed/quality trade-offs
2. **Modify Schemas**: Adapt structured outputs to your needs
3. **Add Tools**: Create domain-specific tools
4. **Tune Prompts**: Customize system prompts for your use case
5. **Add Validation**: Implement additional input/output validation

## What's Next?

These examples provide a foundation. Now you can:
- Combine patterns from multiple examples
- Build domain-specific agents
- Deploy to production
- Add monitoring and observability

**See also**:
- [10_Advanced_Topics.md](./10_Advanced_Topics.md) for production deployment
- [12_Troubleshooting_Guide.md](./12_Troubleshooting_Guide.md) for debugging help

---

**Congratulations!** 🎉 You now have complete, working examples to build production-grade AI applications with Holistic AI!
