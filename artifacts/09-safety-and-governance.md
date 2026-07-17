# NovaMart Variant B - Safety and Governance Validation

Owner: TRẦM QUỐC THUẬN (112287, 24R-Humana)

Verdict: **PASS CONDITIONALLY for shadow mode and non-safety copilot evaluation. Replenishment influence and definitive food-safety/allergen answers are not approved until the release gates in Section 12 pass.**

## 1. Prohibited-condition challenge

| Rejection condition | Evidence in the proposed design | Validator result |
|---|---|---|
| Cannot complete before the operational deadline | The sidecar hard-closes at 02:00; peak stage arithmetic calculates 198.6 minutes against a 240-minute sidecar wall. MERLIN retains 120 minutes. Actual MERLIN P99 and 6x benchmark evidence are absent. | **Conditional.** Feasible target, not proven. Reject production influence if MERLIN P99 >90 minutes for the 02:00 gate or any 6x stage misses its floor. |
| Depends on perfectly accurate inventory | Every recommendation requires an age and calibrated interval; uncalibrated/high-uncertainty rows are suppressed. The calculation explicitly recognizes about 1.5488M of 7.04M stocked records may be wrong at the 22% error baseline. | **Pass by design.** Any published row without an interval is a critical violation. |
| Prevents MERLIN ordering when AI fails | MERLIN's scheduler, min/max engine and EDI remain independent; absence of an override is normal; kill switch blocks sidecar writes. | **Pass by design, conditional on failure drills.** Any AI-induced MERLIN wait rejects the integration. |
| Uses an LLM to generate forecasts | Forecasting is statistical ML and replenishment is deterministic optimization. A language model is isolated to input classification/entity extraction/translation and non-safety assistance. | **Pass.** Numeric and displayed-safety prohibited-call counts must remain zero. |
| Modifies POS | No AI runtime, route, code, library or credential touches POS; copilot uses a separate non-POS device/cache. | **Pass.** Any production POS-to-AI dependency is an automatic rejection. |
| Replaces MERLIN | MERLIN remains supplier-order system of record and only EDI sender. | **Pass.** No dual order authority exists. |
| Exceeds budget without defensible ROI | Planning target is $3.8M/year run cost under $4M; one-time delivery envelope is $3.2M. The accounting basis and provider rates are not signed. Annual break-even is 1.58% of identified upside, not a promised return. | **Conditional.** Reject if the $3.2M includes first-year run cost or the signed annual projection exceeds $4M. |
| Gives uncited or generative allergen answers | Safety domains use deterministic exact approved structured-field/passage display in the approved locale plus fixed pre-approved wording and a visible citation. An LLM never composes, paraphrases, or translates displayed safety content; otherwise refusal/escalation. | **Pass by corrected design, not yet approved operationally.** Safety answer mode remains refusal-only until source/accountability/benchmark gates pass. |

## 2. Safety scope and non-negotiable invariants

These controls apply to allergen presence, cross-contact warnings, storage, shelf life, sellability, product withdrawal/recall and food-safety SOP questions:

1. The language model is never an authority or safety renderer. It may classify/translate the user's input and extract candidate SKU/locale entities, but it may not compose, paraphrase, translate, reorder, or otherwise generate the displayed safety answer. It may assist with separately routed non-safety questions.
2. A safety display is allowed only when evidence is exact, current, approved, signed, market/language appropriate and conflict-free. Output is the exact approved structured field or passage plus a fixed pre-approved locale template and visible citation; no free text is generated.
3. Absence of evidence is not evidence of absence. The copilot must never infer "allergen free" from a missing field.
4. A source conflict cannot be resolved by source ranking alone. It triggers refusal, hold/escalation and content-owner review.
5. Offline operation uses the same evidence rules. Connectivity loss does not relax freshness, revocation or citation controls.
6. The customer/associate must be told to inspect the current physical package label for allergen questions; the copilot does not replace the label or a food-safety officer.
7. Safety incidents disable only the copilot safety-answer capability. They do not stop MERLIN ordering or POS.

The workshop source establishes why these controls are binding: `docs/workshop-1-ai-system-design.pdf`, page 2 names the 1,400 versioned legally binding SOP/food-safety documents and the product master allergen/storage fields; page 3 states criminal liability and potentially fatal harm; page 5 requires design for flaky connectivity.

## 3. Authoritative food-safety sources

