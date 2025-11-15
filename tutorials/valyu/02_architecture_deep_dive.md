# Tutorial 2: Valyu 2 Architecture Deep Dive

## Table of Contents
- [System Architecture](#system-architecture)
- [Core Components](#core-components)
- [Code Structure](#code-structure)
- [Data Flow](#data-flow)
- [Integration Patterns](#integration-patterns)
- [Design Principles](#design-principles)

## System Architecture

Understanding how Valyu 2 is architected helps you use it effectively and debug issues when they arise.

### High-Level Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    Application Layer                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   LangChain  │  │   LangGraph  │  │    Custom    │         │
│  │     Agents   │  │     Agents   │  │  Applications│         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                 │
│         └──────────────────┼──────────────────┘                 │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             │ LangChain Tool API
                             │
┌────────────────────────────┼────────────────────────────────────┐
│              Integration Layer (langchain-valyu)                │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │              ValyuSearchTool                      │          │
│  │  - Parameter validation                           │          │
│  │  - LangChain BaseTool interface                   │          │
│  │  - Error handling                                 │          │
│  └──────────────────────────┬───────────────────────┘          │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │            ValyuContentsTool                      │          │
│  │  - URL validation                                 │          │
│  │  - Batch processing                               │          │
│  └──────────────────────────┬───────────────────────┘          │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │         ValyuRetriever (RAG)                      │          │
│  │  - LangChain BaseRetriever interface              │          │
│  │  - Document formatting                            │          │
│  └──────────────────────────┬───────────────────────┘          │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             │ Valyu SDK API
                             │
┌────────────────────────────┼────────────────────────────────────┐
│                    Core SDK Layer (valyu)                       │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │               Valyu Client                        │          │
│  │  - API authentication                             │          │
│  │  - Request/response handling                      │          │
│  │  - Rate limiting                                  │          │
│  │  - Retry logic                                    │          │
│  └──────────────────────────┬───────────────────────┘          │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             │ HTTPS
                             │
┌────────────────────────────┼────────────────────────────────────┐
│                  Valyu API Service                              │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │            API Gateway                            │          │
│  │  - Authentication                                 │          │
│  │  - Rate limiting                                  │          │
│  │  - Request routing                                │          │
│  └──────────────────────────┬───────────────────────┘          │
│                             │                                    │
│  ┌──────────────────────────▼───────────────────────┐          │
│  │         Search Engine Core                        │          │
│  │  - Query understanding (AI)                       │          │
│  │  - Multi-source orchestration                     │          │
│  │  - Relevance ranking (AI)                         │          │
│  └─────┬────────────────────┬──────────────────┬────┘          │
│        │                    │                  │                │
│  ┌─────▼──────┐  ┌─────────▼────┐  ┌─────────▼───┐            │
│  │   Web      │  │ Proprietary  │  │   Cache     │            │
│  │  Crawler   │  │   Sources    │  │   Layer     │            │
│  └────────────┘  └──────────────┘  └─────────────┘            │
└────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. **Tools Layer** (`core/valyu_tools/tools.py`)

This is where the LangChain integration lives. Two main tools:

#### **ValyuSearchTool**

```python
class ValyuSearchTool(BaseTool):
    """Valyu deep search tool for searching proprietary and web sources."""
    
    name: str = "valyu_deep_search"
    description: str = (
        "A wrapper around the Valyu deep search API to search for relevant "
        "content from proprietary and web sources. "
        "Input is a query and search parameters. "
        "Output is a JSON object with the search results."
    )
    args_schema: Type[BaseModel] = ValyuToolInput
    valyu_api_key: Optional[str] = Field(default=None)
```

**Key Responsibilities:**
- Implements `LangChain.BaseTool` interface
- Defines input schema with Pydantic validation
- Handles API key management
- Wraps the Valyu SDK client
- Provides `_run()` method for synchronous execution

**Input Schema (ValyuToolInput):**
```python
class ValyuToolInput(BaseModel):
    query: str                              # Required
    search_type: str = "all"                # all, proprietary, web
    max_num_results: int = 10               # 1-20
    relevance_threshold: float = 0.5        # 0.0-1.0
    max_price: float = 50.0                 # Cost limit in USD
    is_tool_call: bool = True               # Optimize for LLM
    start_date: Optional[str] = None        # YYYY-MM-DD
    end_date: Optional[str] = None          # YYYY-MM-DD
    included_sources: Optional[List[str]]   # Filter sources
    excluded_sources: Optional[List[str]]   # Exclude sources
    response_length: Optional[Union[int, str]]  # Content length
    country_code: Optional[str] = None      # Geographic bias
    fast_mode: bool = False                 # Quick results
```

#### **ValyuContentsTool**

```python
class ValyuContentsTool(BaseTool):
    """Valyu contents extraction tool for extracting clean content from web pages."""
    
    name: str = "valyu_contents_extract"
    description: str = (
        "A wrapper around the Valyu contents API to extract clean content "
        "from web pages. Input is a list of URLs. "
        "Output is a JSON object with the extracted content from each URL."
    )
    args_schema: Type[BaseModel] = ValyuContentsToolInput
    
    # User-configurable parameters
    summary: Optional[Union[bool, str]] = None
    extract_effort: Optional[Literal["normal", "high", "auto"]] = "normal"
    response_length: Optional[Union[int, str]] = "short"
```

**Key Responsibilities:**
- Extract clean content from URLs
- Support batch processing (up to 10 URLs)
- Optional summarization
- Configurable extraction effort

**Input Schema:**
```python
class ValyuContentsToolInput(BaseModel):
    urls: List[str]  # Max 10 URLs per request
```

### 2. **Retrievers Layer** (`core/valyu_tools/retrievers.py`)

Retrievers are designed for Retrieval-Augmented Generation (RAG) workflows.

#### **ValyuRetriever**

```python
class ValyuRetriever(BaseRetriever):
    """Retriever for Valyu deep search API."""
    
    k: int = 10                              # Number of documents
    search_type: str = "all"
    relevance_threshold: float = 0.5
    # ... all search parameters
    
    def _get_relevant_documents(self, query: str) -> List[Document]:
        """Get relevant documents from Valyu search."""
        # Calls Valyu API
        # Returns LangChain Document objects
```

**Key Features:**
- Implements `LangChain.BaseRetriever` interface
- Returns `Document` objects with page_content and metadata
- Compatible with LangChain RAG chains
- Metadata includes: title, URL, source, relevance_score, price

**Document Structure:**
```python
Document(
    page_content="The actual text content...",
    metadata={
        "title": "Document Title",
        "url": "https://source.com/page",
        "source": "source.com",
        "relevance_score": 0.85,
        "price": 0.02,
        "length": 5000,
        "data_type": "web"
    }
)
```

#### **ValyuContentsRetriever**

```python
class ValyuContentsRetriever(BaseRetriever):
    """Retriever for Valyu contents extraction API."""
    
    urls: List[str] = Field(default_factory=list)
    summary: Optional[Union[bool, str]] = None
    extract_effort: Literal["normal", "high", "auto"] = "normal"
    response_length: Union[int, str] = "short"
```

**Use Case:** When you already know the URLs and want to extract content.

### 3. **SDK Client Layer** (External: `valyu` package)

The core Valyu SDK provides the low-level API client:

```python
from valyu import Valyu

client = Valyu(api_key="your-key")

# Search
response = client.search(
    query="quantum computing",
    max_num_results=5
)

# Contents
response = client.contents(
    urls=["https://example.com"]
)
```

**Responsibilities:**
- HTTP request/response handling
- Authentication
- Error handling
- Rate limiting
- Retry logic
- Response parsing

## Code Structure

### File Organization

```
core/valyu_tools/
├── __init__.py           # Exports public API
├── tools.py              # ValyuSearchTool, ValyuContentsTool
└── retrievers.py         # ValyuRetriever, ValyuContentsRetriever
```

### Module Exports (`__init__.py`)

```python
from .retrievers import ValyuRetriever, ValyuContentsRetriever
from .tools import ValyuSearchTool, ValyuContentsTool

__all__ = [
    "ValyuRetriever",
    "ValyuContentsRetriever",
    "ValyuSearchTool",
    "ValyuContentsTool",
]
```

This clean export structure means users can import with:

```python
from valyu_tools import ValyuSearchTool, ValyuRetriever
```

### Helper Functions

Both `tools.py` and `retrievers.py` include helper functions:

#### **`get_valyu_client()`**

```python
def get_valyu_client(api_key: str = None) -> Valyu:
    """Get a Valyu client instance."""
    if not api_key:
        api_key = os.environ.get("VALYU_API_KEY", "")
    return Valyu(api_key=api_key)
```

**Purpose:** Centralized client creation with environment variable fallback.

#### **`_get_valyu_metadata()`**

```python
def _get_valyu_metadata(result) -> dict:
    """Extract metadata from Valyu search result."""
    metadata = {
        "title": getattr(result, "title", None),
        "url": getattr(result, "url", None),
        "source": getattr(result, "source", None),
        "price": getattr(result, "price", None),
        "length": getattr(result, "length", None),
        "data_type": getattr(result, "data_type", None),
        "relevance_score": getattr(result, "relevance_score", None),
    }
    return metadata
```

**Purpose:** Safe extraction of metadata from API responses.

## Data Flow

### Search Flow (ValyuSearchTool)

```
1. Agent Decision
   ├─> "I need to search for quantum computing"
   └─> Calls ValyuSearchTool with parameters

2. Tool Validation
   ├─> Pydantic validates input schema
   ├─> Checks query is not empty
   ├─> Validates max_num_results (1-20)
   ├─> Validates relevance_threshold (0.0-1.0)
   └─> Validates max_price is positive

3. Client Initialization
   ├─> Gets API key from parameter or environment
   ├─> Creates Valyu client instance
   └─> Client validates API key format

4. API Request
   ├─> Client.search() called with parameters
   ├─> HTTP POST to Valyu API
   ├─> Request includes:
   │   ├─> Authentication headers
   │   ├─> Query parameters
   │   └─> Search configuration
   └─> Waits for response

5. Valyu Processing (Server-side)
   ├─> Query understanding (AI)
   ├─> Multi-source search
   ├─> Content extraction
   ├─> Relevance ranking
   └─> Result formatting

6. Response Handling
   ├─> Client receives JSON response
   ├─> Validates response structure
   ├─> Parses into Python objects
   └─> Returns response object

7. Tool Return
   ├─> Tool formats response as dict
   ├─> Includes:
   │   ├─> Results list
   │   ├─> Metadata per result
   │   └─> Query metadata
   └─> Returns to agent

8. Agent Processing
   ├─> Receives structured search results
   ├─> Extracts relevant information
   ├─> Formulates answer
   └─> Returns to user
```

### RAG Flow (ValyuRetriever)

```
1. RAG Chain Initialization
   ├─> Define retriever
   ├─> Define LLM
   └─> Create chain

2. User Query
   └─> "What is quantum computing?"

3. Retriever Invoked
   ├─> _get_relevant_documents(query) called
   └─> Calls Valyu search API

4. Search Execution
   ├─> Same as search flow (steps 4-6 above)
   └─> Returns results

5. Document Conversion
   ├─> For each result:
   │   ├─> Extract content → page_content
   │   ├─> Extract metadata → metadata dict
   │   └─> Create Document object
   └─> Return List[Document]

6. RAG Chain Processing
   ├─> Formats documents into context
   ├─> Creates prompt with context + query
   ├─> Sends to LLM
   └─> LLM generates answer using context

7. Response
   └─> Answer with source citations
```

## Integration Patterns

### Pattern 1: Direct Tool Usage (Simple)

```python
from valyu_tools import ValyuSearchTool

tool = ValyuSearchTool(valyu_api_key="your-key")
result = tool._run(query="What is AI?", max_num_results=3)

print(result)
```

**When to use:**
- Quick scripts
- Testing
- Non-agent applications

### Pattern 2: LangChain Agent Integration (Common)

```python
from langchain_valyu import ValyuSearchTool
from react_agent import create_react_agent

tool = ValyuSearchTool()
agent = create_react_agent(
    tools=[tool],
    model_name='claude-3-5-sonnet'
)

result = agent.invoke({
    'messages': [('user', 'What is quantum computing?')]
})
```

**When to use:**
- Building AI agents
- Need reasoning and tool selection
- Multi-step tasks

### Pattern 3: RAG Pipeline (Advanced)

```python
from langchain_valyu import ValyuRetriever
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI

retriever = ValyuRetriever(k=5, relevance_threshold=0.7)
llm = ChatOpenAI(model="gpt-4")

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    return_source_documents=True
)

result = qa_chain({"query": "What is quantum computing?"})
```

**When to use:**
- Question answering systems
- Knowledge base queries
- Need source citations

### Pattern 4: Custom Workflow

```python
from valyu import Valyu

client = Valyu(api_key="your-key")

# Custom search logic
def smart_search(query, depth=1):
    results = []
    
    # Initial search
    response = client.search(query=query, max_num_results=5)
    results.extend(response.results)
    
    # Deep dive on top result
    if depth > 1 and results:
        top_url = results[0].url
        content = client.contents(urls=[top_url])
        results.append(content.results[0])
    
    return results
```

**When to use:**
- Complex search strategies
- Multi-step retrieval
- Custom processing logic

## Design Principles

### 1. **Separation of Concerns**

Each layer has a clear responsibility:

- **Tools Layer**: LangChain integration, agent interface
- **Retrievers Layer**: RAG workflows, document handling
- **SDK Layer**: API communication, low-level operations

### 2. **Fail-Safe Defaults**

All parameters have sensible defaults:

```python
max_num_results: int = 10      # Balanced
relevance_threshold: float = 0.5  # Not too strict
max_price: float = 50.0        # Reasonable budget
```

### 3. **Flexible Configuration**

Multiple ways to configure:

```python
# Method 1: Constructor
tool = ValyuSearchTool(valyu_api_key="key")

# Method 2: Environment variable
os.environ["VALYU_API_KEY"] = "key"
tool = ValyuSearchTool()

# Method 3: .env file
# VALYU_API_KEY=key
tool = ValyuSearchTool()
```

### 4. **Type Safety**

Pydantic models ensure type correctness:

```python
class ValyuToolInput(BaseModel):
    query: str                    # Must be string
    max_num_results: int = 10     # Must be int
    relevance_threshold: float    # Must be float
```

### 5. **Metadata Preservation**

All relevant information is preserved:

```python
{
    "title": "...",
    "url": "...",
    "source": "...",
    "relevance_score": 0.85,
    "price": 0.02,
    "data_type": "web"
}
```

### 6. **Error Handling**

Graceful degradation at every layer:

```python
# Safe attribute access
title = getattr(result, "title", None)

# Try-except blocks
try:
    response = client.search(query)
except Exception as e:
    logger.error(f"Search failed: {e}")
    return default_response
```

## Summary

In this tutorial, you learned:

- ✅ Valyu 2 has a layered architecture: Application → Integration → SDK → API
- ✅ Core components include Tools (agents), Retrievers (RAG), and SDK (low-level)
- ✅ Code is organized into `tools.py` and `retrievers.py` with clean exports
- ✅ Data flows through validation → API request → server processing → response
- ✅ Four integration patterns: Direct, Agent, RAG, Custom
- ✅ Design principles: separation of concerns, fail-safe defaults, type safety

**Next Steps:**

- **[Tutorial 3: Getting Started with Valyu Search](./03_getting_started_search.md)** - Hands-on practice
- **[Tutorial 4: Advanced Search Parameters](./04_advanced_search.md)** - Master search options

Now you understand how Valyu 2 works under the hood! Let's get hands-on. 🔧
