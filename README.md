# Multi-Agent Customer Service System with MCP Integration

A production-ready multi-agent customer service system built with Google's Agent Development Kit (ADK) and Model Context Protocol (MCP). This system demonstrates advanced agent-to-agent (A2A) communication, tool integration, and intelligent routing for customer support workflows.

## 🌟 Features

- **Multi-Agent Architecture**: Router, Customer Data Agent, and Support Agent working in coordination
- **MCP Protocol Integration**: Standardized tool access to customer database operations
- **Agent-to-Agent Communication**: Seamless coordination between specialized agents
- **Real-time Tool Execution**: Direct database operations through MCP server
- **Comprehensive Customer Management**: CRUD operations, ticket creation, and history tracking
- **Production-Ready**: Built with error handling, logging, and observability

## 🏗️ Architecture

```
┌─────────────────┐
│  Router Agent   │  ← Orchestrates all incoming requests
└────────┬────────┘
         │
    ┌────┴────┐
    ↓         ↓
┌─────────┐ ┌─────────┐
│Customer │ │Support  │  ← Specialized agents
│Data     │ │Agent    │
│Agent    │ │         │
└────┬────┘ └─────────┘
     │
     ↓
┌─────────────────┐
│  MCP Server     │  ← Tool provider
│  (Flask + SSE)  │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  SQLite DB      │  ← Data storage
│  (Customers +   │
│   Tickets)      │
└─────────────────┘
```

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- Google API Key for Gemini models
- ngrok authtoken (optional, for public access)

### Installation

1. **Clone the repository**
```bash
git clone <repository-url>
cd customer-service-mcp
```

2. **Install dependencies**
```bash
pip install google-adk google-generativeai flask flask-cors pyngrok termcolor
```

3. **Set up environment variables**
```python
# Set your API keys
os.environ['GOOGLE_API_KEY'] = 'your-google-api-key'
os.environ['NGROK_AUTHTOKEN'] = 'your-ngrok-token'  # Optional
```

### Running the System

#### Step 1: Start the MCP Server

Run the `mcp_server.ipynb` notebook:

```python
# This will:
# 1. Initialize the SQLite database
# 2. Start the Flask MCP server
# 3. Expose it via ngrok (optional)
# 4. Display the server URL
```

Output:
```
✅ MCP Server is running!
🔗 Local URL: http://127.0.0.1:5000
🌍 Public URL: https://your-app.ngrok-free.dev
🔍 MCP Endpoint: https://your-app.ngrok-free.dev/mcp
```

#### Step 2: Start the Agent System

Run the `a2a_team.ipynb` notebook:

```python
# Configure the MCP server URL
MCP_SERVER_URL = "http://127.0.0.1:5000/mcp"

# Agents are automatically initialized
# Start querying!
await ask_agent_team("Get customer information for ID 5")
```

## 🤖 Agent Capabilities

### Router Agent (Orchestrator)
- Analyzes user intent
- Routes requests to appropriate specialist agents
- Synthesizes final responses
- Handles clarifications

**Skills:**
- Customer query orchestration
- Multi-step workflow management
- Intent classification

### Customer Data Agent (Specialist)
- Direct access to customer database via MCP tools
- Handles all data operations

**Skills:**
- `retrieve_customer`: Fetch customer records by ID or filters
- `update_customer`: Modify customer information with validation
- `validate_customer_input`: Validate fields before updates

**Available MCP Tools:**
- `get_customer(customer_id)`: Retrieve specific customer
- `list_customers(status?)`: List all/filtered customers
- `update_customer(customer_id, name?, email?, phone?)`: Update records
- `create_ticket(customer_id, issue, priority)`: Create support tickets
- `get_customer_history(customer_id)`: Fetch ticket history

### Support Agent (Specialist)
- Handles general customer support queries
- Provides troubleshooting and guidance
- Escalates complex issues

**Skills:**
- `handle_support_query`: Respond to support questions
- `request_customer_context`: Fetch data from Customer Data Agent
- `escalate_issue`: Route to appropriate teams
- `provide_recommendation`: Suggest solutions

## 📝 Example Usage

### Example 1: Data Retrieval
```python
await ask_agent_team("Get customer information for ID 5")
```