The following is the required authority policy. It is not considered operational until the Market Legal Owner and Food-Safety Content Owner approve the exact systems, publishers and jurisdictions.

| Question domain | Authoritative source set | Eligibility rule | What the answer cites | Conflict/missing behavior | Accountable owner |
|---|---|---|---|---|---|
| Country recall, legal prohibition or mandatory withdrawal | Current country regulator directive/recall feed on the approved publisher allowlist, plus NovaMart's officially issued market withdrawal notice | Exact market, effective time, product/batch match where supplied, verified publisher/signature, not revoked | Regulator/notice ID, issuing authority, effective timestamp, product/batch scope and locale | Refuse definitive sellability; instruct hold/not-sell; escalate immediately | Market Food-Safety Duty Officer LÊ NGUYỄN SỸ BÌNH (122980, ARD) |
| Packaged-product allergen declaration and cross-contact warning | Current physical pack label; approved manufacturer label/specification; NovaMart product-master allergen record only when traceably derived from the same approved manufacturer version | Exact SKU/GTIN, market formulation, language, pack/version/batch when relevant; no contradictory recall/withdrawal | SKU/GTIN, manufacturer document/label version, allergen field or label panel, effective/approval date | Any mismatch, missing field, translation uncertainty or label/master conflict means no definitive answer; inspect label, hold if needed, escalate | Product Data Steward PHẠM THỊ THANH HUYỀN (164955, PH2) |
| Storage condition, shelf life and product handling | Current approved manufacturer specification/product-master version plus the applicable current NovaMart SOP | Exact product/category, market, storage state, effective version and signed approval | Product/SOP ID, version, section/field, effective date and locale | Refuse and escalate if sources conflict or product state is unknown | Food-Safety Content Owner TRẦM QUỐC THUẬN (112287, 24R-Humana) |
| Store process, sanitation, temperature control and general food-safety procedure | Current effective NovaMart legally binding SOP/food-safety policy for that market and language, subordinate to a current regulator directive | Document state `EFFECTIVE`, approved translation, applicable store/process and no active superseding notice | SOP title/ID, version, section, effective date, market and language | Refuse an operational instruction if no exact applicable section or a regulator conflict exists | Food-Safety Content Owner TRẦM QUỐC THUẬN (112287, 24R-Humana) |
| Current sellability after temperature excursion, damaged pack, expiry ambiguity or possible contamination | Current regulator/NovaMart withdrawal notice plus exact applicable SOP and product storage/shelf-life facts | Observed condition must match the rule's required facts; the copilot may not invent missing temperature, time or damage data | All governing document IDs/versions/sections and the facts supplied by the associate | If any required fact is unavailable, answer "do not sell pending human decision" and escalate | Store Duty Manager TRẦN TRỌNG PHÚ (197320, PH2) |

Precedence by authority is regulator/legal directive first, then current approved NovaMart policy, then approved manufacturer/product facts. This precedence determines which rule governs when scopes are compatible; it does **not** authorize the copilot to pick a winner when content conflicts. A conflict is a safety event.

If NovaMart has no approved country recall/withdrawal source connected to the governed publisher, the copilot cannot answer recall or current-sellability questions. It may only refuse and provide the escalation route.

## 4. Citation contract

Every safety claim shown to an associate must carry a visible citation containing:

- authority/publisher and human-readable source title;
- document, label, notice or product-record ID;
- exact version/revision and effective date/time;
- market and language/locale;
- section, page, paragraph, table, label panel or product-master field;
- exact SKU/GTIN and batch/lot where the source is batch-specific;
- content hash and signed-pack manifest ID in the machine audit record;
- retrieval timestamp and `online` or `offline` mode.

The deterministic renderer accepts only approved field/passage IDs and locale-template IDs. It displays the exact source text/structured value without model-authored paraphrase, adds only fixed approved wording, and verifies exact product/market/version plus output/source hash. Release acceptance is 100% eligible safety exact-display and citation correctness, 100% required-refusal precision and recall, and 100% safety policy correctness on the multilingual benchmark. A benchmark containing eligible cases cannot pass through universal refusal; there is no production sampling allowance.

## 5. Confidence, refusal and escalation

### 5.1 Evidence-state policy

LLM probability, token likelihood and verbal confidence are ignored for safety authorization. The policy engine assigns only these states:

