# MCP Temperature Client - Complete Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Component Flow](#component-flow)
4. [Libraries & Dependencies](#libraries--dependencies)
5. [Detailed Component Analysis](#detailed-component-analysis)
6. [Execution Flow](#execution-flow)
7. [Configuration](#configuration)
8. [Getting Started](#getting-started)

## Project Overview

The MCP Temperature Client is an intelligent conversational client built on the **Model Context Protocol (MCP)** that provides temperature conversion services through natural language interactions. It demonstrates how to build a sophisticated client that can connect to multiple MCP servers and orchestrate tool usage through an AI agent.

### Key Features
- **Natural Language Processing**: Understands temperature conversion requests in plain English
- **Multiple Server Support**: Connects to both HTTP and stdio MCP servers
- **Intelligent Agent**: Uses Google's ADK (Agent Development Kit) with Gemini for smart tool orchestration
- **Rich UI**: Beautiful terminal interface with syntax highlighting and interactive displays
- **Session Management**: Maintains conversation context across interactions
- **Real-time Debugging**: Shows detailed MCP client-server interactions

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP Temperature Client                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │     CLI     │  │   Formatters │  │   Config Loader     │ │
│  │   (main.py) │  │ (formatters) │  │   (config_loader)   │ │
│  └─────────────┘  └──────────────┘  └─────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                 MCP Client                              │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │ │
│  │  │ Session     │  │Agent Wrapper │  │    Runner       │ │ │
│  │  │ Service     │  │   (ADK)      │  │  (Execution)    │ │ │
│  │  └─────────────┘  └──────────────┘  └─────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                Server Launchers                         │ │
│  │  ┌─────────────┐              ┌─────────────────────────┐ │ │
│  │  │    HTTP     │              │       Stdio            │ │ │
│  │  │  Launcher   │              │     Launcher           │ │ │
│  │  └─────────────┘              └─────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                   MCP Servers                           │ │
│  │  ┌─────────────┐              ┌─────────────────────────┐ │ │
│  │  │Temperature  │              │    Terminal Server     │ │ │
│  │  │   Server    │              │   (Command Runner)     │ │ │
│  │  │   (HTTP)    │              │      (Stdio)           │ │ │
│  │  └─────────────┘              └─────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Flow

### 1. Initialization Flow
```
User starts CLI
       │
       ▼
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│Load Config  │───▶│Start Servers │───▶│Initialize Client│
└─────────────┘    └──────────────┘    └─────────────────┘
       │                  │                      │
       ▼                  ▼                      ▼
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│servers.json │    │HTTP Server   │    │Session Service  │
│             │    │Stdio Server  │    │Agent Wrapper    │
└─────────────┘    └──────────────┘    │Runner           │
                                       └─────────────────┘
```

### 2. Message Processing Flow
```
User Input: "Convert 25°C to Fahrenheit"
            │
            ▼
    ┌─────────────┐
    │   CLI       │
    │ (main.py)   │
    └─────────────┘
            │
            ▼
    ┌─────────────┐
    │MCP Client   │
    │send_message │
    └─────────────┘
            │
            ▼
    ┌─────────────┐
    │   Runner    │
    │ run_async   │
    └─────────────┘
            │
            ▼
    ┌─────────────┐
    │Agent Wrapper│
    │   (ADK)     │
    └─────────────┘
            │
            ▼
┌─────────────────────────┐
│Agent decides to use:    │
│celsius_to_fahrenheit    │
│tool from HTTP server    │
└─────────────────────────┘
            │
            ▼
    ┌─────────────┐
    │Temperature  │
    │   Server    │
    │   (HTTP)    │
    └─────────────┘
            │
            ▼
    ┌─────────────┐
    │Tool Response│
    │  77°F       │
    └─────────────┘
            │
            ▼
    ┌─────────────┐
    │  Formatter  │
    │Display Result│
    └─────────────┘
```

---

## Libraries & Dependencies

### Core Libraries

#### **Model Context Protocol (MCP)**
- **Purpose**: Enables communication between clients and AI-powered tools
- **Usage**: Base protocol for all server-client interactions
- **Files**: Throughout the project, especially in `src/client/` and `servers/`

#### **Google ADK (Agent Development Kit)**
- **Purpose**: Provides intelligent agent capabilities with Gemini integration
- **Components Used**:
  - `Agent`: Core AI agent for processing requests
  - `Runner`: Execution engine for agent tasks
  - `InMemorySessionService`: Session management
- **Files**: `src/agent/agent_wrapper.py`, `src/client/mcp_client.py`

#### **FastMCP**
- **Purpose**: Rapid development framework for MCP servers
- **Usage**: Powers the HTTP temperature server
- **Features**: Streamable HTTP transport, automatic tool registration
- **Files**: `servers/http/temperature_server.py`

#### **Rich**
- **Purpose**: Beautiful terminal formatting and display
- **Components Used**:
  - `Console`: Terminal output management
  - `Panel`: Bordered content display
  - `Table`: Tabular data formatting
  - `Syntax`: Code syntax highlighting
- **Files**: `src/utils/formatters.py`

#### **Pydantic**
- **Purpose**: Data validation and serialization
- **Usage**: Input/output model validation for temperature conversions
- **Files**: `servers/http/temperature_server.py`

#### **Click**
- **Purpose**: Command-line interface creation
- **Usage**: CLI argument parsing for servers
- **Files**: `servers/http/temperature_server.py`

#### **HTTPX**
- **Purpose**: HTTP client for server health checks
- **Usage**: Verifying server availability during startup
- **Files**: `servers/http/server_launcher.py`

---

## Detailed Component Analysis

### 1. CLI Module (`cli/main.py`)
**Purpose**: Main entry point and user interface

**Key Responsibilities**:
- Initialize and start MCP servers
- Create and configure MCP client
- Handle user input and display responses
- Manage application lifecycle

**Key Functions**:
```python
class MCPClientCLI:
    async def start_servers() -> bool
    async def initialize_client() -> bool  
    async def handle_user_input()
    async def cleanup()
```

### 2. MCP Client (`src/client/mcp_client.py`)
**Purpose**: Core client orchestration and communication

**Key Components**:
- **Session Management**: Tracks user conversations
- **Agent Integration**: Bridges MCP tools with ADK agent
- **Message Processing**: Handles user requests and agent responses

**Key Methods**:
```python
class MCPClient:
    async def initialize()           # Setup client components
    async def send_message()         # Process user input
    async def close()               # Cleanup resources
    def get_available_tools()       # List accessible tools
```

### 3. Agent Wrapper (`src/agent/agent_wrapper.py`)
**Purpose**: Manages ADK agent and MCP server connections

**Key Responsibilities**:
- Load server configurations
- Connect to multiple MCP servers (HTTP & stdio)
- Filter available tools
- Build and manage ADK agent
- Monitor server health

**Key Methods**:
```python
class AgentWrapper:
    async def build()               # Initialize agent with MCP tools
    def is_ready()                 # Check if agent is operational
    def get_server_status()        # Get connection status
    async def close()              # Cleanup connections
```

### 4. Server Launchers (`servers/http/server_launcher.py`)
**Purpose**: Manage MCP server lifecycle

**Key Features**:
- Start servers as subprocesses
- Health monitoring with HTTP requests
- Graceful shutdown handling
- Process output capture

**Server Management Flow**:
```python
class ServerLauncher:
    def start_temperature_server()  # Launch HTTP server
    def _wait_for_server()         # Health check via HTTP
    def stop_servers()             # Graceful shutdown
```

### 5. Temperature Server (`servers/http/temperature_server.py`)
**Purpose**: HTTP MCP server providing temperature conversion tools

**Available Tools**:
- `celsius_to_fahrenheit`
- `fahrenheit_to_celsius`
- `celsius_to_kelvin`
- `kelvin_to_celsius`
- `fahrenheit_to_kelvin`
- `kelvin_to_fahrenheit`

**Features**:
- Input validation with Pydantic models
- Formula information in responses
- Physical bounds checking (absolute zero)
- Streamable HTTP transport

### 6. Response Formatters (`src/utils/formatters.py`)
**Purpose**: Beautiful terminal output and debugging displays

**Formatting Types**:
- **JSON Responses**: Syntax-highlighted API responses
- **MCP Interactions**: Real-time client-server communication
- **Temperature Tables**: Organized conversion results
- **Error Messages**: Consistent error display
- **Welcome Banner**: Application introduction

---

## Execution Flow

### Detailed Step-by-Step Flow

#### Phase 1: Initialization
1. **CLI Startup** (`cli/main.py`)
   - Parse command line arguments
   - Configure logging
   - Display welcome banner

2. **Server Launch** (`servers/http/server_launcher.py`)
   - Read server configurations from `config/servers.json`
   - Start HTTP temperature server on localhost:8000
   - Start stdio terminal server (if configured)
   - Perform health checks on each server

3. **Client Initialization** (`src/client/mcp_client.py`)
   - Create session service for conversation management
   - Initialize agent wrapper with tool filtering
   - Build ADK agent with connected MCP tools
   - Create runner for agent execution

#### Phase 2: User Interaction
1. **Input Processing**
   - User types natural language request
   - CLI captures input and passes to MCP client

2. **Agent Processing**
   - Runner sends message to ADK agent
   - Agent analyzes request using Gemini
   - Agent determines which MCP tools to use

3. **Tool Execution**
   - Agent calls appropriate temperature conversion tool
   - HTTP request sent to temperature server
   - Server processes conversion and returns result

4. **Response Formatting**
   - Tool response returned to agent
   - Agent formulates natural language response
   - Formatter displays result with rich formatting

#### Phase 3: Cleanup
1. **Graceful Shutdown**
   - Close MCP server connections
   - Terminate server processes
   - Cleanup session resources

---

## Configuration

### Server Configuration (`config/servers.json`)
```json
{
  "servers": [
    {
      "name": "local_http",
      "transport": "http",
      "url": "http://localhost:8000",
      "timeout": 30
    },
    {
      "name": "local_stdio", 
      "transport": "stdio",
      "command": ["python", "servers/stdio/terminal_server.py"],
      "timeout": 30
    }
  ]
}
```

### Tool Filtering
Configure which tools are available to the agent:
```python
DEFAULT_TOOLS = [
    'celsius_to_fahrenheit',
    'fahrenheit_to_celsius',
    'celsius_to_kelvin', 
    'kelvin_to_celsius',
    'fahrenheit_to_kelvin',
    'kelvin_to_fahrenheit',
    'run_command'
]
```

---

## Getting Started

### Prerequisites
```bash
# create virtual env using uv package manager
uv venv

# Activate virtual environment (On macOS/Linux)
source .venv/bin/activate   # On Windows: .venv\Scripts\activate

# Install dependencies:
uv pip install -r requirements.txt
```

### Create .env file:
```bash
# Google AI API Configuration
GOOGLE_API_KEY=your_google_api_key_here
# MCP Configuration
MCP_CONFIG_PATH=config/servers.json
# Logging
LOG_LEVEL=INFO
# Workspace
WORKSPACE_DIR=workspace
```

### Running the Application
```bash
uv run python cli/main.py
```

### Example Interactions with Debug Information

Multiple Conversions with File Operations:

```bash
You: Convert 100°F to Celsius and Kelvin, then save results to a file temp_conv.txt
```

### Example Interactions
```
User: "Convert 25 degrees Celsius to Fahrenheit"
Agent: 25°C converts to 77°F using the formula °F = (°C × 9/5) + 32

User: "What is absolute zero in all temperature scales?"
Agent: Absolute zero is:
- -273.15°C (Celsius)
- -459.67°F (Fahrenheit)  
- 0 K (Kelvin)

User: "Convert 100°F to both Celsius and Kelvin"
Agent: 100°F converts to:
- 37.78°C (Celsius)
- 310.93 K (Kelvin)
```

### Debug Commands
- `status` - Show system and server status
- `debug on/off` - Toggle detailed MCP interaction logging

---

## Key Design Patterns

### 1. **Separation of Concerns**
- CLI handles user interaction
- Client manages orchestration
- Servers provide tools
- Formatters handle display

### 2. **Async/Await Pattern**
- All I/O operations are asynchronous
- Enables concurrent server management
- Supports streaming responses

### 3. **Factory Pattern**
- Server launchers create and manage processes
- Agent wrapper builds configured agents
- Formatters create display components

### 4. **Observer Pattern**
- Runner streams agent events
- Client observes and processes events
- Formatters display real-time updates

This architecture provides a robust, scalable foundation for building intelligent MCP clients that can work with multiple servers and provide rich user experiences.
