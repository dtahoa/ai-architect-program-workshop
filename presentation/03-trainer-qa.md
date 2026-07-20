# NovaMart Group 4 - Trainer Q&A

Presentation integrator: **ĐINH XUÂN DŨNG**

| # | Trainer challenge | Defensible answer | Evidence |
|---:|---|---|---|
| 1 | Can you meet 04:00? | The design closes AI writes at 02:00 and requires measured MERLIN P99 <=90 min plus >=30 min reserve. Until measured, influence stays off. | A2 NFR-01 |
| 2 | What about 6x Lunar New Year? | Test 162M forecasts and 42.24M optimizations at 30k/s and 15k/s for three consecutive nights; failure returns to shadow. | A2 NFR-02; supporting 08 |
| 3 | What if AI recovers at 03:30? | It remains closed. Late output is diagnostic only; no rerun or compensating EDI occurs. | A1 flow; A6 §8 |
| 4 | How can 78% inventory be used? | As an uncertain interval with age, calibration, feasibility at both bounds, absolute caps, and suppression—not as shelf truth. | A2 NFR-03 |
| 5 | Can AI failure stop a truck? | No. MERLIN never waits for AI; no override is normal and MERLIN still generates/reviews/sends orders. | A1; A4 |
| 6 | Did you indirectly touch POS or EDI? | No AI route, code, credential, or dependency reaches POS, EDI, suppliers, or MERLIN core. MERLIN alone sends fixed-schema EDI. | A1 diagrams; A2 NFR-05 |
| 7 | Why sidecar rather than event-driven? | Sidecar scores 4.80 and uses supported batch/table/queue contracts. Event-driven needs an undocumented legacy event boundary. | A3 Decision 1 |
| 8 | Is shadow the architecture? | No. Sidecar is the architecture; shadow-assist is the zero-write rollout state. | A3; A6 §6/10 |
| 9 | Is the kill switch executable? | New epoch, stop permits, same-transaction rejection, independent SAP revoke, exact-row cleanup; 30 s/60 s/5 min targets and drills. | A6 §9 |
| 10 | Will review overload managers? | Pilot admission is risk-ranked and capped at 100/night. Excess uses MERLIN; unsafe queue defaults keep shadow-only. | A2 NFR-10; A6 §11 |
| 11 | Are you using an LLM for forecasting? | No. Forecasting is statistical ML and quantities are deterministic. Language models are confined to input handling/non-safety assistance. | Registry D-05; A3 Decision 3 |
| 12 | Is cost credible? | Approximately $3.8M/year and $0.0003856/forecast are planning estimates. Tagged bills and Finance approval are gates. | A2 NFR-09; supporting 08 |
| 13 | How is residency enforced? | Release 1 exports zero member-level loyalty records; classification/DLP blocks affected AI scope while MERLIN continues. | A2 NFR-08; supporting 09 |
| 14 | Can copilot paraphrase an allergen answer? | No. It displays exact approved evidence plus fixed wording/citation or refuses and escalates. Universal refusal also fails. | A2 NFR-07; supporting 09 |
| 15 | Why exactly these five risks? | Inclusion threshold is 15 with irreversible harm as tie-break; omitted contenders score 10 or less. | A5 omitted-contender note |
| 16 | Were matrix weights forced? | Every decision totals 100, uses shared 1/3/5 anchors and a formula, and includes sensitivity. | A3 scoring method |
| 17 | What trade-off did the ADR accept? | Data duplication, reconciliation, certified adapter work, and two-system operations in exchange for isolation and rollback. | A4 Negative consequences |
| 18 | What could be removed? | Optional language assistance, deferred substitution/markdown, and broader influence can be removed. MERLIN continuity and the minimal sidecar remain. | Registry scope; A6 |
| 19 | Could a smaller design work? | Yes: validation, statistical forecast, deterministic optimizer, one certified adapter, disable control, monitoring/audit. Other capabilities are separate or deferred. | A1 container; manifest |
| 20 | What evidence is still missing? | MERLIN P99, adapter/review contracts, three 6x passes, quality/calibration, cost bills, kill drills, residency/safety benchmarks, and funded operators. | A2 production gate; A6 §14 |

Answer pattern: **decision -> number or boundary -> failure behavior -> evidence status**. Never convert a target into a claimed measurement.