| Evidence state | Conditions | Allowed behavior |
|---|---|---|
| `ELIGIBLE` | Exact product/domain/market/locale match; current effective signed source; revocation manifest <=4 h old; no conflict; exact approved field/passage and locale-template IDs available | Deterministically render the exact approved value/passage plus fixed label-check wording and citation; no LLM output |
| `REFUSE_STALE` | Safety revocation manifest >4 h, content pack >7 days, superseded/revoked/expired source, or freshness cannot be verified | Do not answer; show escalation and safe hold/check instruction |
| `REFUSE_MISSING` | No exact evidence, missing required facts, non-exact SKU/market/language or incomplete citation | Do not infer; refuse and escalate |
| `REFUSE_CONFLICT` | Two approved sources disagree, package differs from product master, or translation changes safety meaning | Do not rank/generate a resolution; advise hold/not-sell and create P1 content incident |
| `REFUSE_INTEGRITY` | Signature/hash invalid, unknown publisher, modified pack or audit unavailable | Disable safety answers on the device/service and escalate |

There is no medium-confidence safety answer. Non-safety planogram/product-knowledge questions may have separate confidence tiers, but they cannot reuse those tiers to authorize food-safety claims.

### 5.2 Refusal behavior

The localized refusal must clearly say that the system cannot verify the answer; it must not imply that the product is safe. For allergen uncertainty it instructs the associate/customer to inspect the current package label and contact the Store Duty Manager or Market Food-Safety Duty Officer. For sellability uncertainty it instructs the associate not to sell/use the item until a human decision. If exposure or symptoms may have occurred, it displays the approved local emergency procedure; it does not offer medical advice.

### 5.3 Human escalation

1. Store Duty Manager receives the immediate local escalation and can isolate/hold the item under existing SOP.
2. Market Food-Safety Duty Officer acknowledges an active safety escalation within <=5 minutes and targets a decision within <=15 minutes when the required facts are available.
3. Food-Safety Content Owner resolves source/version conflicts and can revoke a document or disable a question domain.
4. Product Data Steward corrects exact-SKU product data with dual approval.
5. Legal/Compliance is paged for regulator conflict, potentially affected customers, or suspected criminal/regulatory exposure.

Warning is acknowledgement >3 minutes; critical is >5 minutes, no available duty officer, or an associate receives no safe hold/escalation instruction.

## 6. Document and offline-pack controls

### 6.1 Lifecycle

Every governed item follows `DRAFT -> REVIEWED -> APPROVED -> EFFECTIVE -> SUPERSEDED/REVOKED`. Only `EFFECTIVE` content enters an evidence index or signed offline pack.

Required metadata:

- immutable content ID and version;
- publisher/authority and approved source system;
- market, language, product/category scope and effective/expiry time;
- original-language hash, translation link and translation approver;
- Food-Safety and Legal approval identities/timestamps;
- supersedes/superseded-by links and revocation reason;
- generated index version, chunk IDs and pack manifest hash/signature.

Safety content requires two-person approval: the Food-Safety Content Owner and the applicable Market Legal/Compliance Owner. A product allergen change also requires the Product Data Steward.

### 6.2 Publication and revocation

- Central revocation/policy publication target: <=15 minutes from approved emergency change.
- Pilot device signed-pack coverage before activation: 100%.
- Normal pack age target: <=24 h; stable-content maximum while offline: 7 days.
- Safety revocation-manifest maximum age: 4 h. A device older than 4 h refuses current safety/recall/sellability answers even if its document pack is otherwise within 7 days.
- Any invalid signature/hash disables safety answers on that device immediately.
- Distribution is to the separate copilot device only; no POS deployment or dependency is permitted.

Offline acceptance separates usefulness from safe refusal. On a labelled benchmark, >=90% of eligible non-safety questions must return a correct cited answer with >=95% answer/citation accuracy; ineligible non-safety refusal precision and recall must each be >=95%. Safety policy correctness, eligible exact-display/citation correctness, and required-refusal precision/recall must each be 100%. Signed-pack coverage is 100% and offline p95 <=500 ms. A 100% refusal implementation fails even if every refusal is fast. A stale recall manifest correctly requires refusal but does not count as an eligible answer.

## 7. Safety audit record and accountability

For every safety query, refusal and escalation, the immutable audit must record:

