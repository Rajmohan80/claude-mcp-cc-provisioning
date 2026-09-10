# WxCC MCP Showcase — AI-Driven Contact Center Provisioning

> **Claude → ACP → Cisco Webex Contact Center**  
> From customer business requirements to a live IVR in minutes, using only natural language and MCP tool calls.

---

## What This Is

This repository documents a working proof-of-concept where **Claude** (Anthropic's AI assistant) acts as an intelligent orchestrator that:

1. Translates raw customer business requirements into a structured Excel design workbook
2. Uses that approved workbook as the source of truth
3. Provisions a live Cisco Webex Contact Center (WxCC) environment entirely through MCP tool calls — no Control Hub clicks

The only manual step is publishing the IVR flow in WxCC Flow Designer (a current sandbox constraint).

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        OPERATOR'S LAPTOP                            │
│                                                                     │
│   ┌───────────────────┐        ┌──────────────────────────────┐    │
│   │   Claude Desktop  │◄──────►│   ACP MCP Server (port 8100) │    │
│   │   (MCP Client)    │  MCP   │   FastMCP 3.4.5              │    │
│   │                   │  JSON  │   Python · FastAPI           │    │
│   │  Natural language │  over  │   TinyDB audit log           │    │
│   │  ↕                │  STDIO │   RBAC (control_hub.yaml)    │    │
│   │  Claude Sonnet    │        └──────────────────────────────┘    │
│   │  (claude.ai LLM)  │                    │                        │
│   └───────────────────┘                    │ HTTPS REST             │
│                                            │                        │
└────────────────────────────────────────────┼───────────────────────┘
                                             │
                                             ▼
                              ┌──────────────────────────┐
                              │   Cisco Webex CC Cloud   │
                              │   api.wxcc-us1.cisco.com │
                              │                          │
                              │  • Queues                │
                              │  • Agents                │
                              │  • Routing strategies    │
                              │  • Flow import           │
                              │  • Entry point binding   │
                              └──────────────────────────┘
```

---

## Key Components

### 1. Claude LLM — The Brain
**Model:** Claude Sonnet (Anthropic)  
**Access:** [claude.ai](https://claude.ai) / Claude Desktop  

Claude performs three distinct roles in this workflow:
- **Requirements analyst** — reads raw customer notes, asks clarifying questions, structures them into a provisioning spec
- **Design author** — populates the Excel workbook with queue names, agent counts, routing logic, IVR script
- **Provisioning orchestrator** — drives the ACP MCP server to configure WxCC via tool calls, with no manual API work

### 2. Claude Desktop — The MCP Client
**Role:** Local MCP host connecting Claude to the ACP server  

Claude Desktop runs on the operator's Windows laptop. It connects to the ACP MCP server over STDIO (local process), making all MCP tools available directly in the Claude chat interface.

Configuration (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "acp": {
      "command": "python",
      "args": ["-m", "uvicorn", "src.core.mcp.server.app:app"],
      "cwd": "D:\\project-acp"
    }
  }
}
```

### 3. ACP (Agentic Control Plane) — The MCP Server
**Location:** `D:\project-acp\`  
**Framework:** FastMCP 3.4.5 + FastAPI  
**Port:** 8100 (MCP) · 9000 (OAuth issuer)  

ACP is a governed MCP server that wraps the WxCC REST API in safe, auditable tool calls. Every tool invocation:
- Passes through RBAC policy in `control_hub.yaml`
- Checks the caller's OAuth scope
- Writes an immutable audit line to TinyDB

**RBAC Scopes:**
| Scope | What it allows |
|---|---|
| `knowledge:read` | Query queues, agents, stats |
| `diagnostics:run` | Run routing recommendations, simulations |
| `actions:execute` | Create queues, assign agents, import flows, bind entry points |

**MCP Tools Registered:**
| Tool | Scope | Description |
|---|---|---|
| `get_wxcc_queue_stats` | `knowledge:read` | Live queue metrics |
| `get_wxcc_agent_list` | `knowledge:read` | Agent roster |
| `get_wxcc_routing_recommendation` | `diagnostics:run` | AI routing suggestion |
| `get_wxcc_queue_list` | `knowledge:read` | All queues |
| `create_wxcc_queue` | `actions:execute` | Create a new queue |
| `assign_agent_to_queue` | `actions:execute` | Add agent to queue |
| `generate_wxcc_flow` | `actions:execute` | Generate IVR flow JSON |
| `import_wxcc_flow` | `actions:execute` | Import flow to WxCC |
| `bind_flow_to_entry_point` | `actions:execute` | Activate flow on entry point |

### 4. Cisco Sandbox Lab
**Platform:** Cisco dCloud WxCC sandbox  
**Region:** US1  
**API base:** `https://api.wxcc-us1.cisco.com`  

The sandbox provides a full WxCC environment with:
- Pre-configured entry points
- Control Hub for agent/queue management
- Flow Designer for IVR authoring
- Agent Desktop for call testing

> **Note:** Some Block 8 provisioning endpoints (queue create, agent assign, flow import, entry point bind) return HTTP 404 in the dCloud sandbox. These return `status: sandbox_limitation` and are documented per step. The tool logic and governance are fully functional; the limitation is the sandbox API surface.

---

## The Workflow — Step by Step

### Phase 1: Requirements → Design Workbook

```
Customer → (raw notes) → Claude → (structured spec) → Excel Workbook → Human Approval
```

The customer describes their contact center in plain language:

> *"We need two queues — Sales and Support. Sales open 9–6 weekdays, Support 24/7. Aim for 30s answer time. Two agents each to start. IVR: press 1 for Sales, press 2 for Support."*

Claude reads this and asks any missing questions (overflow handling, music on hold, queue priority). Once complete, it populates the **TechRetail WxCC Design Workbook** — an Excel file with tabs for:

- **Requirements** — raw customer statements mapped to WxCC concepts
- **Queues** — name, routing type, SLA targets, hours of operation
- **Agents** — login names, skills, queue membership
- **IVR Script** — menu options, TTS prompts, DTMF routing logic
- **Entry Points** — mapping to queues and flows

The human reviews and approves the workbook. This is the **contract** between business intent and technical provisioning.

### Phase 2: Approved Workbook → MCP Provisioning

```
Approved Workbook → Claude reads it → Claude calls ACP MCP tools → WxCC configured
```

With the approved workbook as context, Claude drives the ACP MCP server through 9 steps:

| Step | MCP Tool Called | What Happens |
|---|---|---|
| 00 | *(pre-config)* | Tokens loaded, connectivity verified |
| 01 | `get_wxcc_queue_stats` | Baseline metrics captured |
| 02 | `get_wxcc_agent_list` | Confirms available agents (User1, User2) |
| 03 | `get_wxcc_routing_recommendation` | AI recommends Circular routing |
| 04 | `get_wxcc_queue_list` | Lists existing queues for collision check |
| 05 | `create_wxcc_queue` | Creates TechRetail-Sales + TechRetail-Support |
| 06 | `assign_agent_to_queue` | Assigns User1→Sales, User2→Support |
| 07 | `generate_wxcc_flow` | Generates TechRetail_SimpleIVR flow JSON |
| 08 | `import_wxcc_flow` | Imports flow to WxCC Flow Designer |
| 09 | `bind_flow_to_entry_point` | Binds flow to Entry Point-1 |

Every tool call is logged with timestamp, token identity, outcome, and HTTP status.

### Phase 3: Manual Flow Publishing (Current Constraint)

After import, the IVR flow exists in WxCC Flow Designer as a **draft**. Publishing requires clicking **Publish** in the Flow Designer UI — this step cannot be automated via REST API in the current sandbox.

The operator opens Flow Designer, selects **TechRetail_SimpleIVR**, and clicks Publish. The flow immediately becomes live on Entry Point-1.

---

## IVR Flow Design

**Flow:** TechRetail_SimpleIVR  
**Entry Point:** Entry Point-1  

```
Caller dials in
     │
     ▼
NewContact node
     │
     ▼
Menu_atm (TTS: "For Sales press 1, for Support press 2")
     │
     ├─── Digit 1 ──► QueueContact → TechRetail-Sales ──► Agent1
     │
     └─── Digit 2 ──► QueueContact → TechRetail-Support ──► Agent2
          (Error / No-Input → Sales fallback)
```

**TTS Prompt:** *"Thank you for calling TechRetail. For Sales, press 1. For Support, press 2."*  
**No-Input Timeout:** 15 seconds → fallback to TechRetail-Sales  
**Queues:** Circular routing, both queues  

---

## Governance & Audit Trail

Every ACP tool call produces an audit record:

```json
{
  "timestamp": "2026-07-15T14:32:11Z",
  "tool": "create_wxcc_queue",
  "caller": "claude-desktop-session",
  "scope_used": "actions:execute",
  "rbac_group": "Allowed",
  "outcome": "sandbox_limitation",
  "http_status": 404,
  "queue_name": "TechRetail-Sales"
}
```

The full audit log is at `D:\project-acp\data\audit.json`.

---

## Repository Structure

```
wxcc-acp-showcase/
├── README.md                    ← This file
├── docs/
│   ├── 00-pre-config.md         ← Environment setup & token verification
│   ├── 01-queue-stats.md        ← Step 1: Baseline queue metrics
│   ├── 02-agent-list.md         ← Step 2: Agent roster
│   ├── 03-routing-recommendation.md  ← Step 3: AI routing recommendation
│   ├── 04-queue-list.md         ← Step 4: Existing queue inventory
│   ├── 05-create-queue.md       ← Step 5: Queue creation
│   ├── 06-assign-agent.md       ← Step 6: Agent assignment
│   ├── 07-generate-flow.md      ← Step 7: IVR flow generation
│   ├── 08-import-flow.md        ← Step 8: Flow import
│   └── 09-bind-entry-point.md   ← Step 9: Entry point activation
└── workbook/
    └── TechRetail_WxCC_Design_Workbook.xlsx   ← Approved design contract
```

---

## Security Notes

- Bearer tokens are stored in `.env` (gitignored — never committed)
- Org ID and Entry Point ID are masked in all documentation as `<ORG_ID_REDACTED>` and `<ENTRY_POINT_ID>`
- All API calls go over HTTPS to Cisco's US1 endpoint
- RBAC policy enforced at the MCP server layer — Claude cannot exceed its granted scopes

---

## Tech Stack

| Layer | Technology |
|---|---|
| AI Model | Claude Sonnet (Anthropic) |
| MCP Client | Claude Desktop |
| MCP Server | ACP — FastMCP 3.4.5 + FastAPI (Python) |
| Policy Engine | TinyDB + YAML RBAC (`control_hub.yaml`) |
| CC Platform | Cisco Webex Contact Center (WxCC) |
| Lab | Cisco dCloud sandbox |
| Design Workbook | Microsoft Excel |

---

## What This Demonstrates

- **MCP as an enterprise integration layer** — structured, governed, auditable AI tool use
- **Requirements → provisioning in one conversation** — no switching between systems
- **RBAC-scoped AI actions** — Claude can only do what its token allows
- **Immutable audit trail** — every AI action is logged with identity and outcome
- **Human-in-the-loop** — workbook approval before any provisioning begins

---

*Built on Cisco dCloud sandbox · Powered by Claude (Anthropic) · Governed by ACP MCP Server*
