# Multi-Agent Systems & CrewAI - Detailed Notes

## What is a Multi-Agent System?

A **Multi-Agent System** is a setup where multiple specialized AI agents collaborate to accomplish a complex task — each agent has its own role, expertise, and tools, and they work together like a team.

---

## Analogy: A Film Production Crew

```
┌─────────────────────────────────────────────────────────────────┐
│                 FILM CREW ANALOGY                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Making a film requires multiple specialists:                   │
│                                                                  │
│   🎬 Director         = Orchestrator / Manager Agent             │
│   📝 Screenwriter     = Research & Writing Agent                 │
│   🎥 Cinematographer  = Data Collection Agent                    │
│   🎨 Editor           = Summarization & Output Agent             │
│   🎵 Sound Designer   = Quality Assurance Agent                  │
│                                                                  │
│   No single person does everything.                              │
│   Each specialist focuses on what they do best.                  │
│   They pass work between each other in a defined workflow.       │
│                                                                  │
│   ═══════════════════════════════════════════════                 │
│   Multi-Agent System = A crew of specialized AI agents           │
│   each handling their part of a complex task                     │
│   ═══════════════════════════════════════════════                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why Multi-Agent Over Single Agent?

```
┌──────────────────────────────────────────────────────────────────┐
│           SINGLE AGENT vs MULTI-AGENT                              │
├──────────────────────────┬───────────────────────────────────────┤
│   SINGLE AGENT           │   MULTI-AGENT SYSTEM                   │
├──────────────────────────┼───────────────────────────────────────┤
│ One agent does everything│ Specialists for each sub-task          │
│ Gets confused on complex │ Each agent stays focused               │
│ tasks                    │                                        │
│ Context window overload  │ Smaller context per agent              │
│ Hard to debug            │ Easier to debug (isolate agents)       │
│ Single point of failure  │ Fault-tolerant                         │
│ Limited expertise        │ Deep expertise per role                │
│ Simple to set up         │ More complex but more capable          │
└──────────────────────────┴───────────────────────────────────────┘

