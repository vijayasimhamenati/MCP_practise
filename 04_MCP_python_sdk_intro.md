**Model Context Protocol (MCP): Interview Preparation Guide (Part 4 - Python SDK & FastMCP)**

Here is the architectural reference diagram mapping the Client-Server zones: watermarked_img_12915730740863125352.png.

**1. Core Definition: FastMCP vs. Low-Level Server**
When developing an MCP Server using the official Python SDK, engineers choose between two primary implementation classes. The **Low-Level Server Class** (formerly just `Server`) is the foundational implementation that requires explicit, manual definitions for routing, tool listing, and schema generation. **FastMCP** is a high-level wrapper built on top of this class. It abstracts away the boilerplate infrastructure, allowing developers to expose Python functions as MCP Tools instantly using decorator patterns, automatically inferring schemas from type hints and docstrings.

**2. Structured Notes & Key Concepts**

* **The Low-Level `Server` Class:** Highly verbose and complex. It requires the developer to manually write the logic for the "tools list request," explicitly define the JSON input schemas, and write custom routing logic to handle what happens when a specific tool is called. The instructor compares this to writing in C or C++—powerful but labor-intensive.
* **The `FastMCP` Class:** The modern standard for Python MCP development. It abstracts the routing and schema generation entirely. The instructor compares this to Python—a cleaner abstraction sitting on top of the lower-level architecture.
* **Implicit vs. Explicit Definition:** With FastMCP, the tool's Name (function name), Description (docstring), and Input Schema (type hints) are derived *implicitly*. In the low-level server, these must be defined *explicitly* in large JSON structures.
* **Industry Best Practice:** Default to FastMCP for 95%+ of server deployments. It is actively maintained by the open-source community and handles almost all standard use cases while keeping codebases modular and clean.

**3. Code Implementation: The FastMCP Abstraction**
For engineers bridging backend architecture and AI, FastMCP utilizes the exact same intuitive decorator patterns seen in modern frameworks like FastAPI. Instead of manually mapping routes and JSON schemas, you wrap standard functions.

**The FastMCP Way (Modern & Clean):**

```python
from mcp.server.fastmcp import FastMCP

# Instantiate the server
mcp = FastMCP("weather-server")

# The decorator automatically parses the function name, docstring, and type hints 
# to generate the Tool List and Input Schema for the LLM.
@mcp.tool()
def get_weather(location: str) -> str:
    """Provides the current weather for a specified location."""
    # Custom API logic goes here
    return f"Weather for {location} is warm and sunny."

```

**The Low-Level Way (Legacy/Verbose):**

```python
# (Conceptual representation of the boilerplate required)
# You must manually define the schema, handle the list_tools request, and route the call_tool request.
server = Server("weather-server")

@server.list_tools()
async def list_tools():
    return [
        Tool(
            name="get_weather",
            description="Provides the current weather for a specified location.",
            inputSchema={
                "type": "object",
                "properties": {"location": {"type": "string"}},
                "required": ["location"]
            }
        )
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "get_weather":
        # Custom logic goes here
        return f"Weather for {arguments['location']} is warm and sunny."

```

**4. AI Engineer Interview Q&A**

**Basic Concept Questions**

* **Q: When building an MCP Server in Python, what is the difference between FastMCP and the low-level Server class?**
* **A:** The low-level Server class requires manual boilerplate for every capability—you have to explicitly write the JSON input schemas and handle the routing for listing and calling tools. FastMCP is a high-level wrapper that abstracts this away, allowing you to turn standard Python functions into MCP tools using decorators.


* **Q: Which SDK class should you default to for new MCP projects?**
* **A:** You should always default to FastMCP. It covers over 95% of standard use cases, produces highly readable code, and is the primary focus of community support. The low-level server should be reserved strictly for edge cases requiring custom protocol manipulation.



**Advanced Architecture & Development Questions**

* **Q: How exactly does FastMCP generate the "Input Schema" for the LLM without you writing any JSON?**
* **A:** FastMCP relies on Python's introspection capabilities. When you apply the `@mcp.tool()` decorator, the framework inspects the function's type hints (e.g., `location: str`) and Pydantic models to automatically generate the standard JSON-RPC schema. It simultaneously extracts the function's docstring to serve as the tool's description for the LLM's system prompt.


* **Q: If you are tasked with migrating a legacy, hardcoded tool script into an MCP Server to be used by an agentic framework, how would you architect the refactor?**
* **A:** I would wrap the existing script in a FastMCP class. By ensuring the legacy functions have strict Python type hints and descriptive docstrings, I can simply apply the `@mcp.tool()` decorator to expose them. This immediately converts the legacy script into a modular MCP Server without rewriting the underlying execution logic, perfectly bridging the backend processes with the AI agents.