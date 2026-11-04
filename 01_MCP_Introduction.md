**1. Structured Notes & Key Concepts**

The **Model Context Protocol (MCP)** operates as a universal, open standard to connect AI models securely to external data sources and tools. You can conceptualize it as the "USB-C port for AI"—a standardized interface that eliminates the need to write custom integration code for every new API or framework.

* **The Fragmentation Problem:** Historically, executing a **Tool Call** required writing custom integration logic for specific frameworks (e.g., OpenAI SDK, LangChain). When APIs updated, these brittle scripts broke. Furthermore, relying on remote servers for tool execution made it nearly impossible for **LLMs** to securely interact with a user's local machine (e.g., analyzing local codebase in VS Code or organizing desktop files).
* **The MCP Solution:** MCP abstracts the connection layer. Developers build an **MCP Server** once, and it instantly becomes compatible with any **Host Process** (like Claude Desktop or Cursor) that features a built-in **MCP Client**.
* **Core Architecture Components:**
* **Host Process:** The user-facing application (e.g., ChatGPT, Claude Desktop, IDEs) that orchestrates the user interface and communicates with the underlying LLM.
* **MCP Client:** A 1:1 protocol client embedded within the Host Process that routes requests to the server using just a few lines of JSON configuration.
* **MCP Server:** A lightweight server that securely exposes capabilities to the client. *Supplemental detail: Servers typically expose three primitives: **Tools** (executable functions), **Resources** (read-only data like local files), and **Prompts** (reusable template structures).*
* **Transport Protocols (Supplemental):** MCP connections rely on two primary transport layers: **STDIO** (Standard Input/Output) for secure, local machine interactions, and **SSE** (Server-Sent Events) for remote, network-based API servers.



**2. System Architecture & Flow**

The standard data lifecycle in an MCP-enabled application abstracts the "red arrows" (custom code) into a standardized green pathway:

**[User]** ➔ *Submits prompt*
⬇
**[Host Process (e.g., Claude Desktop)]** ⟷ *Queries* ⟷ **[Cloud LLM]**
⬇ *(LLM decides a tool is needed)*
**[MCP Client]**
⬇ *(Passes JSON RPC request via STDIO or SSE)*
**[MCP Server]**
⬇ *(Executes logic)*
**[External Systems]** (e.g., Gmail API, Local SQLite DB, File System)

**3. Code Implementation**

Based on the architecture discussed, here is how you connect them in practice.

**Client-Side: The JSON Configuration**
Instead of writing hundreds of lines of API logic, you simply point your Host's MCP Client to the Server using a configuration file:

```json
{
  "mcpServers": {
    "local-filesystem": {
      "command": "python",
      "args": ["-m", "mcp_simple_server"]
    }
  }
}

```

**Server-Side: Production-Ready MCP Tool Definition**
Using the official Python SDK, here is how you define a single, atomic tool on the MCP server that the client can immediately access:

```python
from mcp.server.fastmcp import FastMCP

# Initialize the MCP Server
mcp = FastMCP("local-filesystem")

# Define a tool using the decorator. 
# The docstring is automatically parsed and sent to the LLM to explain how to use the tool.
@mcp.tool()
def read_local_file(file_path: str) -> str:
    """Read the contents of a local file safely."""
    try:
        with open(file_path, "r") as f:
            return f.read()
    except Exception as e:
        return f"Error reading file: {str(e)}"

if __name__ == "__main__":
    # Runs the server using STDIO transport by default
    mcp.run()

```

**4. AI Engineer Interview Q&A**

**Basic Level:**

* **Q: What exact problem does the Model Context Protocol (MCP) solve?**
* **A:** MCP standardizes how AI agents interact with external tools and data. Without MCP, developers must write custom, framework-specific integration code (e.g., OpenAI vs. LangChain) for every API. MCP standardizes this, acting like a universal "USB-C" connection where building a server once makes it compatible with any MCP-enabled host.


* **Q: What is the difference between an MCP Client and an MCP Server?**
* **A:** The **MCP Client** lives inside the **Host Process** (like an IDE or chat interface) and manages the routing of LLM requests. The **MCP Server** sits in front of the external APIs or local resources, securely exposing functions and data back to the client.



**Advanced Level:**

* **Q: If an agentic framework like LangGraph needs to execute an MCP tool, how would you architect that connection securely?**
* **A:** I would instantiate an **MCP Client** within a LangGraph node. If interacting with local resources (like reading a codebase), I'd use the **STDIO** transport layer to keep the execution entirely local, avoiding network exposure. If the tool interacts with external SaaS APIs, I would isolate the **MCP Server** in a containerized Docker environment and connect the client to it via **SSE** (Server-Sent Events) over HTTPS, strictly validating input schemas via Pydantic before execution.


* **Q: How does MCP address the vulnerability of LLMs executing arbitrary code on a local machine?**
* **A:** MCP abstracts the execution layer away from the LLM. The LLM never writes or executes the actual API request; it merely returns a JSON object matching the schema of the **Tools** exposed by the **MCP Server**. The server itself, which is under the developer's complete control and can be heavily sandboxed, is responsible for the actual execution and validation of the parameters.