When to use Multi-Agent:
✅ Complex tasks with distinct phases (research → write → review)
✅ Tasks requiring different expertise areas
✅ Workflows that benefit from checks and balances
✅ When you need higher quality output through collaboration
```

---

## What is CrewAI?

**CrewAI** is a Python framework for building multi-agent systems. It provides:

- Role-based agent design
- Task delegation and sequencing
- Inter-agent communication
- Tool integration per agent
- Multiple process types (sequential, hierarchical)

---

## CrewAI Core Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CrewAI BUILDING BLOCKS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐    │
│   │  🧑 AGENT                                                    │    │
│   │  ───────                                                     │    │
│   │  A specialized team member with:                             │    │
│   │  • Role — what it does (e.g., "Senior Researcher")           │    │
│   │  • Goal — what it aims to achieve                            │    │
│   │  • Backstory — context that shapes behavior                  │    │
│   │  • Tools — what tools it can use                             │    │
│   │  • LLM — which model powers it                               │    │
│   └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐    │
│   │  📋 TASK                                                     │    │
│   │  ──────                                                      │    │
│   │  A specific piece of work to be done:                        │    │
│   │  • Description — what needs to be done                       │    │
│   │  • Expected Output — what the result should look like        │    │
│   │  • Agent — who is responsible                                │    │
│   │  • Context — input from other tasks                          │    │
│   └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│   ┌─────────────────────────────────────────────────────────────┐    │
│   │  👥 CREW                                                     │    │
│   │  ──────                                                      │    │
│   │  The team that brings it all together:                       │    │
│   │  • Agents — list of agents in the crew                       │    │
│   │  • Tasks — list of tasks to execute                          │    │
│   │  • Process — how tasks are executed (sequential/hierarchical)│    │
│   │  • Manager — optional manager agent for hierarchical process │    │
│   └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## CrewAI Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CrewAI EXECUTION FLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                        ┌──────────────┐                              │
│                        │   USER INPUT  │                              │
│                        └──────┬───────┘                              │
│                               │                                      │
│                               ▼                                      │
│                    ┌─────────────────────┐                           │
│                    │      👥 CREW         │                           │
│                    │   (Orchestrator)     │                           │
│                    └──────────┬──────────┘                           │
│                               │                                      │
│              ┌────────────────┼────────────────┐                     │
│              │                │                │                      │
│              ▼                ▼                ▼                      │
│     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│     │  🧑 Agent 1   │ │  🧑 Agent 2   │ │  🧑 Agent 3   │              │
│     │  Researcher  │ │  Writer      │ │  Reviewer    │              │
│     │              │ │              │ │              │              │
│     │  Tools:      │ │  Tools:      │ │  Tools:      │              │
│     │  - Search    │ │  - None      │ │  - Lint      │              │
│     │  - Scrape    │ │              │ │  - Validate  │              │
│     └──────┬───────┘ └──────┬───────┘ └──────┬───────┘              │
│            │                │                │                      │
│            ▼                ▼                ▼                      │
│     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│     │  📋 Task 1    │ │  📋 Task 2    │ │  📋 Task 3    │              │
│     │  "Research   │ │  "Write a    │ │  "Review &   │              │
│     │   the topic" │ │   blog post" │ │   improve"   │              │
│     └──────┬───────┘ └──────┬───────┘ └──────┬───────┘              │
│            │                │                │                      │
│            └────────────────┼────────────────┘                      │
│                             │                                        │
│                             ▼                                        │
│                    ┌─────────────────┐                               │
│                    │  FINAL OUTPUT   │                               │
│                    └─────────────────┘                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Process Types in CrewAI

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROCESS TYPES                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. SEQUENTIAL (Default)                                         │
│  ────────────────────────                                        │
│                                                                  │
│  Task 1 ────▶ Task 2 ────▶ Task 3 ────▶ Output                  │
│  (Agent A)    (Agent B)    (Agent C)                             │
│                                                                  │
│  • Each task runs one after another                              │
│  • Output of one task feeds into the next                        │
│  • Simple, predictable flow                                      │
│  • Best for: linear workflows (research → write → edit)          │
│                                                                  │
│  ─────────────────────────────────────────────────────────────── │
│                                                                  │
│  2. HIERARCHICAL                                                 │
│  ────────────────                                                │
│                                                                  │
│              ┌──────────────┐                                    │
│              │   MANAGER    │                                    │
│              │   AGENT      │                                    │
│              └──────┬───────┘                                    │
│                     │  delegates & coordinates                   │
│          ┌──────────┼──────────┐                                 │
│          ▼          ▼          ▼                                  │
│     ┌────────┐ ┌────────┐ ┌────────┐                             │
│     │Agent A │ │Agent B │ │Agent C │                             │
│     └────────┘ └────────┘ └────────┘                             │
│                                                                  │
│  • A manager agent delegates tasks dynamically                   │
│  • Manager decides who does what and when                        │
│  • More flexible, handles complex workflows                      │
│  • Best for: complex projects needing coordination               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Code Example: Blog Writing Crew

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool

# ──────────────────────────────────────
# Step 1: Define Tools
# ──────────────────────────────────────
search_tool = SerperDevTool()

# ──────────────────────────────────────
# Step 2: Define Agents
# ──────────────────────────────────────
researcher = Agent(
    role="Senior Research Analyst",
    goal="Find comprehensive and accurate information about {topic}",
    backstory="""You are an expert researcher with years of experience
    in finding relevant, accurate, and up-to-date information.
    You excel at synthesizing complex topics into clear findings.""",
    tools=[search_tool],
    verbose=True
)

writer = Agent(
    role="Content Writer",
    goal="Write an engaging and informative blog post about {topic}",
    backstory="""You are a skilled content writer known for creating
    engaging, well-structured articles that make complex topics
    accessible to a general audience.""",
    verbose=True
)

editor = Agent(
    role="Editor & Quality Reviewer",
    goal="Review and polish the blog post for clarity, accuracy, and engagement",
    backstory="""You are a meticulous editor with an eye for detail.
    You ensure content is grammatically correct, factually accurate,
    well-structured, and engaging for readers.""",
    verbose=True
)

# ──────────────────────────────────────
# Step 3: Define Tasks
# ──────────────────────────────────────
research_task = Task(
    description="""Research the topic '{topic}' thoroughly.
    Find key facts, statistics, recent developments, and expert opinions.
    Provide a structured summary of your findings.""",
    expected_output="A detailed research summary with key points, stats, and sources",
    agent=researcher
)

writing_task = Task(
    description="""Using the research provided, write a compelling blog post
    about '{topic}'. Include an attention-grabbing intro, well-organized body
    with subheadings, and a strong conclusion.""",
    expected_output="A well-written blog post of 800-1000 words in markdown format",
    agent=writer,
    context=[research_task]  # Gets output from research_task
)

editing_task = Task(
    description="""Review and edit the blog post for:
    - Grammar and spelling errors
    - Clarity and readability
    - Factual accuracy
    - Engaging tone
    - Proper structure""",
    expected_output="A polished, publication-ready blog post",
    agent=editor,
    context=[writing_task]  # Gets output from writing_task
)

# ──────────────────────────────────────
# Step 4: Create the Crew
# ──────────────────────────────────────
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, writing_task, editing_task],
    process=Process.sequential,  # Tasks run one after another
    verbose=True
)

# ──────────────────────────────────────
# Step 5: Execute!
# ──────────────────────────────────────
result = crew.kickoff(inputs={"topic": "AI Agents in 2025"})
print(result)
```

