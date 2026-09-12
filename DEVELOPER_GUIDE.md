# OpenSwarm — Developer Guide

> **What it is:** A fully open-source, production-ready multi-agent system that coordinates 8 specialized AI agents from a single terminal prompt. Built on [Agency Swarm](https://github.com/VRSEN/agency-swarm) (which sits on top of the OpenAI Agents SDK), it can produce complete deliverables — slide decks, research reports, Word docs, PDFs, data visualizations, images, and videos — that vanilla Claude Code cannot generate on its own.

---

## What This Repo Does

OpenSwarm gives you a team of 8 specialists coordinated by an orchestrator. The orchestrator never does the work itself — it routes your request to the right agent(s), potentially in parallel, then combines their outputs.

| Agent | Capability |
|---|---|
| **Orchestrator** | Pure routing: interprets intent, hands off to one or more specialists, never executes itself |
| **Virtual Assistant** | Writing, scheduling, messaging, admin tasks + 10,000+ integrations via Composio (Gmail, Slack, GitHub, HubSpot, Notion, Jira, etc.) |
| **Deep Research** | Evidence-based web research with citations, scholar search, balanced analysis |
| **Data Analyst** | Structured data analysis, KPIs, charts, statistical models — runs inside a live IPython kernel |
| **Slides Agent** | Generates polished HTML slide decks, exports to `.pptx` |
| **Docs Agent** | Creates and converts Word documents, PDFs, Markdown |
| **Image Generation Agent** | Image generation and editing via Gemini 2.5 Flash / Gemini 3 Pro and fal.ai |
| **Video Generation Agent** | Video generation via Sora (OpenAI), Veo (Google), and Seedance (fal.ai); edits and combines clips |

All agents can hand off to each other directly. For example, the Virtual Assistant hands off to Deep Research for market analysis, which can then hand off to the Slides Agent to generate a deck from its findings.

---

## Six Ways to Use It as a Developer

### 1. One-Line Install (Fastest Path)

```bash
npm install -g @vrsen/openswarm
openswarm
```

A guided wizard runs, asks for your API keys, and drops you into an interactive terminal UI backed by all 8 agents. No Python setup required — Node.js 20+ pulls Python 3.10+ automatically.

**Best for:** Getting the full swarm running in under 60 seconds to evaluate its capabilities.

---

### 2. Local Python — TUI Mode

```bash
git clone https://github.com/VRSEN/openswarm.git
cd openswarm
cp .env.example .env  # fill in your keys
python swarm.py
```

Runs the same terminal UI locally with `show_reasoning=True` — you can watch the orchestrator's routing decisions and inter-agent messages in real time. Useful for debugging communication flows or understanding how the agents coordinate.

**Required keys (at minimum one):**

```env
OPENAI_API_KEY=          # GPT-5.2, Sora video
ANTHROPIC_API_KEY=       # Claude via LiteLLM
GOOGLE_API_KEY=          # Gemini image + Veo video

# Optional
COMPOSIO_API_KEY=        # 10,000+ integrations
COMPOSIO_USER_ID=
SEARCH_API_KEY=          # web + scholar search
FAL_KEY=                 # Seedance video, background removal
```

**Model selection** is controlled by a single env var:

```env
DEFAULT_MODEL=gpt-5.2                         # OpenAI
DEFAULT_MODEL=litellm/claude-sonnet-4-6       # Anthropic
DEFAULT_MODEL=litellm/gemini/gemini-3-flash   # Google
```

---

### 3. Deploy as an API Server

```bash
python server.py   # FastAPI on localhost:8080
```

`server.py` wraps the agency in a FastAPI app via `agency_swarm.integrations.fastapi`. This exposes the swarm as an HTTP API you can call from any backend, frontend, or other service.

```python
# server.py — what it does internally
run_fastapi(
    agencies={"open-swarm": create_agency},
    port=8080,
    enable_logging=True,
    allowed_local_file_dirs=["./uploads"],
)
```

**Best for:** Embedding OpenSwarm into your own application — a SaaS product, internal tool, or automation pipeline that needs to trigger multi-agent work via REST.

---

### 4. Docker Deployment

```bash
cp .env.example .env   # fill in keys
docker-compose up --build
```

The included `Dockerfile` and `docker-compose.yml` containerize the full stack. Good for production, staging environments, or serving the API behind a reverse proxy.

---

### 5. Fork and Build Your Own Swarm

This is the repo's explicit intended use for developers. The agent folder structure is standardized:

```
my_agent/
├── __init__.py
├── my_agent.py         # Agent class, model config
├── instructions.md     # System prompt
└── tools/
    ├── ToolOne.py      # Each tool = one BaseTool subclass
    └── ToolTwo.py
```

You can customize by:

- **Replacing agents** — swap out the Slides Agent for, say, an SEO Writer Agent
- **Adding tools** — each tool is a standalone `BaseTool` subclass with a `run()` method
- **Adding MCP servers** — plug any MCP server into an agent via `mcp_servers=[MCPServerStdio(...)]` without writing custom tool code
- **Changing the model per-agent** — each agent reads `DEFAULT_MODEL` but you can override it
- **Updating communication flows** in `swarm.py` — orchestrator→specialist or peer-to-peer handoffs

The `CLAUDE.md` / `AGENTS.md` files are written specifically so that Claude Code, Cursor, or Codex can read them and autonomously reshape the swarm into a domain-specific agency from a single prompt:

> _"Turn this into an SEO optimization swarm"_ → Claude Code reads the rules, creates new agents, writes tools, rewires communication flows.

**Popular patterns to fork into:**

| Swarm | Agents |
|---|---|
| SEO Swarm | Keyword Researcher + Competitor Analyst + Blog Writer |
| Sales Swarm | Lead Researcher + Outreach Writer + Proposal Generator |
| Marketing Swarm | Campaign Planner + Creative Asset Generator + Analytics |
| Dev Ops Swarm | Code Reviewer + Test Generator + Deployment Manager |

---

### 6. Use It as a Meta-Agent Builder (via Claude Code / Cursor)

The repo contains `.cursor/rules/agency-swarm-workflow.mdc` and detailed `CLAUDE.md` instructions that turn any AI coding assistant into a specialized agency builder. If you open this repo in Claude Code or Cursor and say:

> _"Build me a customer support swarm with a ticket classifier, knowledge base lookup agent, and escalation agent"_

The AI reads the workflow rules and:
1. Creates each agent folder with correct structure
2. Writes `instructions.md` for each agent
3. Implements `BaseTool` classes for each tool
4. Wires up `swarm.py` with the right communication flows
5. Tests each tool individually

---

## How It Compares to Vanilla Claude Code

| Capability | Vanilla Claude Code | OpenSwarm |
|---|---|---|
| **Concurrency** | Single agent, one task at a time | Parallel specialists — research + slides + data analysis run simultaneously |
| **Deliverable types** | Code files, text | `.pptx`, `.docx`, `.pdf`, charts, images, videos — actual binary artifacts |
| **Context isolation** | One shared context window | Each specialist has its own context; large outputs don't pollute the main thread |
| **External integrations** | MCP servers configured manually | Composio built-in: 10,000+ services with OAuth handled by the framework |
| **LLM provider** | Anthropic only | OpenAI, Anthropic, Google Gemini — switchable per model env var |
| **Data analysis** | Generates code for you to run | Runs code in a live IPython kernel, returns actual computed results and chart files |
| **Web research** | No built-in tool (needs MCP) | `WebSearchTool` + `ScholarSearch` built into Deep Research Agent |
| **Deployment** | CLI only | CLI, FastAPI server, Docker |
| **Customizability** | CLAUDE.md for context | Full agent code: instructions, tools, communication flows all forkable |
| **Agent specialization** | One generalist handles everything | 8 specialists — each with domain-specific instructions and tools optimized for one job |
| **Cost control** | No — one big context for everything | Task routing means each specialist only sees the slice it needs |

### When to use Claude Code instead

- Writing, reviewing, or refactoring code — Claude Code's native purpose
- Quick one-shot questions or searches
- Tasks that don't produce binary file artifacts

### When OpenSwarm wins

- You need an actual `.pptx` file, not markdown describing one
- You want parallel execution (research happening while a doc is being drafted)
- The task spans multiple domains (research → analysis → presentation)
- You want to connect to Gmail, Slack, GitHub, etc. without manual MCP setup
- You want to embed multi-agent capability in your own application via REST API

---

## Quick Decision Tree

```
Do you need a binary artifact (slide deck, video, PDF)?
├── Yes → OpenSwarm
└── No → Does it span multiple domains (e.g. research + writing + data)?
    ├── Yes → OpenSwarm (parallel delegation)
    └── No → Does it need an external integration (Gmail, Slack, GitHub)?
        ├── Yes → OpenSwarm (Composio)
        └── No → Claude Code is simpler and sufficient
```

---

## Key Files to Read First

| File | Purpose |
|---|---|
| `swarm.py` | Agency definition — agents, communication flows, TUI entry point |
| `shared_instructions.md` | Shared system prompt injected into every agent |
| `config.py` | LiteLLM model routing — how provider switching works |
| `orchestrator/instructions.md` | Routing logic: when to use `SendMessage` (parallel) vs `Handoff` (single-agent) |
| `.env.example` | All available API keys with descriptions |
| `.cursor/rules/agency-swarm-workflow.mdc` | The full guide for building new agents and tools |
| `server.py` | FastAPI wrapper for REST deployment |