- event/query ID, timestamp, store, device and pseudonymous associate identity;
- market/locale and online/offline state;
- redacted query text or a privacy-preserving representation under the approved retention policy;
- detected domain/intent and exact SKU/GTIN/batch supplied;
- retrieved source IDs, versions, sections/fields, chunk hashes and pack/revocation manifest IDs;
- evidence state, conflict/freshness/signature results and deterministic citation-verifier output;
- displayed answer/refusal and citations;
- input classifier/entity/translation model and prompt versions if used, plus retrieval, policy, exact renderer/template, output/source hash and citation-verifier versions; record that no generative model produced the displayed safety text;
- escalation ID, recipients, acknowledgement, resolution, approvers and any content revocation;
- audit commit ID and retention/legal-hold class.

Audit commitment precedes display of a definitive safety answer. If audit is unavailable, the system returns a refusal and escalation; it never shows an untraceable answer. Minimum retention is 5 years to align with the stated POS retention until Legal approves a market-specific longer/shorter safety schedule. Access is least-privilege and every read/export is audited.

### Accountability matrix

| Decision/control | Accountable | Responsible | Consulted/notified |
|---|---|---|---|
| Authoritative source/policy approval | Food-Safety Content Owner TRẦM QUỐC THUẬN (112287, 24R-Humana) | Content Governance Team NGUYỄN HÒA (122989, PH2) | Market Legal, Product Data Steward |
| Product allergen/version accuracy | Product Data Steward PHẠM THỊ THANH HUYỀN (164955, PH2) | Product Master Team TRẦN THANH PHỤNG (218924, PH2) | Manufacturer/Supplier, Food Safety |
| Market legal interpretation and retention | Market Legal/Compliance Owner PHẠM THỊ THANH HUYỀN (164955, PH2) | Legal Operations TRẦN THANH PHỤNG (218924, PH2) | Data Protection Officer |
| Live safety escalation/hold decision | Market Food-Safety Duty Officer LÊ NGUYỄN SỸ BÌNH (122980, ARD) | Store Duty Manager TRẦN TRỌNG PHÚ (197320, PH2) | Food-Safety Content Owner |
| Copilot technical safety controls | Copilot Product Owner TRẦN THANH PHỤNG (218924, PH2) | AI Service Team NGUYỄN HÒA (122989, PH2) | Model Risk, Security, Food Safety |
| Model/prompt/policy approval | Model Risk Owner TRẦN TRỌNG PHÚ (197320, PH2) | ML/Copilot Engineering TRẦN TRỌNG PHÚ (197320, PH2) | Food Safety, Legal, Security |
| Audit integrity and access | Audit Service Owner TRẦN TRỌNG PHÚ (197320, PH2) | Platform/Security Operations PHẠM THỊ THANH HUYỀN (164955, PH2) | Internal Audit, Data Protection |
| Incident declaration and regulator/customer response | Market Legal/Compliance Owner PHẠM THỊ THANH HUYỀN (164955, PH2) | Food-Safety Incident Commander LÊ NGUYỄN SỸ BÌNH (122980, ARD) | Executive, Communications, Store Operations |

Placeholders are acceptable in workshop artifacts but not in an operational release. Missing named accountability keeps safety-answer mode disabled.

## 8. Data residency, privacy and retention

Release 1 uses no member-level loyalty data or personal loyalty-derived features in the regional sidecar. Controls and targets:

| Control | Target | Failure behavior | Warning | Critical | Owner |
|---|---|---|---|---|---|
| Cross-border personal loyalty movement | 0 records and 0 bytes | Block/quarantine partition; bypass affected AI scope | Any unclassified loyalty field/flow | Any personal record/feature crosses the country boundary | Data Protection Officer PHẠM THỊ THANH HUYỀN (164955, PH2) |
| Regional feature schema | 100% fields classified; 0 member IDs/quasi-identifiers not approved | Schema gate blocks run | Classification coverage <100% in test | Unknown/personal field in production regional snapshot | Data Governance Owner PHẠM THỊ THANH HUYỀN (164955, PH2) |
| Safety-query privacy | Redact customer names/contact/health details; pseudonymize associate ID; least-privilege audit | Refuse/store minimal event if safe audit cannot be created | PII detector precision/recall <approved benchmark | Unencrypted sensitive query export or unauthorized access | Privacy Engineering Owner ĐINH XUÂN DŨNG (123015, 24R-Humana) |
| POS retention | Existing authoritative POS data retained >=5 years; sidecar does not alter source lifecycle | Retention exception incident; AI remains detached from POS | Failed retention job/verification | Early deletion or AI change to POS retention | POS Data Owner PHẠM THỊ THANH HUYỀN (164955, PH2) |
| Decision/safety audit retention | >=5 years minimum, immutable, legal hold capable | Block new AI actions if audit service cannot commit | Audit availability <99.99% | Missing/mutable/deleted in-retention record | Audit Service Owner TRẦN TRỌNG PHÚ (197320, PH2) |

