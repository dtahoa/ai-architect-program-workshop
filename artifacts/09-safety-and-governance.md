# Supporting Evidence - Safety, Residency, and Governance

This is supporting evidence for Artifacts 2 and 6, not a seventh submission artifact.

## Deterministic safety contract

1. A named Food-Safety Content Owner approves the source hierarchy, SKU/market/locale applicability, exact field or passage, fixed locale wording, citation format, and revocation policy.
2. The displayed allergen, recall, sellability, storage, shelf-life, or food-safety response is the exact approved current field/passage plus fixed approved wording and citation.
3. An LLM may classify the input or help with non-safety language. It may not compose, paraphrase, translate, summarize, or repair displayed safety content.
4. Missing, stale, conflicting, unsigned, wrong-SKU, or inexact evidence produces refusal, a hold/check instruction, and human escalation.
5. Every display or refusal records source/version/hash, policy/template version, market/locale, outcome, actor/escalation, and timestamp.

Acceptance requires 100% safety policy correctness, eligible exact-display/citation correctness, and required-refusal precision/recall; generated safety displays must remain zero. Universal refusal fails because eligible cases must be answered exactly. Escalation acknowledgement target is <=5 minutes and signed safety-pack revocation is <=4 hours. These are gates, not measured results.

## Residency and privacy

- Member-level loyalty records and features exported across a country boundary: **0** in Release 1.
- Release 1 uses approved store-SKU aggregates and excludes personal loyalty identifiers from regional sidecar data and audit records.
- Any unclassified or cross-border personal record blocks the affected AI scope while MERLIN continues.
- POS retention and availability remain incumbent obligations; the sidecar neither deletes POS data nor connects to POS.

## Decision governance

- Statistical forecast models and deterministic optimizer rules have separate registries, versions, approvals, metrics, and rollback.
- Every influenced row must have data, model, rules, constraints, uncertainty, publication, and post-generation order correlation evidence before action/audit completion.
- Missing audit evidence blocks the AI action, not MERLIN.
- Markdown, online substitution, and member-level personalization remain non-production until interfaces, fairness, residency, capacity, safety, ownership, and cost gates are separately approved.
