# Step 2 — Agent List (`get_wxcc_agent_list`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `get_wxcc_agent_list` |
| Block | 6A — Live WxCC Read |
| Required scope | `knowledge:read` |
| Allowed groups | `admins`, `cc_admins`, `cc_supervisors` |
| Blocked groups | `engineers`, `viewers`, `cc_agents` |
| API | GraphQL Search `POST /search?orgId=<ORG_ID_REDACTED>` |

---

## MCP Tool Call

```
Tool: get_wxcc_agent_list
Parameters: {} (service token auto-resolved)
```

---

## Result (live sandbox data)

```json
{
  "allowed": true,
  "org_id": "<ORG_ID_REDACTED>",
  "agents": [
    {
      "agent_id": "91c73c31-eb4b-4c1e-9b4b-3ed4e19bd819",
      "name": "User1 Agent1",
      "email": "user1@<ORG_DOMAIN>",
      "status": "Available",
      "group": "cc_agents"
    },
    {
      "agent_id": "07f49dc5-ae27-4cad-ad54-b93c86dd82f9",
      "name": "User2 Agent2",
      "email": "user2@<ORG_DOMAIN>",
      "status": "Available",
      "group": "cc_supervisors"
    },
    {
      "agent_id": "deff0239-6ef3-4315-8a25-60d383e1b89e",
      "name": "Admin User",
      "email": "admin@<ORG_DOMAIN>",
      "status": "Idle",
      "group": "cc_admins"
    }
  ],
  "agent_count": 3
}
```

---

## What This Proves

- Live agent roster with real-time status from WxCC sandbox
- Agent IDs resolved here are reused in Step 6 (`assign_agent_to_queue`)
- `cc_agents` are Blocked from seeing the full agent list — correct RBAC

---

## Governance Policy (control_hub.yaml)

```yaml
get_wxcc_agent_list:
  description: "Return agent and supervisor list with status from WxCC sandbox"
  required_scope: "knowledge:read"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Allowed
    cc_agents:      Blocked
```
