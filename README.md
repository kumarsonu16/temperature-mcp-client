# MCP Temperature Client

> An intelligent conversational client built on the Model Context Protocol (MCP) that provides temperature conversion services through natural language interactions.

## 🚀 Quick Start

### Prerequisites
Ensure you have Python 3.9+ and required dependencies installed:

```bash
uv pip install -r requirements.txt
```

### Running the Application

1. **Activate virtual environment:**
   ```bash
   source .venv/bin/activate
   ```

2. **Start the client:**
   ```bash
   uv run python cli/main.py
   ```

3. **Start conversing:**
   ```
   > Convert 25°C to Fahrenheit
   > What is absolute zero in all temperature scales?
   > Convert 100°F to both Celsius and Kelvin
   ```

---

## 📖 Understanding Core Components

### Session Management

**Why do we need sessions?**

Sessions in the MCP Client provide essential functionality for maintaining intelligent conversations:

- **🧠 Context Tracking**: Remembers previous messages and maintains conversation continuity
- **🔒 Conversation Isolation**: Keeps multiple users/conversations separate without data mixing
- **📝 Stateful Operations**: Enables workflows that require remembering previous actions
- **💬 Multi-turn Dialogs**: Supports ongoing, multi-step conversations beyond single requests

> **Summary:** Sessions provide context, isolation, and continuity for intelligent user-agent interactions.

---

### ADK Agent vs Direct MCP Access

**Can you work with MCP servers directly?**

Yes, you can interact with MCP servers directly through HTTP requests or CLI tools, but you lose significant benefits.

#### ⚡ ADK Agent Advantages:

| Feature | Direct MCP | ADK Agent |
|---------|------------|-----------|
| **Interface** | Multiple endpoints | Single unified API |
| **Intelligence** | Manual tool selection | Automatic orchestration |
| **Language** | JSON/HTTP requests | Natural language |
| **Context** | Stateless | Multi-turn conversations |
| **Discovery** | Manual configuration | Dynamic tool discovery |
| **Error Handling** | Basic HTTP errors | Enhanced debugging |

#### 🎯 Key Benefits:
- **Unified Interface**: Aggregates tools from multiple MCP servers
- **Intelligent Orchestration**: Automatically selects and sequences tools
- **Natural Language Support**: Understands plain English requests
- **Context Management**: Maintains conversation history and state
- **Tool Filtering**: Dynamically discovers and filters available tools
- **Enhanced Debugging**: Provides detailed error handling and logging

---

## 🔧 Component Deep Dive

### AgentWrapper: The Brain Coordinator

**What does AgentWrapper do?**
- 🔌 Manages ADK agent connections to multiple MCP servers
- 🛠️ Loads and filters available tools (temperature conversions, etc.)
- 🎛️ Builds intelligent agents that can answer user queries
- 📊 Monitors server health and connection status

#### 📋 Complete Example Scenario

**1. Configuration Setup**
```json
// config/servers.json
{
  "servers": [
    {
      "name": "local_http",
      "transport": "http", 
      "url": "http://localhost:8000"
    },
    {
      "name": "local_stdio",
      "transport": "stdio",
      "command": ["python", "servers/stdio/terminal_server.py"]
    }
  ]
}
```

**2. Agent Initialization**
```python
# Filter to only temperature conversion tools
tool_filter = ["celsius_to_fahrenheit", "fahrenheit_to_celsius"]
agent_wrapper = AgentWrapper(tool_filter=tool_filter)
```

**3. Building the Agent**
```python
await agent_wrapper.build()
# This automatically:
# ✅ Loads server configurations
# ✅ Connects to each server
# ✅ Discovers available tools  
# ✅ Applies your filter
# ✅ Creates ADK agent with filtered tools
```

**4. Using the Agent**
```python
if agent_wrapper.is_ready():
    # Now the agent can handle requests like:
    # "Convert 100°C to Fahrenheit"
    # "What is -40°F in Celsius?"
    # The agent intelligently selects the right MCP tools
```

**5. Health Monitoring**
```python
status = agent_wrapper.get_server_status()
print(status)  
# Output: {'local_http': 'connected', 'local_stdio': 'connected'}
```

