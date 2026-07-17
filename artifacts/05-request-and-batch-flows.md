# NovaMart Variant B - Request and Batch Flows

Owner: LÊ NGUYỄN SỸ BÌNH (122980, ARD)

## 1. Flow contract

- MERLIN owns every supplier order and is the only system that uses the supplier EDI connector.
- MERLIN never waits for the AI sidecar. No AI response is part of the incumbent scheduler's success condition.
- POS has no request or dependency relationship with the sidecar or copilot.
- Forecast ML, deterministic replenishment optimization, optional non-safety language processing, and deterministic safety rendering are separate paths, permissions, metrics, and audit records. An LLM never composes displayed allergen/food-safety/recall/sellability/storage/shelf-life content.
- A run is publishable only as a complete, current, validated unit. Row-level success never permits a partial cross-boundary publish.
- Shadow-assist uses the same data and computation path but ends before the override-table adapter.

## 2. Nightly forecast and replenishment flow

### Proposed deadline budget

| Time | Maximum stage budget | Exit condition | Failure/bypass |
|---|---:|---|---|
| 22:00-22:20 | 20 min | Immutable nightly snapshots are ready; schema, count, freshness, residency, and checksums pass | Stop AI run; MERLIN proceeds |
| 22:20-00:20 | 120 min | Required forecast partitions complete with approved model/version and confidence outputs | Retry only bounded failed partitions; no LLM fallback |
| 00:20-01:20 | 60 min | Deterministic optimization completes; every candidate passes hard constraints | Suppress invalid rows; fail whole publish scope if completeness policy fails |
| 01:20-01:30 | 10 min | Final allowed internal retry completes | After 01:30, no new retry starts |
| 01:30-01:45 | 15 min | Reconciliation, risk ranking, audit manifest, and current-run checksum pass | Discard incomplete or untraceable run |
| 01:45-02:00 | 15 min | Prepare/checksum outside the live read set; then use a supported <=5 s epoch-guarded activation, or a measured direct transaction whose cap fits the certified <=60 s lock | Stale epoch, expired permit, unsupported staging/activation, insufficient rate, or any partial failure aborts/rolls back; no retry after 02:00 |
| 02:00-04:00 | 120 min reserved for incumbent path | MERLIN generates orders, routes native exceptions, and transmits fixed-schema EDI | Sidecar cannot delay, restart, or extend this path |

The 02:00 gate is a proposed design value, not an observed legacy fact. Production influence remains disabled until the existing MERLIN P99 order-generation, review, and EDI duration plus a 30-minute operating buffer fits between the AI hard close and 04:00. If it does not, the AI gate moves earlier; the 04:00 cut-off never moves.

### Step-by-step narration

