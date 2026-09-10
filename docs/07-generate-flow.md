# Step 7 — Generate IVR Flow (`generate_wxcc_flow`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `generate_wxcc_flow` |
| Block | 7 — Flow Generation |
| Required scope | `actions:execute` |
| Allowed groups | `admins`, `cc_admins` only |
| Blocked groups | `engineers`, `viewers`, `cc_supervisors`, `cc_agents` |
| Approval | Required before import |

---

## MCP Tool Call

```
Tool: generate_wxcc_flow
Parameters:
  flow_name: "TechRetail-SimpleIVR"
  include_article50: true
  sales_queue: "Queue-1"
  support_queue: "Queue-1"
```

---

## Result Summary

```json
{
  "allowed": true,
  "flow_name": "TechRetail-SimpleIVR",
  "node_count": 13,
  "link_count": 13,
  "article50": true,
  "connector": "TechRetail-GCP-CCAI",
  "status": "GENERATED — review then call import_wxcc_flow",
  "approval_required": true,
  "approval_message": "Flow 'TechRetail-SimpleIVR' generated with 13 nodes. Article 50 disclosure: YES. Sales bot: TechRetail-Sales-Bot. Support bot: TechRetail-Support-Bot. Approve import to push this flow to Webex Contact Center."
}
```

---

## Flow Node Inventory

| Node ID | Type | Name |
|---------|------|------|
| `start` | START | Call Start |
| `art50_*` | PLAY_MESSAGE | AI Disclosure (Art 50) |
| `bizhours_*` | BUSINESS_HOURS | Business Hours Check |
| `welcome_*` | PLAY_MESSAGE | Welcome Prompt |
| `collect_*` | COLLECT_DIGITS | Intent Collector |
| `router_*` | CONDITION | Intent Router |
| `salesvav2_*` | VIRTUAL_AGENT_V2 | TechRetail-Sales-Bot |
| `supportvav2_*` | VIRTUAL_AGENT_V2 | TechRetail-Support-Bot |
| `salesq_*` | QUEUE_CONTACT | TechRetail-Sales Queue |
| `supportq_*` | QUEUE_CONTACT | TechRetail-Support Queue |
| `noinput_*` | PLAY_MESSAGE | No Input Handler |
| `afterhours_*` | PLAY_MESSAGE | After Hours |
| `end_*` | END_FLOW | End Flow |

---

## Article 50 AI Disclosure

The first node after START plays a mandatory disclosure:

> *"Welcome to TechRetail. Please be aware that parts of this interaction may be handled by an AI system. You may request to speak with a human agent at any time by saying 'agent' or pressing zero."*

Applied by default per EU AI Act Article 50. Best practice for all deployments.

---

## Call Flow Logic

```
PSTN Call → AI Disclosure → Business Hours Check
                               ├── Closed → After Hours Message → END
                               └── Open  → Welcome Prompt → Collect Intent
                                              ├── "1" / "Sales" → Sales Bot (VAV2)
                                              │     ├── Resolved → END
                                              │     └── Escalate → Sales Queue
                                              ├── "2" / "Support" → Support Bot (VAV2)
                                              │     ├── Resolved → END
                                              │     └── Escalate → Support Queue
                                              └── No Input → Fallback → Sales Queue
```

---

## Governance Policy (control_hub.yaml)

```yaml
generate_wxcc_flow:
  description: "Generate WxCC Flow Designer JSON with VAV2 + Dialogflow CX nodes"
  required_scope: "actions:execute"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Blocked
    cc_agents:      Blocked
```
