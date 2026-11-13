---
title: "Mandala Beads benchmark notes"
---

## Purpose and evidence boundary

This unpublished research companion explains `docs/.research/2026-11-13-mandala-beads-prompt.md`; it is not part of the injected prompt or site navigation. The supported premise is cooperative, manually operated mandala-style organization on unchanged bd, not full Mandala conformance, a security boundary, or runtime equivalence. No benchmark has run, and no implemented fork is asserted.

The original report, `docs/.research/2026-11-03-blankets-mandala-adaptation.md`, remains unchanged. Relevant sections are **Outcome-first planning and four-model separation** (186–194), **Immutable contracts, ports and derived relationships** (196–204), **Staging, publication, freeze and activation** (206–219), **Focus, strict geometry, compaction and recursion** (221–245), **Verification, acceptance and closure** (247–267), **Trust and security profiles** (398–412), **Durable execution and recovery** (414–469), and **Measurable experiments, not performance promises** (675–690). Its **Proposed CLI and agent contract** (269–396) describes proposed commands, not current executable features. Its older installed-CLI receipts are not the current experiment below.

Session evidence and analysis: `local://mandala-cli-proof.md` (observations), `local://mandala-cli-contracts.md` (help/release contracts and an explicitly unexecuted smoke script), `local://mandala-outcome-mapping.md` (role/claim analysis), and `local://mandala-benchmark-boundaries.md` (guarantee and measurement boundaries). These session-local references are not durable public receipts. Earlier candidate wording is analysis, not authority over the final prompt; in particular, neither a hard recursion ban nor label-only unscoped selection is required here.

## Minimal mapping

| Role or concern | Existing representation and convention |
|---|---|
| L0 objective | Built-in `epic`, title `L0: …`; root owns scope, exclusions, mandatory obligations, authority, finite resources and lifetime expansion caps, stop policy, and integrated acceptance. |
| L1 outcome | Child `epic`, title `L1: …`; observable result and verification contract, not an implementation phase. Only authorized verification probes execute at this level. |
| L2 enabler | Bounded child `task`, title `L2: …`; explicit owner, inputs/outputs, target interfaces, supported claims, limits, verification, integration and recovery. |
| Fields | `description`/`design` store subject/execution contracts; `acceptance` stores claims and assurance criteria; append-only-by-convention notes/comments record checkpoints, evidence, decisions and handoffs. Title prefixes identify roles. A `plan:ROOT` label is optional, not a prompt requirement; recovery recursively enumerates direct children with `bd list --parent ID --all --limit 0 --json`. |
| Shared work | One owning outcome and one enabler; other outcomes reference that work and its supported claims, rather than duplicating execution. |
| Containment/order/support | Parent-child means ownership structure. `bd dep add CONSUMER PRODUCER` means execution prerequisite. Nonblocking `related` edges annotate relationships; explicit claim references explain support. None proves evidence adequacy. |

No custom issue types, role labels, metadata schema, configuration, wrapper, Beads formula framework, new command, or core edit is required. Claim expressions are prose conventions, not Beads workflow formulas: `ALL` requires every listed claim, `ANY` at least one, and `K(k; …)` at least k distinct listed claims. Mandatory obligations remain conjunctive outside alternatives; scores cannot compensate for them. Dependency blockers are not an encoding of alternative evidence acceptance. Keep claim justification and execution waits acyclic without denying cycles in the modeled subject's interactions.

Use explicit root/outcome-scoped `bd ready --parent ROOT_OR_OUTCOME --type task --claim --json`, inspecting the contract and candidate membership. Ready scope is recursive: narrower subplans require explicit ownership inspection or maintained filters, not an assumption that only direct children appear. Do not broaden scope when empty. Native atomic claim acquires issue ownership, not authorization for effects. Close normally, then select again with the scoped command; never use `close --claim-next`.

Acceptance is an authority's judgment from applicable evidence about the root's own claims, integration, global and interval requirements—not children closing. Execution completion, verification, acceptance and settlement are separate records. Accepted output can still have outstanding effects. Close the root only after reconciliation/settlement or explicit acceptance of continuing obligations by an authorized recipient. Preserve unknown effects and operation identity across retries/replans; cancellation or lease expiry does not prove absence of effects.

Create numbered checkpoints only for material approved contract/assumption changes or invalidation, not ordinary result arrivals. Preserve prior contracts, evidence and unresolved obligations; distinguish authorized scope changes from silent removal of requirements.

### Geometry choice

