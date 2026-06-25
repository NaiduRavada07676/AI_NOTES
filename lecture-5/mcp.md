# Model Context Protocol (MCP) - Detailed Notes

## What is MCP?

**Model Context Protocol (MCP)** is an open standard (created by Anthropic) that defines how AI models/agents connect to external data sources and tools. Think of it as a **universal plug** that lets any AI model talk to any tool or data source in a standardized way.

---

## Analogy: The USB Standard

```
┌─────────────────────────────────────────────────────────────┐
│                    USB ANALOGY FOR MCP                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   BEFORE USB (Before MCP):                                   │
│   ─────────────────────────                                  │
│   Every device had its own connector:                        │
│   - Printer → Parallel port                                  │
│   - Mouse → PS/2 port                                        │
│   - Camera → Proprietary cable                               │
│   - Phone → Brand-specific charger                           │
│                                                              │
│   AFTER USB (After MCP):                                     │
│   ────────────────────────                                   │
│   ONE standard connector for everything:                     │
│   - Printer → USB                                            │
│   - Mouse → USB                                              │
│   - Camera → USB                                             │
│   - Phone → USB                                              │
│                                                              │
│   ═══════════════════════════════════════                     │
│   MCP = USB for AI tools & data sources                      │
│   ═══════════════════════════════════════                     │
│                                                              │
│   Before MCP: Every AI app had custom integrations           │
│   After MCP:  One protocol to connect any AI to any tool     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## The Problem MCP Solves

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   WITHOUT MCP (N × M Problem):                                   │
│   ─────────────────────────────                                  │
│                                                                  │
│   AI Apps          Tools/Data Sources                            │
│   ────────         ─────────────────                             │
│   ChatGPT    ──┬── GitHub                                        │
│              ──┼── Slack                                          │
│              ──┼── Database                                       │
│   Claude     ──┼── Google Drive                                   │
│              ──┼── Jira                                           │
│              ──┘                                                  │
│   Cursor     ──┬── (Each needs custom integration!)              │
│              ──┘                                                  │
│                                                                  │
│   = N apps × M tools = N×M custom integrations 😱                │
│                                                                  │
│   ─────────────────────────────────────────────────────────────  │
│                                                                  │
│   WITH MCP (N + M Problem):                                      │
│   ─────────────────────────                                      │
│                                                                  │
│   AI Apps              MCP              Tools/Data               │
│   ────────         ┌─────────┐         ───────────              │
│   ChatGPT   ──────▶│         │◀────── GitHub                    │
│   Claude    ──────▶│   MCP   │◀────── Slack                     │
│   Cursor    ──────▶│Protocol │◀────── Database                  │
│   Kiro      ──────▶│         │◀────── Google Drive              │
│                     └─────────┘                                  │
│                                                                  │
│   = N + M integrations (each side implements once!) 🎉           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## MCP Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MCP ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐         ┌────────────────┐                       │
│  │   MCP HOST     │         │  MCP SERVER    │                       │
│  │  (AI App)      │         │  (Tool/Data)   │                       │
│  │                │         │                │                       │
│  │  ┌──────────┐  │  JSON   │  ┌──────────┐  │                       │
│  │  │  MCP     │◀─┼─────────┼─▶│  MCP     │  │                       │
│  │  │  CLIENT  │  │  RPC    │  │  SERVER  │  │                       │
│  │  └──────────┘  │         │  └──────────┘  │                       │
│  │                │         │                │                       │
│  │  Examples:     │         │  Examples:     │                       │
│  │  - Claude      │         │  - File system │                       │
│  │  - Cursor      │         │  - GitHub API  │                       │
│  │  - Kiro        │         │  - Database    │                       │
│  │  - VS Code     │         │  - Slack       │                       │
│  └────────────────┘         └────────────────┘                       │
│                                                                      │
│  Communication: JSON-RPC 2.0 over stdio or HTTP (SSE)                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Three Core Primitives of MCP

```
┌─────────────────────────────────────────────────────────────────┐
│                  MCP CORE PRIMITIVES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  1. TOOLS  🔧                                               │ │
│  │  ─────────────                                              │ │
│  │  Actions the AI can perform                                 │ │
│  │  (Model-controlled — AI decides when to call)               │ │
│  │                                                             │ │
│  │  Examples:                                                  │ │
│  │  - search_web(query)                                        │ │
│  │  - create_github_issue(title, body)                         │ │
│  │  - query_database(sql)                                      │ │
│  │  - send_email(to, subject, body)                            │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  2. RESOURCES  📄                                           │ │
│  │  ────────────────                                           │ │
│  │  Data the AI can read (like GET endpoints)                  │ │
│  │  (Application-controlled — app decides when to fetch)       │ │
│  │                                                             │ │
│  │  Examples:                                                  │ │
│  │  - file://project/src/main.py                               │ │
│  │  - db://users/schema                                        │ │
│  │  - git://repo/log                                           │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  3. PROMPTS  💬                                             │ │
│  │  ─────────────                                              │ │
│  │  Pre-built prompt templates for common tasks                │ │
│  │  (User-controlled — user selects which prompt to use)       │ │
│  │                                                             │ │
│  │  Examples:                                                  │ │
│  │  - "Summarize this codebase"                                │ │
│  │  - "Review this PR"                                         │ │
│  │  - "Explain this error"                                     │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## How MCP Communication Works