1. **Pre-run control check.** At 21:55 the on-prem control plane reads the kill-switch state, approved pilot scope, model and ruleset deployment status, and prior-run cleanup result. A disabled or unhealthy state sets the run to `SHADOW_ONLY` or `BYPASS` before any MERLIN write can be attempted.
2. **Supported snapshot handoff.** At 22:00 the existing MERLIN nightly-batch extension emits a dated, read-only snapshot/start signal. The warehouse provides its approved nightly aggregate. Neither export changes the source schedule. No component queries POS.
3. **Ingestion validation.** The validator records snapshot IDs, source times, schema versions, row counts, checksums, missing-store/SKU rates, and country classification. It labels warehouse and stock data as stale/uncertain. A critical partition or residency failure halts publication for the affected pilot scope.
4. **Feature construction.** The feature pipeline builds store-SKU-day history, promotion/calendar/weather features, last-sync age, shrink behavior, and inventory-uncertainty bands. Release 1 excludes member identifiers and personal loyalty features.
5. **Forecasting ML.** The forecasting service uses an approved statistical model to produce expected demand plus prediction quantiles/confidence. It processes 27M outputs for conservative validation even though 640 x 11,000 is 7.04M stocked combinations. A 6x feature-load test is required. It never uses an LLM to generate numeric forecasts.
6. **Deterministic optimization.** The optimizer takes forecast distributions and applies case pack, shelf life, lead time, DC capacity, bounded inventory, and versioned absolute limits. It has no current-run MERLIN baseline. Initial-pilot quantity is a non-negative whole case pack capped at 2 case packs/store-SKU and the lower of approved shelf-life sell-through, inventory-interval, DC-allocation, and category limits.
7. **Complete-set and risk gate.** The store reconciles all partitions, suppresses out-of-policy/high-uncertainty cases, ranks on absolute quantity/value, uncertainty, product/category risk and capacity, applies pilot scope, and signs the manifest. Missing provenance blocks publication. A relative MERLIN delta is not available or used pre-publication.
8. **Shadow branch.** In shadow-assist, the recommendation is frozen and the adapter is not called. A current-run comparison is shown only after a documented supported queue/reporting output supplies MERLIN's generated quantity/order ID; without that output, the artifact claims timing/quality shadow evidence but no current-run MERLIN comparison.
9. **Influence branch.** After every gate passes, eligible rows enter the adapter. If the existing supported contract provides non-live bulk staging and constant-time pointer/partition activation, rows are staged/checksummed before an epoch-guarded <=5 s activation. Otherwise a direct transaction uses the measured cap `floor(R_cert x (L_cert - T_overhead_p99) x 0.80)`. The 70,400 ceiling needs a raw >=1,173.4 rows/s inside 60 s; with a 10 s overhead reserve and 20% headroom it needs >=1,760 rows/s. Unsupported or slower behavior means a lower cap or shadow-only. Exact keys/hashes are recorded before activation.
10. **Hard close.** At 02:00 the adapter circuit opens to all writes, whether or not the AI run succeeded. Late results are retained for diagnosis only and cannot affect that day's orders.
11. **MERLIN execution.** The unmodified min/max engine runs on its independent schedule. It reads any supported override rows present for the run, otherwise uses its existing result. The sidecar is not a prerequisite.
12. **Native review and post-generation join.** MERLIN sends generated exceptions through its supported queue. The certified contract must expose generated order ID, quantity and correlation keys; the adapter joins them to the exact publication ledger and only then computes a MERLIN-vs-AI delta/adds admitted provenance. If those read fields/correlation/defaults are unavailable, no fourth feed is invented and automated influence stays disabled.
13. **Supplier transmission.** MERLIN's existing EDI connector transmits the authoritative order using the fixed supplier schema before 04:00. The sidecar has no network route or credential to the supplier EDI endpoint.
14. **Outcome and learning.** MERLIN order IDs, queue decisions, later deliveries, sales, waste, and forecast outcomes are linked to the immutable sidecar trace. Training uses a separately approved historical process; a live run never self-modifies its model.

### Sequence view

```mermaid
sequenceDiagram
    autonumber
    participant N as Existing nightly batch
    participant I as Ingestion validator
    participant F as Forecast ML
    participant O as Deterministic optimizer
    participant R as Recommendation store
    participant A as Override-table adapter
    participant M as MERLIN engine
    participant Q as MERLIN review queue
    participant E as MERLIN EDI

    Note over N,E: MERLIN's incumbent schedule does not wait for the sidecar
    N->>I: Supported dated snapshot and start signal at 22:00
    I->>I: Validate schema, freshness, completeness and residency
    alt Critical input failure
        I-->>R: Mark run BYPASS; record evidence
        Note over A: No adapter call
    else Inputs valid
        I->>F: Versioned feature partitions
        F->>O: Forecast quantiles and confidence
        O->>R: Bounded quantities, constraints and reason codes
        R->>R: Apply absolute caps; reconcile complete set and manifest
        alt Shadow mode or policy gate fails
            R-->>R: Freeze recommendation; publish nothing
        else Approved influence mode and complete by 01:45
            R->>A: Eligible current-run set
            A->>A: Validate current epoch in supported activation/direct transaction
            alt Commit fails or clock reaches 02:00
                A-->>A: Roll back AI-owned writes and open circuit
            else Commit succeeds before 02:00
                A->>M: Current-run rows are available in supported override table
            end
        end
    end
    M->>M: Generate authoritative orders using legacy logic and any valid overrides
    M->>Q: Route generated order ID/quantity after generation
    Q-->>R: Certified post-generation correlation fields, if supported
    Q-->>M: Existing human review outcome where applicable
    M->>E: Authoritative fixed-schema order
    E-->>E: Transmit before 04:00
```

## 3. Fail-open and deadline state transitions

