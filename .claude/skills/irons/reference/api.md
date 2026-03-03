# irons REST API Reference

## Base Information

- **Base URL:** `https://api.iron.sh`
- **Auth:** `Authorization: Bearer <api_key>` header
- **Response format:** JSON, timestamps in ISO 8601 UTC

## Resource ID Prefixes

| Resource | Prefix | Example |
|----------|--------|---------|
| VM | `vm_` | `vm_k3mf9xvw2p` |
| Egress Rule | `egr_` | `egr_9xm2kfp4` |
| Egress Event | `ee_` | `ee_w8n3vqx1` |

## Pagination

All list endpoints return a pagination envelope:

```json
{
  "data": [],
  "has_more": false,
  "cursor": null
}
```

Pass `cursor` as a query parameter to fetch the next page.

## Errors

```json
{
  "error": {
    "code": "vm_not_found",
    "message": "No VM with ID vm_k3mf9xvw2p"
  }
}
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request / validation error |
| 401 | Missing or invalid API key |
| 404 | Resource not found |
| 409 | Conflict (e.g. VM already running) |
| 422 | Unprocessable entity |
| 429 | Rate limited |
| 500 | Internal server error |

---

## VMs

### VM Status Values

| status | status_detail | Description |
|--------|---------------|-------------|
| pending | queued, creating, starting | Being provisioned |
| running | running, ready | Up and accepting connections |
| stopped | stopping, stopped | Shut down |
| destroyed | destroying, destroyed | Terminated |
| failed | failed | Encountered error |

### Create VM

`POST /v1/vms`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | yes | Unique name among non-destroyed VMs |
| public_key | string | yes | SSH public key for access |

```bash
curl -X POST https://api.iron.sh/v1/vms \
  -H "Authorization: Bearer $IRONS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-dev-env",
    "public_key": "ssh-ed25519 AAAA..."
  }'
```

Response: `201 Created`

```json
{
  "id": "vm_k3mf9xvw2p",
  "name": "my-dev-env",
  "status": "pending",
  "status_detail": "creating",
  "created_at": "2026-02-28T12:00:00Z",
  "updated_at": "2026-02-28T12:00:00Z"
}
```

### List VMs

`GET /v1/vms`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| status | string | — | Filter: pending, running, stopped, destroyed, failed |
| status_detail | string | — | Filter: queued, creating, starting, running, ready, stopping, stopped, destroying, destroyed, failed |
| name | string | — | Exact name match |
| cursor | string | — | Pagination cursor |
| limit | integer | 20 | Results per page (max 100) |
| sort | string | created_at | Sort field: created_at, updated_at, name |
| order | string | desc | asc or desc |

### Get VM

`GET /v1/vms/{vm_id}`

Returns the full VM object.

### Destroy VM

`DELETE /v1/vms/{vm_id}`

VM must be stopped first. Returns `409 Conflict` with code `vm_not_stopped` if running.

Response: `204 No Content`

### Start VM

`POST /v1/vms/{vm_id}/start`

Response: `200 OK` with full VM object (status: running).

### Stop VM

`POST /v1/vms/{vm_id}/stop`

Response: `200 OK` with full VM object (status: stopped).

### Get SSH Connection Info

`GET /v1/vms/{vm_id}/ssh`

```json
{
  "host": "203.0.113.10",
  "port": 2222,
  "username": "root",
  "command": "ssh -p 2222 root@203.0.113.10"
}
```

---

## Egress Policy

### Get Policy

`GET /v1/egress/policy`

```json
{
  "mode": "enforce"
}
```

### Update Policy

`PUT /v1/egress/policy`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| mode | string | yes | `enforce` or `warn` |

---

## Egress Rules

### List Rules

`GET /v1/egress/rules`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| host | string | — | Exact host match |
| cidr | string | — | Exact CIDR match |
| name | string | — | Exact name match |
| cursor | string | — | Pagination cursor |
| limit | integer | 50 | Results per page (max 200) |
| sort | string | created_at | Sort: created_at, host |
| order | string | desc | asc or desc |

```json
{
  "data": [
    {
      "id": "egr_9xm2kfp4",
      "name": "github-api",
      "host": "api.github.com",
      "cidr": null,
      "comment": "GitHub API access",
      "created_at": "2026-02-28T12:00:00Z"
    }
  ],
  "has_more": false,
  "cursor": null
}
```

### Create Rule

`POST /v1/egress/rules`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | no | Unique human-readable name |
| host | string | no | Domain to allow (e.g. `api.github.com`, `*.npmjs.org`) |
| cidr | string | no | CIDR range to allow (e.g. `10.0.0.0/8`) |
| comment | string | no | Note about rule purpose |

One of `host` or `cidr` is required. They are mutually exclusive.

Response: `201 Created`

### Delete Rule

`DELETE /v1/egress/rules/{rule_id}`

Response: `204 No Content`

---

## Audit Logs

### List Egress Events

`GET /v1/audit/egress`

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| vm_id | string | — | Filter by VM ID |
| verdict | string | — | Filter: allowed, denied |
| allowed | boolean | — | Filter by whether request was permitted |
| mode | string | — | Filter: enforce, warn |
| host | string | — | Exact host match |
| cidr | string | — | CIDR match |
| protocol | string | — | Filter: http, tls, tcp |
| since | string | — | ISO 8601 timestamp lower bound |
| until | string | — | ISO 8601 timestamp upper bound |
| cursor | string | — | Pagination cursor |
| limit | integer | 50 | Results per page (max 500) |
| order | string | desc | asc or desc (always sorted by timestamp) |

```json
{
  "data": [
    {
      "id": "ee_w8n3vqx1",
      "timestamp": "2026-02-28T12:05:30Z",
      "vm_id": "vm_k3mf9xvw2p",
      "host": "api.github.com",
      "cidr": null,
      "protocol": "tls",
      "verdict": "allowed",
      "allowed": true,
      "mode": "enforce"
    }
  ],
  "has_more": true,
  "cursor": "ee_p3xn7vm2"
}
```

---

## Authentication (Device Flow)

### Request Device Code

`POST /v1/auth/device/code`

No auth required.

```json
{
  "data": {
    "code": "ABCD-1234",
    "verification_uri": "https://app.iron.sh/activate",
    "expires_at": "2026-02-28T12:15:00Z"
  }
}
```

### Poll Authorization

`GET /v1/auth/device/poll?code=ABCD-1234`

| Status | Meaning |
|--------|---------|
| pending | User hasn't authorized yet — keep polling |
| authorized | Success — `token` field is present |
| expired | Code expired — request a new one |

```json
{
  "data": {
    "status": "authorized",
    "token": "iron_live_a1b2c3d4e5f6..."
  }
}
```