```
┌─────────────────────────────────────────────────────────────────┐
│              MCP COMMUNICATION FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   HOST (Claude/Kiro)              SERVER (GitHub MCP)            │
│   ──────────────────              ───────────────────            │
│         │                                │                       │
│         │  1. Initialize Connection      │                       │
│         │───────────────────────────────▶│                       │
│         │                                │                       │
│         │  2. Server declares capabilities│                      │
│         │◀───────────────────────────────│                       │
│         │    "I have these tools:        │                       │
│         │     - create_issue             │                       │
│         │     - list_repos               │                       │
│         │     - search_code"             │                       │
│         │                                │                       │
│         │  3. Client calls a tool        │                       │
│         │───────────────────────────────▶│                       │
│         │    {tool: "create_issue",      │                       │
│         │     args: {title: "Bug fix"}}  │                       │
│         │                                │                       │
│         │  4. Server returns result      │                       │
│         │◀───────────────────────────────│                       │
│         │    {result: "Issue #42 created"}│                      │
│         │                                │                       │
│         ▼                                ▼                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Transport Mechanisms

```
┌──────────────────────────────────────────────────────────────┐
│                 MCP TRANSPORT OPTIONS                          │
├────────────────┬─────────────────────────────────────────────┤
│   Transport    │   Description                                │
├────────────────┼─────────────────────────────────────────────┤
│   stdio        │   Local process communication                │
│                │   Server runs as child process               │
│                │   Best for: local tools, CLI tools           │
│                │   Example: File system, local DB             │
├────────────────┼─────────────────────────────────────────────┤
│   HTTP + SSE   │   Remote server communication               │
│   (Streamable) │   Server runs on a remote machine           │
│                │   Best for: cloud services, shared tools    │
│                │   Example: GitHub API, Slack                 │
└────────────────┴─────────────────────────────────────────────┘
```

---

## MCP Configuration Example

```json
{
  "mcpServers": {
    "github": {
      "command": "uvx",
      "args": ["mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token-here"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
    },
    "postgres": {
      "command": "uvx",
      "args": ["mcp-server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

---

## Building a Simple MCP Server (Python)

```python
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP("my-tools")

# Define a tool
@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b

# Define a resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting."""
    return f"Hello, {name}! Welcome to MCP."

# Run the server
if __name__ == "__main__":
    mcp.run()
```

---

## MCP vs Function Calling vs API

```
┌──────────────────────────────────────────────────────────────────┐
│              COMPARISON: MCP vs Function Calling vs REST API       │
├────────────────┬──────────────────┬──────────────┬───────────────┤
│   Aspect       │   MCP            │  Func Call   │  REST API     │
├────────────────┼──────────────────┼──────────────┼───────────────┤
│ Standard       │ Open protocol    │ Vendor-locked│ HTTP standard │
│ Discovery      │ Auto-discovery   │ Pre-defined  │ Docs needed   │
│ Who uses it    │ AI apps          │ Single LLM   │ Any app       │
│ Portability    │ Any AI client    │ One provider │ Universal     │
│ Designed for   │ AI-tool connect  │ LLM actions  │ App-to-app    │
│ Bidirectional  │ Yes              │ No           │ No            │
└────────────────┴──────────────────┴──────────────┴───────────────┘
```

---

## The MCP Ecosystem

```
┌─────────────────────────────────────────────────────────────────┐
│                     MCP ECOSYSTEM                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   HOSTS (Clients)          SERVERS (Tools/Data)                  │
│   ────────────────         ────────────────────                  │
│                                                                  │
│   🖥️  Claude Desktop       📁  Filesystem                       │
│   🖥️  Cursor               🐙  GitHub                           │
│   🖥️  Kiro                 💬  Slack                             │
│   🖥️  Windsurf             🗄️  PostgreSQL                       │
│   🖥️  VS Code (Copilot)   📊  Google Sheets                    │
│   🖥️  Cline               🌐  Brave Search                     │
│   🖥️  Continue            🐳  Docker                            │
│   🖥️  Zed                 ☁️  AWS                               │
│                            📧  Gmail                             │
│                            📝  Notion                            │
│                                                                  │
│   Any host can connect to any server using MCP! 🔌               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────┐
│                    KEY TAKEAWAYS                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. MCP is the "USB-C for AI" — a universal connector        │
│     between AI apps and tools/data sources                   │
│                                                              │
│  2. It solves the N×M integration problem by providing       │
│     a single standard protocol                               │
│                                                              │
│  3. Three primitives: Tools (actions), Resources (data),     │
│     Prompts (templates)                                      │
│                                                              │
│  4. Servers expose capabilities, Clients (hosts) consume     │
│     them — both sides implement once                         │
│                                                              │
│  5. Transport options: stdio (local) or HTTP+SSE (remote)    │
│                                                              │
│  6. Growing ecosystem — 1000s of MCP servers already exist   │
│                                                              │
│  7. Open standard — not locked to any vendor                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Further Reading

- [MCP Official Docs](https://modelcontextprotocol.io/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [MCP Servers Repository](https://github.com/modelcontextprotocol/servers)
- [Building MCP Servers Guide](https://modelcontextprotocol.io/quickstart/server)
