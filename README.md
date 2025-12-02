# Multi-Agent Customer Service System  
### (LangGraph + MCP + Agent-to-Agent Communication)

This project implements a **multi-agent customer service workflow** using:

- **LangGraph** for orchestrating multi-step, stateful agent interactions  
- **MCP (Model Context Protocol)** for tool-based database access  
- **Specialized AI agents** (Router, Customer Data Agent, Support Agent)  
- **Robust JSON-RPC over SSE**, ngrok tunneling, and safe A2A routing  

The system demonstrates real-world **multi-agent coordination**, **tool-grounded reasoning**, and **safe state management** in a production-oriented architecture.

---

# 🔧 Agent Overview

## **Router Agent (Orchestrator)**  
- Receives customer queries  
- Analyzes query intent  
- Routes tasks to appropriate specialist agents  
- Coordinates multi-step workflows  

## **Customer Data Agent (Specialist)**  
- Accesses customer database via MCP  
- Retrieves customer information  
- Updates customer records  
- Performs field validation  

## **Support Agent (Specialist)**  
- Handles general customer support queries  
- Can escalate complex issues  
- Requests customer context  
- Generates user-facing recommendations and solutions  

---

# 🐍 Python Environment Setup

This project requires **Python 3.10**.

## 1. Create Virtual Environment

python -m venv .venv

## 2. Activate Environment

### macOS/Linux:

source .venv/bin/activate

### Windows:

.venv\Scripts\activate

## 3. Install Dependencies

pip install -r requirements.txt

---

# 🌐 Start the MCP Server

Run the MCP server notebook:

mcp_server.ipynb

This launches your MCP server at:

http://127.0.0.1:5000/mcp

---

# 🤖 Running the Multi-Agent A2A System

The file **`a2a.ipynb`**:

- Builds the LangGraph multi-agent system  
- Connects automatically to the MCP server (ngrok or local)  
- Implements Router → Customer Data Agent → Support Agent → Router cycles  
- Allows you to run queries end-to-end with ask_query()

To run:

python a2a.ipynb

You will see:

- Final user-facing response  
- Full A2A trace showing agent-to-agent communication  
- MCP tool interactions  
- Scenario classification and routing steps  

---

# 📘 Usage Examples

## Scenario 1: Task Allocation  
**Query:**  
“I need help with my account, customer ID 12345”

**A2A Flow:**
1. Router receives query  
2. Router → Customer Data Agent: “Get customer info for ID 12345”  
3. Customer Data Agent returns MCP data  
4. Router analyzes customer tier/status  
5. Router → Support Agent for response generation  
6. Support Agent generates final message  
7. Router returns consolidated answer  

---

## Scenario 2: Negotiation & Escalation  
**Query:**  
“I want to cancel my subscription but I’m having billing issues”

**A2A Flow:**
1. Router detects multiple intents  
2. Router → Support Agent: initial handling  
3. Support Agent → Router: “I need billing context”  
4. Router → Customer Data Agent  
5. Customer Data Agent returns billing info  
6. Router coordinates the final response  

---

## Scenario 3: Multi-Step Coordination  
**Query:**  
“What’s the status of all high-priority tickets for premium customers?”

**A2A Flow:**
1. Router decomposes query  
2. Router → Customer Data Agent: fetch premium customers  
3. Customer Data Agent returns list  
4. Router → Support Agent: fetch high-priority tickets  
5. Support Agent queries MCP  
6. Agents coordinate to format a summary report  
7. Router synthesizes final answer  

---
