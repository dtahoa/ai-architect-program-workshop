# Artifact 4 - ADR-001: Fail-Open AI Sidecar

Owner: **PHẠM THỊ THANH HUYỀN**
Status: Accepted for replay/shadow; production influence remains gated.

## Context

NovaMart needs forecasting without replacing MERLIN. MERLIN remains order authority; its engine, POS, supplier EDI, and 04:00 cut-off cannot change. Inventory is 78% accurate and AI failure cannot stop deliveries. Only nightly batch, override-table, and post-generation review-queue extensions are supported.

## Decision drivers

Ordering continuity, supported interfaces, failure/release isolation, 27M normal and conservative 6x scale, deterministic quantities, shadow, audit, and rollback.

## Options considered

| Option | Disposition and reason |
|---|---|
| External sidecar | **Selected:** supported isolation, fail-open, scale, and rollback |
| Event-driven boundary | Rejected: requires an unsupported MERLIN event contract |
| Synchronous API gateway | Rejected: makes AI an order-path dependency |
| In-process plugin | Rejected: changes the untested core |
| Shadow-only | Retained for rollout; not an architecture |

## Decision

Deploy an asynchronous sidecar. It reads approved immutable snapshots, runs statistical forecast ML, applies deterministic quantity constraints, and may write bounded rows only through a certified adapter. MERLIN generates, reviews, and sends every order. Post-generation queue fields may return order ID, quantity, and correlation; no pre-publication MERLIN baseline is assumed. Stop retries at 01:30, require completeness at 01:45, and close writes at 02:00.

## Positive consequences

MERLIN continues without AI; POS/EDI stay isolated; AI scales independently; shadow tests the real path; removal needs no MERLIN-code rollback.

## Negative consequences

Duplicated snapshots, reconciliation, SAP certification, two-system monitoring/on-call cost, and intentionally bounded value are accepted.

## Risks

Unmeasured MERLIN timing, adapter locking/cleanup, inaccurate inventory, peak model failure, and unsafe safety content may block influence.

## Rollback

Set a new disable epoch, stop permits, reject writes within 60 seconds, independently revoke SAP access, and clean only exact unconsumed AI rows. Consumed rows use native pre-EDI handling. Verify safe within five minutes; re-entry is shadow-only after cause closure.

## When to revisit

Revisit if interfaces change, measured P99 cannot preserve 30 minutes, adapter safety cannot be certified, repeated 6x/quality/cost gates fail, or an incident disproves fail-open behavior. MERLIN authority remains fixed.