| Trigger | State transition | Publication outcome | Ordering outcome |
|---|---|---|---|
| Publication latch absent/disabled/unreachable/expired/non-current | Any -> `BYPASS` | No permit; adapter/DB transaction rejects | MERLIN min/max continues |
| Snapshot absent, stale beyond approved policy, schema-invalid, materially incomplete, or residency-invalid | `VALIDATING` -> `BYPASS` | None for affected publish scope | MERLIN continues with incumbent inputs and rules |
| Forecast/optimization partition failure before 01:30 | `COMPUTING` -> bounded retry | None until complete | MERLIN remains independent |
| Required partition still missing at 01:30 | `COMPUTING` -> `BYPASS` | None; partial set discarded | MERLIN continues |
| Low confidence or high inventory uncertainty on a row | Row -> `SUPPRESSED` | No override for that row | MERLIN result is used |
| Manifest/provenance/checksum missing | `RECONCILING` -> `BYPASS` | None | MERLIN continues |
| Complete shadow run | `RECONCILING` -> `SHADOW_COMPLETE` | Stored outside MERLIN only | MERLIN unchanged |
| Adapter fails, epoch changes, permit expires, or DB/network access is revoked | `PUBLISHING` -> atomic abort/rollback -> `BYPASS` | Staged rows stay inert; direct transaction rolls back; session terminates | MERLIN continues after supported transaction is released |
| Clock reaches 02:00 | Any publish state -> `CLOSED` | No new or retried AI write | Incumbent path has reserved time to 04:00 |
| AI becomes available at 03:30 | Remains `CLOSED` | No late publish and no re-run | MERLIN continues its current authoritative run |

## 4. Category-manager review request flow

1. Before publication, the optimizer marks candidates with absolute quantity/value, forecast confidence, inventory uncertainty, category risk, promotion state, capacity, and constraint reason codes. It does not consume or recreate a current MERLIN min/max result.
2. High-risk, excessive-absolute-quantity/value, fresh, safety-related, low-confidence, and policy-violating recommendations are suppressed; they do not become overrides.
3. In shadow mode, recommendations remain outside MERLIN. Comparison occurs only after a documented queue/reporting output supplies generated quantity/order ID; otherwise no current-run comparison is claimed.
4. In influence mode, after MERLIN generates orders the supported review queue must emit generated order ID/quantity/correlation keys. The adapter joins to the exact publication ledger, computes any delta post-generation, and annotates only admitted orders. Missing fields/correlation keep influence disabled.
5. Admission is limited to the smaller of risk-ranked candidates and measured human capacity. The initial pilot cap is 100 AI exceptions/night across 20 stores and 2 low-risk categories. Lower-ranked items use the incumbent MERLIN result; they are not queued for later accumulation.
6. The category manager sees MERLIN quantity/order ID only from the certified post-generation contract, plus AI quantity, post-generation delta, absolute value at risk, prediction interval, uncertainty, constraints/reasons, data age, versions, and expiry.
7. Accept, reject, edit, and expiry use the native MERLIN queue semantics and are audited with actor and timestamp.
8. Medium-risk AI influence is prohibited until discovery proves the queue's timeout/default behavior cannot block EDI and can safely preserve or restore the incumbent quantity. If that behavior is unavailable, medium-risk cases remain suppressed rather than creating a new integration.

## 5. Kill-switch flow

1. The durable on-prem latch stores signed `{epoch,state,run_id,scope_hash,manifest_hash,not_after}`. Valid progression is `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`; every transition increments epoch and invalid/unreachable/expired state denies.
2. The adapter has no standing write credential. For `ENABLED_PUBLISH`, a local broker issues a single-run/scope/manifest permit valid <=60 s. The certified activation/direct-write path enforces a same-transaction epoch guard immediately before commit; stale state atomically aborts.
3. Either Duty Manager sets `DISABLING` at a new epoch. Latch quorum, broker, DB/network gateway, orchestrator, and session monitor acknowledge within <=30 s. No new write may be accepted after <=60 s; missing acknowledgement automatically invokes the independent SAP stop.
4. The independent SAP break-glass identity revokes permit issuance and adapter DB/network access and terminates active sessions without depending on the adapter/control service. Uncommitted work rolls back and staged rows remain inert.
5. A separate cleanup identity/runbook reconciles the immutable pre-activation exact-key/hash ledger with MERLIN audit evidence, then removes only unconsumed AI-owned rows. Manual rows are never selected. Consumed rows use native pre-EDI correction only; no compensating EDI or post-04:00 rerun exists.
6. `VERIFIED_SAFE` within <=5 min requires no permits, sessions, transactions, or unexplained activated rows plus exact checksum/cleanup and acknowledgement audit IDs. Failure leaves access revoked and state `DISABLING` while MERLIN continues.
7. Re-enable requires cause closure, clean evidence, a current-data `ARMED_SHADOW` run, approvals, and a new epoch; it cannot jump directly to publish.

Drills before pilot and monthly inject control/latch loss, stale epoch cache, check/write race, adapter compromise/failure, missing acknowledgement, out-of-band revoke/session termination, and cleanup failure. Any failure blocks influence.

## 6. Associate-copilot request flow (read-only pilot)

