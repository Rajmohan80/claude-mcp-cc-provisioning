# Step 0 — Pre-Configuration

## What This Step Covers

Before any MCP tool call, two services must be running and a service token must be in scope.

---

## Services

| Service | Command | Port | Purpose |
|---------|---------|------|---------|
| OAuth Issuer | `scripts\run_oauth.bat` | 9000 | Mints JWT tokens for each user group |
| MCP Server | `scripts\run_mcp.bat` | 8100 | Governs and executes all 17 tools |

---

## Token Resolution

All provisioning tools accept an optional `token` parameter. When omitted, the server calls `_get_service_token()`:

```python
def _get_service_token() -> str:
    """Mint a cc_admins JWT from the OAuth issuer at startup."""
    resp = httpx.post(
        f"{OAUTH_ISSUER_URL}/token",
        data={
            "grant_type": "client_credentials",
            "client_id": "service-account",
            "client_secret": "service-secret",
            "scope": "actions:execute diagnostics:run knowledge:read",
            "group": "cc_admins",
        },
    )
    return resp.json()["access_token"]
```

The service token is minted once at server startup and reused for all demo calls. In production, each user presents their own JWT from the org's IdP.

---

## Identity Map (control_hub.yaml)

```yaml
identity_map:
  "admin@<ORG_DOMAIN>":  cc_admins
  "user2@<ORG_DOMAIN>":  cc_supervisors
  "user1@<ORG_DOMAIN>":  cc_agents
```

---

## Governance Check Flow

Every tool call runs this sequence before touching any API:

```
Token presented (or service token used)
        ↓
JWT decoded → group resolved (e.g. cc_admins)
        ↓
control_hub.yaml consulted:
  - Is this tool Allowed for this group?
  - Does the token carry the required scope?
        ↓
  PASS → tool executes → audit line written
  FAIL → refusal returned → audit line written
```

Audit lines are written regardless of pass/fail — every attempt is recorded.

---

## Environment Variables (masked)

```bash
WXCC_ORG_ID=<ORG_ID_REDACTED>
WXCC_BASE_URL=https://api.wxcc-us1.cisco.com
WXCC_ACCESS_TOKEN=<BEARER_TOKEN>
OAUTH_ISSUER_URL=http://localhost:9000
MCP_SERVER_PORT=8100
```

Never commit `.env` to git. Use `.env.example` as the template.