The market, regulated field set, treatment of pseudonymous/derived features, approved regions and cross-border support access are still open. Until Legal resolves them, exclusion of member-level loyalty is the only approved regional treatment.

## 9. Forecast, optimizer and replenishment governance

### 9.1 Forecast ML

- Approved model/version, training snapshot, feature version, code revision and approvers are immutable in the registry.
- Release gate: enterprise WAPE <=25%, promotion WAPE <=35%, representative market/category/inventory-quality slices, calibrated intervals and three consecutive 6x timing/cost passes.
- Warning: enterprise WAPE >25%, promotion WAPE >35%, 90% interval coverage <90% or drift statistic beyond the approved baseline.
- Critical: enterprise WAPE >30%, promotion WAPE >45%, interval coverage <85% or any unregistered model. Affected scope returns to shadow; no sidecar row is published and MERLIN independently computes its result.
- No online self-learning. Training and deployment are separate approved processes.

### 9.2 Deterministic optimizer

- Every recommendation is reproducible from forecast distribution, inventory interval, case pack, shelf life, lead time, DC capacity, approved absolute category limits and ruleset version. A current MERLIN min/max result is not available or recreated pre-publication.
- Hard-constraint violation tolerance is zero.
- Initial-pilot recommendation is a non-negative whole case pack capped at 2 case packs/store-SKU and the lower of shelf-life sell-through, interval-feasible quantity, DC allocation and category absolute limit. A relative delta is computed only after a certified post-generation queue/reporting output supplies MERLIN's generated quantity.
- A recommendation must remain feasible across the approved inventory interval. If not, suppress.
- The LLM has no route or credentials to the optimizer or override adapter.

### 9.3 Human review and accountability

- Pilot publication cap: 5,000 low-risk rows/night; AI review cap: 100 exceptions/night.
- Admission is `min(risk-ranked demand, measured reviewer capacity)`; excess has no sidecar override/annotation and MERLIN independently computes/uses its result. It never accumulates.
- Accept/reject/edit/expiry records include actor, reason and MERLIN order reference.
- Review is not a way to launder an unsafe result. Fresh, safety-sensitive, low-confidence, excessive absolute quantity/value, or untraceable recommendations are suppressed pre-publication; an extreme post-generation delta is reviewable only when the certified queue supplies both quantities.
- Until native queue timeout/default semantics are proved non-blocking, medium-risk influence is prohibited.

## 10. Operational safety controls

### 10.1 Replenishment kill switch

- A durable quorum-backed on-prem latch stores signed `{epoch,state,run_id,scope_hash,manifest_hash,not_after}`. Valid transitions are `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`; every transition increments epoch. Missing/unreachable/unsigned/expired/non-current state denies.
- The adapter has no standing write credential. A local broker issues a single-run/scope/manifest permit valid <=60 seconds only for `ENABLED_PUBLISH`. The certified supported activation/direct-write transaction must evaluate the current epoch immediately before commit in the same transaction; stale/expired state atomically aborts/rolls back. If unsupported, influence is prohibited.
- Either Duty Manager creates a new `DISABLING` epoch. Latch quorum, broker, DB/network gateway, orchestrator and session monitor acknowledge <=30 seconds; no new write may be accepted after <=60 seconds.
- Missing acknowledgement invokes an independent SAP break-glass stop with separate authority: revoke permit issuance and adapter DB/network access and terminate active sessions. It does not rely on the adapter/control service; uncommitted work rolls back and staged rows stay inert.
- A separate cleanup identity/runbook reconciles the immutable pre-activation exact-key/hash ledger with MERLIN audit evidence and removes only unconsumed AI rows. Broad deletes/manual-row changes are prohibited.
- `VERIFIED_SAFE` <=5 minutes requires no valid permit, active writer/session/transaction or unexplained row, plus exact cleanup/checksum and acknowledgement evidence. Failure leaves access revoked/state `DISABLING`; MERLIN is not told to wait.
- Re-enable requires cause closure, clean proof, a current-data `ARMED_SHADOW` run and dual approval for a new epoch; it cannot jump directly to publish.