---

## How Data Flows Between Agents

```
┌─────────────────────────────────────────────────────────────────┐
│                   DATA FLOW IN SEQUENTIAL PROCESS                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────┐                                          │
│  │  👨‍🔬 RESEARCHER      │                                          │
│  │                    │                                          │
│  │  Input: "AI Agents"│                                          │
│  │                    │                                          │
│  │  Uses: search_tool │                                          │
│  │                    │                                          │
│  │  Output:           │                                          │
│  │  "Key findings:    │                                          │
│  │   1. Agent market  │                                          │
│  │      growing 40%   │                                          │
│  │   2. ReAct is the  │                                          │
│  │      main pattern" │                                          │
│  └─────────┬──────────┘                                          │
│            │ context=[research_task]                              │
│            ▼                                                     │
│  ┌────────────────────┐                                          │
│  │  ✍️  WRITER          │                                          │
│  │                    │                                          │
│  │  Input: Research   │                                          │
│  │  findings above    │                                          │
│  │                    │                                          │
│  │  Output:           │                                          │
│  │  "# AI Agents:     │                                          │
│  │   The Future...    │                                          │
│  │   [full blog post]"│                                          │
│  └─────────┬──────────┘                                          │
│            │ context=[writing_task]                               │
│            ▼                                                     │
│  ┌────────────────────┐                                          │
│  │  📝 EDITOR          │                                          │
│  │                    │                                          │
│  │  Input: Blog post  │                                          │
│  │  draft above       │                                          │
│  │                    │                                          │
│  │  Output:           │                                          │
│  │  "# AI Agents:     │                                          │
│  │   The Future...    │                                          │
│  │   [polished post]" │                                          │
│  └────────────────────┘                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## CrewAI Tools

```
┌──────────────────────────────────────────────────────────────────┐
│                    POPULAR CrewAI TOOLS                            │
├──────────────────┬───────────────────────────────────────────────┤
│   Tool           │   What it does                                 │
├──────────────────┼───────────────────────────────────────────────┤
│ SerperDevTool    │ Google search via Serper API                   │
│ ScrapeWebsite    │ Scrape content from URLs                      │
│ FileReadTool     │ Read local files                              │
│ DirectoryRead    │ Read directory contents                       │
│ CodeInterpreter  │ Execute Python code                           │
│ PDFSearchTool   │ Search within PDF documents                    │
│ YoutubeSearch    │ Search YouTube videos                         │
│ GithubSearchTool │ Search GitHub repositories                    │
│ Custom Tools     │ Build your own with @tool decorator           │
└──────────────────┴───────────────────────────────────────────────┘
```

### Creating Custom Tools

```python
from crewai.tools import tool

@tool("Calculate Profit")
def calculate_profit(revenue: float, cost: float) -> str:
    """Calculate profit given revenue and cost."""
    profit = revenue - cost
    margin = (profit / revenue) * 100
    return f"Profit: ${profit:.2f}, Margin: {margin:.1f}%"
