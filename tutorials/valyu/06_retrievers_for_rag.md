# Tutorial 6: Valyu Retrievers for RAG Applications

## Table of Contents
- [What is RAG?](#what-is-rag)
- [ValyuRetriever Basics](#valyuretriever-basics)
- [Building RAG Chains](#building-rag-chains)
- [ValyuContentsRetriever](#valucontentsretriever)
- [Advanced RAG Patterns](#advanced-rag-patterns)

## What is RAG?

**Retrieval-Augmented Generation (RAG)** combines information retrieval with language model generation.

```
Traditional LLM:     Question → LLM → Answer (from training data)

RAG:                 Question → Retrieve relevant docs → LLM + docs → Answer
```

**Benefits:**
- ✅ Up-to-date information
- ✅ Reduced hallucinations
- ✅ Source citations
- ✅ Domain-specific knowledge

## ValyuRetriever Basics

### Basic Setup

```python
from langchain_valyu import ValyuRetriever

# Create retriever
retriever = ValyuRetriever(
    k=5,                      # Number of documents to retrieve
    relevance_threshold=0.6   # Minimum relevance score
)

# Retrieve documents
docs = retriever.get_relevant_documents("quantum computing")

# Each doc has page_content and metadata
for doc in docs:
    print(f"Content: {doc.page_content[:200]}...")
    print(f"Source: {doc.metadata['url']}")
    print(f"Relevance: {doc.metadata['relevance_score']}")
    print()
```

### Document Structure

```python
Document(
    page_content="The actual text content...",
    metadata={
        "title": "Document Title",
        "url": "https://source.com",
        "source": "source.com",
        "relevance_score": 0.85,
        "price": 0.02,
        "length": 5000,
        "data_type": "web"
    }
)
```

### Configuration Options

```python
retriever = ValyuRetriever(
    k=10,                           # Number of documents
    search_type="all",              # "all", "web", "proprietary"
    relevance_threshold=0.7,        # Quality filter
    max_price=1.0,                  # Cost limit
    fast_mode=False,                # Speed vs quality
    response_length="medium",       # Content length
    valyu_api_key="your-key"        # Optional
)
```

## Building RAG Chains

### Simple QA Chain

```python
from langchain_valyu import ValyuRetriever
from langchain_openai import ChatOpenAI
from langchain.chains import RetrievalQA

# Setup components
retriever = ValyuRetriever(k=5, relevance_threshold=0.7)
llm = ChatOpenAI(model="gpt-4", temperature=0)

# Create QA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    return_source_documents=True
)

# Ask question
result = qa_chain({"query": "What is quantum computing?"})

print("Answer:", result['result'])
print("\nSources:")
for doc in result['source_documents']:
    print(f"- {doc.metadata['title']}: {doc.metadata['url']}")
```

### Custom RAG Chain

```python
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnablePassthrough
from langchain.schema.output_parser import StrOutputParser

# Create prompt template
template = """Answer the question based only on the following context:

Context:
{context}

Question: {question}

Provide a detailed answer with citations to the sources."""

prompt = ChatPromptTemplate.from_template(template)

# Build chain
def format_docs(docs):
    """Format documents for context."""
    return "\n\n".join([
        f"[{doc.metadata['title']}]\n{doc.page_content}"
        for doc in docs
    ])

chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# Use chain
answer = chain.invoke("What is quantum computing?")
print(answer)
```

### Conversational RAG

```python
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory

# Setup memory
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True,
    output_key='answer'
)

# Create conversational chain
conv_chain = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=retriever,
    memory=memory,
    return_source_documents=True
)

# Have a conversation
response1 = conv_chain({"question": "What is quantum computing?"})
print("Q1:", response1['answer'])

response2 = conv_chain({"question": "What are its applications?"})
print("Q2:", response2['answer'])  # Maintains context
```

## ValyuContentsRetriever

Extract content from known URLs for RAG.

### Basic Usage

```python
from langchain_valyu import ValyuContentsRetriever

# Create retriever with URLs
retriever = ValyuContentsRetriever(
    urls=[
        "https://docs.example.com/page1",
        "https://docs.example.com/page2"
    ],
    response_length="medium"
)

# Retrieve documents
docs = retriever.get_relevant_documents("any query")  # Query not used, returns all URLs
```

### Use Case: Documentation QA

```python
def create_docs_qa(doc_urls):
    """Create QA system for documentation."""
    from langchain_valyu import ValyuContentsRetriever
    from langchain_openai import ChatOpenAI
    from langchain.chains import RetrievalQA
    
    # Setup retriever
    retriever = ValyuContentsRetriever(
        urls=doc_urls,
        extract_effort="high",
        response_length="large"
    )
    
    llm = ChatOpenAI(model="gpt-4")
    
    # Create QA chain
    qa = RetrievalQA.from_chain_type(
        llm=llm,
        retriever=retriever,
        return_source_documents=True
    )
    
    return qa

# Example
docs = [
    "https://docs.python.org/3/tutorial/",
    "https://docs.python.org/3/library/",
]

qa_system = create_docs_qa(docs)
result = qa_system({"query": "How do I use list comprehensions?"})
```

## Advanced RAG Patterns

### Pattern 1: Multi-Source RAG

Combine search and contents retrieval.

```python
from langchain.retrievers import MergerRetriever

# Create multiple retrievers
web_retriever = ValyuRetriever(
    k=5,
    search_type="web",
    relevance_threshold=0.7
)

proprietary_retriever = ValyuRetriever(
    k=5,
    search_type="proprietary",
    relevance_threshold=0.6
)

# Merge retrievers
merged_retriever = MergerRetriever(
    retrievers=[web_retriever, proprietary_retriever]
)

# Use in RAG chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=merged_retriever
)
```

### Pattern 2: Re-ranking

Retrieve more docs, then re-rank.

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor

# Base retriever (get many docs)
base_retriever = ValyuRetriever(k=20, relevance_threshold=0.5)

# Compressor (re-rank with LLM)
compressor = LLMChainExtractor.from_llm(llm)

# Compression retriever
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)

# Use in chain
docs = compression_retriever.get_relevant_documents(
    "quantum computing applications"
)
```

### Pattern 3: Time-Aware RAG

Use recent information for time-sensitive queries.

```python
from datetime import datetime, timedelta

def time_aware_retriever(days_back=30):
    """Create retriever for recent information."""
    start_date = (datetime.now() - timedelta(days=days_back)).strftime("%Y-%m-%d")
    
    return ValyuRetriever(
        k=10,
        relevance_threshold=0.7,
        start_date=start_date,
        search_type="web"
    )

# Use for recent news
recent_retriever = time_aware_retriever(days_back=7)
qa_chain = RetrievalQA.from_chain_type(llm=llm, retriever=recent_retriever)

result = qa_chain({"query": "Latest AI developments"})
```

### Pattern 4: Source-Specific RAG

RAG for specific domains.

```python
def academic_rag_system():
    """RAG system for academic research."""
    retriever = ValyuRetriever(
        k=10,
        relevance_threshold=0.8,
        included_sources=[
            "arxiv.org",
            "scholar.google.com",
            "ieee.org",
            "nature.com"
        ],
        response_length="large"
    )
    
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    return RetrievalQA.from_chain_type(
        llm=llm,
        retriever=retriever,
        return_source_documents=True
    )

# Use for research questions
academic_qa = academic_rag_system()
result = academic_qa({"query": "Recent advances in quantum error correction"})
```

### Pattern 5: Agent with RAG

Combine agent reasoning with RAG.

```python
from react_agent import create_react_agent
from langchain.tools.retriever import create_retriever_tool

# Create retriever tool
retriever = ValyuRetriever(k=5, relevance_threshold=0.7)

retriever_tool = create_retriever_tool(
    retriever,
    name="valyu_search",
    description="Search for information using Valyu. Input should be a search query."
)

# Create agent with retriever tool
agent = create_react_agent(
    tools=[retriever_tool],
    model_name='claude-3-5-sonnet'
)

# Agent can reason about when to use retrieval
result = agent.invoke({
    'messages': [('user', 'What is quantum computing and how is it used in cryptography?')]
})
```

### Pattern 6: Iterative RAG

Refine answers through multiple retrievals.

```python
def iterative_rag(question, iterations=2):
    """Iteratively refine answer."""
    retriever = ValyuRetriever(k=5)
    llm = ChatOpenAI(model="gpt-4")
    
    current_question = question
    context = []
    
    for i in range(iterations):
        # Retrieve documents
        docs = retriever.get_relevant_documents(current_question)
        context.extend(docs)
        
        # Generate answer
        prompt = f"""Based on this context, answer the question. If you need more information, ask a follow-up question.

Context:
{chr(10).join([doc.page_content for doc in context])}

Question: {current_question}

Answer:"""
        
        response = llm.predict(prompt)
        
        # Check if follow-up needed (simplified)
        if i < iterations - 1 and "?" in response:
            current_question = response.split("?")[0] + "?"
        else:
            return response
    
    return response

# Example
answer = iterative_rag("What is quantum computing?", iterations=2)
```

## Summary

In this tutorial, you learned:

- ✅ RAG combines retrieval with generation for better answers
- ✅ ValyuRetriever provides LangChain-compatible retrieval
- ✅ Build QA chains, conversational systems, and more
- ✅ ValyuContentsRetriever for known URL extraction
- ✅ Advanced patterns: multi-source, re-ranking, time-aware, agent-based

**Next Steps:**

- **[Tutorial 7: Building Your First Agent](./07_building_your_first_agent.md)** - Complete agent implementation
- **[Tutorial 8: Real-World Use Cases](./08_real_world_use_cases.md)** - Production patterns

Build powerful RAG applications with Valyu! 🎯