**Response:**
```
Here is the information for customer ID 5:

**Name:** Charlie Brown
**Email:** charlie.brown@email.com
**Phone:** +1-555-0105
**Status:** active
**Created At:** 2025-11-27 23:12:44
**Updated At:** 2025-11-27 23:12:44
```

### Example 2: Support Query
```python
await ask_agent_team("I'm customer 12345 and need help upgrading my account")
```

**Agent Flow:**
1. Router → Customer Data Agent (verify customer)
2. Customer Data Agent → Support Agent (handle upgrade)
3. Support Agent → Router → User (provide guidance)

### Example 3: Complex Workflow
```python
await ask_agent_team("I've been charged twice, please refund immediately!")
```

**Response:**
```json
{
  "response": "I understand your concern about being double-charged and the urgency of a refund. 
              I'll need to access your account details to investigate this issue thoroughly 
              and process your request. Please bear with me while I look into this for you."
}
```

## 🔍 MCP Protocol Details

### Protocol Version
- **MCP Version**: 2024-11-05
- **Transport**: Server-Sent Events (SSE) over HTTP
- **Message Format**: JSON-RPC 2.0

### Available MCP Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `get_customer` | Retrieve customer by ID | `customer_id: int` |
| `list_customers` | List all/filtered customers | `status?: "active" \| "disabled"` |
| `update_customer` | Update customer info | `customer_id: int, name?: str, email?: str, phone?: str` |
| `create_ticket` | Create support ticket | `customer_id: int, issue: str, priority: str` |
| `get_customer_history` | Get ticket history | `customer_id: int` |

### MCP Server Endpoints

- **MCP Endpoint**: `POST /mcp` - Main protocol endpoint
- **Health Check**: `GET /health` - Server status
- **Protocol**: JSON-RPC 2.0 over SSE

## 📊 Event Trace Example

```
📡 FULL AGENT EVENT TRACE
──────────────────────────────────────────────────────────

[Event 1]
  author       : router_agent
  model_version: gemini-2.5-flash
  🛠 Function Call (part 1)
     name : transfer_to_agent
     args : {"agent_name": "customer_data_agent"}

[Event 2]
  author       : router_agent
  🔀 transfer_to_agent
     router_agent → customer_data_agent

[Event 3]
  author       : customer_data_agent
  🛠 Function Call (part 1)
     name : get_customer
     args : {"customer_id": 5}

[Event 4]
  📦 Function Response (part 1)
     from : get_customer
     response : {"success": true, "customer": {...}}
```

## 🛠️ Technical Implementation

### Agent Cards (A2A Protocol)

Each agent exposes an Agent Card with:
- **Capabilities**: Streaming support, input/output modes
- **Skills**: Detailed skill descriptions with examples
- **Transport**: JSON-RPC preferred protocol
- **Version**: Semantic versioning

### Key Technologies

- **Google ADK**: Agent orchestration and management
- **Gemini 2.5 Flash**: LLM for agent reasoning
- **Flask**: MCP server implementation
- **SQLite**: Customer data storage
- **ngrok**: Public tunnel (optional)
- **SSE**: Real-time event streaming

## 📈 Learnings & Challenges

### Key Learnings

1. **A2A Communication**: Gained deep understanding of structured message passing, tool-grounded actions, and state management in multi-agent systems

2. **MCP Protocol**: Learned how MCP enables standardized tool integration and how agents can coordinate through external data sources

3. **Agent Specialization**: Discovered the importance of clear role separation and how specialized agents improve system reliability

### Challenges Overcome

1. **Coordination Failures**: Debugging errors across router, specialists, and MCP required comprehensive event logging and message tracing

2. **State Management**: Ensuring consistent state across agent transfers required careful design of context passing

3. **Tool Integration**: Mapping MCP tool responses to agent actions required robust error handling and validation

4. **Observability**: Implemented detailed event traces to understand multi-step workflows and agent decision-making

## 🔐 Best Practices

1. **Error Handling**: All MCP operations wrapped in try-catch blocks
2. **Validation**: Input validation before database operations
3. **Logging**: Comprehensive event logging for debugging
4. **State Management**: Clear context passing between agents
5. **Tool Design**: Well-defined tool schemas with descriptions