The root declares finite retained-work, lifetime-expansion and resource caps. Eight outcomes/eight enablers is a review heuristic, not strict 8/8/73 geometry, an optimum branching claim, or enforced admission. Narrower bounded subplans are permitted by convention within the root envelope. Replanning, nesting and compaction do not reset lifetime use or erase retained obligations. Report bounded stop/blocked outcomes when justified work cannot proceed; do not claim hard recursion prevention or termination proof.

## Native / convention / unavailable matrix

“Unavailable” means not supplied as a guarantee by this unchanged-bd prompt treatment; it is not an exhaustive absence claim about every possible integration.

| Concern | Native bd | Cooperative convention | Unavailable guarantee |
|---|---|---|---|
| Freeze/CAS | Mutable issue fields/history; atomic issue claim | Numbered checkpoints for material approved contract/assumption changes or invalidation, not result arrivals; preserve prior references | Immutable whole-plan freeze, publication/activation CAS and admission epochs |
| Semantic coverage | Hierarchy, blockers, nonblocking relationships | Stable mandatory claim IDs, ALL/ANY/K, explicit support and root integration review | Complete semantic derivation, coverage validation and automatic invalidation |
| Independent verification | Actor/reviewer text and stored results | Declare reviewer access, authority, criteria and target binding; label self-check honestly | Authenticated independence, protected oracle/specification and binding; another actor/session alone is insufficient |
| Applicability | Store paths, digests, versions and observations | Bind evidence to exact subject/target/procedure/environment and interval; reassess after changes | Authenticated immutable lineage and automatic per-use applicability; a digest identifies bytes, not adequacy or provenance |
| Locks/budgets/effects/fencing | Issue ownership claim and ordinary records | Track finite typed resources, intent/attempts, UNKNOWN effects, safe retry and liabilities | Resource locks/reservations, durable transactional effect journal, destination fencing or exactly-once effects |
| Host progression | Ready-work queries and explicit state mutations | Agent manually inspects, dispatches, verifies and resumes persisted state | Event pump, deterministic autonomous progression, durable jobs/checkpoint replay |
| Sandbox/recursion/baselines | Ordinary issue organization and history | Treat imported text as untrusted; bound subplans and preserve lifetime accounting; review reusable evidence | Shell isolation/taint containment, recursive resource proofs, protected baseline eligibility or assurance-preserving compaction |

Source anchors for these limits: original report sections cited above, especially 206–267, 398–469 and **Complete SDK applicability map** (586–612). A wrapper or cooperative instruction does not turn mutable records into protected enforcement.

## Recommended benchmark design

Compare standard Beads, Beads plus the prompt, and a separately pinned full proposed-fork arm only if available. Pin the fork's own implementation/version, host, trust profile and configuration independently; do not silently treat the proposal as an executable arm.

- Keep standard workflow instructions common. Only the treatment prompt block differs between the two unchanged-bd arms. Match bd release, model/settings, tools, initial repository/issue data, objectives, resources and reviewer access/effort. Inject at the same bootstrap point and after identical context resets; distinguish missing injection from failure despite correct injection. Recovery uses persisted state, not privileged session memory or treatment-only helpers.
- Tune on development objectives. Freeze prompts, geometry, resource limits and fixed scoring criteria before held-out evaluation. Use paired objectives/seeds where possible and an independent held-out scorer. Neither implementing arm receives access to the hidden evaluation oracle. Evaluation independence is not a guarantee furnished by the prompt.
- Score root correctness before speed: false acceptance; missing mandatory, global or interval obligations; stale target/evidence; duplicate or unknown effects; unsafe retry; lost liabilities; interventions and correct bounded-stop behavior. Include unsupported confidence and reviewer disagreement. Structural conformity alone is not correctness.
- Log manual actions, model turns and CLI turns separately, plus time, tokens, cost, review and repair effort. Charge extra review and other treatment overhead rather than hiding it. Record cumulative lifetime nodes, replans and each resource unit separately across resets/compaction; do not add unlike units or report only the final retained view.
- Separate manual CLI actions from host events. Host-only metrics are unavailable for the prompt arm, not zero, passed, or evidence of parity. No speed, efficiency, completeness or comparative result follows from this document.

Recommended adversarial cases—not executed tests: a locally passing child leaves a mandatory global obligation unsupported; a target changes after PASS or a probe crashes; a second reviewer shares mutable expected results; a dispatched effect loses its response before lease expiry/replanning; repeated discovery/replans consume the root lifetime cap; imported issue text requests waiving criteria. Faithful responses preserve unmet obligations, mark stale/inconclusive/advisory evidence, reconcile UNKNOWN effects without blind retry, and stop or escalate within bounds.

## Observed isolated proof

