# Changelog

## 0.3.0 — 2026-04-17

Locked in Friday alignment meeting: BrentBot surfaces "no SF evidence" flags
in the daily plan (not Penny UI only).

### Added
- `commitment_completed_no_evidence` webhook (Penny → BrentBot): fired once when
  a commitment is marked completed with no prior `status_evidence` received.
  BrentBot renders a daily plan flag: "Marked complete in Penny — no Salesforce
  activity found. Verify with rep."
- `CommitmentCompletedNoEvidencePayload` schema

### Scope impact
- Penny build ticket 2 (status_evidence receiver): also implements the
  "no SF evidence" scheduled flag job that triggers this webhook
- BrentBot ticket 3 (inbound webhook): adds receiver for
  `commitment_completed_no_evidence` + daily plan rendering

## 0.2.0 — 2026-04-17

Corrections and clarifications from Friday alignment review (Brent).

### Fixed
- `meddpicc_field` enum: `identify_pain` → `identified_pain`
- `CommitmentDetectedResponse.status` enum narrowed to `[green]` only in Phase 1; error path clarified (return 404, not amber, if rep unresolvable)

### Added
- Retry semantics documented in info description (5xx backoff, 4xx permanent failure, 404 queued retry)
- Bidirectional rate limits (both directions capped at 10 req/sec)
- `StatusEvidenceResponse` body for `status_evidence` 200 — includes `current_status` so BrentBot can detect a transition without a separate webhook round-trip
- "No SF evidence" flag behavior documented in info description (Penny-side check; BrentBot surfaces separately via execution verification)
- Phase 2 note on `evidence_type` enum (`calendar_event`, `email_activity` deferred)

### Clarified
- `opportunity_id` nullable with explanation: post-call notes can be account-scoped before an opportunity exists; SF Activity attaches to Account directly when null

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
