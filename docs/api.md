# QnEvt API Reference

This document describes the public HTTP endpoints used by the QnEvt web tools, developer portal, node console, and admin panel.

Base URL examples:

- Local development: `http://localhost:3000`
- Production: use your deployed QnEvt origin

All responses are JSON unless noted otherwise.

## Authentication

User endpoints use a JWT:

```http
Authorization: Bearer <jwt>
```

Developer entropy endpoints use an API key:

```http
Authorization: Bearer <qrng_api_key>
```

Telemetry push endpoints use either a user push key or a node token:

```http
Authorization: Bearer <qrng_push_or_node_token>
```

## Status

### `GET /api/status`

Returns public platform status, counters, and daily public API quota state.

Typical fields:

- `deviceStatus`
- `throughputKbps`
- `totalBytes`
- `globalDailyPublicCalls`
- `publicDailyLimit`
- `nextDailyResetAt`

## Randomness

### `GET /api/public/random`

Returns random bytes from the public physical entropy reservoir. No login is required, but public rate and daily quota limits apply.

Query parameters:

| Name | Default | Limit | Description |
| --- | --- | --- | --- |
| `bytes` | `16` | `128` | Number of bytes to return. |
| `format` | `hex` | `hex`, `base64`, `uint8` | Output encoding. |

Example:

```http
GET /api/public/random?bytes=16&format=hex
```

Successful response:

```json
{
  "ok": true,
  "bytes": 16,
  "format": "hex",
  "data": "84e0f61f5a0c3fb1b47dd6c6c5f70c1a",
  "batch_id": "qnb_0123456789abcdef",
  "source": "quantum",
  "timestamp": "2026-05-29T12:00:00.000Z"
}
```

### `GET /api/random`

Authenticated developer endpoint for larger random byte requests.

Headers:

```http
Authorization: Bearer <qrng_api_key>
```

Query parameters:

| Name | Default | Limit | Description |
| --- | --- | --- | --- |
| `bytes` | `32` | `4096` | Number of bytes to return. |
| `format` | `hex` | `hex`, `base64`, `uint8` | Output encoding. |

## Proof Registry

### `POST /api/proofs`

Stores a proof in the cloud registry. Anonymous proofs may require Cloudflare Turnstile when enabled. Logged-in proofs are associated with the user account.

Body:

```json
{
  "version": "1.0",
  "proof_type": "audit_sampling",
  "input_hash": "64_hex_sha256",
  "entropy_batch": "qnb_0123456789abcdef",
  "mixing_algorithm": "SHA256-Counter Fisher-Yates (Seeded by Entropy)",
  "output_hash": "64_hex_sha256",
  "timestamp": "2026-05-29T12:00:00.000Z",
  "signature": "unsigned_client_side"
}
```

Response:

```json
{
  "ok": true,
  "public_id": "32_hex_public_id"
}
```

### `GET /api/proofs/:public_id`

Fetches a visible proof by public ID.

### `GET /api/public/proofs`

Lists recent visible public proofs.

### `GET /api/user/proofs`

Lists the authenticated user's saved proofs.

Headers:

```http
Authorization: Bearer <jwt>
```

### `GET /api/batch/:batch_id`

Looks up an entropy batch used by a proof.

Response:

```json
{
  "ok": true,
  "batch": {
    "batch_id": "qnb_0123456789abcdef",
    "entropy_hash": "64_hex_sha256",
    "health_status": "PASS",
    "created_at": "2026-05-29T12:00:00.000Z"
  }
}
```

## User Accounts

### `GET /api/auth/turnstile-config`

Returns whether Cloudflare Turnstile is enabled and the public site key.

### `POST /api/auth/send-code`

Sends a registration verification code.

Body:

```json
{
  "email": "user@gmail.com",
  "turnstileToken": "optional_when_disabled"
}
```

### `POST /api/auth/register`

Creates a user account after email verification.

### `POST /api/auth/login`

Returns a JWT for the developer portal.

### `GET /api/user/profile`

Returns authenticated account profile and quota usage.

## API Keys

### `GET /api/keys`

Lists API keys for the authenticated user.

### `POST /api/keys`

Creates a new API key.

Body:

```json
{
  "label": "Production Server"
}
```

### `PUT /api/keys/:id/status`

Enables or disables an API key.

### `DELETE /api/keys/:id`

Deletes an API key.

## Node And Telemetry

### `POST /api/nodes/register`

Registers a hardware node and returns a node token.

### `GET /api/nodes`

Lists registered nodes for public telemetry views.

### `POST /api/push-random`

Pushes physical entropy telemetry into the server reservoir.

Headers:

```http
Authorization: Bearer <qrng_push_or_node_token>
```

Body fields vary by bridge implementation, but should include random bytes or block hashes, health status, and optional sequence/client metadata.

## Admin Endpoints

Admin endpoints require an admin JWT:

```http
Authorization: Bearer <admin_jwt>
```

Common endpoints:

- `POST /api/admin/login`
- `GET /api/admin/config`
- `POST /api/admin/config`
- `GET /api/admin/metrics`
- `POST /api/admin/clear-cache`
- `GET /api/admin/users`
- `PUT /api/admin/users/:id/tier`
- `GET /api/admin/nodes`
- `GET /api/admin/invitations`
- `POST /api/admin/invitations/generate`
- `GET /api/admin/sample/info`
- `GET /api/admin/sample/download`
- `POST /api/admin/update-audit`

## Error Shape

Errors use this format:

```json
{
  "error": "Human readable error message"
}
```

Important status codes:

- `400` invalid input
- `401` missing or invalid credential
- `402` quota exceeded for authenticated API usage
- `403` insufficient role
- `404` resource not found
- `429` public rate limit or proof creation limit exceeded
- `503` entropy reservoir or database unavailable