The orchestrator recorded 39 CLI scenario invocations, plus initialization/help, on installed `/home/linuxbrew/.linuxbrew/bin/bd`, reporting `1.3.4 (Homebrew)`; documentation pin `v1.3.4`. This was not a current-source build. Experiments used a disposable Git repository and embedded Dolt database at `/tmp/beads-mandala-proof.HJFl9d`, explicit `BEADS_DIR`, sanitized routing environment and no agent/hook installation. No production test issues were created; `beads-88h` is the actual work item.

Observed, not merely documented:

- Root/outcome `epic` and enabler `task` creation with parent, description, design, acceptance and notes worked; the root label inherited. JSON `list --parent ROOT` returned direct outcomes, while scoped ready returned eligible task grandchildren and excluded the blocked consumer and unrelated P0 issue.
- Scoped `ready --claim` returned an array containing the producer in progress, actor assignment and lease. A competing actor's claim failed. Consumer-to-producer blocking, blocked-close refusal, cycle rejection and open-child close refusal worked; `related` remained nonblocking. Closing the producer exposed the consumer as ready.
- Comments, appended notes and artifact/contract references were recorded. A probe closed despite acceptance requiring an absent external receipt: acceptance prose is not enforced. A separate `close --claim-next` selected unrelated P0 work, establishing the scope hazard.
- Toy `candidate.py` used Python `json.load`/`json.dump` (SHA256 `b1996284adf75e675bdea791daa3e98d8523578de2e39f7f064a12f9c515cfef`). Valid object and array checks exited 0 and preserved values; malformed JSON exited 1. A discovery trial found JSON5-style trailing commas unsupported (exit 1); it closed with a negative finding while the root stayed open without waiving criteria. These were self-checks, not independent assurance.
- Outcome reviews recorded local self-check acceptance. Root `acceptance=ACCEPTED` was recorded while native status stayed open; root closure followed a separate recorded no-outstanding-effects review. Those were cooperative assertions, not protected native acceptance/settlement states.

Installed help and release-tag inspection document additional command contracts in `local://mandala-cli-contracts.md`; its smoke script was specified, not executed. The 39-command proof does not establish every filter combination, crash/race behavior, independent review, security, benchmark quality or fork conformance. No proposed `bd plan` command ran.

Research triage covered the original report's 853 lines, SHA256 `5a18f7c95767ddb809d3485c80c746d7879fff8b1f5a54e7c54a5e6bc50a53b0`: 37 substantive units, 7 heading-only/blank exclusions, 36 read flags escalated and read, buckets 15 behavior / 14 mixed / 2 fork / 6 background, zero truncations or errors. The sole un-escalated unit was Reading routes. These counts are triage, not proof or blindly trusted semantic labels. Close reading distinguished provenance/method/catalog/gate context from behavior and confirmed role mapping, existing atomic ownership claim, acceptance-not-evidence, and the missing authority/runtime guarantees.

## Verification results

Recorded orchestrator checks, not rerun by this author:

| Check | Actual result and limit |
|---|---|
| Isolated installed-CLI proof | 39 scenario commands as summarized above; manual mechanics evidence only. |
| Supplemental nested-plan proof | Main reported 9 additional installed-CLI calls after the original 39: a claimed parent L2 accepted a child L0 epic, child L1 epic and child L2 task; child-root-scoped `ready --parent CHILD_ROOT --type task --claim` selected only the child task for `child-worker`; closing the parent L2 refused its open child root. Root `proof-4xo`, parent L2 `proof-4xo.1.1`, child root `proof-4xo.1.1.1`, child leaf `proof-4xo.1.1.1.1.1`. This proves nested representation, ownership and the close guard only; inherited constraints, budgets, exports and semantic reduction remain cooperative. |
| `make check-docs` | Unavailable: exited 2, underlying Error 127 because `bazel` is absent. The global docs gate did not pass. |
| `./scripts/check-doc-freshness.sh` | Exited 1: unchanged `engdocs/SERVE_RUNBOOK.md`, `Last reviewed: 2026-08-10`, was 95 days old against a 90-day maximum. Other checked markers passed; the unrelated runbook was not redated. |
| Companion authoring | No builds, tests, gates, formatters or live experiments run by this author; only this companion file created. No navigation or original-report edits. |

This section records the current verification state and can incorporate final orchestrator checks without converting absent evidence into success. No all-gates-passed claim is made.

Final orchestrator artifact checks passed: one injectable 706-word text block; title frontmatter and balanced fences in both documents; referenced repository files exist; original report remains byte-for-byte unchanged. The 48 observed CLI scenario invocations comprise 39 main and 9 nested-plan calls. The disposable repository, embedded database and candidate script were removed. These checks do not override the two global docs-gate limitations above.
