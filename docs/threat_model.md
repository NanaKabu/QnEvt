# QnEvt Threat Model

QnEvt provides verifiable entropy provenance for workflows that depend on randomness. This document defines what the platform protects, what it does not protect, and which assumptions must hold for proofs to be meaningful.

## Assets

Primary assets:

- Raw physical entropy entering the reservoir
- Entropy batch records and hashes
- Proof registry records
- User API keys, push keys, node tokens, and JWTs
- Admin credentials and platform settings
- Generated secrets, sampled CSV outputs, and fair random winner lists

## Trust Boundaries

### Browser

Secret generation, audit sampling, fair random draws, hashing, and proof creation are performed in the user's browser. The server should not receive raw secrets or uploaded audit datasets during normal proof generation.

### QnEvt Server

The server authenticates users, manages API keys, stores proof metadata, tracks entropy batches, manages quota, and serves public verification data.

### Hardware Node And Bridge

The firmware and bridge are responsible for acquiring entropy and pushing telemetry to the server. The server treats the node token or push key as the credential boundary for telemetry ingestion.

### Administrator

Admin and superadmin users can modify platform settings, user quotas, audit reports, and proof visibility. Compromise of admin credentials can affect presentation and registry visibility.

## Attacker Goals

An attacker may try to:

- Predict generated random outputs
- Replace physical entropy with deterministic data
- Tamper with a proof after publication
- Register unauthorized nodes
- Exhaust public entropy quota
- Steal API keys or push tokens
- Publish misleading audit reports
- Hide or alter proof registry records
- Inject script through user-controlled fields

## In Scope

QnEvt is designed to mitigate:

- Accidental or malicious reuse of entropy blocks through server-side health checks
- Tampering with published proof metadata through hash commitments
- Public API abuse through rate and daily quota limits
- Replay or stale telemetry through client sequence tracking where bridge metadata is available
- Loss of provenance for generated proofs through batch IDs and registry lookups

## Out Of Scope

QnEvt does not claim to solve:

- Malware on the user's browser or workstation
- A fully compromised server or database administrator
- A compromised hardware node that can also forge expected health metadata
- Side-channel leakage from generated secrets after they are shown to the user
- Human misuse, such as publishing the raw secret or storing it in logs
- Legal certification claims beyond the uploaded audit report evidence

## Main Threats And Controls

### Predictable Random Output

Risk: Random outputs may be predictable if the entropy source fails or if only weak local randomness is used.

Controls:

- Browser CSPRNG is used for local randomness.
- Live QnEvt entropy batches are mixed when available.
- Entropy batches are logged by ID and hash for later audit.
- Verification can identify whether a proof used `local_simulation`, `qnevt_live_pool`, or a live `qnb_...` batch.

### Entropy Source Substitution

Risk: A node or bridge could push deterministic data while claiming it is physical entropy.

Controls:

- Telemetry push requires a push key or node token.
- Continuous health checks detect duplicate blocks and obvious stuck streams.
- Batch records include health status.
- Admin audit reports provide external statistical evidence.

Residual risk: Current browser-side proof signatures are marked `unsigned_client_side`; stronger node-signed batch attestations should be added before high-assurance production use.

### Proof Tampering

Risk: A published proof may be edited before being shared.

Controls:

- `input_hash` and `output_hash` commit to the input and output.
- Stored registry proofs can be fetched by public ID.
- The verifier can compare an uploaded proof against the registry copy when a public ID is available.

### API Key Theft

Risk: A stolen API key can consume quota or request entropy.

Controls:

- Keys can be disabled or deleted.
- Quota limits are checked on authenticated requests.
- Usage counts are tracked.

Recommended improvement: Store only hashed API keys server-side and implement IP whitelist enforcement.

### Node Token Theft

Risk: A stolen node token or push key can inject telemetry.

Controls:

- Push token validation is server-side.
- Redis cache entries are invalidated when nodes or push keys are removed.
- Single-device push tracking can reject competing clients.

Recommended improvement: require admin authentication for node registration and add token rotation.

### Admin Misconfiguration

Risk: Incorrect admin settings can point users to wrong docs, break login, or publish misleading audit data.

Controls:

- Settings are stored centrally.
- Admin changes are recorded in audit events.
- Public pages load document links from managed settings.

## Security Assumptions

QnEvt assumes:

- TLS terminates correctly before requests reach the app.
- Browser Web Crypto APIs are available and not compromised.
- Admin credentials and environment secrets are kept private.
- Database backups and server logs are protected.
- Hardware node credentials are provisioned through a trusted channel.

## Recommended Hardening Roadmap

1. Hash API keys at rest and only reveal new keys once.
2. Require admin authentication for node registration.
3. Add explicit proof payloads that support full deterministic recomputation.
4. Add node-signed entropy batch attestations and public verification keys.
5. Enforce API key IP allowlists.
6. Add automated tests for proof verification, auth, quota, and admin settings.