**6. Graceful Cleanup**
```python
await agent_wrapper.close()  # Properly disconnect from all servers
```

#### 📊 AgentWrapper Lifecycle

| Step | Action | Result |
|------|--------|--------|
| **Initialize** | Set up tool filter, prepare agent wrapper | Agent wrapper ready for configuration |
| **Build** | Discover servers, connect, filter tools, create ADK agent | Fully functional intelligent agent |
| **Use** | Agent processes queries using MCP tools | Natural language responses with tool results |
| **Status** | Check server and tool availability | Real-time connection health monitoring |
| **Close** | Gracefully disconnect from all servers | Clean resource cleanup |

---

### Runner: The Execution Engine

**What is the Runner?**

The Runner acts as the execution engine that makes everything work together:

```python
self.runner = Runner(
    agent=self.agent_wrapper.agent,        # The intelligent ADK agent with MCP tools
    app_name=self.app_name,               # Application identifier for context
    session_service=self.session_service   # Session manager for user context
)
```

#### 🔄 Runner Responsibilities:

- **💬 Conversation Execution**: Processes user messages through the agent
- **🗂️ Session Management**: Maintains conversation history and user context  
- **⚡ Response Streaming**: Provides real-time event streaming via `run_async()`
- **🛠️ Tool Orchestration**: Coordinates MCP tool calls and response handling

#### 🌊 Example Message Flow:

```
User Input: "Convert 25°C to Fahrenheit"
              │
              ▼
   ┌─────────────────────┐
   │   1. Runner         │ ◄── Receives user message
   │   receive_message   │
   └─────────────────────┘
              │
              ▼
   ┌─────────────────────┐
   │   2. Agent          │ ◄── Processes with AI intelligence  
   │   analyze_request   │
   └─────────────────────┘
              │
              ▼
   ┌─────────────────────┐
   │   3. Tool Selection │ ◄── Chooses celsius_to_fahrenheit
   │   celsius_to_f      │
   └─────────────────────┘
              │
              ▼
   ┌─────────────────────┐
   │   4. MCP Server     │ ◄── Executes temperature conversion
   │   HTTP Call         │
   └─────────────────────┘
              │
              ▼
   ┌─────────────────────┐
   │   5. Stream Events  │ ◄── Returns: thinking → tool call → result → answer
   │   Real-time Updates │
   └─────────────────────┘
```

> **Key Insight:** The Runner is the "execution engine" that orchestrates the entire conversation flow from user input to formatted response.

---

## 🎯 Getting Started Examples

### Temperature Conversion Examples
```bash
# Simple conversions
> Convert 0°C to Fahrenheit
> What is 212°F in Celsius?

# Multiple scale conversions  
> Convert 100°F to both Celsius and Kelvin
> Show me absolute zero in all temperature scales

# Complex queries
> If water freezes at 0°C, what temperature is that in Fahrenheit and Kelvin?
```

### Debug Commands
```bash
> status        # Show system and server status
> debug on      # Enable detailed MCP interaction logging  
> debug off     # Disable detailed logging
```

---

## 📚 Additional Resources

- **📖 [Complete Documentation](DOCUMENTATION.md)** - Detailed architecture and component analysis
- **⚙️ [Configuration Guide](config/servers.json)** - Server setup and customization
- **🧪 [Test Suite](tests/)** - Comprehensive testing examples

---

*Built with ❤️ using Model Context Protocol, Google ADK, and FastMCP*


**Why do we need the session**  

The session in MCPClient is needed to:

- Track user interactions: It keeps context for each user and conversation, so the agent can remember previous messages and maintain continuity.
- Isolate conversations: Each session is separate, allowing multiple users or conversations to run independently without mixing data.
- Enable stateful operations: Some tools or workflows may require remembering previous actions or data, which is managed via the session.
- Support multi-turn dialogs: Sessions allow the agent to handle ongoing, multi-step conversations rather than just single, stateless requests.

* In summary, sessions provide context, isolation, and continuity for user-agent interactions.



