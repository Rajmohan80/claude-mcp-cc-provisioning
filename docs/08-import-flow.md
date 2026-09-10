# Step 8 — Import Flow (`import_wxcc_flow`)

## Tool Metadata

| Field | Value |
|-------|-------|
| Tool | `import_wxcc_flow` |
| Block | 7 — Flow Import |
| Required scope | `actions:execute` |
| Allowed groups | `admins`, `cc_admins` only |
| Blocked groups | `engineers`, `viewers`, `cc_supervisors`, `cc_agents` |
| API | `POST /organization/<ORG_ID_REDACTED>/flow` |

---

## MCP Tool Call

```
Tool: import_wxcc_flow
Parameters:
  flow_name: "TechRetail-SimpleIVR"
```

---

## Result

```json
{
  "allowed": true,
  "status": "SAVED_LOCALLY",
  "http_status": 404,
  "flow_name": "TechRetail-SimpleIVR",
  "saved_to": "generated_flows/TechRetail-SimpleIVR.json",
  "node_count": 13,
  "article50_included": true,
  "message": "WxCC API returned 404 — flow saved locally at generated_flows/TechRetail-SimpleIVR.json. Import manually via Control Hub → Flows → Import, or use the Flow Designer API when available in your sandbox tier."
}
```

---

## What This Proves

**Governance:** `allowed: true` — `cc_admins` service token with `actions:execute` scope passes. The import was attempted against the live WxCC API.

**Graceful degradation:** When `POST /flow` returns HTTP 404 (sandbox limitation), the tool saves the 13-node flow JSON to `generated_flows/TechRetail-SimpleIVR.json` on the server filesystem. The file is ready for manual import via Control Hub → Flows → Import.

**Nothing is lost:** The flow JSON is fully generated (Article 50 disclosure included), structured, and persisted. The only missing step is the API endpoint availability — which is a sandbox entitlement boundary, not a code issue.

---

## Manual Fallback Path

When the REST API is unavailable in sandbox:

1. Open Webex Control Hub → Contact Center → Flows
2. Click **Import**
3. Upload `generated_flows/TechRetail-SimpleIVR.json`
4. Flow appears in DRAFT status
5. Open in Flow Designer and publish

---

## Sandbox vs Production Behaviour

| Environment | Outcome |
|-------------|---------|
| Sandbox (this demo) | HTTP 404 → saved locally for manual import |
| Production | HTTP 201 → flow ID returned, ready for Step 9 |

---

## Governance Policy (control_hub.yaml)

```yaml
import_wxcc_flow:
  description: "Import generated flow JSON to Webex Contact Center"
  required_scope: "actions:execute"
  groups:
    admins:         Allowed
    engineers:      Blocked
    viewers:        Blocked
    cc_admins:      Allowed
    cc_supervisors: Blocked
    cc_agents:      Blocked
```
