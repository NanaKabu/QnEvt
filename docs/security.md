# QnEvt Security Notes

This document gives operational security guidance for deploying and administering QnEvt.

## Deployment Secrets

Set real production values for:

- `JWT_SECRET`
- `ADMIN_EMAIL`
- `ADMIN_PASSWORD_HASH`
- `SUPERADMIN_KEY`
- MySQL credentials
- Redis credentials, if Redis is enabled
- SMTP credentials, if email verification is enabled
- Cloudflare Turnstile site and secret keys, if bot protection is enabled

Do not run production with generated in-memory secrets. A process restart would invalidate sessions and may weaken incident response.

## Transport Security

Run QnEvt behind HTTPS in production. The server sets common security headers, but TLS must be provided by your reverse proxy, load balancer, or platform.

Recommended proxy settings:

- Preserve `X-Forwarded-For` so rate limits can use the client IP.
- Disable caching for admin pages and API responses.
- Limit request body size at the proxy as well as in Node.js.

## Account Security

Recommendations:

- Use a strong admin password and rotate it after initial deployment.
- Enable Cloudflare Turnstile for login, registration, and anonymous proof creation.
- Keep the custom admin path private, but do not treat it as the only security boundary.
- Limit admin access by network policy when possible.

## API Key Handling

User API keys authorize entropy requests and quota usage.

Current behavior:

- Users can create, disable, and delete keys in the developer portal.
- Quota is checked before authenticated `/api/random` calls.
- Usage count is incremented on API key use.

Recommended production hardening:

- Store only a hash or HMAC of each API key.
- Display the raw key only once at creation time.
- Enforce IP allowlists before serving entropy.
- Add key rotation and last-used metadata.

## Push Key And Node Token Handling

Push keys and node tokens authorize telemetry ingestion.

Operational rules:

- Treat push keys and node tokens like production secrets.
- Do not paste them into public issue trackers or screenshots.
- Revoke keys immediately when a bridge host is retired.
- Prefer separate credentials per physical node.

Recommended production hardening:

- Require authenticated admin access for manual node registration.
- Add token rotation and expiration.
- Bind node credentials to expected chip IDs where practical.

## Proof Privacy

QnEvt proofs are designed to avoid uploading raw sensitive material.

Important notes:

- Secret generation should never upload the raw secret.
- Audit sampling should hash datasets locally in the browser.
- Fair random proofs should disclose only the participant commitment unless the user intentionally publishes the list.
- Public proof records should not contain confidential business data.

## Audit Report Uploads

Admin audit report uploads should be treated as public evidence.

Allowed report file types are restricted by the server, but operators should still:

- Upload only non-sensitive reports.
- Review generated PDFs or logs before publishing.
- Remove obsolete or incorrect reports from the admin panel.
- Keep raw 1 GB sample files protected.

## Logging

Avoid logging:

- Raw generated secrets
- API keys
- Push keys
- Node tokens
- Full entropy payloads beyond intended audit samples
- Password reset or verification codes in production

If development logs include verification codes, disable that behavior before public deployment.

## Incident Response

If credentials are suspected to be compromised:

1. Rotate `JWT_SECRET` and force users to log in again.
2. Rotate admin credentials and `SUPERADMIN_KEY`.
3. Disable suspicious API keys.
4. Revoke affected push keys or node tokens.
5. Review proof visibility and audit events.
6. Rebuild the entropy reservoir from trusted nodes.

## Responsible Disclosure

Report suspected vulnerabilities privately to the project maintainer or the configured contact email. Include:

- Affected endpoint or page
- Steps to reproduce
- Expected impact
- Suggested mitigation, if known
