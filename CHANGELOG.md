# Changelog

## 0.1.0 — 2026-04-17

Initial contract covering Phase 1 scope.

### Webhooks
- `commitment_detected` (BrentBot → Penny)
- `status_evidence` (BrentBot → Penny)
- `commitment_status_changed` (Penny → BrentBot)

### Key decisions reflected
- `rep_email` as canonical join key (lowercase-normalized)
- HMAC-SHA256 webhook auth, separate secrets per direction
- `idempotency_key` on every payload, format: `{commitment_id}:{sha256(payload)}`
- `trace_id` propagated across all webhooks for a given commitment
- `context.transcript_excerpt` max 500 chars
- Penny is source of truth for status; BrentBot sends `status_evidence`, never overwrites
- New commitments start `green` (owner assigned at creation in Phase 1)
- Rate limit: 10 req/sec BrentBot → Penny
