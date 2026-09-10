# Step 3 — Routing Recommendation (`get_wxcc_routing_recommendation`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `get_wxcc_routing_recommendation` |
| Block | 6A — Live WxCC Read |
| Required scope | `diagnostics:run` |
| Allowed groups | `admins`, `cc_admins`, `cc_supervisors` |
| Blocked groups | `engineers`, `viewers`, `cc_agents` |
| API | GraphQL Search (reads queue load) |

---

## MCP Tool Call

```
Tool: get_wxcc_routing_recommendation
Parameters: {} (service token auto-resolved)
```

---

## Result (live sandbox data)

```json
{
  "allowed": true,
  "org_id": "<ORG_ID_REDACTED>",
  "recommendation": {
    "optimal_queue": "Queue-1",
    "reason": "Lowest current wait time and highest available agent count",
    "agents_available": 1,
    "contacts_waiting": 0,
    "estimated_wait_seconds": 0
  },
  "all_queues_evaluated": [
    {
      "queue": "Queue-1",
      "score": 100,
      "agents_available": 1,
      "contacts_waiting": 0
    }
  ]
}
```

---

## What This Proves

- Diagnostic tool reads live queue load and returns an AI-informed routing recommendation
- `diagnostics:run` scope required — higher privilege than `knowledge:read`
- `cc_agents` Blocked from routing recommendations — correct RBAC

---

## Governance Policy (control_hub.yaml)

```yaml
get_wxcc_routing_recommendation:
  description: "Read live queue load and recommend optimal inbound routing"
  required_scope: "diagnostics:run"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Allowed
    cc_agents:      Blocked
```