### 10.2 Copilot safety circuit breaker

Separate controls can disable one market, language, content version or safety domain while leaving general non-safety retrieval available. Any unsupported claim, invalid signature, conflict surge or unavailable duty escalation disables the affected safety scope immediately. This circuit breaker never changes MERLIN, POS, ordering or EDI.

### 10.3 Exercise schedule

- Before pilot: full outage, late result, latch/control loss, stale epoch, check/write race, adapter failure/compromise, missing acknowledgement, independent credential/session revocation, exact-row cleanup failure, audit outage, residency canary, offline eligible-answer/refusal and source-conflict exercises.
- Monthly during rollout: the fail-safe kill path including out-of-band stop, and one safety-control drill.
- Before and during Lunar New Year: three 6x load/failure rehearsals, then staffed daily readiness checks for 14 days.
- After every adapter, model, rules, source hierarchy, signing or citation-verifier change: affected regression and failure suite before re-enable.

## 11. Incident thresholds

| Severity | Trigger examples | Required action | Acknowledgement target |
|---|---|---|---:|
| Safety P0 | Unsupported/wrong allergen claim, wrong SKU citation, tampered source, possible customer exposure | Disable affected safety scope, direct safe action/emergency process, page Food Safety/Legal/Security, preserve evidence, assess recall/notification | <=5 min |
| Ordering P0 | AI blocks MERLIN, removes manual rows, causes missed 04:00 cut-off or sends/changes EDI | Global publication kill, SAP incident, preserve MERLIN authority, no post-cutoff sidecar compensation | <=5 min |
| Residency P0 | Personal loyalty data crosses the prohibited border | Stop/quarantine flow, disable affected pipeline, Data Protection and Legal incident | <=5 min |
| P1 | Adapter rollback failure, audit unavailable, safety duty unavailable, cost projected >$4M, interval coverage <85% | Bypass/disable affected scope, owner remediation and release review | <=10 min |
| P2 | Latency warning, WAPE warning, elevated refusal/suppression, page/toil budget breach | Investigate during support window; do not relax safety/deadline gates | <=30 min |

## 12. Governance release gates

The design remains shadow-only, and the copilot remains refusal-only for definitive safety answers, until all gates are evidenced:

1. Country-specific regulator/recall publishers, NovaMart source systems and domain precedence are approved by Food Safety and Legal.
2. Exact owners in the accountability matrix and nightly/market duty rotations are named, trained and funded.
3. Multilingual exact-SKU safety benchmark contains eligible-answer and required-refusal cases and passes with 100% policy correctness, 100% eligible exact-field/passage/template/citation correctness, 100% refusal precision/recall, zero generated/unsupported output, and correct escalation. Universal refusal fails.
4. Signed document lifecycle, emergency revocation <=15 minutes centrally, device pack coverage 100%, 4-hour revocation freshness enforcement and tamper tests pass.
5. Immutable audit commitment, five-year retention, privacy controls and retrieval of complete evidence records pass.
6. Market/field residency rules and 0-byte cross-border controls pass with a canary; personal loyalty remains excluded.
7. Measured MERLIN P99 reserve, certified post-generation order fields/correlation, no pre-publication MERLIN-baseline dependency, same-transaction epoch guard, supported staged activation or measured direct cap/rate/overhead, ownership/locking/rollback/cleanup, independent revoke, 6x capacity/cost and fail-open drills pass.
8. Inventory intervals reach >=90% empirical coverage and no published row lacks an interval or trace.
9. Finance separately approves the $3.2M delivery basis and <=$3.8M operating target with all support costs included.
10. Native human-review capacity and safe timeout/default behavior are measured; the 100/night cap remains until evidence supports change.

## 13. Final safety and governance verdict

The corrected architecture avoids every prohibited structural condition: it preserves MERLIN, keeps POS untouched, fails open, models inventory uncertainty, separates statistical ML/deterministic optimization from optional non-safety language use, excludes personal loyalty from the regional Release 1 path, and forbids both uncited and generative displayed safety answers.

The unresolved controls are material, not editorial. Actual source authorities, named accountable people, multilingual safety evidence, residency field definitions, MERLIN/adapter measurements, support staffing and signed cost treatment do not yet exist in the artifacts. The correct result is therefore **PASS CONDITIONALLY for replay, shadow mode and non-safety copilot evaluation; REJECT for replenishment influence or definitive safety/allergen answers until all ten gates pass**.
