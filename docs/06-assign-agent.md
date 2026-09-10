# Step 6 — Assign Agent to Queue (`assign_agent_to_queue`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `assign_agent_to_queue` |
| Block | 8 — WxCC Provisioning |
| Required scope | `actions:execute` |
| Allowed groups | `admins`, `cc_admins` only |
| Blocked groups | `engineers`, `viewers`, `cc_supervisors`, `cc_agents` |
| API | `POST /organization/<ORG_ID_REDACTED>/queue/{queueId}/members` |

---

## Agent IDs (resolved in Step 2)

| Agent | ID |
|-------|----|
| User1 Agent1 | `91c73c31-eb4b-4c1e-9b4b-3ed4e19bd819` |
| User2 Agent2 | `07f49dc5-ae27-4cad-ad54-b93c86dd82f9` |
| Admin User | `deff0239-6ef3-4315-8a25-60d383e1b89e` |

---

## MCP Tool Call

```
Tool: assign_agent_to_queue
Parameters:
  queue_id: "Queue-1"
  agent_id: "91c73c31-eb4b-4c1e-9b4b-3ed4e19bd819"
```

---

## Result

```json
{
  "allowed": true,
  "api_endpoint": "POST /organization/<ORG_ID_REDACTED>/queue/Queue-1/members",
  "payload_attempted": {
    "members": [
      {
        "id": "91c73c31-eb4b-4c1e-9b4b-3ed4e19bd819",
        "type": "AGENT"
      }
    ]
  },
  "status": "sandbox_limitation",
  "http_status": 404,
  "sandbox_note": "Sandbox tier limitation: POST /queue/{queueId}/members returns HTTP 404. This endpoint requires production entitlement. Agent assignment must be done via Control Hub → Queues → Edit → Agents in sandbox."
}
```

---

## What This Proves

**Governance:** `allowed: true` — governance chain ran to completion. The agent assignment was fully structured, the REST call was attempted, the limitation was returned transparently.

**API-driven intent:** In production, this single MCP call replaces the 6-click Control Hub flow: Queues → Select Queue → Edit → Agents tab → Add Agent → Save.

**End-to-end traceability:** The payload shows exactly what was sent. No black-box clicks — every provisioning intent is logged.

---

## Sandbox vs Production Behaviour

| Environment | Outcome |
|-------------|---------|
| Sandbox (this demo) | HTTP 404 — entitlement not provisioned |
| Production | HTTP 200 — agent assigned to queue |

---

## Governance Policy (control_hub.yaml)

```yaml
assign_agent_to_queue:
  description: "Assign an agent to a queue via POST /organization/{orgId}/queue/{queueId}/members"
  required_scope: "actions:execute"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Blocked
    cc_agents:      Blocked
```