The copilot is a separate associate tool and has no POS, ordering, pricing, or EDI integration.

```mermaid
sequenceDiagram
    autonumber
    actor S as Store associate
    participant L as Store-local cache and retrieval
    participant V as Signed evidence index
    participant P as Safety policy and domain router
    participant D as Deterministic safety renderer
    participant G as Non-safety language service
    participant H as Food-safety human escalation
    participant U as Audit log

    S->>L: Ask planogram, SOP, product or safety question
    L->>L: Verify signed pack, version and allowed staleness
    alt Pack invalid, expired for safety use, or no evidence
        L-->>S: Refuse definitive answer; show safe escalation
        L->>H: Escalation with query and missing/conflicting source details
        L->>U: Record refusal, pack version and reason
    else Approved evidence exists and WAN is unavailable
        L->>P: Route domain and verify exact SKU/market/locale
        alt Safety domain and evidence eligible
            P->>D: Exact approved field/passage and fixed template ID
            D-->>S: Deterministic exact display with citation
        else Non-safety eligible
            L-->>S: Approved cached extract/template with citation
        else Policy expects refusal
            L-->>S: Refusal and escalation
        end
        L->>U: Record offline policy outcome and cited evidence
    else Approved evidence exists and WAN is available
        L->>V: Retrieve versioned passages and product facts
        V->>P: Exact evidence, query and domain facts
        alt Safety domain and evidence exact/current/conflict-free
            P->>D: Approved structured field/passage + fixed template ID
            D-->>S: Exact deterministic display + source/version/section
        else Safety evidence missing/stale/conflicting/inexact
            P-->>S: Refuse definitive answer and escalate
            P->>H: Conflict/missing source references
        else Non-safety question
            P->>G: Non-safety query plus bounded evidence only
            G-->>S: Cited non-safety assistance
        end
        P->>U: Record domain, renderer/template, versions, citations and outcome
    end
```

Safety rules:

- The governed SOP/food-safety set and product master are the only candidate safety sources. Their precedence and accountable approver must be resolved before production use.
- Allergen, sellability, storage, recall, and food-safety answers require exact approved evidence and visible source title/version/section.
- An LLM may classify/translate the input or assist with non-safety questions. It cannot compose, paraphrase, translate, or otherwise generate the displayed safety answer. Safety output is an exact approved structured field/passage in the approved locale plus fixed pre-approved wording and citation.
- Missing, conflicting, unsigned, or disallowed-stale evidence always produces refusal and named human escalation.
- Offline acceptance separates usefulness from safe refusal: >=90% of benchmark-eligible non-safety questions must return a correct cited answer with >=95% answer/citation accuracy; ineligible non-safety refusal precision/recall must each be >=95%. Safety policy correctness, eligible exact-display/citation correctness, and required-refusal precision/recall must each be 100%. A 100% refusal implementation fails.
- Offline p95 must be <=500 ms; online p95 <=1.5 s. Signed-pack coverage and latency are separate from answer/refusal quality.

## 7. Deferred capability flows

- **Fresh markdown:** offline recommendations may be evaluated three times per day, but there is no production price write. Activation requires an authoritative pricing interface, discrimination controls, decision audit, human accountability, cost proof, and a measured 3x/day completion SLO.
- **Online substitution:** no production request flow is claimed. Activation requires the real online-order boundary, a cached serving design proven at p95 <500 ms, safe inventory semantics, residency review, and an owning operations team.

Deferral prevents the architecture from inventing APIs or silently claiming that an NFR is met by a component that does not exist.

## 8. Traceability record

For every forecast/recommendation, persist at minimum:

- run ID, market, store, SKU and forecast/order date;
- input snapshot IDs, source timestamps, schema versions, completeness result and checksums;
- feature pipeline version and inventory-uncertainty band;
- model ID/version, training approval, forecast quantiles, confidence and drift status;
- optimizer and ruleset version, hard-constraint results, suppression reason and approved pilot scope;
- AI recommendation, absolute cap inputs/results and economic/risk rank; no pre-publication MERLIN quantity or delta;
- shadow/publish decision, publication epoch/permit, adapter transaction, exact published keys and hashes;
- latch state/epoch, acknowledgements, credential/session revocation and safe-state evidence;
- post-generation MERLIN quantity/order ID/delta, native review decision/actor/time and EDI reference only when a certified supported queue/reporting output supplies them;
- monitoring alerts, rollback or cleanup result, and later outcome links.

Audit records are append-only and access-controlled. Personal loyalty identifiers are not recorded in Release 1. Missing trace evidence blocks publication but never blocks MERLIN.
