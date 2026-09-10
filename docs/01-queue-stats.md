# Step 1 — Live Queue Statistics (`get_wxcc_queue_stats`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `get_wxcc_queue_stats` |
| Block | 6A — Live WxCC Read |
| Required scope | `knowledge:read` |
| Allowed groups | `admins`, `cc_admins`, `cc_supervisors`, `cc_agents` |
| Blocked groups | `engineers`, `viewers` |
| API | GraphQL Search `POST /search?orgId=<ORG_ID_REDACTED>` |

---

## MCP Tool Call

```
Tool: get_wxcc_queue_stats
Parameters: {} (service token auto-resolved)
```

---

## Result (live sandbox data)

```json
{
  "allowed": true,
  "org_id": "<ORG_ID_REDACTED>",
  "queues": [
    {
      "queue_id": "Queue-1",
      "contacts_in_queue": 0,
      "oldest_contact_age_seconds": 0,
      "agents_available": 1,
      "agents_connected": 0,
      "longest_wait_seconds": 0,
      "service_level_percent": 100
    }
  ],
  "summary": {
    "total_queues_reported": 1,
    "total_contacts_waiting": 0,
    "total_agents_available": 1
  }
}
```

---

## What This Proves

- Live read from WxCC GraphQL Search API via MCP tool
- Governance: `cc_agents` group has `knowledge:read` → Allowed
- Real-time queue depth and SLA visible from Claude without browser

---

## Governance Policy (control_hub.yaml)

```yaml
get_wxcc_queue_stats:
  description: "Return live queue statistics from Webex Contact Center sandbox"
  required_scope: "knowledge:read"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Allowed
    cc_agents:      Allowed
```
