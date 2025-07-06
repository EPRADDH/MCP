# MCP
MCP-Experiments :MCP (Model Configuration Protocol) to enable seamless, standardized interaction between Large Language Models (LLMs) and external tools, APIs, databases, and services
## Table of Contents

- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Step-by-Step Usage](#step-by-step-usage)
- [Troubleshooting & Suggestions](#troubleshooting--suggestions)
- [References](#references)

---

## Project Overview

This lab enables you to:
- Use the OpenAI Agents SDK to create agents.
- Integrate MCP servers for browser automation (Playwright) and filesystem access.
- Run agents that can browse the web, fetch data, and write files.
- Trace and debug agent runs using OpenAI's trace tools.

---

## Prerequisites

- **Python 3.10+**
- **Node.js (v18+ recommended)**
- **npm and npx**
- **Docker** (optional, for containerized runs)
- **API Keys** for OpenAI, Google, HuggingFace, etc. (see `.env`)

---

## Environment Setup

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd MCP
```

### 2. Install Python Dependencies

We use [uv](https://github.com/astral-sh/uv) for fast, reproducible installs.

```bash
uv init
uv sync
```

### 3. Install Node.js and MCP Tools

Install Node.js and npm if not already installed.  
Then, install the required MCP servers globally:

```bash
npm install -g @playwright/mcp @modelcontextprotocol/server-filesystem
```

### 4. Set Up Environment Variables

Copy `.env.example` to `.env` and fill in your API keys:

```bash
cp .env.example .env
# Edit .env with your keys
```

---

## Project Structure

```
.
├── 1_lab1.ipynb         # Main Jupyter notebook for the lab
├── .env                 # Environment variables (API keys, configs)
├── pyproject.toml       # Python project dependencies
├── uv.lock              # Lock file for reproducible installs
└── sandbox/             # Directory for agent file outputs
```

---

## Configuration

The `.env` file contains all necessary API keys and configuration options.  
**Never share your `.env` file publicly.**

Example entries:
```
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...
LANGSMITH_TRACING=true
...
```

---

## Step-by-Step Usage

### 1. Launch Jupyter Notebook

```bash
jupyter notebook
```
Open `1_lab1.ipynb` in your browser.

---

### 2. Imports and Environment Loading

The notebook starts by importing required modules and loading environment variables:

```python
from dotenv import load_dotenv
from agents import Agent, Runner, trace
from agents.mcp import MCPServerStdio
import os

load_dotenv(override=True)
```

---

### 3. MCP Server Setup

- **Fetch Server:** Demonstrates fetching tools via MCP.
- **Playwright Server:** Enables browser automation.
- **Filesystem Server:** Enables file read/write in a sandboxed directory.

Example:
```python
playwright_params = {"command": "npx", "args": ["@playwright/mcp@latest"]}
files_params = {"command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", sandbox_path]}
```

---

### 4. Running the Agent

The agent is configured with instructions and connected to both the browser and filesystem MCP servers:

```python
instructions = """
You browse the internet to accomplish your instructions...
"""

async with MCPServerStdio(params=files_params, ...) as mcp_server_files:
    async with MCPServerStdio(params=playwright_params, ...) as mcp_server_browser:
        agent = Agent(
            name="investigator",
            instructions=instructions,
            model="gpt-4.1-nano",
            mcp_servers=[mcp_server_files, mcp_server_browser]
        )
        with trace("investigate") as t:
            try:
                result = await Runner.run(agent, "Find a great recipe for apple Pie, then summarize it in markdown to applepie.md")
                print(result.final_output)
            except Exception as e:
                print("❌ Exception occurred:", e)
                if hasattr(t, "record_exception"):
                    t.record_exception(e)
```

---

### 5. Tracing and Debugging

- All agent runs are traced using OpenAI's trace tools.
- If an error occurs, it is printed and (if supported) recorded in the trace for easier debugging.

Check your traces at:  
[https://platform.openai.com/traces](https://platform.openai.com/traces)

---

## Troubleshooting & Suggestions

- **API Key Errors:** Double-check your `.env` file for correct keys.
- **Node/npx Not Found:** Ensure Node.js and npm are installed and in your PATH.
- **Dependency Issues:** Use `uv sync` to ensure all Python dependencies are installed.
- **Sandbox Directory:** The `sandbox/` directory is used for file operations. Ensure it exists and is writable.
- **Docker:** For a fully reproducible environment, consider running the lab in Docker. See below for a sample Dockerfile.

### Sample Dockerfile

```Dockerfile
FROM python:3.11-slim

RUN apt-get update && \
    apt-get install -y curl && \
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs

RUN npm install -g @playwright/mcp @modelcontextprotocol/server-filesystem

COPY pyproject.toml uv.lock ./
RUN pip install uv && uv pip install -r uv.lock

COPY . /app
WORKDIR /app

CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root"]
```

---

## References

- [OpenAI Agents SDK](https://github.com/openai/openai-agents)
- [Model Context Protocol (MCP)](https://mcp.so)
- [Playwright MCP](https://www.npmjs.com/package/@playwright/mcp)
- [LangSmith Tracing](https://smithery.ai/)
- [uv Python Installer](https://github.com/astral-sh/uv)
- [OpenAI Trace Dashboard](https://platform.openai.com/traces)

---

## Suggestions

- **Keep your `.env` file secure.**
- **Use Docker for consistent, portable environments.**
- **Explore MCP marketplaces for more tools and integrations.**
- **Check the trace dashboard for debugging and optimization.**
- **Experiment with different agent instructions and tasks!**

---

Happy experimenting with MCP and OpenAI Agents!