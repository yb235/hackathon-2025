# Working with Tools - Extending Agent Capabilities

Tools are what make agents truly powerful. This tutorial teaches you everything about tools: using built-in tools, creating custom tools, and best practices for tool design.

## Table of Contents
- [What Are Tools?](#what-are-tools)
- [Built-in Tools](#built-in-tools)
- [Creating Custom Tools](#creating-custom-tools)
- [Tool Design Principles](#tool-design-principles)
- [Tool Input Schemas](#tool-input-schemas)
- [Tool Error Handling](#tool-error-handling)
- [Advanced Tool Patterns](#advanced-tool-patterns)
- [Testing Tools](#testing-tools)
- [Best Practices](#best-practices)
- [Complete Examples](#complete-examples)

## What Are Tools?

**Tools** are functions that agents can call to perform specific actions or retrieve information.

### Simple Analogy
Think of an agent as a person and tools as their devices:
- **Search Tool** = Smartphone (look things up)
- **Calculator Tool** = Calculator (do math)
- **Email Tool** = Email app (send messages)
- **Database Tool** = Filing cabinet (get records)

### Why Tools Matter

Without tools:
```python
User: "What's the weather in Paris?"
Agent: "I don't have access to current weather data."
```

With tools:
```python
User: "What's the weather in Paris?"
Agent: [Uses weather tool] "It's 18°C and sunny in Paris!"
```

### Tool Lifecycle

```
1. DEFINITION
   You define what the tool does
   ↓
2. REGISTRATION
   You give the tool to the agent
   ↓
3. DISCOVERY
   Agent sees tool name/description
   ↓
4. DECISION
   Agent decides when to use it
   ↓
5. INVOCATION
   Agent calls tool with arguments
   ↓
6. EXECUTION
   Tool runs and returns results
   ↓
7. OBSERVATION
   Agent processes the results
```

## Built-in Tools

Holistic AI includes powerful built-in tools from Valyu:

### 1. ValyuSearchTool - Deep Web Search

**What it does**: Searches proprietary and web sources for relevant information.

**When to use**: 
- Finding current information
- Research tasks
- Fact-checking
- Discovery

**Basic usage**:
```python
from valyu_tools import ValyuSearchTool

# Create tool
search_tool = ValyuSearchTool()

# Use in agent
agent = create_react_agent(
    tools=[search_tool],
    model_name='claude-3-5-sonnet'
)

# Agent can now search!
result = agent.invoke({
    "messages": [HumanMessage("What are the latest AI trends?")]
})
```

**Parameters**:
```python
search_tool._run(
    query="search query",              # What to search for
    search_type="all",                 # 'all', 'web', or 'proprietary'
    max_num_results=10,                # How many results (1-20)
    relevance_threshold=0.5,           # Min relevance (0.0-1.0)
    max_price=50.0,                    # Max cost in dollars
    start_date="2024-01-01",          # Optional date filter
    end_date="2024-12-31",            # Optional date filter
    country_code="US",                 # Optional country focus
    fast_mode=False                    # Faster but shorter results
)
```

### 2. ValyuContentsTool - Content Extraction

**What it does**: Extracts clean content from web pages.

**When to use**:
- Reading articles
- Extracting data from websites
- Content analysis
- Information gathering

**Basic usage**:
```python
from valyu_tools import ValyuContentsTool

# Create tool
content_tool = ValyuContentsTool()

# Use in agent
agent = create_react_agent(
    tools=[content_tool],
    model_name='claude-3-5-sonnet'
)

# Agent can now read web pages!
result = agent.invoke({
    "messages": [HumanMessage(
        "Read and summarize https://example.com/article"
    )]
})
```

**Parameters**:
```python
content_tool._run(
    urls=[
        "https://example.com/page1",
        "https://example.com/page2"
    ]  # Up to 10 URLs per request
)
```

**Configuration**:
```python
content_tool = ValyuContentsTool(
    summary=True,              # Generate summary
    extract_effort="high",     # 'normal', 'high', or 'auto'
    response_length="medium"   # 'short', 'medium', 'large', 'max'
)
```

### Using Multiple Tools Together

Combine tools for powerful workflows:

```python
from valyu_tools import ValyuSearchTool, ValyuContentsTool

agent = create_react_agent(
    tools=[
        ValyuSearchTool(),
        ValyuContentsTool()
    ],
    model_name='claude-3-5-sonnet'
)

# Agent can search AND read pages
result = agent.invoke({
    "messages": [HumanMessage(
        "Find recent articles about quantum computing and summarize the top result"
    )]
})

# What happens:
# 1. Agent uses ValyuSearchTool to find articles
# 2. Agent uses ValyuContentsTool to read top article
# 3. Agent summarizes the content
```

## Creating Custom Tools

Now let's create your own tools! 

### Step 1: Basic Tool Structure

Every tool needs:
1. **Name**: Unique identifier
2. **Description**: What it does (helps agent decide when to use it)
3. **Input Schema**: What parameters it accepts
4. **Run Method**: The actual function code

### Step 2: Simple Calculator Tool

```python
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

# Step 1: Define input schema
class CalculatorInput(BaseModel):
    expression: str = Field(
        description="Mathematical expression to evaluate (e.g., '2 + 2', '10 * 5')"
    )

# Step 2: Create the tool class
class CalculatorTool(BaseTool):
    name: str = "calculator"
    description: str = (
        "A calculator for evaluating mathematical expressions. "
        "Use this when you need to perform calculations. "
        "Input should be a valid Python math expression."
    )
    args_schema: Type[BaseModel] = CalculatorInput
    
    def _run(self, expression: str) -> str:
        """Execute the tool"""
        try:
            # Safely evaluate expression
            result = eval(expression, {"__builtins__": {}}, {})
            return f"Result: {result}"
        except Exception as e:
            return f"Error: Invalid expression - {str(e)}"

# Step 3: Use it in an agent
calculator = CalculatorTool()

agent = create_react_agent(
    tools=[calculator],
    model_name='claude-3-5-sonnet'
)

# Test it
result = agent.invoke({
    "messages": [HumanMessage("What is 234 * 567?")]
})

print(result["messages"][-1].content)
# Agent will use calculator: "Result: 132678"
```

### Step 3: Weather Tool Example

```python
import requests
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type, Optional

# Input schema
class WeatherInput(BaseModel):
    city: str = Field(description="City name (e.g., 'London', 'Paris')")
    country: Optional[str] = Field(
        default=None,
        description="Optional 2-letter country code (e.g., 'GB', 'FR')"
    )

# Weather tool
class WeatherTool(BaseTool):
    name: str = "get_weather"
    description: str = (
        "Get current weather information for a city. "
        "Returns temperature, conditions, humidity, and wind speed."
    )
    args_schema: Type[BaseModel] = WeatherInput
    api_key: str = Field(default="")  # Your API key
    
    def _run(self, city: str, country: Optional[str] = None) -> str:
        """Get weather data"""
        try:
            # Build location string
            location = f"{city},{country}" if country else city
            
            # Call weather API (example with OpenWeatherMap)
            url = "https://api.openweathermap.org/data/2.5/weather"
            params = {
                "q": location,
                "appid": self.api_key,
                "units": "metric"
            }
            
            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()
            
            data = response.json()
            
            # Extract relevant info
            temp = data["main"]["temp"]
            feels_like = data["main"]["feels_like"]
            humidity = data["main"]["humidity"]
            description = data["weather"][0]["description"]
            wind_speed = data["wind"]["speed"]
            
            # Format response
            return (
                f"Weather in {city}:\n"
                f"Temperature: {temp}°C (feels like {feels_like}°C)\n"
                f"Conditions: {description}\n"
                f"Humidity: {humidity}%\n"
                f"Wind Speed: {wind_speed} m/s"
            )
            
        except requests.exceptions.RequestException as e:
            return f"Error fetching weather data: {str(e)}"
        except KeyError as e:
            return f"Error parsing weather data: {str(e)}"

# Use it
weather_tool = WeatherTool(api_key="your-api-key-here")

agent = create_react_agent(
    tools=[weather_tool],
    model_name='claude-3-5-sonnet'
)
```

### Step 4: Database Query Tool

```python
import sqlite3
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type

class DatabaseQueryInput(BaseModel):
    query: str = Field(
        description="SQL SELECT query to execute (read-only)"
    )

class DatabaseTool(BaseTool):
    name: str = "query_database"
    description: str = (
        "Query the product database using SQL. "
        "Use SELECT statements to retrieve data. "
        "Available tables: products(id, name, price, category), "
        "customers(id, name, email), orders(id, customer_id, product_id, date)"
    )
    args_schema: Type[BaseModel] = DatabaseQueryInput
    db_path: str = Field(default="database.db")
    
    def _run(self, query: str) -> str:
        """Execute database query"""
        try:
            # Security: Only allow SELECT queries
            if not query.strip().upper().startswith("SELECT"):
                return "Error: Only SELECT queries are allowed"
            
            # Connect and execute
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            cursor.execute(query)
            
            # Get results
            results = cursor.fetchall()
            columns = [description[0] for description in cursor.description]
            
            conn.close()
            
            # Format results
            if not results:
                return "No results found"
            
            # Create table format
            output = " | ".join(columns) + "\n"
            output += "-" * len(output) + "\n"
            for row in results:
                output += " | ".join(str(val) for val in row) + "\n"
            
            return output
            
        except sqlite3.Error as e:
            return f"Database error: {str(e)}"
        except Exception as e:
            return f"Error: {str(e)}"
```

## Tool Design Principles

### 1. Single Responsibility

Each tool should do ONE thing well:

✅ Good:
```python
class SearchTool: # Just searches
class EmailTool:  # Just sends emails
class SaveTool:   # Just saves files
```

❌ Bad:
```python
class UtilityTool:  # Does search, email, save, calculate, etc.
```

### 2. Clear Descriptions

Help the agent understand when to use your tool:

✅ Good:
```python
description = (
    "Get current weather for any city worldwide. "
    "Returns temperature, conditions, humidity, and wind. "
    "Use this when the user asks about weather or temperature."
)
```

❌ Bad:
```python
description = "Weather tool"  # Too vague
```

### 3. Descriptive Parameter Names

✅ Good:
```python
class SearchInput(BaseModel):
    search_query: str = Field(description="What to search for")
    max_results: int = Field(description="Maximum results to return (1-20)")
```

❌ Bad:
```python
class SearchInput(BaseModel):
    q: str  # What does 'q' mean?
    n: int  # What is 'n'?
```

### 4. Sensible Defaults

```python
class SearchInput(BaseModel):
    query: str  # Required
    max_results: int = Field(default=10)  # Optional with default
    language: str = Field(default="en")   # Optional with default
```

### 5. Robust Error Handling

Always handle errors gracefully:

```python
def _run(self, url: str) -> str:
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.text
    except requests.Timeout:
        return "Error: Request timed out"
    except requests.HTTPError as e:
        return f"Error: HTTP {e.response.status_code}"
    except Exception as e:
        return f"Error: {str(e)}"
```

## Tool Input Schemas

Input schemas use **Pydantic** for validation:

### Basic Types

```python
from pydantic import BaseModel, Field
from typing import Optional, List

class MyToolInput(BaseModel):
    # String
    text: str = Field(description="Text input")
    
    # Integer
    count: int = Field(description="Number of items", ge=1, le=100)
    
    # Float
    threshold: float = Field(description="Threshold value", ge=0.0, le=1.0)
    
    # Boolean
    include_metadata: bool = Field(default=False, description="Include metadata")
    
    # Optional
    optional_param: Optional[str] = Field(default=None, description="Optional parameter")
    
    # List
    items: List[str] = Field(description="List of items")
    
    # Enum
    from enum import Enum
    class Mode(str, Enum):
        FAST = "fast"
        ACCURATE = "accurate"
    
    mode: Mode = Field(default=Mode.FAST, description="Processing mode")
```

### Validation

```python
from pydantic import BaseModel, Field, validator

class SearchInput(BaseModel):
    query: str = Field(description="Search query")
    max_results: int = Field(description="Max results", ge=1, le=20)
    
    @validator('query')
    def query_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Query cannot be empty')
        return v
    
    @validator('max_results')
    def reasonable_limit(cls, v):
        if v > 20:
            return 20  # Cap at 20
        return v
```

## Tool Error Handling

Handle errors properly to help the agent recover:

### Pattern 1: Return Error Message

```python
def _run(self, url: str) -> str:
    try:
        # Tool logic
        return success_result
    except SpecificError as e:
        # Return helpful error message
        return f"Error: {str(e)}. Try providing a different URL."
```

### Pattern 2: Validate Inputs

```python
def _run(self, email: str) -> str:
    # Validate before processing
    if "@" not in email:
        return "Error: Invalid email format. Please provide a valid email address."
    
    try:
        # Send email
        return "Email sent successfully"
    except Exception as e:
        return f"Error sending email: {str(e)}"
```

### Pattern 3: Provide Context

```python
def _run(self, file_path: str) -> str:
    try:
        with open(file_path, 'r') as f:
            return f.read()
    except FileNotFoundError:
        # Helpful error with context
        return (
            f"Error: File '{file_path}' not found. "
            "Please check the path and try again. "
            "Available files: [list files in directory]"
        )
```

## Advanced Tool Patterns

### Pattern 1: Tool with State

```python
class DatabaseTool(BaseTool):
    name = "database_query"
    description = "Query database"
    args_schema = QueryInput
    
    # Tool maintains connection
    _connection: Optional[sqlite3.Connection] = None
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self._connection = sqlite3.connect("db.sqlite")
    
    def _run(self, query: str) -> str:
        cursor = self._connection.cursor()
        cursor.execute(query)
        return cursor.fetchall()
    
    def __del__(self):
        if self._connection:
            self._connection.close()
```

### Pattern 2: Async Tool

```python
from langchain_core.tools import BaseTool
import asyncio
import aiohttp

class AsyncSearchTool(BaseTool):
    name = "async_search"
    description = "Async web search"
    
    async def _arun(self, query: str) -> str:
        """Async execution"""
        async with aiohttp.ClientSession() as session:
            async with session.get(f"https://api.search.com?q={query}") as resp:
                return await resp.text()
    
    def _run(self, query: str) -> str:
        """Sync wrapper"""
        return asyncio.run(self._arun(query))
```

### Pattern 3: Tool Chaining

```python
class SearchAndSummarizeTool(BaseTool):
    name = "search_and_summarize"
    description = "Search and automatically summarize results"
    
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.search_tool = ValyuSearchTool()
        self.content_tool = ValyuContentsTool()
    
    def _run(self, query: str) -> str:
        # Step 1: Search
        results = self.search_tool._run(query=query, max_num_results=3)
        
        # Step 2: Extract URLs
        urls = extract_urls_from_results(results)
        
        # Step 3: Get content
        contents = self.content_tool._run(urls=urls)
        
        # Step 4: Summarize
        summary = summarize_contents(contents)
        
        return summary
```

## Testing Tools

Always test your tools before using them with agents:

### Unit Testing

```python
def test_calculator_tool():
    tool = CalculatorTool()
    
    # Test valid input
    result = tool._run("2 + 2")
    assert "4" in result
    
    # Test invalid input
    result = tool._run("invalid")
    assert "Error" in result
    
    print("All tests passed!")

test_calculator_tool()
```

### Integration Testing

```python
def test_tool_with_agent():
    tool = CalculatorTool()
    agent = create_react_agent(tools=[tool])
    
    result = agent.invoke({
        "messages": [HumanMessage("Calculate 15 * 23")]
    })
    
    response = result["messages"][-1].content
    assert "345" in response or "Result: 345" in response
    
    print("Integration test passed!")

test_tool_with_agent()
```

## Best Practices

### ✅ Do's

1. **Clear Names**: Use descriptive, action-oriented names
   ```python
   ✅ get_weather, send_email, search_database
   ❌ tool1, helper, util
   ```

2. **Good Descriptions**: Explain when to use the tool
   ```python
   ✅ "Get current weather data for any city. Use when user asks about weather."
   ❌ "Weather"
   ```

3. **Handle Errors**: Return helpful error messages
   ```python
   ✅ "Error: Invalid city name. Please provide a valid city."
   ❌ throw Exception
   ```

4. **Validate Inputs**: Check parameters before processing
   ```python
   ✅ Check if email is valid before sending
   ✅ Verify file exists before reading
   ```

5. **Return Structured Data**: Make it easy for agents to parse
   ```python
   ✅ "Temperature: 18°C\nConditions: Sunny"
   ✅ JSON: {"temp": 18, "conditions": "sunny"}
   ❌ "its 18 degrees and sunny"
   ```

### ❌ Don'ts

1. **Don't Make Tools Too Complex**: Keep them focused
2. **Don't Ignore Errors**: Always handle exceptions
3. **Don't Use Generic Names**: Be specific
4. **Don't Forget Timeouts**: Set reasonable timeouts for API calls
5. **Don't Expose Sensitive Data**: Sanitize outputs

## Complete Examples

### Example 1: File System Tool

```python
import os
from pathlib import Path
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type, Optional

class FileReadInput(BaseModel):
    file_path: str = Field(description="Path to file to read")
    max_lines: Optional[int] = Field(default=None, description="Max lines to read")

class FileReadTool(BaseTool):
    name: str = "read_file"
    description: str = (
        "Read contents of a text file. "
        "Use when you need to access file contents. "
        "Returns the file content as text."
    )
    args_schema: Type[BaseModel] = FileReadInput
    allowed_dir: str = Field(default="./data")  # Security: restrict to specific dir
    
    def _run(self, file_path: str, max_lines: Optional[int] = None) -> str:
        try:
            # Security: Ensure file is in allowed directory
            full_path = Path(self.allowed_dir) / file_path
            if not full_path.is_relative_to(self.allowed_dir):
                return "Error: Access denied - file outside allowed directory"
            
            if not full_path.exists():
                return f"Error: File '{file_path}' not found"
            
            # Read file
            with open(full_path, 'r', encoding='utf-8') as f:
                if max_lines:
                    lines = [f.readline() for _ in range(max_lines)]
                    content = ''.join(lines)
                else:
                    content = f.read()
            
            return f"File: {file_path}\n\n{content}"
            
        except PermissionError:
            return f"Error: Permission denied for '{file_path}'"
        except UnicodeDecodeError:
            return f"Error: '{file_path}' is not a text file"
        except Exception as e:
            return f"Error reading file: {str(e)}"
```

### Example 2: API Integration Tool

```python
import requests
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Type, Dict, Any

class APICallInput(BaseModel):
    endpoint: str = Field(description="API endpoint path (e.g., '/users/123')")
    method: str = Field(default="GET", description="HTTP method (GET, POST, etc.)")
    data: Dict[str, Any] = Field(default={}, description="Request body data")

class APITool(BaseTool):
    name: str = "call_api"
    description: str = (
        "Make HTTP requests to the company API. "
        "Use to retrieve or update data from internal systems."
    )
    args_schema: Type[BaseModel] = APICallInput
    base_url: str = Field(default="https://api.company.com")
    api_key: str = Field(default="")
    
    def _run(self, endpoint: str, method: str = "GET", data: Dict[str, Any] = {}) -> str:
        try:
            url = f"{self.base_url}{endpoint}"
            headers = {
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json"
            }
            
            response = requests.request(
                method=method,
                url=url,
                headers=headers,
                json=data if data else None,
                timeout=30
            )
            
            response.raise_for_status()
            
            return f"Status: {response.status_code}\n{response.text}"
            
        except requests.Timeout:
            return "Error: API request timed out"
        except requests.HTTPError as e:
            return f"Error: HTTP {e.response.status_code} - {e.response.text}"
        except Exception as e:
            return f"Error: {str(e)}"
```

## Key Takeaways

✅ **Tools Extend Capabilities**: They give agents access to external systems
✅ **Simple Structure**: Name, description, schema, and run method
✅ **Clear Communication**: Good descriptions help agents use tools effectively
✅ **Error Handling**: Always return helpful error messages
✅ **Testing**: Test tools independently before using with agents
✅ **Security**: Validate inputs and restrict access appropriately

## What's Next?

Continue learning about:
- **State Management**: How agents maintain context
- **Message Flow**: Understanding conversation history
- **Structured Output**: Getting validated JSON responses

**Continue to**: [06_State_Management.md](./06_State_Management.md)

## Additional Resources

- **Tool Examples**: `core/valyu_tools/tools.py`
- **LangChain Tools**: [python.langchain.com/docs/modules/tools/](https://python.langchain.com/docs/modules/tools/)
- **Tutorial Notebooks**: `tutorials/02_custom_tools.ipynb`

---

**Awesome!** 🎉 You now know how to create and use tools to build powerful agents that can interact with the real world!
