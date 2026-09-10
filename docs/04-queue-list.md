# Step 4 — Queue List (`get_wxcc_queue_list`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `get_wxcc_queue_list` |
| Block | 8 — WxCC Provisioning |
| Required scope | `knowledge:read` |
| Allowed groups | `admins`, `cc_admins`, `cc_supervisors` |
| Blocked groups | `engineers`, `viewers`, `cc_agents` |
| API | `GET /organization/<ORG_ID_REDACTED>/queue` |

---

## MCP Tool Call

```
Tool: get_wxcc_queue_list
Parameters: {} (service token auto-resolved)
```

---

## Result (live sandbox data)

```json
{
  "allowed": true,
  "api_endpoint": "GET /organization/<ORG_ID_REDACTED>/queue",
  "queue_count": 1,
  "queues": [
    {
      "id": "Queue-1",
      "name": "Queue-1",
      "channel_type": "TELEPHONY",
      "description": ""
    }
  ],
  "note": ""
}
```

---

## What This Proves

- First Block 8 tool call — provisioning REST API responding for read operations
- `GET /queue` succeeds in sandbox tier (read endpoint available)
- Queue ID `Queue-1` confirmed for use in Steps 5 and 6

---

## Governance Policy (control_hub.yaml)

```yaml
get_wxcc_queue_list:
  description: "List all queues in the WxCC org via GET /organization/{orgId}/queue"
  required_scope: "knowledge:read"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Allowed
    cc_agents:      Blocked
```
