# BrentBot ↔ Penny Integration Contracts

OpenAPI 3.1 contracts for the BrentBot ↔ Penny Phase 1 integration.

**Shared primitive:** A `Commitment` — a promise a sales rep makes to a customer, extracted by BrentBot from post-call notes and tracked to completion by Penny.

## Webhooks

| Event | Direction | When |
|-------|-----------|------|
| `commitment_detected` | BrentBot → Penny | BrentBot extracts a commitment from a post-call note |
| `status_evidence` | BrentBot → Penny | BrentBot finds a Salesforce activity that corroborates a commitment |
| `commitment_status_changed` | Penny → BrentBot | A commitment transitions RAG status in Penny |

## Auth

**Webhooks:** HMAC-SHA256 signature in `X-Signature` header (`sha256=<hex>`). Signed payload: `sent_at + "." + raw_body`. Reject payloads where `sent_at` is >5 minutes old. Separate secrets per direction.

**REST:** `Authorization: Bearer <api_key>`. One key per environment.

## Identity

`rep_email` (lowercase work email) is the canonical join key. Each system maps it to its own internal user ID on receipt.

## Key Rules

- **Penny is source of truth for commitment status.** BrentBot sends evidence; Penny decides whether to transition.
- New commitments always arrive with an owner assigned and start `green`.
- Every payload carries `idempotency_key` and `trace_id`. Receivers deduplicate on `(source, idempotency_key)`.
- Rate limit: BrentBot will not exceed 10 webhook requests/second to Penny.

## Phase 1 Scope

Only `audience: customer` + `category: deliverable` commitments are in scope. All other types are Phase 2.

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