```

---

## Multi-Agent Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                MULTI-AGENT COLLABORATION PATTERNS                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PIPELINE (Sequential)                                        │
│  ─────────────────────────                                       │
│     A ────▶ B ────▶ C ────▶ Output                               │
│     Good for: Content creation, data processing                  │
│                                                                  │
│  2. DEBATE / ADVERSARIAL                                         │
│  ────────────────────────                                        │
│     A ◀────▶ B                                                   │
│     │        │                                                   │
│     └───┬────┘                                                   │
│         ▼                                                        │
│      JUDGE (C)                                                   │
│     Good for: Decision making, finding flaws                     │
│                                                                  │
│  3. SUPERVISOR / HIERARCHICAL                                    │
│  ──────────────────────────────                                  │
│         MANAGER                                                  │
│        /   |   \                                                 │
│       A    B    C                                                │
│     Good for: Complex projects, dynamic delegation               │
│                                                                  │
│  4. PEER COLLABORATION                                           │
│  ──────────────────────                                          │
│     A ◀──▶ B ◀──▶ C                                              │
│     Good for: Brainstorming, iterative refinement                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real-World Multi-Agent Examples

### 1. Content Marketing Crew

```
Researcher → Writer → SEO Optimizer → Editor → Publisher
```

### 2. Software Development Crew

```
Product Manager → Developer → Code Reviewer → Tester → DevOps
```

### 3. Financial Analysis Crew

```
Data Collector → Analyst → Risk Assessor → Report Writer
```

### 4. Customer Support Crew

```
Classifier → Specialist Agent → Quality Checker → Responder
```

---

## CrewAI vs Other Multi-Agent Frameworks

```
┌──────────────────────────────────────────────────────────────────┐
│           MULTI-AGENT FRAMEWORK COMPARISON                        │
├────────────────┬────────────────┬────────────────┬───────────────┤
│   Feature      │   CrewAI       │   AutoGen      │  LangGraph    │
├────────────────┼────────────────┼────────────────┼───────────────┤
│ Ease of use    │ ⭐⭐⭐⭐⭐        │ ⭐⭐⭐           │ ⭐⭐⭐          │
│ Role-based     │ ✅ Built-in    │ ❌ Manual      │ ❌ Manual     │
│ Process types  │ Seq + Hier     │ Custom         │ Graph-based   │
│ Tool support   │ Rich ecosystem │ Good           │ Excellent     │
│ Learning curve │ Low            │ Medium         │ High          │
│ Flexibility    │ Medium         │ High           │ Very High     │
│ Best for       │ Role-based     │ Conversations  │ Complex flows │
│                │ workflows      │ between agents │ & state mgmt  │
└────────────────┴────────────────┴────────────────┴───────────────┘
```

---

## Getting Started with CrewAI

```bash
# Installation
pip install crewai crewai-tools

# Create a new project
crewai create crew my-project

# Project structure created:
# my-project/
# ├── src/
# │   └── my_project/
# │       ├── config/
# │       │   ├── agents.yaml    # Agent definitions
# │       │   └── tasks.yaml     # Task definitions
# │       ├── crew.py            # Crew assembly
# │       ├── main.py            # Entry point
# │       └── tools/
# │           └── custom_tool.py # Custom tools
# ├── pyproject.toml
# └── README.md

# Run the crew
crewai run
```

---

## YAML Configuration (CrewAI Way)

### agents.yaml

```yaml
researcher:
  role: "Senior Research Analyst"
  goal: "Find accurate and comprehensive information about {topic}"
  backstory: "You are an expert researcher with 10+ years of experience..."
  tools:
    - search_tool

writer:
  role: "Content Writer"
  goal: "Create engaging content based on research findings"
  backstory: "You are a skilled writer known for clear, engaging articles..."
```

### tasks.yaml

```yaml
research_task:
  description: "Research {topic} thoroughly and provide key findings"
  expected_output: "Structured research summary with sources"
  agent: researcher

writing_task:
  description: "Write a blog post using the research findings"
  expected_output: "800-word blog post in markdown"
  agent: writer
  context:
    - research_task
```

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│                      KEY TAKEAWAYS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Multi-agent systems = team of specialized AI agents          │
│     collaborating on complex tasks                               │
│                                                                  │
│  2. CrewAI makes it easy with role-based agents, tasks,          │
│     and crews — like assembling a real team                      │
│                                                                  │
│  3. Two process types:                                           │
│     - Sequential: A → B → C (predictable pipeline)              │
│     - Hierarchical: Manager delegates dynamically                │
│                                                                  │
│  4. Each agent can have its own tools, role, goal, and backstory │
│                                                                  │
│  5. Tasks flow data between agents via the 'context' parameter   │
│                                                                  │
│  6. Best for: content pipelines, research workflows,             │
│     development teams, analysis workflows                        │
│                                                                  │
│  7. The future of AI is not a single powerful agent —            │
│     it's a CREW of specialized agents working together! 🚀       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Further Reading

- [CrewAI Documentation](https://docs.crewai.com/)
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [Multi-Agent Patterns Paper](https://arxiv.org/abs/2402.01680)
- [AutoGen Docs](https://microsoft.github.io/autogen/)
- [LangGraph Multi-Agent](https://langchain-ai.github.io/langgraph/)