# Can i work with MCP server without building the agent and what's advanatage of ADK agent?

You can interact with an MCP server directly (for example, by sending HTTP requests to its endpoints or using its CLI tools), but you won’t get the benefits of the ADK agent.

Advantages of using the ADK agent:
* Unified interface: The agent aggregates tools from multiple MCP servers, so you interact with all tools through a single, consistent API.
* Intelligent orchestration: The agent can choose, sequence, and combine tools automatically to fulfill complex user requests.
* Natural language support: The agent understands and processes user queries in natural language, making it easier to use.
* Context management: The agent maintains session and conversation context, enabling multi-turn interactions and stateful workflows.
* Tool filtering and discovery: The agent can filter available tools and discover new ones dynamically from connected servers.
* Error handling and debugging: The agent provides enhanced error handling, logging, and debugging features.
* Summary:
Direct MCP server access is possible, but the ADK agent adds intelligence, convenience, and advanced capabilities for building robust, user-friendly applications.

# Can you explain this with an examples

What does AgentWrapper do?
* Manages the ADK agent and its connection to one or more MCP servers.
* Loads available tools from those servers (like temperature conversion).
* Optionally filters which tools are available.
* Builds an agent that can use these tools to answer user queries.

Example Scenario
1. Configuration
Suppose your config file lists two MCP servers:

One HTTP server at http://localhost:8000
One stdio server running a local Python script

2. Initialization
You create an AgentWrapper and specify you only want temperature conversion tools:

```bash
tool_filter = ["celsius_to_fahrenheit", "fahrenheit_to_celsius"]
agent_wrapper = AgentWrapper(tool_filter=tool_filter)
```

3. Building the Agent
You call await agent_wrapper.build().
This does the following:

Loads server configs.
Connects to each server.
Discovers available tools, applies your filter.
Creates an ADK agent with those tools.

4. Using the Agent
Now, you can use the agent to process user requests:


```bash
   if agent_wrapper.is_ready():
    # The agent can now answer questions like:
    # "Convert 100 Celsius to Fahrenheit"
    # "What is -40 Fahrenheit in Celsius?"
    # The agent will use the loaded MCP tools to perform these conversions.
```

5. Checking Server Status
You can check which servers connected successfully:

```bash
status = agent_wrapper.get_server_status()
print(status)  # e.g., {'local_http': 'connected', 'local_stdio': 'connected'}

```

6. Shutting Down
When done, you can close all connections:

```bash 
await agent_wrapper.close()
```

#### 📋 AgentWrapper Summary

| Step | What Happens |
|------|--------------|
| **Initialize** | Set up tool filter, prepare agent wrapper |
| **Build** | Discover servers, connect, filter tools, create ADK agent |
| **Use** | Agent answers queries using MCP tools |
| **Status** | Check which servers and tools are available |
| **Close** | Gracefully disconnect from all servers |

**In short:**
AgentWrapper lets you easily connect to multiple MCP servers, load and filter tools, and build an intelligent agent that can answer user queries using those tools.


# What is Runner and how does it work  

This code creates a Runner instance that acts as the execution engine for your MCP client. Here's what each parameter does:

```bash
self.runner = Runner(
    agent=self.agent_wrapper.agent,        # The ADK agent with MCP tools
    app_name=self.app_name,               # App identifier for context
    session_service=self.session_service   # Session manager for user context
)
```


What the Runner does:

* Executes conversations: When you call send_message(), the Runner takes your message and sends it to the agent for processing.
* Manages sessions: Uses the session service to maintain conversation history and context for each user.
* Streams responses: Provides the run_async() method that yields real-time events as the agent processes your request.
* Orchestrates tool calls: Coordinates when the agent needs to call MCP tools (like temperature conversion) and handles the responses.

Example flow:

You send: "Convert 25°C to Fahrenheit"
1. Runner receives your message
2. Runner asks the agent to process it
3. Agent decides to use the celsius_to_fahrenheit tool
4. Runner executes the tool call via MCP
5. Runner streams back: thinking → tool call → tool response → final answer
* In essence: The Runner is the "execution engine" that makes everything work together.