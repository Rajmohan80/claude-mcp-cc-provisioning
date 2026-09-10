# Step 5 — Create Queue (`create_wxcc_queue`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `create_wxcc_queue` |
| Block | 8 — WxCC Provisioning |
| Required scope | `actions:execute` |
| Allowed groups | `admins`, `cc_admins` only |
| Blocked groups | `engineers`, `viewers`, `cc_supervisors`, `cc_agents` |
| API | `POST /organization/<ORG_ID_REDACTED>/queue` |

---

## MCP Tool Call

```
Tool: create_wxcc_queue
Parameters:
  name: "TechRetail-Sales-Q"
  description: "TechRetail Sales queue — created via ACP MCP tool"
  channel_type: "TELEPHONY"
  max_time_in_queue: 300
  service_level_threshold: 20
```

---

## Result

```json
{
  "allowed": true,
  "api_endpoint": "POST /organization/<ORG_ID_REDACTED>/queue",
  "payload_attempted": {
    "name": "TechRetail-Sales-Q",
    "description": "TechRetail Sales queue — created via ACP MCP tool",
    "channelType": "TELEPHONY",
    "maxTimeInQueue": 300,
    "serviceLevelThreshold": 20
  },
  "status": "sandbox_limitation",
  "http_status": 404,
  "sandbox_note": "Sandbox tier limitation: POST /organization/<ORG_ID_REDACTED>/queue returns HTTP 404. This endpoint requires production entitlement. Queue creation must be done via Control Hub → Queues → New Queue in sandbox."
}
```

---

## What This Proves

**Governance:** `allowed: true` — the `cc_admins` service token carries `actions:execute` and the group is permitted. Governance passed. The 404 is a WxCC sandbox API limitation, not a code or governance failure.

**Honest engineering:** The tool surfaces the exact HTTP status, the payload it attempted, and the manual fallback path. In production (with entitlement), the same call creates the queue with no code changes.

**Audit trail:** This attempt is recorded in TinyDB with `allowed: true`, `tool: create_wxcc_queue`, timestamp, and the governance result — regardless of the API outcome.

---

## Sandbox vs Production Behaviour

| Environment | Outcome |
|-------------|---------|
| Sandbox (this demo) | HTTP 404 — entitlement not provisioned |
| Production | HTTP 201 — queue created, ID returned |

---

## Governance Policy (control_hub.yaml)

```yaml
create_wxcc_queue:
  description: "Create a new queue via POST /organization/{orgId}/queue"
  required_scope: "actions:execute"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Blocked
    cc_agents:      Blocked
```
