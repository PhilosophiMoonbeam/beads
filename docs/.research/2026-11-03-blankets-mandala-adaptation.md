---
title: "Blankets: adapting the Mandala TaskGraph design to a CLI-first Beads fork"
---

## Recommendation

Preserve Beads as the durable issue graph, identity and history substrate. Add a first-class, deterministic **plan and assurance authority** that references issues, rather than renaming issue statuses or treating metadata as an execution protocol. Mandala contributes outcome contracts, semantic requirement claims, independent verification, revision-aware admission, effect accountability, and separate acceptance and operational closure. Its strongest contribution is making the host—not an agent's narrative—responsible for transitions that follow from frozen contracts. [M01] [M02] [M03c] [M07c]

The recommended architecture is CLI-first and ergonomics-driven: one transactional authority, entitlement-aligned `planops` operation roles, versioned machine-readable commands, independent runner/verifier/effect adapters, **implicit determinism on every CLI transition**, and an optional supervised event pump for background autonomous progression. It requires neither an RLM nor Code Mode, a model-selected scheduler, microservices, nor a new storage engine. Ordinary CLI agents can propose and repair plans; registered native handlers and existing CI can execute bounded work. Critically, **implicit determinism** saves CLI-wielding agents from manual bookkeeping that could only have one valid next step: session-bound CAS cursors, deterministic request-ID derivation, auto-derived relationships/pruned slots/reconciliation manifests, compound `--freeze --activate` pipelines, and **synchronous piggybacked frontier progression** (where completing an L2 task automatically triggers bound verification, updates coverage, evaluates gates, and claims the next ready enabler in one command, extending `bd close --claim-next` and `--continue`). [M01] [M07c] [B22] [B22p] [B25m] A manual CLI workflow is useful and low-ceremony, though **full autonomous Mandala execution parity** additionally requires a reliable background host for asynchronous jobs and the applicable runtime/security conformance evidence. [M13] [M15]

This is an adaptation plan, not an implementation, a release announcement, or evidence that the proposed system works. The report rejects three pitfalls: a metadata-only wrapper cannot enforce atomic cross-entity invariants; building a general RLM platform is not a prerequisite for useful planning and assurance; and exposing raw protocol plumbing as mandatory CLI flags would bury agents in ceremony instead of letting the host compute what frozen contracts already determine. It also rejects equating a closed issue, a structural validation pass, a contract hash, or a model's confidence with objective achievement. [M01] [B01] [B03] [B27]

**Consequential decisions:** D01 explicitly expands the fork's charter; D02 preserves issues and separates four models; D03 uses one plan authority; D04 separates staging/save/freeze/activation; D05 derives coverage from applicable evidence; D06 binds independent specifications and targets; D07 journals effects before dispatch; D08 treats geometry as opt-in policy; D09 delivers CLI-first operation without overstating autonomy; D10 makes managed mutation guards universal at the shared operation boundary; D11–D14 govern artifact lineage, baseline compaction, liveness, and bounded recursion; **D15 mandates implicit determinism and zero-ceremony CLI ergonomics** across both `bd plan` and existing high-frequency `bd` verbs. These decisions are defined below and traced into waves W00–W10 and scenarios S01–S16.

### Reading routes

- Product and evidence: [baselines](#baselines-and-provenance), [method and source index](#research-method-and-mandala-source-index), [capability matrix](#beads-capability-and-gap-matrix).
- Design: [decisions](#adoption-decisions-and-requirements), [architecture](#target-architecture-and-persistence), [planning](#planning-and-revision-contracts), [assurance](#verification-acceptance-and-closure), [CLI](#proposed-cli-and-agent-contract), [security](#trust-and-security-profiles), [durability](#durable-execution-and-recovery).
- Delivery and review: [worked migration](#worked-cli-first-authentication-migration), [implementation waves](#dependency-ordered-implementation-waves), [SDK mapping](#complete-sdk-applicability-map), [conformance](#conformance-scenarios), [v4 refinements](#nine-v4-refinements), [risks and experiments](#risks-open-decisions-and-success-experiments), [research receipts](#research-verification-evidence-and-limits), [source catalog](#beads-source-catalog).

## Baselines and provenance

| Baseline | Observed identity | What it establishes |
|---|---|---|
| Beads checkout | `9f03f32e7a7e5f30d173b0f16d7bc6985d862ad1`; initial working tree reported clean, branch ahead by one commit (`trail init` over `dbe6699bb`) | Exact source basis for the capability analysis; carries `Unreleased` post-`v1.3.2` `main` commits (including schema migration `0073` `arrival_seq`, endpoint credential pinning, and `dbe6699bb` documenting the `v1.3.3` hotfix cherry-pick split vs `v1.4.0`). [B29] |
| Beads default version | `1.3.2`, subject to build-time override | [Version source][B00a] still defaults to `1.3.2` on `main`; does not establish an explicit `1.3.3-rc.1` tag in this checkout. |
| Documentation release pin | `v1.3.2` | [Release pin][B00b] is separate from current source and installed CLI. |
| Installed smoke binary | `bd version 1.3.2 (Homebrew)` | Fifteen behavior checks (P01–P15) in disposable, isolated embedded-Dolt Git repositories; not a build of this checkout. |
| Mandala | `648589d05a7ca79405b6bcf4fd02d8e38a12dac9`; ADD-RLM-MANDALA-01 v4.0, dated 2026-11-02 | Normative architecture and conformance design, not a runnable TaskGraph implementation. The split edition retains the numbered source semantics. [M00] |
| Intended fork release basis | User described `1.3.3-rc.1`, with release/rebrand after upstream `1.3.3` stable | Reconcile the exact upstream `v1.3.3` hotfix tag against `main` (`9f03f32e7`) before implementation or release claims; do not silently relabel `1.3.2` as the release candidate. [B29] |

The working product name here is **blankets**. Existing Beads concepts and commands retain their actual names. Every new command in this report is explicitly **PROPOSED**, shown in a `text` fence, and uses `bd plan` as a design namespace to keep the proposal independent of a future executable name. No code, package, executable, storage directory, generated documentation, or issue prefix is rebranded by this report. Rebranding remains a separate post-upstream-stable decision, after compatibility and migration behavior are established.

Citation links are source-local, revision-pinned by this table, not self-pinning URLs. From this file, `../../` reaches this Beads checkout; `../../../mandala/` assumes a sibling checkout named `mandala` at the pinned revision. These private/local links may not resolve in a standalone clone or published site. Copying the report requires carrying or remapping that source layout; no public remote address for Mandala is asserted. `#Lx-Ly` identifies exact source coordinates, not lines in a research bundle. Short citation keys resolve to the source index or catalog below.

Evidence labels used throughout: **SOURCE** means inspected code/design at those revisions; **SMOKE** means observed installed Homebrew behavior; **PROPOSED** means a future contract; **INFERENCE** means a bounded conclusion, not a proof of absence across all code. Normative Mandala requirements describe the desired design, not existing Beads functionality.

## Research method and Mandala source index

The research first sectioned the split design corpus at heading boundaries, retaining heading context; oversized units would split at paragraph boundaries. The supplied classifier used a maximum 18,000-character section cap and probabilistic judgments over relevance to CLI adaptation. Close reading was required for TaskGraph/mixed units, low-confidence verdicts, unresolved questions, errors, and truncations. This was triage, not automated specification extraction or conformance proof.

| Classification result | Units | Interpretation |
|---|---:|---|
| `taskgraph_contract` | 20 | Direct planning/assurance contracts. |
| `mixed_graph_runtime` | 15 | Transferable contracts mixed with host/runtime mechanisms. |
| `runtime_only` | 4 | Initial label, subject to close-read correction. |
| `unrelated_other` | 12 | Includes architectural context and heading-only material; not all are irrelevant. |
| Total | 51 | Original machine labels, not corrected semantic totals. |
| Close-read units | 44 | All units except seven heading-only/index-pointer sections. |
| Prefilter removals / truncations / failures | 0 / 0 / none | No missing text or classifier execution failure reported. |

The prefilter allowed relevant textual document types and excluded conventional generated/vendor/build components; no source unit was removed by it. The seven sections not escalated were the root `README.md:1–3` and title/pointer-only `1–4` spans of sections 03, 04, 05, 07, 08, and 11. Their substantive numbered-document bodies were included. The architectural `docs/README.md` was close-read despite the initial unrelated label.

Close reading corrected three important judgments without rewriting the original counts: the four architectural-index units are design context; section 10's immutable identity, authorization and lineage are transferable rather than merely runtime-only; section 11's root accounting at lines 80–93 applies to ordinary workers, not just model costs. It also corrected a misleading brief phrase: Beads already has atomic **work ownership claiming**; it lacks the proposed **semantic requirement claim ledger** in the inspected seams. Citation defects in the briefs were not inherited: `readyclaimer.go` ends at 98, `moleculestepper.go` at 209, and the cited close/execution implementations live under `internal/storage/issueops`, not nonexistent public-package files.

The following index covers the complete architectural index and all sixteen numbered sources. Diagrams are orientation, not stronger authority than their detailed text. [M00]

| Key | Exact source and coverage | Adaptation destination |
|---|---|---|
| M00 | [Architecture index, provenance and routes, 1–72][M00] | Baseline, service map, completeness boundary. |
| M01 | [01 Purpose and governing rules, 1–22][M01] | Distinctions, deterministic host, no structural-completeness claim. |
| M02 | [02 Logical services and authority, 1–22][M02] | Logical services, authority intersection, rigor. |
| M03a | [03 Hierarchy and compaction, 30–53][M03a] | L0/L1/L2, slots, immutable settled baselines, lifetime caps. |
| M03b | [03 Recursive depth governance, 75–106][M03b] | Coordinates, boundary isomorphism, attenuation, termination and modality. |
| M03c | [03 Four models and claim coverage, 107–136][M03c] | Separate models, coverage, mandatory constraints and composition. |
| M04 | [04 Planning procedure, 29–49][M04] | Ordered planning, discovery-only frontier, justified iteration. |
| M05a | [05 Common contracts, 5–22][M05a] | Immutable schema, target ports, owner/authority/resources/recovery. |
| M05b | [05 Bindings, recipes and integration, 23–48][M05b] | Exact/future/observation inputs, registration, explicit integration. |
| M06 | [06 Decisions and discovery, 1–18][M06] | Approval separation, experiment validity, liveness. |
| M07a | [07 Relationships and draft publication, 34–65][M07a] | Derived relations, all findings, save/freeze rollback. |
| M07b | [07 Activation/readiness, 66–77][M07b] | Active revision/epoch, reconciliation, admission checks. |
| M07c | [07 Autonomous frontier and interrupts, 78–102][M07c] | Five host transitions, nine interrupt classes. |
| M08a | [08 Oracle governance/evidence, 28–50][M08a] | Separate spec and target binding, authenticated verdict/applicability. |
| M08b | [08 Gates, acceptance and closure, 51–76][M08b] | Snapshot permits, protected acceptance, operational settlement. |
| M09 | [09 Data boundaries and lifecycle, 29–69][M09] | Bootstrap, projections, ephemeral turns, durable waits, recursion. |
| M10 | [10 Artifacts, taint and sandbox, 1–19][M10] | Identity/access, monotonic observations, conditional sandbox guarantees. |
| M11a | [11 Operations/replay, 43–67][M11a] | Semantic IDs, lexical keys, intent-before-dispatch, unknown effects. |
| M11b | [11 Resources/batches/accounting, 68–93][M11b] | Locks/subleases, domain manifests, root reservations. |
| M12 | [12 State, cancellation and recovery, 1–23][M12] | Orthogonal states, epoch spine, recovery and reporting. |
| M13 | [13 Complete logical SDK, 1–34][M13] | Capability/method and error contracts. |
| M14 | [14 Worked lifecycle, 1–17][M14] | Authentication migration, canary, compaction, settlement. |
| M15 | [15 Conformance/readiness, 1–29][M15] | Sixteen adversarial demonstration areas and production prerequisites. |
| M16 | [16 Design basis/refinements, 1–15][M16] | Nine v4 refinements, explicitly mapped below. |

## Beads capability and gap matrix

These are source-grounded capabilities of the inspected checkout, except rows explicitly marked SMOKE. The gap column is an INFERENCE limited to the issue/type/storage/role/CLI/HTTP seams examined; it does not assert an exhaustive absence proof over every extension. Existing primitives should be reused, not rediscovered as proposed features.

| Area | Existing capability: SOURCE unless marked | Gap and adaptation |
|---|---|---|
| Issue identity/content | Mutable issue fields include type/status, acceptance prose, owner/assignee, estimates, time constraints and arbitrary JSON metadata. Metadata validation establishes JSON well-formedness. [B01] [B01v] | Preserve issue IDs and history. Add separate plan contracts, semantic claims, bindings and evidence; acceptance prose is not an oracle. |
| Ready work | SQL filters status, pin/blocked projections, ephemeral inclusion, deferral, type/assignee/label/metadata and hierarchy. [B02] | Useful work eligibility, not authorization against a plan revision, approvals, exact inputs, budgets or resource locks. |
| Atomic claiming | `ready --claim` selects, CASes and hydrates the winning issue in one transaction. Durable wins grant a lease; ephemeral wins do not. Empty eligibility is a normal result. [B03] | Reuse the transaction and ownership pattern. Do not describe semantic claim coverage as a new version of this existing feature. |
| CAS and metadata coordination | `RowVersion`/wire `revision` is an opaque equality-only, deliberately partial lifecycle/ownership token. Single-key metadata CAS supports expected absence/value and explicitly is not authentication. [B01] [B04] | A row token is not aggregate plan-content identity. Add distinct draft/head/epoch CAS and exact contract digests; compare touched issue rows independently. |
| Public roles/layering | Experimental `issueops` exposes role-shaped guarded operations, including Lifecycle and ReadyClaimer. New capability gets its own role/accessor. [B03] [B05] [B05l] | Reuse role boundaries and explicit capability discovery, pin API versions. Do not present the experimental package as a stable blankets SDK. |
| Dependency meanings | `blocks`, `parent-child`, `conditional-blocks`, `waits-for` affect readiness; only the first three belong to the combined static cycle set. Subject relationships are not an assurance graph. [B06] | Preserve existing edge behavior, including nonblocking relationship kinds. Add typed plan relations rather than silently redefining parent-child as pure containment. Include future-output, verification and child waits in managed-plan cycle checks. |
| Raw and guarded batches | Line-oriented `bd batch` runs one transaction and rolls back on error, but its `close` branch directly calls raw `Transaction.CloseIssue`; help explicitly documents the close-policy bypass. Guarded BatchApplier has distinct request keys, per-item rules and expected-version guards. [B07] [B07r] [B08] [B08c] | No uniform close-policy boundary can be assumed. Managed-plan guards must cover raw, guarded, embedded, proxied, update-status and batch paths before activation is enabled. |
| Graph creation | Symbolic graph keys translate to one bounded ApplyBatch request; it will not split an oversized graph across transactions. The source documents a waits-for shape that can stall ready work despite passing its static cycle gate. [B09] [B09a] | Reuse symbolic import/atomic graph creation where compatible, not as saved drafts or full execution-graph validation. |
| Formulas/protos/molecules | `cook` compiles a formula to JSON by default, optionally persists a proto; pouring materializes issue graphs. MoleculeStepper explicitly discovers/optionally claims a next ready step and reports all-step closure. [B10] [B11] [B12] | Pin template digest and resolved variables for candidate plan import. A formula is not a frozen plan, independent oracle, registered executable recipe or autonomous scheduler. |
| Gates | Existing gates are async human/timer/GitHub/bead waits that block workflow steps. [B13] | Keep that meaning. New assurance gates produce snapshot-scoped NOT_READY/DENY/PERMIT; a manually closed gate is not authenticated verification. |
| Close semantics | Public Lifecycle documents blocker/child checks, template and ownership guards, force semantics, and atomic refusal. [B05l] | Work closure remains distinct from objective acceptance and operational settlement. Force cannot create evidence or waive non-waivable plan obligations. |
| Leases | Claim leases are node-local liveness/reclaim data; foreign replica observations can be stale. [B01] [B03] | Reuse ownership recovery only. A claim TTL is neither a destination fencing token nor proof an old external writer stopped. |
| Mutation journal | Off by default; normal mutations share its transaction; raw SQL is not journaled. Branch/replica-local, ignored by Dolt merges, automatically retention-bounded; CLI follow polls. [B14] | Not the effect ledger or autonomous event pump. Add durable intents/outbox and retention protection; schedule from authoritative plan state rather than assuming journal completeness. |
| History and federation | Dolt is the underlying storage/version/sync/merge engine. Issue-version dual-write is separately scoped; ordinals are local and the source warns about disconnected writers. [B15] [B16] | Preserve graph/history movement, but never treat offline merges as admission consensus. Aggregate frozen revision identity is separate from issue-version ordinals. |
| HTTP/client | Versioned issue/workspace routes advertise capabilities; ready and event list/watch exist. `serve` is an issue API, not a durable execution host. [B17] [B17e] [B18] | Add plan roles/routes/client accessors end-to-end, fail closed on missing capability; do not claim SSE or polling executes work. |
| Trust and endpoint pinning | Bearer secret grants the full surface without per-client identity/scopes; actor strings are caller-asserted provenance; server supplies no TLS. [B18] [B19] Recent `main` commits add user-private first-use endpoint pinning (`~/.config/beads/trusted-endpoints.json`) so shared repo configs cannot redirect credentials. [B29] | Authenticated approval/evidence authority and constrained effect sinks are new requirements; extend `trusted-endpoints.json` pinning to verifier, oracle-synthesis and effect-adapter endpoints referenced in shared plans. |
| Piggybacked progression and session cursors | `bd close --claim-next` and `--continue` close a step, auto-close a completed molecule root, claim the next ready step inside the transaction, update `.beads/last-touched`, and print shell-safe `remedy` commands; non-interactive bare `bd close` refuses missing IDs before opening the store. [B22] [B22p] [B22b] [B22m] [B23] | Generalize into synchronous piggybacked autonomic progression across all state-settling `bd plan` and managed `bd close` calls, backed by a workspace session cursor (`.beads/plan-cursor.json`). |
| Context priming, wisps and squash | `bd prime` recovers workflow context and memories after compaction; `bd wisp` creates ephemeral vapor issues (now stamped with `arrival_seq`); `bd mol squash` condenses completed ephemeral children into a permanent closed `Digest` issue and deletes the children. [B24] [B24w] [B24s] [B29] | Extend `bd prime` to inject the active plan bootstrap. Contrast `mol squash` (destructive prose digest) with Mandala's `compactSettledBaseline` (non-destructive zero-slot baseline preserving typed output refs, effect dispositions and invalidation lineage). |
| Sibling `memoryops` plane and role governance | `memoryops` establishes a sibling leaf package outside `issueops`, aliases `beadserrors.ErrValidation`, and derives omitted keys (`"EMPTY MEANS DERIVE IT"`). `ADDING_AN_ISSUEOPS_ROLE.md` codifies the 13-step role checklist (~17 files per role) and `TestEveryRoleMethodHasAContractCase`. [B25] [B25m] [B26] [B26c] | Model `planops` on `memoryops` as a sibling leaf package with four entitlement-aligned `storage.Storage` roles rather than ten separate store accessors, avoiding ~170+ boilerplate wrapper files while keeping logical sub-services modular inside `PlanAuthority`. |
| Transaction retries | UOW can rerun the work callback after SQL serialization failures. [B20] | Persist intent/reservations inside the transaction; dispatch external effects after commit, never inside a replayable callback. |
| Installed behavior: SMOKE | Fifteen checks (P01–P15) exercised dependency readiness, cycles, close guards, acceptance prose, ownership conflict, stale assignee guard, rollback, raw batch bypass, producer-close progression, `--claim-next` + `last-touched` handoff, non-interactive bare-close refusal, wisp `mol squash` digest condensation, and `bd remember` key derivation. | See exact receipts below. They corroborate narrow behavior in installed 1.3.2, not checkout conformance or future design correctness. |

## Adoption decisions and requirements

The following is one coherent architecture; alternatives in research briefs are not competing command surfaces. “Deferred” means a conditional capability with a complete dependency path and refusal semantics, not silently omitted Mandala behavior.

| Decision | Disposition and rationale | Required invariant |
|---|---|---|
| D01 Fork charter | **Adopt explicitly.** Blankets owns planning/assurance/admission policy; upstream Beads explicitly leaves orchestration policy outside core. [B16] | R01: retain issue primitives while documenting the fork-specific semantic boundary before schema/command expansion. |
| D02 Four models | **Adopt.** Work, subject, execution and assurance are different models; reference one another through typed/versioned IDs. [M03c] | R02: containment implies neither execution order nor logical support; whole-objective integration is independently checked. |
| D03 Single authority | **Adopt/adapt.** One store/service serializes each active plan, with shared target locks across plans. Use existing storage transactions, not custom storage-engine recovery. [M02] [M07b] [B16] | R03: no two offline clones may independently act as the same scheduling authority; replication is not consensus. |
| D04 Lifecycle | **Adopt.** Process-local staging, explicit inert save, immutable inactive freeze, then separate activation. [M07a] | R04: publish whole versions under CAS; no implicit save or activation and no intermediate issue creation leaking from staging. |
| D05 Coverage | **Adopt.** Quantified/approved-rubric claims, ALL/ANY/K-of-N expression AST, mandatory conjunctions, support lineage and derived coverage. [M03c] | R05: only applicable evidence establishes claims; planned paths, pruning, completion and confidence never do. |
| D06 Oracle | **Adopt/adapt.** Independently governed immutable spec **and target binding**, approved registry preferred; optional isolated synthesis. [M08a] | R06: freeze both before dependent output evaluation; implementation authority cannot select its own trusted oracle. |
| D07 Execution | **Adopt/adapt.** Native adapters with durable operation/attempt identity and intent-before-dispatch; optional pinned script runtime. [M11a] | R07: unknown/partial effects retain liabilities; no unsafe repeat and no universal external exactly-once promise. |
| D08 Focus geometry | **Adapt.** Default bounded-focus review and configurable caps; opt-in strict Mandala geometry. Reject universal 8/8/73 as an empirical fact. [M03a] [M03b] | R08: strict bounds include retained work; all profiles keep obligations, resource limits and lifetime accounting. |
| D09 Agent/runtime | **Adapt.** CLI agents use bootstrap/next-action; deterministic host advances bound work. RLM, Code Mode and model synthesis are optional adapters. [M09] [M13] | R09: manual operation is labeled manual; autonomy requires a reliably invoked event pump and zero redundant controller turns on deterministic transitions. |
| D10 Shared guards | **Adopt.** All managed mutation paths call the same authoritative guards, including batch/raw lifecycle routes. [B07] [B08] | R10: wrappers, force flags, imports or an alternate transport cannot bypass managed-plan constraints. Privileged direct SQL/local tampering is outside the guarantee. |
| D11 Artifact flow | **Adopt identity/access; conditionally adopt host taint.** Immutable refs, permissions and provenance are core; whole-process taint requires a mediating host. [M10] | R11: possession/hash/source declarations are not permission or authenticity; no shell-wrapper sandbox claim. |
| D12 Replanning/compaction | **Adopt after settlement primitives.** Retain or compact eligible settled work without erasing effects or resetting lifetime budgets. [M03a] [M07b] | R12: no baseline is executable; transitive child closure and current applicability are independently checked. |
| D13 Decisions/liveness | **Adopt.** Discovery validity, authorized alternatives, finite waits and stop policies. [M06] | R13: negative discovery may succeed as an experiment while the objective remains unestablished; equivalent replan loops cannot manufacture progress. |
| D14 Recursion | **Conditional adaptation.** Typed child plans first; isolated model frames only for a model adapter. Known homogeneous work uses a batch; known fitting work stays in-plane. [M03b] [M09] | R14: no recursive admission until ports, reduction witnesses, reserves, ancestry fences, subleases and descendant settlement are implemented. |
| D15 Implicit determinism and agent ergonomics | **Adopt as a primary CLI design rule.** Compute every contract-determined transition, default, reconciliation, and CAS/request token in the host/CLI rather than forcing manual multi-command agent choreography; integrate managed plans into `bd prime`, `bd ready`, `bd show`, `bd close` and `bd doctor`. [M01] [M07a] [M07c] [B22] [B25m] [B27] | R15: omitting optional CAS/request flags uses the actor's last-observed session cursor (`.beads/plan-cursor.json`) and deterministic request-ID derivation without weakening authority CAS; state-settling CLI mutations synchronously run the five deterministic frontier transitions and optionally auto-claim the next ready enabler. |

## Target architecture and persistence

### One engine, several logical roles

The plan authority is a Go application/service layer above the existing storage boundary. Its implementation can run in-process for a CLI command or behind a server, using the same semantic roles and transactions. Logical planner, evidence, admission and operational responsibilities do not require microservices. The storage engine continues to own storage crash safety and SQL concurrency. [M15] [B16]

**PROPOSED logical flow:**

```text
CLI / ordinary agent / optional model controller
  -> session cursor (.beads/plan-cursor.json) + deterministic request-ID derivation
  -> authenticated or explicitly local-trust planops roles
  -> PlanAuthority
       planner + revision CAS + evidence applicability + gates
       admission + root reservations + locks/epochs + intent/outbox
       synchronous deterministic frontier progression on every state change
  -> existing storage/UOW interface -> Dolt-backed authoritative store
  -> after commit: runner / verifier / artifact / effect adapters
  -> authenticated results + reconciliation -> same PlanAuthority
Optional supervised event pump invokes that authority after committed events.
```

The engine—not the CLI adapter—owns freeze, activation, admission, permits, acceptance and closure. CLI preflight provides explanations and ergonomic defaults but cannot be the policy boundary. Internally, `PlanAuthority` decomposes into ten logical responsibilities (`PlanReader`, `PlanDraftEditor`, `PlanPublisher`, `PlanActivator`, `PlanAdmission`, `EvidenceRegistrar`, `VerificationCoordinator`, `DecisionAuthority`, `OperationCoordinator` and `PlanFinalizer`). At the `storage.Storage` and `uow` boundary, however, Beads' role-addition rules show that every distinct role interface costs ~17 files across store accessors, hook/telemetry decorators, fake sentinels, 3-leg conformance wirings (`TestEveryRoleMethodHasAContractCase` and `unwiredContractEntrypoints`), OpenAPI specs, and `serveRoleSource` compile-time stubs, and that roles should split only where the question shape or caller entitlement differs. [B26] [B26c] Following `memoryops`—the repository's established pattern for a sibling domain plane outside `issueops` that aliases `beadserrors.ErrValidation` so `errors.Is` works uniformly across CLI, HTTP and conformance suites [B25]—blankets should introduce a top-level `planops` leaf package exposing **four entitlement-aligned storage roles**:

1. **`planops.Reader`** (unwrapped read surface): bootstrap, show, context projections, next-action, coverage, gate evaluation and operation/batch status (`PlanReader`).
2. **`planops.Planner`** (controller-entitled planning writes): draft apply/save/validate/fork, freeze, activate, baseline compaction, subject-model publish and recipe registration (`PlanDraftEditor`, `PlanPublisher`, `PlanActivator`).
3. **`planops.Executor`** (worker/scheduler-entitled execution writes): work claim/complete, synchronous or background frontier advance, operation intent/reconcile/cancel/handoff, batch submission and child-plan spawn (`PlanAdmission`, `OperationCoordinator`).
4. **`planops.Assurance`** (verifier/authority-entitled governance writes): decision proposals/resolutions, verification spec freeze/run, authenticated evidence import, release promote/recover, protected acceptance and operational closure (`DecisionAuthority`, `VerificationCoordinator`, `EvidenceRegistrar`, `PlanFinalizer`).

Keep narrow roles/capabilities rather than expanding `issueops.Lifecycle`. Add equivalent HTTP routes (`internal/httpapi/spec/openapi.v0.yaml` first, then `make api-gen`) and client accessors with the role; capability/version preflight must refuse incompatible servers. [B03] [B05] [B17] [B26c]

Runner, verifier and effect adapters remain separate responsibilities even if one process hosts them. A runner produces candidate outputs; a verifier supplies authenticated observations under an approved procedure; an effect adapter owns destination semantics and reconciliation. No adapter can self-approve authority or acceptance. All effects—including verifier probes, integration, disclosure, release and recovery—use the same admission/ledger/accounting protocol. [M02] [M09] [M11a]

### Persistence boundaries

| Proposed data | Authority and integrity boundary |
|---|---|
| Plan identity/head | Stable plan ID, root issue ID, authority ID, schema/policy versions, expected draft/head version and active revision/epoch. Plan CAS does not reuse issue RowVersion. |
| Published inert draft | Versioned candidate and findings, stable draft/mutation/entity IDs, parent lineage and request deduplication. May be incomplete but cannot authorize work. |
| Frozen revision | Immutable canonical manifest/digest, claims/contracts, subject version refs, node membership, bindings, support/relations, approved scope decisions, policy and lineage. Immutable child records reference this revision. |
| Assurance | Immutable verification specs, target bindings, approvals, verdict records and acceptance judgments; separate versioned per-use applicability and coverage projections. |
| Execution | Durable logical runs/operations, immutable attempt records, append-only journal, transactional outbox, exact run bindings and result selection. Mutable state projections derive from those records. |
| Resources | Root typed accounts/reservations/liabilities, pass-key allocations, locks/fence generations, child delegations, epoch ancestry, batch/item manifests, cancellation and handoff records. |
| Artifacts | Immutable content refs/manifests and attested lineage; payload service may be external. Store access/trust policy and verifier provenance, not merely a hash in notes. |

Advisory metadata can bootstrap proposed claims, imported template provenance and non-authoritative links while schemas evolve. Existing metadata CAS can coordinate one key. It cannot by itself enforce plan-wide claim preservation, unique operation/pass keys, immutable evidence, atomic reservation plus admission, or foreign-key integrity across a graph. [B01] [B04] D01 therefore justifies first-class validated tables/indexes/foreign keys where those invariants require them, explicitly departing from upstream's metadata-first schema boundary. [B16s] Examples are uniqueness on `(authority, request_id)` and operation identity, uniqueness on live-pass keys, foreign keys from nodes to revisions and issue IDs, and referentially closed evidence dependencies. Canonical frozen/evidence bytes should be stored without JSON-number rewriting; the current history implementation's canonical-byte storage is a useful precedent, not a ready-made plan store. [B15s]

Keep durable semantic records distinct from the optional clone-local issue mutation journal. At initial deployment, one authoritative database owns active admissions and operational ledgers. Replicas may receive immutable plan/evidence/history records for reading, but imported operational rows confer no dispatch authority; resuming elsewhere requires an explicit authority transfer with fencing and reconciliation. Do not let a Dolt merge manufacture a current lease, active revision or trustworthy evidence signer. [B14] [B15]

## Planning and revision contracts

### Outcome-first planning and four-model separation

R02 and R05 preserve Mandala's L0 objective, verification-only L1 outcomes and L2 bounded enablers as plan roles, not aliases for Beads issue types. The root owns scope, exclusions, mandatory constraints, rigor, resource envelope, stop policy and acceptance. An L1 names an observable outcome and verification contract, not a phase checklist. L2 references work issues for actions, discovery, fault-tolerant batches, guardrails or child plans. Explicitly authorized verification probes are permitted at L1; implementation, deployment, migration and recovery remain L2 or contracted host operations. [M03a] [M05a]

The work hierarchy assigns responsibility. The subject model captures interfaces, interactions, state, trust boundaries, failure modes and operating conditions; its cycles can be legitimate. The execution model contains typed prerequisites and bounded iterations, not circular waits. The assurance model contains claim support, composition, assumptions and invalidation with no circular justification. Shared work has one owner and multiple consumers/support relations rather than duplicated enablers. Root integration checks cover emergent properties and incompatible child assumptions. [M03c]

Each semantic claim includes stable ID/version, target scope, observable predicate or approved qualitative rubric, threshold/unit, operating conditions, required observations/interval, acceptance authority and mandatory status. Expressions support conjunction, alternatives and K-of-N; all mandatory constraints remain conjunctive and cannot be compensated by a weighted score. Coverage is `PLANNED`, `ESTABLISHED`, `DEFERRED` or `BLOCKED`. Deferred/blocked obligations retain owner, reason, resolution authority, trigger/deadline and next action. Support paths are planned justification, not evidence. Only the authority derives ESTABLISHED from applicable evidence; ordinary progress changes projections without rewriting the frozen revision. Removing a requirement needs an authorized scope decision, not deletion, pruning or a favorable verdict. [M03c] [M06]

Follow the planning order in M04: frame objective; model the subject; define acceptance and target ports before implementation; identify consequential uncertainty; decompose backward; make composition/resources explicit; review sufficiency/economy; freeze the justified frontier; execute/observe; accept under protection or stop truthfully. A discovery-only revision is valid when admitted work is useful/safe and later obligations remain explicit; it does not imply the entire outcome is feasible or funded. [M04]

### Immutable contracts, ports and derived relationships

A frozen node contract pins identity/version/lineage, scope, responsibility, acceptance/claims, assumptions/decisions, typed inputs/outputs, authority, resources/deadlines, integration/recovery and context completeness. L0/L1 contract target ports describe outcome interfaces independent of L2 implementation choices; L2 outputs declare which target ports they fulfill. Frozen oracle bindings target those ports or canonical subject interfaces rather than implementation node IDs. [M05a]

Inputs are exact immutable artifact bindings, declared future outputs, or authorized observations with selector/time/freshness/snapshot rules. A future binding records producer/revision/interface/port/schema, acceptance condition and deterministic selection rule; it does not invent a nonexistent output hash. Unavailable required future output blocks admission. At admission, exact available inputs and declared deferred observation rules enter an immutable run manifest with contract/recipe/runtime/decision versions and target preconditions. Each deferred observation resolves once into an append-only binding before dependent effects. Replay reuses it; fresh observation is a distinct recorded identity requiring revalidation. No uncontrolled `latest`, silent producer swap or ambiguous schema resolution is allowed. [M05b]

Derive containment, artifact consumption, output/verification/decision/stage prerequisites, pairwise canonical lock conflicts, claim support and invalidation from declarative contracts. Explicit `after` is for non-derivable control flow, not redundant restatement of inputs. Resource conflict does not impose arbitrary order. Validate the expanded event graph including verification, integration, gates and child waits; reject ancestor-completion/resource-release deadlocks. Pre-release permission must not wait on final acceptance if final acceptance requires post-release observations. [M07a]

Recipes for native execution pin handler/version/digest, entry point, tool/schema/dependency constraints, capabilities, ports, effects and declared bounded branches. Formula digest plus variables supplies template provenance, not execution authorization. Generated code is candidate output until explicitly registered, contracted, frozen and admitted; script generation alone grants no permission to run it. A future script adapter additionally implements the AST/call-stack rules in M05b/M11a. Integration contracts name destination, expected version/state, compatibility, authorized operation, owner, post-integration checks and failure path. Component passes do not become composition passes. [M05b]

### Staging, publication, freeze and activation

R04 distinguishes four different things:

1. **Process-local staging:** a request/turn initializes an isolated buffer from expected absence, a published draft version or parent frozen revision. A declarative changeset or delta repair edits only this buffer. Exiting, throwing, cancelling or failing publication discards unsaved staging. A normal CLI invocation does not leave a hidden server session behind.
2. **Published inert draft:** an explicit save atomically CASes the whole candidate into a durable draft version, retaining findings. This is how edits survive separate CLI invocations. It may be incomplete and cannot be admitted, scheduled, accepted or mistaken for a frozen contract.
3. **Frozen revision:** whole-candidate validation plus atomic publication creates immutable content-addressed state, optionally materializing referenced new issues/edges in that same transaction. Freeze does not activate, confer approvals or resolve unavailable future inputs.
4. **Activation:** a separate CAS on expected active revision/epoch installs a frozen revision after reconciliation of prior operations. One active authority governs each plan; compatible old work needs narrow explicit continuation authorization. [M07a] [M07b]

For a CLI, a single apply request stages a changeset and chooses validate-only (`--publish none`), inert checkpoint (`--publish save`), or immutable freeze (`--publish freeze`). Multi-command work saves inert checkpoints, then opens a new request-local buffer against the returned draft version. This preserves Mandala's process-local draft semantics without pretending an exited shell command retains memory. A validate-only request returns all findings and discards staging at exit; a subsequent repair can replay the changeset or target an explicitly saved draft. Failed freeze returns findings then discards unsaved edits, leaving all earlier successful explicit saves intact. When the candidate binds pre-approved registry oracles and no prior in-flight operations require manual reconciliation, adding `--activate` to a `--publish freeze` apply executes the distinct `Freeze -> BindRegistryOracles -> DeriveReconciliation -> Activate -> AdvanceReadyFrontier` authority transitions sequentially in one CLI invocation, stopping fail-closed at the exact boundary if any validation, oracle, or CAS check fails (R04, R15).

Schema, structural, slot, coverage and cross-mutation defects accumulate as stable findings rather than stopping at the first bad edit. The candidate must retain enough invalid data to identify the affected mutation/entity. Authority, capability, information-flow and other safety failures reject immediately; unparseable transport is a request error, not permission to defer safety. A staging acknowledgment says only staged. Whole-candidate checks cover schemas, identities, claim dispositions, relationships, bindings, execution cycles, decisions, recipe compatibility, integration/recovery/verification, bounds/resources and lineage. No failed validation silently weakens a contract. Proposals/approval requests referring to staged entities remain buffered or bind an already-published version, preventing dangling references. [M06] [M07a]

Activation derives retained, baseline-promoted and new-work partitions automatically by diffing the frozen revision against its parent revision; the controller specifies explicit reconciliation directives (`--reconcile FILE`) **only** for prior operations that remain in-flight or unsettled at activation time: continue compatibly, cancel before more effects, quarantine, recover or escalate. When activating an initial revision (`--expected-active none`) or a successor revision whose prior operations have all reached terminal settlement, `--reconcile` is omitted and the host auto-derives the reconciliation manifest (R15). A source cannot both occupy an L2 slot and become its successor baseline. Freeze and activation both recheck settlement, outputs, lineage, applicability, slot/lifetime bounds and expected versions. An old worker completing successfully cannot authorize its own integration after epoch change. [M07b]

### Focus, strict geometry, compaction and recursion

The default blankets profile uses concise outcome views, configurable fan-out/lifetime/resource bounds and explicit broad-plan review. It does not reject a coherent plan merely for exceeding eight outcomes. The strict Mandala profile is opt-in, versioned and enforces these source ceilings; eight is a coordination/presentation bound, not an empirically optimal branching factor. [M03a] [M03b]

| Depth | Strict L1 ceiling | Strict L2 per L1 | Work slots including L0/L1 | Conservative policy |
|---:|---:|---:|---:|---|
| 0 | 8 | 8 | 73 | Root plane. |
| 1 | 4 | 8 | 37 | Recursive admission only with full R14 support. |
| 2 | 2 | 4 | 11 | Enforce parent delegable pool and minimum child quantum. |
| 3 | 1 | 4 | 6 | Leaf by default. |
| Terminal depth up to 4 | 1 | 2 | 4 | Explicit policy override required to reach depth 4; terminal profile forbids further spawn. |

M03b defines attenuation formulas and also a terminal-row override; do not infer one universal depth-four rule from the formulas alone. Freeze the chosen policy, resolve terminal-row precedence explicitly, and default to depth-three leaf behavior until approved. Non-strict profile claims adaptation, not strict geometry conformance. All profiles enforce declared limits, retained-plus-new accounting and explicit lifetime counters.

Unused bounded slots default to PRUNED when mapped claims have valid dispositions; a shared rationale/trigger suffices. PRUNED is not an issue status or a claim waiver. Completed/blocked/verified retained nodes still consume slots. Settled baselines consume none but are non-executable lineage, never hidden batches, recovery or guardrails. Promotion requires host-authenticated completion, applicable passing verification, exact resolved outputs and settlement of every attempt/operation/item/job/effect; suspended work, partial/unknown effects, unresolved recovery and continuing obligations reject it. Recursive sources require all descendant planes operationally CLOSED and no continuing obligation attached to the compacted node. [M03a]

A baseline preserves outputs, artifact refs, operation/effect identities and dispositions, original evidence, support rules/provenance, assumptions, invalidation and historical lineage. Recursive baselines transitively bind descendant revisions/baselines and union their dependencies/effects through an immutable Merkle closure. Current applicability is reevaluated per use; a Merkle proof authenticates included identity, not completeness of the original semantic obligations. Compaction neither redispatches producers nor refunds lifetime instantiations/resources. The root lifetime cap counts total new L2 instantiations across revisions/depths independently of presentation slots. [M03a] [M05a]

Beads already has a conceptual precursor in `bd wisp` and `bd mol squash`: ephemeral vapor steps execute without polluting git-synced durable history until `mol squash` condenses them into a permanent closed `Digest` issue attached to the parent root and deletes the ephemeral child rows. [B24w] [B24s] However, `mol squash` is a **destructive prose condensation**: deleting the children discards structured per-step output bindings, operation/effect dispositions, and per-claim invalidation dependencies. Blankets evolves the familiar `squash` mental model into non-destructive **Settled Baseline Compaction** (`compactSettledBaseline`): historical issue and operation rows remain immutable in storage while the successor revision replaces eligible L2 work slots with zero-slot typed baseline records that preserve exact consumer output resolution and per-use evidence invalidation. [M03a] [B24s]

R14 uses a typed child-plan delegation, without requiring an LLM. A child has canonical ancestry/coordinate, parent-enabler/root boundary mapping, immutable seeds, bijective declared export ports, inherited mandatory constraints, narrowed authority/deadline, budget/reserves and cancellation/integration obligations. Parent siblings consume child exports, not interior nodes. Known homogeneous work uses domain batches; known work fitting the plane stays there. A child is justified by a bounded discover/decide/refine problem or genuine context isolation gain, not as a way around capacity. [M03b] [M09]

Strict recursion enforces a well-founded vector of remaining depth, budget, lifetime instantiations and obligations; reserve recovery, parent verification and local integration before delegating. With alpha = 1 minus those reserve fractions, child allocations sum to at most alpha times the parent pool, and each exceeds the approved minimum viable quantum and node allocation. Different resource units remain separate. If no viable child fits, reject admission. The source's logarithmic heuristics require calibrated overhead/latency inputs; they are policy estimates, not demonstrated runtime guarantees. [M03b] [M11b]

A **machine-checkable reduction witness** must use structured finite scope/obligation sets, a proper subset/partition, or a finite decision-resolution obligation prohibited from re-deferral. For free-form natural-language objectives, a reviewer may approve an advisory narrowing claim, but the system must label it advisory and cannot advertise strict semantic termination proof. A normalized contract fingerprint catches identical structured inputs and some respawn cycles, not arbitrary paraphrase equivalence or semantic completeness. Persist call-spine and horizontal spawn history; allow equivalent respawn only after a relevant approved decision or applicable evidence change under bounded adaptation policy. [M03b]

## Verification, acceptance and closure

### Independent specification and independent binding

R06 requires a verification manifest with claims, procedure digest, target/interface bindings, environment/inputs, thresholds or rubric, sample population/count, observation interval, freshness, limitations, authorized probes and independence policy. Point-in-time success cannot erase prohibited events during an interval. The designated verification authority—not implementation code or an actor string—governs oracle adequacy. [M08a]

Prefer a pre-existing approved registry oracle whose exact specification and claim/target binding versions can be reviewed before output evaluation. Human-authored procedures can enter that governed registry. When new model synthesis is used and independence is required, both specification and binding originate in a separate input-restricted VERIFIER_SYNTHESIS frame receiving only immutable frozen L0/L1 contracts and canonical subject interfaces. Approved decisions enter only incorporated into those contracts. No uncommitted draft, shared controller history, separate decision channel, implementation recipe/code/output or contaminated child context seed is allowed. A second process, identity or service alone does not establish independent judgment. [M08a] [M09]

Freeze both objects before dependent L2 output evaluation, whether or not further approval is required; any required approval binds those same immutable versions. The binding can target a declared future outcome port and resolve deterministically later, without tailoring the oracle to candidate contents. Changes create new versions, repeat the same governance/approval path and invalidate affected evidence applicability. Protected test inputs, expected results, environments and dependencies must be inaccessible to unauthorized implementation actors. Without the required trusted provider/isolation, report NOT_READY for that rigor rather than manufacturing independence. [M08a]

### Evidence, gates and protected acceptance

Only authorized verification services issue trusted records. Evidence binds exact claim/contract/spec/binding/artifact versions, run/procedure/environment, observations, all required samples, verdict, time interval, producer/provenance and limitations. Human qualitative judgments use authenticated rubric/reviewer records through the same boundary. A verifier execution failure is not a passing result and is not automatically a measured subject failure. Immutable verdicts are PASS/FAIL/INCONCLUSIVE; discovery interpretation separately records valid support/opposition/inconclusive/invalid execution. [M06] [M08a]

Applicability is per claim/use: CURRENT/STALE/SUPERSEDED, with revocation/provenance failures additionally blocking use. It depends on artifacts, bindings, contracts, assumptions, decisions, procedures, environment, time and descendant invalidation dependencies. Preserve historical verdicts. Unknown dependency coverage conservatively invalidates potentially affected uses; absence of an event is not proof of freshness. Conflicting applicable evidence uses a declared resolution rule, never favorable-result selection. [M08a]

A gate produces NOT_READY for missing observations/work/decisions, DENY for failed requirements or unacceptable evidence, or PERMIT for one named boundary on a consistent snapshot. Both NOT_READY and DENY block that transition; expected incompleteness need not trigger replanning. Permits bind target/stage, revision/contract, dependency versions, evidence/approvals, expiry and consumption rule and are rechecked at effect or acceptance commit. External destinations lacking conditional writes need explicit race handling/reconciliation. Release policy additionally specifies stage/exposure, entry/exit thresholds, duration/samples, abort/containment/recovery and operational owner. [M08b]

Root acceptance evaluates the root's own expression including integration, interval and operational requirements, not “all children closed.” Optional work may remain incomplete only if its effects cannot defeat the outcome. Before committing, the authority stops conflicting admission, inspects active **and suspended** workers, quiesces/reconciles them or establishes authorized ongoing control/handoff, and atomically fences the applicable epoch. A cancellation request or issue lease alone is insufficient. A one-use boundary token may carry the host's attestation of this state, but the accepting transaction must validate and consume it against the same authority/snapshot; an untrusted runner cannot mint it. [M08b]

Acceptance is an immutable judgment about a snapshot/interval with evidence, approvals, boundary scope/epoch/duration and release conditions. Later authorized change can stale current assurance without rewriting historical acceptance. Operational closure is a different transition: descendants, jobs, effects, reservations and continuing obligations must settle or transfer to an accepting owner under an authorized contract. Handoff records name owner, liability/scope, duration/deadline, release condition and expiry/recovery action; “handoff requested” is not acceptance by the recipient. Required guardrails continue until safe termination or transfer. An unaccepted objective with no admissible work, bounded discovery or tracked external wait becomes BLOCKED, never successful by exhaustion. [M08b] [M12]

## Proposed CLI and agent contract

### One command grammar

All new commands below are **PROPOSED**, not executable documentation for current Beads. The sole namespace is `bd plan`; there is no competing `bd exec`, `bd verify`, top-level `accept` or overloaded existing `promote`. All mutations take durable `--request-id`; all semantic writes take the relevant expected-version/epoch guards. Read commands do not acquire execution authority.

**PROPOSED command families:**

```text
bd plan bootstrap PLAN --json
bd plan show PLAN --revision REV --json
bd plan context PLAN --selector SELECTOR --json
bd plan next PLAN --json
bd plan coverage PLAN --json

bd plan draft apply PLAN --base absent|draft:VERSION|revision:REV
  --changeset FILE --publish none|save|freeze --expected-draft VERSION|absent
  --request-id REQUEST --json
bd plan draft validate PLAN --draft-version VERSION --json
bd plan draft fork PLAN --from REV --publish save
  --expected-draft VERSION|absent --request-id REQUEST --json
bd plan activate PLAN --revision REV --expected-active REV|none
  --expected-epoch EPOCH --reconcile FILE --request-id REQUEST --json

bd plan decision propose|request-approval|resolve|status PLAN --record FILE --json
bd plan subject publish PLAN --record FILE --expected-version VERSION
  --request-id REQUEST --json
bd plan recipe register PLAN --manifest FILE --request-id REQUEST --json
bd plan verify spec import|synthesize|freeze PLAN --manifest FILE --json
bd plan verify run|status PLAN --record FILE --json
bd plan evidence import|status PLAN --record FILE --json
bd plan gate evaluate PLAN --boundary BOUNDARY --json

bd plan work claim PLAN --node NODE --expected-revision REV
  --expected-epoch EPOCH --request-id REQUEST --json
bd plan work complete PLAN --run RUN --result FILE --request-id REQUEST --json
bd plan advance PLAN --expected-revision REV --expected-epoch EPOCH
  --max-admissions LIMIT --request-id REQUEST --json
bd plan host run PLAN --policy POLICY --json
bd plan operation status|reconcile|cancel|handoff PLAN --record FILE --json
bd plan batch submit PLAN --manifest FILE --request-id REQUEST --json
bd plan child spawn|output-ref PLAN --record FILE --json
bd plan release promote|recover PLAN --record FILE --request-id REQUEST --json

bd plan accept PLAN --expected-revision REV --expected-epoch EPOCH
  --boundary-token TOKEN --request-id REQUEST --json
bd plan close PLAN --expected-version VERSION --request-id REQUEST --json
```

The family summary elides guards from some individual lines for readability; it does not make them optional. Versioned request records for decision resolution, spec freeze, evidence binding, operation cancellation/handoff and release carry their expected entity version plus applicable plan revision/epoch. A mutation that also updates issue rows carries their expected row tokens and exact plan-relevant content/dependency digests. Managed issue writes invalidate/reject against the affected plan boundary even when the issue's partial row token would not change. Missing expected guards fail closed, never silently select the latest state.

Draft changesets include stable root/node/claim/relationship IDs, additions/patches/removals, pruning, claim support/dispositions and baseline promotions. A validate-only apply uses request-local staging; a separate validate command inspects a saved draft version without publishing. The freeze publication choice performs whole-candidate validation atomically; it does not first save on failure. Delta repair targets stable mutation/entity handles in a new buffer based on an explicit saved version. A first draft save creates no live work issues for incomplete staged definitions; successful freeze may atomically materialize validated new work and bind returned issue IDs. [M07a]

Work claiming is admission, not a shell permission grant. It returns a run/contract/allowed sink scope; work completion submits declared candidate outputs and operation dispositions, not acceptance. Manual agents can edit an ordinary development workspace, but managed integration/release must pass constrained adapters. `advance` executes a bounded amount of deterministic processing when invoked explicitly or piggybacked onto a state-settling mutation; `host run` is the optional supervised pump for asynchronous background jobs. Only the latter or an equivalent reliable background trigger supplies asynchronous autonomous progression; synchronous piggybacked progression eliminates redundant agent round-trips on local CLI transitions without claiming a background daemon is running.

### Implicit determinism and zero-ceremony agent ergonomics

An LLM agent wielding a CLI thrives when the tool absorbs deterministic ceremony and surfaces only genuine epistemic, implementation, or decision choices (D15, R15). Both Mandala (`M01`: *"Any relationship, slot disposition, reconciliation manifest, call-site step identity, or ready-frontier transition that is mathematically determined by frozen contracts MUST be computed and enforced by the host rather than re-authored through redundant imperative LLM calls"*) and Beads (`memoryops/memories.go`: *"EMPTY MEANS DERIVE IT... A request with no key is the NORMAL case"*; `bd close --claim-next` and `--continue`) share this principle. [M01] [M07c] [B22] [B22p] [B25m] Blankets implements four ergonomic mechanisms above the explicit `PlanAuthority` contract without weakening a single CAS, authority, or settlement invariant:

1. **Workspace session cursor and deterministic request-ID derivation (`.beads/plan-cursor.json`):** Every read/claim/publish response (`bootstrap`, `next`, `show`, `draft apply`, `activate`, `work claim`) updates a gitignored, mode-`0600` workspace cursor file `.beads/plan-cursor.json` (keyed by actor/session ID alongside `.beads/last-touched` [B23]) recording the actor's **last-observed** `(plan_id, draft_version, revision_id, epoch, active_run_id)`. When a CLI call omits `--expected-draft`, `--expected-revision`, `--expected-active`, or `--expected-epoch`, the CLI populates the request's CAS fields from that last-observed cursor—never from an unverified `SELECT MAX(...)` read at mutation time—so if another agent or operator advanced the revision/epoch since this agent last observed the plan, `PlanAuthority` still rejects with a structured CAS conflict. When `--request-id` is omitted, the CLI derives a deterministic token ` Hash(actor, observed_cursor, command, canonical_payload)` so retrying an interrupted CLI call is idempotent while a new logical action after cursor advancement receives a fresh token. Following `AllowLastTouchedFallback` (`cmd/bd/last_touched.go:20–43` [B23]), mutating commands in non-interactive agent sessions still require an explicit target ID whenever more than one plan or claimed run is active, preventing empty-variable shell bugs.
2. **Synchronous piggybacked autonomic progression (implicit `advance` + `--claim-next`):** In CLI-first mode without a background `host run` daemon, forcing an agent to run `bd plan work complete`, then `bd plan advance`, then `bd plan gate evaluate`, then `bd plan work claim` wastes three turns on transitions that are 100% determined by the frozen revision. Instead, every state-settling CLI mutation (`activate`, `work complete`, `decision resolve`, `evidence import`, `verify run`) **synchronously runs the deterministic autonomic frontier loop** (`M07c`) inside the same invocation before returning:
   - Resolves fulfilled `targetPorts` (`fulfillsTargetPort`) from settled producer outputs.
   - Executes pre-bound deterministic/local verifiers (or enqueues asynchronous `VERIFIER_JOB`s and reports `VERIFIER_RUNNING`).
   - Binds authenticated evidence and updates the `ClaimCoverage` projection (`PLANNED` → `ESTABLISHED` or `STALE`).
   - Evaluates affected consistent-snapshot `Gate`s (`NOT_READY` / `DENY` / `PERMIT`) and prepares the single-use `boundary_token` when L0 acceptance holds and active/suspended work is quiesced.
   - Recomputes the ready frontier and—when `--claim-next` / `--continue` is enabled (the default on `bd close <id> --continue` and `bd plan work complete --claim-next`, mirroring `ExecuteCloseBatchWithPolicy` [B22b])—**atomically claims the next ready L2 enabler** for the actor, updates `.beads/last-touched` and `.beads/plan-cursor.json`, and returns the claimed node's hydrated `RunManifest` and bounded context projection in the same JSON response.
3. **Compound lifecycle flags, zero-file defaults, and stdin/inline inputs:** While `saveDraft`, `validateAndFreeze`, and `activate` remain distinct authority transitions (`R04`), `bd plan draft apply PLAN --changeset FILE --publish freeze --activate` pipelines `Stage -> ValidateAndFreeze -> BindApprovedRegistryOracles -> DeriveReconciliation -> Activate -> AdvanceReadyFrontier` in one command, halting fail-closed at the exact stage if validation fails or an external approval/synthesis boundary is `NOT_READY`. When activating `r1` (`--expected-active none`) or a replan with zero unsettled in-flight operations, `--reconcile` is omitted and the host auto-derives the reconciliation manifest (`M07b`). Any `--changeset`, `--manifest`, `--record`, or `--result` flag accepts `-` (stdin) or concise inline flags (`--output port=path`, `--verdict PASS`) so agents do not have to litter the repository with scratch JSON files for simple transitions.
4. **Seamless integration with existing `bd` agent verbs:** Per `engdocs/UI_PHILOSOPHY.md` (*"Minimize cognitive overload... Prefer flags on existing commands"* [B27]), agents continue using their familiar `bd` workflow while gaining Mandala guarantees:
   - **`bd prime`** (`cmd/bd/prime.go` [B24]): Automatically detects an active managed plan and injects the **Mandala Minimal Bootstrap** (active `plan_id`, `revision_id`, `epoch`, L0 mandatory claims, coverage counts, active claimed L2 node/run contract, and exact next command) alongside workspace memories on session start or post-compaction recovery.
   - **`bd ready [--claim]`**: Lists plan-admissible L2 enablers (annotated with L1 milestone, contract digest, and satisfied input bindings); `bd ready --claim` routes through `planops.Executor` (`ClaimWork`), enforcing revision/epoch, decision barriers, locks, and budget reservations and returning the hydrated `RunManifest`.
   - **`bd show <id>`**: Displays the L0/L1/L2 contract summary, `targetPorts` / `fulfillsTargetPort` bindings, claim coverage dispositions (`PLANNED`/`ESTABLISHED`/`DEFERRED`/`BLOCKED`), and evidence applicability alongside issue details.
   - **`bd close <id> [--output port=path] [--claim-next|--continue]`**: When `<id>` is a managed L2 enabler, routes through `PlanAuthority.CompleteWork` + synchronous frontier progression + next-enabler auto-claim, updating `.beads/last-touched`. Attempting `bd close` on a managed L1 Milestone or L0 objective refuses direct issue closure and instead evaluates (or reports) the L1/L0 acceptance gate with an exact shell-escaped `remedy` command (`HandleErrorWithHintRespectJSON` [B28]).
   - **`bd doctor [--fix]`**: Consolidates health and recovery diagnostics (`UI_PHILOSOPHY.md` rule 1 [B27]): detects undispatched outbox intents, `UNKNOWN` effects awaiting reconciliation, orphaned draft buffers, expired subleases, and stale evidence projections.

### Compact next-action and diagnostics

An ordinary agent needs a small, stable bootstrap: hierarchy of trusted instructions; plan/objective/delegation; identity and trust profile; exact schema/skill catalog versions; current revision/epoch; resource/stop limits; how to inspect/project/submit/stop; and a missing-context rule. The agent reads `next` (or receives it piggybacked on the prior mutation's response), performs only the offered class of proposal or authorized work, and relies on the session cursor or explicit CAS tokens. Missing information must produce a diagnostic, not guessed continuity. Bounded projections include source/version, selector/version, trust/confidentiality, truncation/completeness and policy. Required input cannot be silently truncated. Reading documentation in a tool does not inform an already-generated model script; model-dependent action waits for the next actual model turn. [M09]

**PROPOSED illustrative envelope, not an observed result:**

```text
{
  "schema_version": "blankets.plan.v1",
  "authority_id": "auth-local",
  "plan_id": "p-auth",
  "revision_id": "r2",
  "epoch": "3",
  "projection_version": "17",
  "trust_profile": "reviewed",
  "mode": "manual",
  "cursor": {"expected_revision": "r2", "expected_epoch": "3", "expected_draft": "none"},
  "state": {"lifecycle": "ACTIVE", "acceptance": "NOT_READY",
            "current_assurance": "INCOMPLETE", "closure": "OPEN"},
  "autonomic_transitions_applied": ["OUTPUT_RESOLVED", "VERIFIER_DISPATCHED", "COVERAGE_UPDATED", "GATE_EVALUATED"],
  "next_actions": [{"action_id": "a-integration", "kind": "WORK_ADMISSION",
                    "node_id": "n-integrate", "reason_codes": ["INPUTS_BOUND"],
                    "expected_revision": "r2", "expected_epoch": "3",
                    "remedy_command": "bd plan work claim p-auth --node n-integrate --json"}],
  "blockers": [],
  "outstanding_effects": [],
  "source_refs": ["contract:integration-v2"],
  "completeness": "COMPLETE_FOR_SELECTOR"
}
```

The envelope separates semantic coverage, structural validity, admission readiness, work completion, verification, release permits, acceptance/current assurance and closure. `next` is advisory and conveys no reusable execution token. It provides stable explanations, `remedy_command` hints (built through `internal/remedy.CommandArg` shell escaping [B22m]), and source refs rather than dumping the whole graph. Ordering is deterministic under a frozen policy: declared priority, fair-share class/admission age, canonical spine/node ID as final tie break. Record the policy and decision reasons; a stable tie break is not a throughput claim or a mandate for milestone-wide waves. Time/age inputs used for decisions must be recorded/authoritative so replay does not reinterpret them.

**PROPOSED finding/error schema:**

```text
{
  "schema_version": "blankets.error.v1",
  "code": "FUTURE_OUTPUT_AMBIGUOUS",
  "severity": "ERROR",
  "identities": {"plan_id": "p-auth", "draft_id": "d-auth",
                 "mutation_id": "m-bind", "entity_id": "n-integrate",
                 "claim_ids": ["c-interaction"], "relationship_ids": ["rel-input"]},
  "retryable": false,
  "effect_state": "NONE",
  "next_action": {"kind": "REPAIR_BINDING", "authority": "plan-editor",
                  "remedy_command": "bd plan draft apply p-auth --base draft:d-auth --changeset - --publish freeze --json"},
  "diagnostic_ref": "diagnostic:binding-conflict"
}
```

Every finding/error uses stable codes and applicable draft/mutation/entity/operation/attempt/step/spine identities, retryability, known effect state, required next action and an authorized diagnostic ref. A failed validation returns the complete findings array in canonical order; immediate authorization/information-flow failure returns a request error without leaking forbidden details. Categories cover schema, authority/approval, taint, missing input/context, unresolved decision, CAS conflict, recipe mismatch, cycle/resource/ancestor conflict, recursive modality/reduction, budget/deadline, unacceptable evidence, pass-key collision, replay divergence, unknown outcome and recovery failure. Retryability never overrides an unresolved effect. [M13]

Deduplication binds request ID to actor authority scope, operation kind and canonical payload. Same request/payload returns its prior result; changed payload under that ID conflicts. This is distinct from execution-pass uniqueness: a second logical worker call with the same key must fail even for identical input, unless it presents the host's same-operation retry/replay authorization. Return durable job handles separately from completed results. Immutable identities/digests and CAS tokens are strings where numeric precision would be unsafe, following the caution in existing issue/metadata contracts. [B01] [B04] [M11a]

## Trust and security profiles

| Profile | Honest guarantees | Required restrictions |
|---|---|---|
| Local advisory | Structural review, durable state, reproducible references and explicitly untrusted submitted observations. | Local users/processes with filesystem/SQL access remain trusted administrators. No authenticated independence or shell containment claim. Imported evidence is advisory until authenticated/endorsed under policy. |
| Reviewed | Approved immutable contracts/spec/bindings, independent review, authenticated CI/verifier results, guarded integration/release. | Separate verifier authority and protected expected results/environment; real principal-to-role policy, signer trust roots, constrained effect sinks and explicit target preconditions. |
| Controlled/high-assurance | Applicable Mandala authority/artifact/runtime guarantees supported by conformance evidence. | All relevant effects and observations mediated by a trusted host; protected credentials, sandbox/frame controls where used, destination limits, signed provenance and revocation. Missing capabilities make the affected operation NOT_READY/denied. |

An actor string, session ID, owner field or current shared bearer token is not authenticated identity or approval authority. A future server profile must bind principal/tenant/root/contract/revision/epoch/attempt/approvals/reservations and derive delegated permissions. Existing bearer authentication needs an identity/authorization layer and transport confidentiality before it can support reviewed approval roles. [B18] [B19] [M02] Likewise, shared workspace configs (`.beads/config.yaml`, `.beads/.env`, synced Dolt `config` rows, or imported plan manifests) must never redirect verifier, oracle-synthesis, or effect-adapter credentials to an attacker-controlled endpoint or inject executable local settings: blankets extends Beads' user-private first-use endpoint pinning (`~/.config/beads/trusted-endpoints.json`) to every remote verifier, synthesis model, and effect destination referenced by a plan. [B29]

Untrusted issue text, artifacts, imported histories and agent outputs cannot grant capabilities, waive criteria, select authoritative evidence or publish trusted oracles. Signatures attest a recognized producer and envelope integrity, not that measurements are adequate or correct. A digest proves identity of bytes, not safety, independence, provenance truth or semantic completeness. Imports retain original provenance and current trust assessment; copying an evidence record into Dolt does not authenticate it. [M08a] [M10]

An ordinary CLI agent can run arbitrary shell commands if its environment permits them. Bootstrap instructions and flags do not control those effects, prevent reading tests, isolate model context, or enforce filesystem/network taint. Guarantees apply to managed state transitions and controlled sinks, not all behavior of an unrestricted process. For stronger guarantees, use constrained credentials/workspaces and adapters at integration/release/disclosure destinations; sandbox filesystem/network/process/package/time/entropy/secret access under a real execution host. Privileged direct SQL, alternate binaries and local database tampering are administrative escape hatches outside the adversary model, not bypasses the CLI can magically prevent. D10 still requires every *supported managed API path* to enforce guards. [M10]

When a sandbox/model adapter exists, R11 includes M10's exact semantics: clean ref-only pipelines retain per-artifact lineage; observing payload or terminal diagnostics journals confidentiality joins and integrity/trust meets before delivery; every later write/tool/ref-only call/projection/child/batch inherits the accumulated bounds. Helpers, constant arguments, fresh invocations, replay or earlier checkpoints cannot wash them. Declassification/endorsement is explicit, scoped and recorded and does not reset later process state. Child output-port refs are channel 1; status/limitations/diagnostics are observed channel 2. Errors/logs also obey disclosure policy. This is a conditional host obligation, not a promised implementation in a CLI wrapper. [M09] [M10]

## Durable execution and recovery

### Identity, journal and crash protocol

R07 identifies a logical operation independently of attempts: authority/root, semantic node/intent, logical run, pinned handler/recipe digest and complete stable step/item path. Static native handlers use declared domain step IDs; batches use domain item keys, never array index, arrival/completion order or runtime counters. Attempt IDs change on safe retry; operation/effect IDs do not. Revision/spine/epoch, adapter version, canonical inputs, target/effect semantics, approvals/reservations/fences and outputs are recorded provenance. Compatible replans explicitly adopt prior operations/results; a new revision cannot authorize repeating a deployment. [M11a]

A pinned script adapter must parse/register immutable AST identity, instrument full lexical call-stack/fork ancestry, and reject unkeyed dynamic loops/fan-out. Native explicit steps transform this contract without requiring ASTs for Go commands; they cannot claim Code Mode AST conformance. Host pass-key allocation rejects concurrent **and sequential** reuse across the live pass, including helpers and overlapping scopes. A recovery cursor or authenticated retry token may resolve only the same journaled operation; matching key and inputs alone is insufficient. Changed input/target/call-site/adapter/recipe bindings halt before dependent execution. [M05b] [M11a]

The authoritative transaction reserves resources, validates authority/epoch/preconditions, records intent and an outbox entry, then commits. External dispatch happens outside the transaction callback, because UOW callbacks can retry. [B20] Before calling a destination, persist an attempt marker meaning dispatch may have happened. A crash after that marker yields UNKNOWN unless reconciliation establishes otherwise, even if the actual send might not have occurred. A known undispatched outbox record can be resumed; a possibly dispatched effect cannot be blindly retried. Response recording is a later transaction. Local issue mutations can settle atomically with the operation ledger only when both are in the same DB transaction. [M11a]

Journal nondeterministic observations, model outputs/generated code when used, selections, clock/random inputs and label transitions before dependent effects. Checkpoints bind recipe/run identity, journal position, operation states and labels and supplement rather than replace the journal. Durable waits release CPU/RAM permits, retaining storage/reservations/effect liabilities and operation identity; readiness reacquires compute and restores/replays without an LLM. This applies to result, child and batch waits in a durable host, not to an ephemeral CLI/controller process that exits. [M09] [M11a]

### Adapter capability matrix

| Adapter/destination class | Admission/retry contract | Missing capability behavior |
|---|---|---|
| Local transactional mutation | Intent/result and subject mutation can share one transaction where storage supports it. | No external call inside callback; do not mistake SQL retries for remote idempotency. |
| Durable idempotency key | Declare key scope, exact payload matching, persistence/expiry, retry window, internal retries and accounting. | Reconcile outside the supported window; token presence alone is insufficient. |
| Conditional/fenced write | Destination enforces expected target version or monotonic fencing generation. | Host fence alone cannot revoke a dispatched unfenced write. |
| Observable non-idempotent | Reconciliation can prove applied or safely absent against defined consistency limits. | No retry until safe absence or other destination semantics are established. |
| Opaque non-idempotent | At most one possible dispatch; explicitly bounded permitted risk if policy allows. | Lost response becomes UNKNOWN; quarantine resource, retain liability and escalate. |
| Verifier/probe | Approved procedure/input/environment and authenticated result; probes declare effects. | Execution failure is not evidence; unauthorized/missing independence blocks. |
| Child/batch/durable worker | Host owns jobs, immutable manifests/keys, checkpoints, cancellation and descendant settlement. | Reject unsupported awaits/spawn/batch instead of pretending process return resumes work. |
| Proxied authority | Version/capability advertised through role, server and client. | Refuse managed operation if any layer lacks it. |

Every adapter contract declares cancellation behavior, consistency limits, reconciliation/recovery and secret/access handling as well as the above properties. A local ledger and arbitrary external effect generally cannot commit atomically. The system offers accountable at-most-one logical intent and destination-specific execution guarantees, **not universal exactly-once effects**. Compensation is a new linked operation and never erases the original effect. [M11a] [M15]

### Resources, cancellation and settlement

Canonical resource keys, shared/exclusive modes, deterministic acquisition order and wait-for checks govern admission. Claim leases are not these locks. Enforce destination fences where supported; otherwise an unknown old writer quarantines conflicting reuse even after lease expiry. A parent awaiting a child releases any required exclusive lock or delegates a scoped SubLease bound to child coordinate/epoch; reject undelegated ancestor waits with ANCESTOR_RESOURCE_CONFLICT. Host fencing stops new admission but cannot retract an already-dispatched write ignored by its destination. [M11b]

Root accounting reserves before admission and tracks actual consumption, remaining reservations and uncertain liabilities. Money/calls/attempts, time/deadlines, concurrency, CPU/RAM/storage/I/O/output, lifetime nodes/revisions/child calls and optional model tokens are different units, not one generic budget. Child allocation subdivides the parent; do not double charge the same use. Protect recovery, parent verification and local integration reserves plus sibling fair share. Waiting releases only the waiting compute permits; admitted child work/storage/effect liabilities remain. Compaction does not reset any cumulative counter. Exhaustion stops new admission but preserves funded/preauthorized containment or explicit escalation. [M11b]

Fault-tolerant execution batches have immutable item manifest, stable domain identities, handler digest, per-item operation IDs, concurrency/checkpoints, retry/reconciliation policy, failure threshold and completion rule. Reordering preserves identity; changed bindings under the same key reject. Threshold/cancel stops new items, drains/reconciles in-flight effects and reports partial outcomes honestly. This is not the all-or-nothing issue CRUD batch. [B07] [M11b]

Maintain orthogonal lifecycle, execution attempt, effect, verification, applicability, gate and acceptance dimensions. COMPLETED means declared result produced with launched operations accounted for, not verified/accepted. Cancellation separately stops undispatched work, interrupts computation where possible and fences new descendant effects, while reconciling those already dispatched. It does not assert effects were absent and cannot cancel required containment by default. [M12]

Hierarchical epochs bind every ancestor's current epoch along the child spine. Superseding a parent fences descendant admissions/commits; a compatible local child replan changes only the child's epoch and leaves siblings/parent boundary intact—the replan damper. The report adopts that observable prefix-fencing contract but makes no independent O(1) implementation/performance claim. Recovery/containment contracts specify what state they can actually establish, authority, budget, verification and escalation. Quarantine needs owner, scope, review deadline and release condition. Accepted handoff transfers explicit liabilities to an accepting owner; forgotten work is never settlement. [M09] [M12]

### Deterministic progression and interrupts

After activation, a capable host performs five deterministic transitions: admission/dispatch on satisfied prerequisites; governed oracle-synthesis trigger when needed; execution of pre-bound verification on producer settlement; authenticated evidence binding/coverage projection; consistent-snapshot gate evaluation and permitted transitions. A registry oracle avoids mandatory model synthesis. An unavailable required approval/synthesis provider remains a blocker/decision request, not a guessed spec. A manual deployment invokes the same transitions explicitly; an autonomous deployment reliably processes them on activation and state change. [M07c]

| Stable interrupt | Controller/owner boundary |
|---|---|
| WAKE_ON_DISCOVERY_COMPLETE | Experiment and validity check settle before deferred work. |
| WAKE_ON_DECISION_REQUIRED | Unresolved barrier, authority amendment/rejection or reconsideration predicate. |
| WAKE_ON_VERIFICATION_FAILURE | Authenticated FAIL/INCONCLUSIVE, not routine absence. |
| WAKE_ON_EXECUTION_FAILURE | Worker/batch/child terminal failure after contracted retries. |
| WAKE_ON_UNKNOWN_EFFECT | Reconciliation cannot settle a possible effect. |
| WAKE_ON_GATE_DENY_OR_DEADLOCK | Gate denial or no runnable/in-flight work/external wait while acceptance is not ready. |
| WAKE_ON_EVIDENCE_INVALIDATION | Support stales/supersedes with no scheduled replacement producer. |
| WAKE_ON_CAPACITY_OR_BUDGET_WATERMARK | Configured soft root watermark or expansion needing baseline compaction. |
| WAKE_ON_TERMINAL_SETTLEMENT | Protected root acceptance and all obligations settle, or stop policy fires. |

An interrupt need not invoke a model: it can present a bounded next-action envelope to a human/ordinary agent. Do not wake a controller merely to restate derived relationships, bind a predictable result, or choose a mathematically determined next node. Waits have owner/deadline/expiry response; lack of progress consumes explicit retry/replan/turn limits and eventually reports a blocker. [M06] [M07c]

## Worked CLI-first authentication migration

This is a **PROPOSED scenario**, not something implemented or verified during research. It adapts M14 without inventing production thresholds. The approving authority must supply the workload population, load, latency/security/availability thresholds, observation intervals and exposure limits before affected evaluation/release. Example identifiers are illustrative, not existing issue IDs.

### Intake, subject and discovery revision

Plan `p-auth` references an existing objective issue `auth-101` without renaming it. The subject model records the authentication service, workloads, legacy proxy, trust roots, identity issuer, certificate rotation, audit flows, state transitions and rollback destination. Observations, assumptions and approved decisions have separate versions. Root claims are `c-allow` (approved identities succeed), `c-deny` (unauthorized identity/trust paths fail), `c-interaction` (required integrations work), `c-latency` (approved load/latency), `c-availability` (approved interval), and `c-operations` (ownership/recovery). Mandatory security and availability constraints remain conjunctive; component checks do not replace root composition evidence. [M14]

L1 `m-integrated-auth` declares service and audit target ports. Another L1 can cover independently meaningful recovery/ownership outcomes, not a “testing phase.” Decision `d-mechanism` compares workload-identity mTLS alternatives including no-change/deferral, issuer/trust assumptions, migration exposure and residual risk. A bounded discovery enabler references work issue `auth-102`; its contract specifies comparison method, controlled test conditions, measurements, validity, stop/resource limits and affected decisions. Later implementation coverage is DEFERRED with owner and trigger, not fabricated recipe code. Registry oracle/spec and target binding for the frozen discovery/outcome contracts are independently governed; unknown future thresholds remain explicit blocking obligations.

**PROPOSED initial commands; files are schema-validated input manifests, not supplied implementations:**

```text
# Explicit protocol sequence (stateless / multi-step repair):
bd plan draft apply p-auth --base absent --changeset auth-discovery.json --publish save --expected-draft absent --request-id auth-save-1 --json
bd plan draft validate p-auth --draft-version 1 --json
bd plan draft apply p-auth --base draft:1 --changeset auth-discovery-repair.json --publish freeze --expected-draft 1 --request-id auth-freeze-1 --json
bd plan verify spec import p-auth --manifest discovery-registry-binding.json --json
bd plan verify spec freeze p-auth --manifest discovery-spec-versions.json --json
bd plan activate p-auth --revision r1 --expected-active none --expected-epoch 0 --reconcile initial-reconciliation.json --request-id auth-activate-1 --json
bd plan next p-auth --json

# Equivalent implicit-determinism agent sequence (D15, R15: pre-approved registry oracle in changeset, zero prior in-flight work, session cursor + piggybacked claim):
bd plan draft apply p-auth --changeset auth-discovery-complete.json --publish freeze --activate --json
bd ready --claim --json
```

The input records for spec operations include their request IDs/version guards. These commands illustrate ordering, not outputs. A saved draft is inert; r1 freeze is inactive; activation admits only authorized discovery. In the implicit-determinism sequence, `--publish freeze --activate` executes the same distinct freeze, registry-oracle bind, auto-derived empty reconciliation (`expected_active == none`), and activation steps, stopping fail-closed if any check fails, and `bd ready --claim` claims the admitted discovery enabler `auth-102` with its hydrated `RunManifest`. In manual mode, the agent claims discovery, uses its allowed test adapter and submits exact results. In host mode the engine dispatches it and runs the bound validity verifier automatically. A valid negative result can satisfy the experiment while the migration claims remain unestablished. Invalid execution supplies no such finding. Discovery completion raises the defined interrupt; an authorized decision either selects a compatible path with bounded risk or stops the migration.

### Approved contracts and two execution waves

Suppose the authority selects workload-identity mTLS and approves all affected thresholds/populations. Fork r1, incorporate the decision in root/L1 contracts, freeze successor r2, then version/freeze/reapprove any affected oracle specification **and binding** from those frozen contracts before dependent output evaluation. If new bindings/spec refs require publication, attach them through a successor frozen revision before activation; do not mutate r2. For clarity, call that executable successor r3. This avoids a circular demand for an oracle synthesized from an uncommitted draft. The chosen default is separate immutable contract then governed-spec attachment revisions, rather than pretending freeze activates either. [M06] [M08a]

In r3, wave A has two independent enablers: build identity/trust policy artifacts in a private workspace and prepare deterministic endpoint-rotation tooling. Both reference existing/new work issues with pinned native handlers, exact input refs, allowed test effects and target ports. A fault-tolerant batch performs uniform endpoint test preparation using endpoint identity keys, not array positions. The legacy proxy adaptation stays in-plane when a known recipe fits; if genuine bounded discovery remains, a child plan is admitted only after W08 capabilities and R14 witnesses/reserves are available. No RLM is required.

Producer settlement resolves exact output bindings and automatically admits pre-bound component verification in host mode. Passing component records establish only their declared claims. Wave B becomes eligible through those bindings: integration into the designated candidate target under expected target version and canonical locks, then independent root composition/security/load verification. The authority rechecks revision/epoch, approvals, resource/budget admission and output selection at dispatch and integration. If running manually, an operator explicitly invokes bounded advance/claim (or relies on piggybacked progression on `work complete` / `bd close --continue`); that intervention is visible in mode/metrics, not called background autonomous.

**PROPOSED representative execution commands:**

```text
# Explicit protocol sequence:
bd plan activate p-auth --revision r3 --expected-active r1 --expected-epoch 1 --reconcile adopt-discovery.json --request-id auth-activate-3 --json
bd plan work claim p-auth --node n-trust-policy --expected-revision r3 --expected-epoch 2 --request-id auth-claim-policy --json
bd plan work complete p-auth --run run-policy --result policy-candidates.json --request-id auth-complete-policy --json
bd plan advance p-auth --expected-revision r3 --expected-epoch 2 --max-admissions 2 --request-id auth-advance-a --json
bd plan gate evaluate p-auth --boundary canary-entry --json

# Equivalent implicit-determinism agent sequence (D15, R15: session cursor + piggybacked verifier/coverage/gate/claim-next on completion):
bd plan activate p-auth --revision r3 --json
bd close auth-103 --output policy_bundle=dist/trust-policy.tar.gz --claim-next --json
```

No command output is predicted as a research result. The manifests must supply exact environment/procedure/target approvals and all guards. In the implicit-determinism sequence, because discovery in `r1` already settled, `bd plan activate p-auth --revision r3` auto-derives the retained/settled reconciliation manifest and claims the first wave-A enabler (`auth-103` / `n-trust-policy`); then `bd close auth-103 --output ... --claim-next` routes through `PlanAuthority.CompleteWork`, synchronously runs the bound component verifier, updates `ClaimCoverage`, evaluates the `canary-entry` gate, and atomically claims the next ready enabler in one round-trip. A JobRef remains pending until its declared result and launched operations settle; an unmanaged issue close does not supply evidence. A denied/missing component verifier blocks the dependent integration path without falsifying unrelated claims.

### Invalidation, replan, release and settlement

Suppose an issuer assumption changes after wave B. Old PASS verdicts remain historical; affected uses become STALE. If no pre-authorized replacement producer exists, the engine emits WAKE_ON_EVIDENCE_INVALIDATION. The editor forks r3, preserves claim IDs/obligations and effect lineage, updates the assumption/contract and stages replacement work. A previously dispatched integration with uncertain outcome is reconciled/quarantined or explicitly continued compatibly; it cannot disappear into a baseline or be repeated under a new revision ID.

For strict-capacity illustration, six eligible completed/verified/settled enablers plus four new ones would require ten slots under one L1 and fail. Promote the six to immutable zero-slot baselines, or only two to leave four retained plus four new. In either case four *new* lifetime instantiations are charged; prior effects/output refs/support/evidence remain. A suspended worker, unresolved batch item, unknown deployment, active guardrail or child subtree not CLOSED rejects compaction. Reevaluate every baseline use against the new issuer assumption; stale support does not become current merely because it was compacted. [M14]

After replacement verification and integration establish the candidate claims, evaluate a pre-release gate for canary permission. It cannot require the post-canary availability interval. Consume a scoped permit through the release adapter, observe health for the approved duration/population, and advance exposure only on current permits. Denial stops rollout; an unsafe state invokes preauthorized containment/recovery, not just a warning. A timeout after possible canary application becomes UNKNOWN and blocks conflicting retry until reconciliation. Post-release records, operational ownership and tested recovery then contribute to root acceptance. [M08b]

Before acceptance, stop conflicting optional-work admission and fence active/suspended continuations. Required ongoing monitoring remains under its authorized guardrail or accepted handoff; optional open issue status is not the deciding factor. Commit an immutable root judgment for the specified snapshot/interval, then settle or transfer remaining operational liabilities to an accepting owner. Only afterward close the plan. A later authorized service change may stale current assurance but preserves that historical judgment.

**PROPOSED finishing commands:**

```text
# Explicit protocol sequence:
bd plan draft apply p-auth --base revision:r3 --changeset issuer-replan.json --publish freeze --expected-draft 2 --request-id auth-freeze-4 --json
bd plan activate p-auth --revision r4 --expected-active r3 --expected-epoch 2 --reconcile issuer-effect-dispositions.json --request-id auth-activate-4 --json
bd plan release promote p-auth --record canary-permit-consumption.json --request-id auth-canary --json
bd plan operation reconcile p-auth --record canary-reconciliation.json --json
bd plan operation handoff p-auth --record accepted-monitoring-handoff.json --json
bd plan accept p-auth --expected-revision r4 --expected-epoch 3 --boundary-token boundary-auth --request-id auth-accept --json
bd plan close p-auth --expected-version 9 --request-id auth-close --json

# Equivalent implicit-determinism agent sequence (D15, R15: compound freeze+activate with explicit unsettled-effect disposition; session cursor supplies CAS/boundary tokens):
bd plan draft apply p-auth --changeset issuer-replan.json --publish freeze --activate --reconcile issuer-effect-dispositions.json --json
bd plan release promote p-auth --record canary-permit-consumption.json --json
bd plan operation reconcile p-auth --record canary-reconciliation.json --json
bd plan operation handoff p-auth --record accepted-monitoring-handoff.json --json
bd plan accept p-auth --json
bd plan close p-auth --json
```

The draft and plan version numbers in this example are illustrative CAS values returned by prior proposed operations, not a guarantee of a particular counter sequence. If evidence or settlement is missing, the truthful terminal report names the blocker, known outcomes, current operating state, unresolved effects, continuing owners and remaining resources; it does not claim migration success.

## Dependency-ordered implementation waves

These are design delivery dependencies, not a Markdown task list or authorization to implement now. Each wave follows schema → `planops` storage/operation roles (following the 13-step `ADDING_AN_ISSUEOPS_ROLE.md` and `memoryops` sibling-leaf contract [B25] [B26] [B26c]) → CLI/server adapters; stacked changes should preserve that ordering. Existing public issueops is experimental, so freeze the fork's versioned interface deliberately. [B05] The exit criteria are future acceptance evidence, not checks run for this report.

| Wave | Dependencies and tangible seams | Data/API/CLI delivery and exit evidence |
|---|---|---|
| W00 Charter, baseline and trust | No code prerequisite. Resolve intended upstream baseline, fork charter [B16], actor/server limitations and endpoint pinning [B18] [B19] [B29]. | Approve D01/D03/D15/profile policy, authority/replication and compatibility model; map all supported managed write routes including raw batch. Exit: no undocumented authority or claim of rc/stable equivalence. |
| W01 Semantic schema and `planops` leaf | W00. `internal/types`, `planops` sibling leaf aliasing `beadserrors.ErrValidation` [B25], storage schema/migrations and driver boundary; preserve current issue tables/IDs. | Stable plan/claim/subject/node/contract/relation/draft/revision IDs and canonical schema versions; FK/unique constraints, request dedupe and aggregate CAS distinct from RowVersion. Exit: import/referential-integrity/schema-upgrade scenarios; no historical close converted to evidence. |
| W02 Planner and transactional publication | W01. Reuse graph parsing/batch primitives [B08] [B08c] [B09] [B09a] through `planops.Reader` and `planops.Planner`, not CLI-only writes. | Request-local staging, all findings, inert save/delta repair, immutable inactive freeze, deterministic derivation/cycle checks, policy bounds, and `.beads/plan-cursor.json` session-cursor/request-ID derivation (R15). CLI draft/show/coverage and `bd prime` plan bootstrap injection [B24]. Exit: S01–S04/S09, conflict/exit rollback and atomic issue/revision publication. |
| W03 Assurance and decisions | W02 frozen contracts; artifact identity/trust manifests from W01. `planops.Assurance` evidence/approval/applicability role; existing gates remain waits [B13]. | Subject CAS, decisions/discovery, governed registry/spec+binding freeze, authenticated evidence import, invalidation, snapshot gates, and `trusted-endpoints.json` verifier pinning [B29]. Exit: S02/S04/S07; forged/stale/incomplete records reject, interval/global checks remain. |
| W04 Managed mutation cutover | W02–W03. Shared lifecycle/update/close/dependency/batch/raw/proxied storage seams [B05l] [B06] [B07r] [B08c] [B09a]. | Managed-membership guard service, plan-relevant content invalidation and issue row guards; existing commands (`bd close`, `bd update`, `bd ready`) either route through `planops` or refuse direct L0/L1 mutation with a shell-escaped `remedy` command [B22m] [B28]. No bypass shim; force never grants acceptance. Exit: every supported front door exercises identical managed invariants, including raw batch-close regression. Activation remains unavailable until W05. |
| W05 Durable admission, piggybacked progression and manual workflow | W03–W04. UOW [B20], `planops.Executor` ledger/outbox/locks/reservations role, ReadyClaimer and `close --claim-next` patterns [B03] [B22] [B22b]. | Activation/auto-derived reconciliation, compound `--publish freeze --activate`, exact run manifests, logical operation/attempt journal, pass keys, cancellation/epoch, adapter capability policy, and synchronous piggybacked frontier progression + `--claim-next` on state-settling CLI mutations (R15). CLI next/work/operation/advance, `bd ready --claim`, `bd close --continue` and `bd doctor` plan health checks [B27]. Exit: S08/S10/S11/S16; crashes around dispatch, stale epoch/target and concurrent root-budget races fail closed. A CLI plan works end-to-end with zero-ceremony ergonomics, explicitly manual for external/async work. |
| W06 Protected acceptance, release and recovery | W05 plus W03 gates/evidence. `planops.Assurance` finalizer/operational methods and constrained destination adapters. | Boundary fencing includes suspended work, permit consumption, containment/recovery, accepted handoff, acceptance/current-assurance/closure separation. Exit: S07/S15/S16, optional-work race, lost release response, guardrail continuity and recipient-handoff refusal. |
| W07 Autonomous single-plane host and batches | W05–W06. Optional event pump invokes same `planops` roles; add durable batch/item schemas and OpenAPI capability/client routes [B26c]. | Reliable activation/state-change triggers, five deterministic transitions/nine interrupts, domain-keyed batches, job status/durable waits with compute release. Full CLI/server parity. Exit: S10/S12/S13; two waves plus verification/gates with zero intermediate controller wakeups, restart/partial-batch recovery and no lost outbox work. Manual-only installations retain honest reduced parity. |
| W08 Conditional child-plan semantics | W07, typed finite scope witnesses/ports from W01–W03, W05 budgets/epochs and W06 settlement. | Canonical spine/coordinate, idempotent child creation, boundary port isomorphism, modality/reduction/respawn checks, root reserves/fair share/lifetime caps, subleases and child-only replan damping. A native/human-reviewed child may use this without a model. Exit: S03/S11/S14; no ancestor wait cycle, no cloned objective delegation or sibling starvation/double charging; parent crash retains descendant ownership. |
| W09 Baselines and strict profile | Single-plane promotion needs W06–W07 settlement; recursive promotion additionally W08. New immutable baseline/descendant closure indexes contrasting with `mol squash` [B24s]. | Retain/promote partition, non-executable lineage, per-use invalidation, unchanged lifetime accounting and opt-in strict slot/depth policies. Exit: full S01 including six-plus-four, changed grandchild assumptions, output-resolution continuity and reject active handoff/guardrail/suspended work. Strict profile advertises only resolved/frozen terminal policy. |
| W10 Optional runtime/security adapters and release qualification | W07–W09; real artifact/sandbox/model services only when selected. | AST lexical instrumentation, isolated synthesis/controller frames, progressive model projections, high-water labels and two-channel child outputs; worker checkpoint/replay for all three await families. Exit: S05/S06/S10/S12/S14 under the chosen runtime, all applicable M15 evidence, calibrated limits/recovery procedures. Rebrand/package cutover only after intended upstream stable and compatibility review; no RLM dependency for native profiles. |

The roadmap covers the entire conditional design, not just a reduced planning subset. Unsupported recursion/sandbox/AST/await guarantees must be capability-gated and explicitly rejected until their wave exits; no command should report success for a capability its host cannot enforce. Full Mandala-profile claims require all applicable runtime demonstrations, not merely W02's structural planner.

### Import, compatibility and rollback

Preserve issue IDs, hierarchy, comments, dependency records and history. Keep `related`, `discovered-from` and other nonblocking edges nonblocking; do not turn support into blockers or existing containment into new semantics. Import existing formulas/protos/molecules as proposed plan material with pinned provenance and human-reviewed role/claim mappings. Existing closed issues become historical work facts only; never synthesize PASS, acceptance, baseline eligibility or trusted evidence from their status. Missing contracts/evidence remain explicit unestablished obligations. [B01] [B06] [B10] [B11] [B12]

Managed enrollment is explicit and transactional. Unmanaged issue workflows retain Beads semantics. Once a plan is managed, migrate every supported caller to the shared guarded roles or reject that operation for managed records; do not keep a compatibility path that bypasses authority. Old clients receive structured capability errors before writing. Direct SQL/import/maintenance paths either use the managed transaction protocol, require an explicit offline administrative maintenance mode that pauses/fences plans, or are outside the supported guarantee with active managed writes refused. Importing operational state never resumes dispatch automatically.

Migrations are additive until data/compatibility exits are proven. Rollback first pauses admissions, fences epochs and drains/reconciles effects; preserve the ledger, evidence and liabilities in a readable archive. A binary downgrade may continue unmanaged issue tracking only if it cannot mutate active managed membership/targets outside the guards. Reverting a schema pointer or Dolt branch cannot undo an external effect, so destructive down-migration is not a recovery plan. Resume requires a compatible authority and explicit reconciliation/transfer. After post-stable rebranding, retain persisted Beads issue identity/history and documented data compatibility rather than rewriting old IDs to match a brand.

## Complete SDK applicability map

M13 defines logical capabilities, not a mandatory language. All listed families are accounted for here; commands reference the single PROPOSED grammar above. **Core** is CLI/service semantics; **host** needs durable adapters/event processing; **conditional** needs the indicated real runtime. No CLI-only deployment claims sandbox or autonomous parity.

| Mandala operations | Placement and transformed contract | Wave / parity limit |
|---|---|---|
| `skills.inspect` | Core approved documentation catalog returns exact schema/interface/policy version+digest through bootstrap/context. | W01–W02; existing prime/formula discovery is only an analogue. |
| `context.project`, `context.requestExtension` | Core bounded source projections with selector/version/classification/completeness; model extension and delivery at a real turn boundary belong to model host. | W02/W10; extension is not applicable without a model frame; CLI paging is not model resumption. |
| `artifacts.readInSandbox`, `artifacts.writeFromSandbox` | Core immutable refs/access/provenance; sandbox reads journal whole-process labels and writes inherit them under a trusted artifact/execution host. | W01/W03/W10; no sandbox claim for ordinary shell. |
| `models.createVersion` | Subject-model publish under expected version, immutable model versions separate from work issues. | W01/W03; cycles allowed in subject model. |
| `decisions.propose`, `requestApproval`, `status` | Core proposal/approval records, authority-separated resolution, staged-reference publication rules. | W03; a decision issue alone is insufficient. |
| `recipe.register` | Native pinned handler manifest and explicit stable steps; script adapter additionally parses pinned AST/full call-stack paths and validates dynamic keys. | W05/W10; native form is an adaptation, not AST parity. |
| `plan.stageChangeset`, `initRoot`, `defineMilestone`, `defineEnabler`, `patchNode`, `removeNode`, `pruneSlots`, `compactSettledBaseline` | Core draft apply operations in request-local staging; aggregate findings; default bounded pruning; authoritative baseline eligibility only at publication. | W02/W09; saved proposed baseline cannot perform promotion. |
| `plan.bindClaim`, `unbindClaim`, `setCoverage`, `addRelationship`, `removeRelationship`, `revertMutation` | Core changeset/delta verbs over support/dispositions/typed relations; implied relations derived. Caller cannot set ESTABLISHED. | W02–W03; support is not evidence. |
| `plan.validateDraft`, `plan.validateAndFreeze` | Validate-only staging/saved-version inspection; freeze is complete validation plus atomic inactive immutable publication; failure discards unsaved buffer. | W02; no implicit activation or weakened findings. |
| `plan.saveDraft` | Explicit whole-version inert save with findings and expected-version CAS, later stable-handle repair. | W02; never grants scheduling authority. |
| `plan.forkForReplan`, `plan.activate` | Core draft fork then separate frozen-revision activation CAS/epoch/reconciliation; retain/baseline/new partition derived. | W02/W05/W09; unsettled work cannot vanish. |
| `exec.dispatchReady` | Core admission request through work claim/advance; optional host autonomously reevaluates after committed changes. | W05/W07; ready listing/polling alone is not dispatch parity. |
| `exec.callTool` | Worker-only named effect adapter with exact input bindings, stable keys, intent journal, current authority/labels and controlled execution options. | W05/W10; arbitrary subprocess cannot inherit a safety guarantee by being called from CLI. |
| `exec.mapBatch` | Host immutable domain-keyed manifest, handler, limits/checkpoints/partial dispositions; inherited process floor where relevant; durable wait suspension. | W07/W10; not issue CRUD batching. |
| `exec.spawnRecursivePlan`, `outputPortRef` | Host idempotent child at canonical spine, port isomorphism, reduction/modality/reserves/epochs/subleases; lazy port refs separate from observed diagnostics. | W08/W10; isolated frames/taint conditional on model/sandbox host, child closure mandatory for recursive baseline. |
| `exec.status`, `awaitResult`, `cancel`, `reconcile` | Core durable observations/cancel/reconciliation; host owns jobs after caller exit and supports result waits/checkpoint/resume without an LLM. | W05/W07/W10; a returned job is not completed output; manual CLI exit is not a suspended continuation. |
| `verify.synthesizeSpec`, `verify.bindSpec`, `verify.freezeSpec`, `verify.run`, `bindEvidence`, `evaluateGate` | Core governed registry/spec+binding, immutable approvals, authenticated verifier records, applicability and consistent gates; optional isolated synthesis receives frozen allowed inputs only. | W03/W06/W07/W10; independence cannot be inferred from a different identity or self-authored receipt. |
| `operations.promote`, `recover` | Release subfamily of the same effect/authority/permit/ledger protocol, with explicit reconciliation and verification. | W06; existing Beads promote/persistence concepts are not release promotion. |
| `plan.accept`, `plan.close` | Core immutable acceptance under protected boundary; close after settlement or accepted handoff. | W06; neither is ordinary issue close. |

Cross-cutting M13 rules apply to all families: schema/version validation, authorized refs for large values, durable job identities for long work, request dedupe/CAS, lexical or domain execution identity, structured safe diagnostics and host ownership after caller exit. For direct CLI control-plane mutations, request IDs plus CAS/turn identity replace invented AST call sites; a future controller-script adapter additionally enforces every-call lexical keys. This is an explicit adaptation, not a claim that command flags are an AST runtime. [M11a] [M13]

## Conformance scenarios

These sixteen stable scenarios cover all sixteen areas in M15. They are proposed future evidence, **not executed tests**. Each should include positive/negative assertions, observed operation counts/effect dispositions and the relevant crash/race schedule, not just a successful command exit. Semantic reviews additionally assess objective adequacy, coverage, decision quality and oracle sufficiency; no unit suite proves those alone. [M15]

| Scenario / M15 area | Required falsifiable demonstration | Requirements / waves |
|---|---|---|
| S01 Decomposition/compaction | Strict six-completed plus four-new fails if all ten retained; six eligible baselines plus four work nodes succeeds, preserving outputs/effects/evidence/support/history. Reject ninth L1/L2, unsettled/suspended/unknown/partial/guardrail/descendant-not-CLOSED promotion, stale support and baseline dispatch. Retention still consumes slots; lifetime charges do not reset. Recursive Merkle closure exposes changed grandchild assumptions. | R05/R08/R12; W02/W08/W09. |
| S02 Planning semantics | All work issues/children close but root mandatory/global composition claim lacks applicable evidence: no acceptance. Correctly evaluate alternatives/K-of-N without offsetting a mandatory failure. Discovery-only completion remains not-ready. | R02/R05; W02–W03/W06. |
| S03 Dependencies | Derive all typed relations and structural reconciliation; reject future-output/verification/gate/child/ancestor wait cycles and ambiguous/substituted inputs; allow subject cycles. Resolve each observation/producer selection once. | R02/R04/R14; W02/W05/W08. |
| S04 Decisions/liveness | Implementation blocked pending evidence and authorized choice; valid negative differs from invalid run. Refuse unauthorized residual-risk expansion, dangling rolled-back proposal, equivalent respawn and expired/unowned wait. | R05/R13/R14; W03/W08. |
| S05 Authority/taint | Managed raw/batch/proxy/force path cannot bypass authority; forged identity/approval and revoked epoch effects reject. Conditional host contrasts clean refs with payload/diagnostic observation, then constant/ref-only/tool/projection/child/batch outputs; labels remain monotonic across helpers/checkpoints/replay and scoped disclosure. Exercise actual sandbox escape controls only in controlled profile. | R03/R10/R11; W04/W05/W10. |
| S06 Artifact security | Unauthorized ref reads/projections/disclosure reject; omitted/fabricated sources cannot downgrade lineage. A hash, imported record or reference possession does not grant access or trust. | R06/R11; W03/W10. |
| S07 Oracle/evidence | Freeze independent spec **and binding** before output evaluation; reject draft/controller-history/implementation/child-seed contamination and same-controller oracle, even via another service. Reject forged/stale/incompatible/incomplete/favorable-only samples. Change target between gate and commit: stale permit refuses. | R05/R06; W03/W06/W10. |
| S08 Revision races | Two publications/activations with same expected head/epoch (including omitted-flag calls using `.beads/plan-cursor.json`): at most one transition publishes, no staged leak. Surviving worker cannot commit after fence; compatible effect adoption never redispatches. | R03/R04/R07/R15; W02/W05. |
| S09 Transactional drafts | Stage multiple schema/coverage/cross-mutation defects including an invalid fifteenth mutation; all findings have stable handles and shell-escaped `remedy_command`s. Immediate safety failure remains immediate. Validate retains in-request staging; failed freeze (and compound `--publish freeze --activate`) discards; explicit inert save survives exit and supports delta repair; prior saves survive later cancellation/conflict. | R04/R10/R15; W02/W04. |
| S10 Replay/step keys | Crash before/after possible dispatch and after effect/before result; unknown reconciliation prevents unsafe retry. Concurrent/sequential pass-key reuse fails even for identical input; replay only resolves same journaled operation; changed bindings/call paths halt. Script adapter tests nested/overlapping fan-out/helper ancestry and rejects unkeyed dynamic loops. | R07; W05/W07/W10. |
| S11 Locks/destinations | Competing exclusive targets cannot both be admitted; stale destination fence rejects where supported. Unfenced unknown old write blocks reuse after lease expiry. Suspended ancestor releases/delegates child-needed lock, otherwise ANCESTOR_RESOURCE_CONFLICT; no cross-depth wait deadlock. | R03/R07/R14; W05/W08. |
| S12 Controller/worker lifecycle | Ephemeral controller exits to caller/frame, not worker replay. Synchronous piggybacked progression on CLI completion and background host pump both progress two work waves, bound verification, projections and gates with no intermediate controller turn. For result/child/batch awaits, release compute, retain liabilities/journal/labels, then resume same continuation without an LLM. | R09/R11/R14/R15; W05/W07/W08/W10. |
| S13 Batches | Reordering preserves domain item IDs; changed input under key refuses. Threshold/cancel stops new admission while in-flight partial effects remain visible/reconciled. No aggregate atomic-success fiction. | R07/R13; W07. |
| S14 Recursion/budgets | Replay spawn returns same child; enforce boundary/finite reduction, modality, reserves/minimum quantum, attenuation/lifetime limits and fair share. No double charging. Parent revocation fences descendants; child-local compatible replan spares siblings. Conditional model host demonstrates fresh frames/seed isolation and two-channel taint. | R08/R14; W08/W10. |
| S15 Acceptance/closure | Race optional work and suspended result completion against acceptance: boundary fences further conflicting effects, preserves required control coverage. Later authorized change stales assurance but not history. Unaccepted handoff or outstanding liability prevents closure. | R05/R07/R12; W06. |
| S16 Exhaustion/cancellation | Stop admissions and fence descendants while preserving reservations/liabilities; already-dispatched effects reconcile and authorized containment/recovery continues. Recovery failure escalates with owner/deadline; only settlement or accepted transfer permits closure. Offline clone merge cannot grant dispatch authority. | R03/R07/R13; W05–W06. |

## Nine v4 refinements

Each numbered refinement from M16 has an explicit disposition; none is hidden as an unspecified future enhancement.

| Refinement | Adaptation and parity boundary | Trace |
|---|---|---|
| F01 High-water taint vs reference pipelining | Adopt immutable lineage/access now; implement exact process/frame joins/meets and ref-only distinction only in a trusted artifact/sandbox adapter. CLI flags cannot provide it. | R11; S05–S06; W10. |
| F02 Lexical keys/pass uniqueness | Adopt stable native/domain keys and host pass-wide uniqueness; conditionally implement pinned-AST full call-stack/fork instrumentation. Runtime counters are rejected in either mode. | R07; S10; W05/W10. |
| F03 Ephemeral turns vs durable coroutines | Adopt ephemeral CLI/controller requests and host-owned durable jobs; durable worker waits require actual checkpoint/replay/compute release, not a long-running shell promise. | R09; S12; W07/W10. |
| F04 Compacted settled baselines | Adopt after authoritative settlement; preserve transitive identity/effects/support and reevaluate applicability, independent lifetime caps. | R12; S01; W09. |
| F05 Isolated synthesis and target binding | Adopt separate independent governance/frozen spec+binding; registry path avoids a mandatory LLM. New model synthesis must enforce input-restricted frames, not just identity separation. | R06; S07; W03/W10. |
| F06 Transactional drafts/delta repair | Adopt request-local buffer, explicit inert save, aggregate findings, whole-version CAS freeze, compound `--publish freeze --activate`, rollback and stable delta handles. | R04/R15; S09; W02. |
| F07 Autonomous frontier/interrupts | Adopt deterministic engine, synchronous piggybacked CLI progression + `--claim-next` on state-settling mutations, and nine named interrupts; optional background host supplies asynchronous invocation. | R09/R15; S12; W05/W07. |
| F08 3D geometry/termination | Adapt as conditional typed child plans with opt-in strict ceilings, finite witnesses, reserves and bounded adaptation. Do not claim hashes solve natural-language equivalence. | R08/R14; S14; W08–W09. |
| F09 Cross-depth concurrency/two channels/subleases/epochs | Adopt observable subtree fencing, parent wait release/delegation, separate output/diagnostic paths and child replan damper; model taint/frame claims conditional on actual runtime. No unmeasured constant-time claim. | R07/R11/R14; S11–S15; W08/W10. |

## Risks, open decisions and success experiments

### Remaining decisions and conservative defaults

| Issue | Evidence/uncertainty | Recommended default |
|---|---|---|
| Exact release basis | Checkout/default/docs are 1.3.2; intended 1.3.3-rc.1 and later stable timing are not established by source provenance. | Rebase/reconcile before implementation qualification; retain this report's pin, do not rebrand or claim release-candidate behavior. |
| Fork charter and release authority | Current charter deliberately excludes orchestration. [B16] | Explicit fork charter and managed namespace, preserving unmanaged Beads semantics. |
| Mandala terminal-depth rule | Depth-three leaf-default, depth-four terminal row and attenuation formula require precedence policy. [M03b] | Version/freeze terminal override; no spawn beyond depth three without approved override; no invented universal fourth-depth policy. |
| Budget heuristic meaning | Source uses parent delegable-pool inequality and a depth-two table shorthand; concrete resource fractions, minimum quantum, overhead and latency are workload-specific. [M03b] [M11b] | Enforce the explicit parent-pool/child-minimum constraints in each unit; configure/calibrate rather than invent constants or add unlike units. |
| Semantic reduction/adequacy | Fingerprints and structural validation cannot decide arbitrary natural-language equivalence, completeness or feasibility. [M01] | Strict proof only over structured finite domains; otherwise advisory reviewer judgment with bounded resource/depth caps and weaker claims. |
| Automatic oracle bootstrap | Freeze precedes isolated synthesis, while executable frontier eventually needs frozen spec/binding references. [M06] [M07c] [M08a] | Freeze outcome contracts first, govern/freeze specs/bindings, then freeze an executable attachment revision before activation; no draft access or mutable attachment shortcut. |
| Admission ownership | `dispatchReady` is a request while active host progression is autonomous. [M13] | One authority owns queue/selection; both explicit request and pump invoke the same idempotent transition, never competing schedulers. |
| Controller keys vs lexical worker keys | M13 every-call lexical language meets CLI requests without an AST. [M11a] [M13] | Durable request ID plus CAS/turn scope for direct CLI; explicit native step keys for jobs; full lexical enforcement only for script adapter. Label transformation, not exact AST parity. |
| Identity, evidence and deployment trust roots | Organization principals, approvers, CI signers, revocation, tenant policy and production destination guarantees are not supplied by the design corpus. | Advisory local trust until configured; reviewed/high-assurance operations fail closed without real authenticated providers and constrained sinks. |
| Retention/privacy vs immutable evidence | Source requires assurance retention alongside deletion/minimization/access policy. [M10] | Freeze retention policy and access/encryption strategy; protect unresolved liabilities, use revocation/tombstoned availability rather than rewriting verdicts. Legal retention specifics require deployment policy. |
| Operational state replication | No offline merge protocol can serialize external effects already performed independently. | Single active authority; explicit fenced transfer and reconciliation before another clone resumes; read-only replicated history is not live authority. |
| Exact storage design | Existing UOW/driver is reusable, but new schema constraints, migrations and adapter recovery have not been implemented/proven here. | Add only justified semantic tables through storage boundary; no custom engine/flock/retry layer. Validate driver transaction semantics during W01/W05. |

The main product risk is turning a useful issue tracker into a ceremony-heavy workflow platform. Counter it with progressive disclosure, small next-action contracts, optional strict geometry, native handlers and an advisory import path that does not masquerade as enforcement. The main correctness risk is partial cutover: one raw batch/status/import path bypassing guards can invalidate the whole assurance story. The main operational risk is unknown external effects hidden by retry/replan/compaction. The main assurance risk is a formally valid but inadequate objective/oracle; independent semantic review remains necessary.

### Measurable experiments, not performance promises

Use representative migrations and small bounded development objectives, with fixed outcome/risk categories and recorded policy versions. Compare ordinary Beads plus instructions, blankets manual mode and blankets host mode; hold approver effort and workload definitions visible. Pre-register success thresholds with the responsible authority rather than inventing favorable numbers in this report.

| Experiment | Measures and required interpretation |
|---|---|
| E01 Agent ambiguity/intervention | Count clarification requests, wrong next actions, duplicate work, unsupported acceptance attempts, repair turns and human interventions per completed objective; inspect transcripts for hidden shell bypasses. A smaller prompt is not success if correctness falls. |
| E02 Coverage/oracle adequacy | Independent reviewers identify missed mandatory/global/negative/interval obligations and oracle-target mismatches; compare review findings before/after structural validation. Report false confidence and disagreement, not just validator pass rate. |
| E03 Deterministic progression | Measure host transitions per controller wakeup, zero-intermediate-turn two-wave completion, admission explanations and quiescent blocker accuracy. Separate manual invocations from autonomous events. |
| E04 Race/crash recovery | Inject the S08/S10/S11/S15 windows with instrumented adapters; measure duplicate destination effects, unknown classification, reconciliation time, lost liabilities and unsafe retries. “One operation row” alone is not one remote effect. |
| E05 Cost/liveness | Record typed reservations/usage, retries, storage/wait overhead, replan/node lifetime counts, recovery reserve use and bounded-stop behavior. No generic sum of tokens, money and seconds. |
| E06 Compatibility | Import historical issues/formulas/molecules, preserve IDs/comments/edges/history and compare unmanaged behavior; attempt every managed mutation front door and old client. Check rollback cannot erase an external liability. |
| E07 Conditional runtime | For sandbox/model/recursive profiles, measure label/isolation violations, child reduction rejection, compute release while suspended, fairness and descendant recovery. Native CLI-only success cannot substitute for these demonstrations. |
| E08 Agent ceremony and turn-count efficiency | Compare explicit protocol CLI invocations against implicit-determinism CLI mode (`.beads/plan-cursor.json`, `--publish freeze --activate`, synchronous piggybacked progression + `--claim-next`, and `remedy_command` hints) across identical objectives: measure CLI invocations per settled L2 node, flag/CAS syntax errors, temporary file churn, and stale-cursor CAS rejection accuracy. Ergonomic shortcuts must never reduce CAS or gate rejection fidelity. |

No performance improvement, semantic completeness, token-efficiency gain, perfect oracle, arbitrary shell safety or universal exactly-once claim is made. Benchmark throughput only after correctness and real adapter limits are established. [M15] [M12]

## Research verification evidence and limits

### Installed CLI receipts

The research exercised the installed Homebrew 1.3.2 binary in disposable isolated embedded-Dolt Git repositories, then removed them. The evidence artifacts are scoped to this research session and not portable repository links. The table preserves actual outcomes and excerpts; it does not substitute hypothetical expected output. Fifteen behavior checks (P01–P15) are summarized here.

| Receipt | Existing command/action observed | Actual result |
|---|---|---|
| P01 Dependency readiness | Add consumer `probe-80o.2` depending on producer `.1`, then `bd ready --type task --json` | Add exited 0; ready array contained `.1` and `.3`, not `.2`. |
| P02 Cycle rejection | Add reverse `.1` → `.2` dependency | Exit 1: `Error: adding dependency would create a cycle`. |
| P03 Child close guard | Close objective `probe-80o` with three open children | Exit 1: `cannot close probe-80o: 3 open child issue(s); close children first or use --force to override`. |
| P04 Blocker close guard | Close consumer `.2` with live producer blocker | Exit 1: `cannot close blocked issue: probe-80o.2 is blocked by [probe-80o.1] (use --force to override)`. |
| P05 Prose is not evidence | Close `.3` whose acceptance text required a receipt, with reason `No verification receipt supplied.` | Exit 0; subsequent show confirmed `status: closed` with the unchanged acceptance text and no supplied receipt. |
| P06 Atomic ownership | `bd ready --claim --type task --actor research-a --json` | Exit 0; `.1` returned `in_progress`, assigned to `research-a`, with heartbeat and lease expiry. This is ownership, not semantic coverage. |
| P07 Competing claimant | `bd update probe-80o.1 --claim --actor research-b --json` | Exit 1: `issue already claimed by research-a`; later show retained that assignee. |
| P08 Stale assignee guard | Update `.1` priority under `--if-assignee not-the-holder` | Exit 13; JSON included `guard_mismatch: true`; subsequent show kept priority 2 and the same revision token. This exercised assignee CAS, not a stale row-revision experiment. |
| P09 Transaction rollback | Batch updates `.3` title, then updates nonexistent issue | Exit 1 with `code: batch_error` at line 2; subsequent show kept original title `Probe unverified result`. |
| P10 Raw batch close bypass | Batch input `close probe-80o.2 batch policy comparison` | Exit 0; JSON `operations: 1`, `status: ok`; show confirmed `.2` closed while `.1` remained `in_progress`. |
| P11 Producer progression | Reopen `.2`, close `.1` as `research-a`, query ready work | Producer close exited 0; ready array then contained `.2`. |
| P12 Piggybacked `--claim-next` + `last-touched` handoff | `bd close erg-0xe --reason 'Step 1 done' --claim-next --actor agent-1 --json` where `erg-msf` depends on `erg-0xe` | Exit 0; single JSON response returned `closed: [erg-0xe]` and `claimed: {id: erg-msf, status: in_progress, assignee: agent-1, lease_expires_at: ...}`; `.beads/last-touched` updated to `erg-msf`. |
| P13 Non-interactive bare-close safety guard | `BD_NON_INTERACTIVE=1 bd close --reason 'Missing ID' --json` | Exit 1 before store open: `no issue ID provided (the last-touched fallback only applies in interactive sessions; pass an explicit issue ID or set BD_LAST_TOUCHED_FALLBACK=1)`. |
| P14 Wisp `mol squash` prose digest vs baseline | Create ephemeral epic `erg-wisp-05x` and child `.1`, close `.1`, then `bd mol squash erg-wisp-05x --summary 'Settled discovery digest' --json` | Exit 0; created permanent closed digest `erg-32r` and deleted ephemeral child (`deleted_count: 1`, `wisp_squash: true`), corroborating why `compactSettledBaseline` must preserve typed output/effect/invalidation records rather than deleting children into prose. |
| P15 Automatic key derivation (`memoryops`) | `bd remember 'Auth service requires mTLS workload identity' --json` with `--key` omitted | Exit 0; deterministically derived `key: "auth-service-requires-mtls-workload-identity"`, corroborating the `"EMPTY MEANS DERIVE IT"` ergonomic contract. |

These observations corroborate source distinctions, especially the documented raw-batch asymmetry, piggybacked `--claim-next` progression, non-interactive fallback safety, and destructive `mol squash` condensation, but prove neither concurrency under adversarial load nor that current checkout `9f03f32` or intended `1.3.3-rc.1` behaves identically. No proposed plan command, verifier, ledger, host, recursion, acceptance boundary or sandbox was run. All such behavior remains future acceptance criteria.

### Documentation gates and document coverage

Parent-reported gate observations before and during this authoring wave:

| Existing gate/action | Observed outcome and scope |
|---|---|
| `make check-docs` | Could not start its Bazel work: `bazel` absent, Error 127, make exit 2. Not a passed docs gate. |
| `./scripts/check-doc-freshness.sh` | Passed reference freshness-marker checks. This is not research citation/coverage validation. |
| `./scripts/check-doc-flags.sh /home/linuxbrew/.linuxbrew/bin/bd` | Unavailable: source pin resolution attempted `v1.3.2`, absent from origin's tag refs; exit 128. |
| Documented pin-bypass flag check with `BD_DOCS_IGNORE_PIN=1` | Command coverage checks passed, but CGO help detection rebuilt a pure-Go binary from current source; generated reference drift against release-pinned docs failed without a PR diff base. It does not isolate installed-binary behavior or establish research-induced drift. No generated docs were changed. |
| `./mint.sh validate` | Passed: `success build validation passed`. Published-site validation does not cover hidden `.research` source references. |
| `./mint.sh broken-links` | Passed: `success no broken links found`. Published-site links are not this report's sibling/source-span links. |

The docsync collector intentionally skips dot-directories, so this report under `.research` is unpublished and is not added to `docs/docs.json`. [B21] A separate report audit checked all 71 local source links and exact line ranges, all sixteen numbered Mandala sources, front matter and heading conventions, seven balanced text fences, and both illustrative JSON envelopes. Cross-document reviews accounted for all nineteen SDK families (48 methods), sixteen conformance areas, nine v4 refinements, and fifteen architectural decisions (D01–D15). These checks establish report integrity and reviewed design consistency, not implemented blankets behavior.

Secondary citation-grounding triage initially processed 63 cited units. For substantive reporting, 23 duplicate source-catalog rows were excluded; structural and uncited text was outside that audit's candidate set. The retained machine verdicts were:

| Citation-grounding label | Substantive units |
|---|---:|
| `current_supported` | 4 |
| `mixed_supported_and_proposed` | 18 |
| `unsupported_or_overstated` | 18 |

Thirty substantive units were escalated by label or uncertainty, with no classifier errors or truncations. Their flags were read against exact cited sources and whole-document proposal context, rather than accepted as findings. In particular, source checks confirmed the molecule's all-step closure field, Lifecycle close guards and string-token precision cautions. Proposal-context false positives were overturned; no material current-code assertion error remained. The source-span audit corrected the M09 endpoint and its display label to line 69. All additional citations (`B22`–`B29`) and live CLI smoke receipts (`P12`–`P15`) were verified directly against the current checkout and disposable embedded-Dolt workspaces.

Research limitations are explicit: classifier judgments were triage; source analysis is bounded to inspected seams; runtime proof is installed 1.3.2 only; the intended release candidate is unestablished; Mandala is normative design only; organization trust roots, production adapters and measured resource limits are unavailable. Those limits do not justify shrinking the target semantics or importing old closure as evidence.

## Beads source catalog

Material Beads claims above use these exact source-local spans. Separate keys identify distinct spans even when they share a file.

| Key | Source span | Evidence role |
|---|---|---|
| B00a | [cmd/bd/version.go:18–21][B00a] | Default version/build override. |
| B00b | [docs/cli-docs.pin:1–12][B00b] | Release documentation pin. |
| B01 | [internal/types/types.go:20–140][B01], [metadata validation:461–483][B01v] | Issue model, acceptance prose, node-local leases, partial RowVersion, generic JSON metadata. |
| B02 | [internal/storage/sqlbuild/ready.go:207–285][B02] | Ready SQL predicate/filtering. |
| B03 | [issueops/readyclaimer.go:1–98][B03] | Existing atomic selection/CAS/hydration and lease contract. |
| B04 | [issueops/metadatacas.go:19–72][B04] | Single-key CAS, provenance-not-auth, numeric representation caution. |
| B05 | [issueops/issueops.go:1–7][B05], [Lifecycle:552–599][B05l] | Experimental API and guarded lifecycle semantics. |
| B06 | [internal/types/types.go:1545–1623][B06] | Readiness versus combined static cycle edge meanings. |
| B07 | [cmd/bd/batch.go:25–81][B07], [raw close dispatch:352–374][B07r] | Transactional script and documented policy bypass. |
| B08 | [issueops/batchapplier.go:1–48][B08], [guarded close item:168–195][B08c] | Distinct symbolic refs and guarded batch operation. |
| B09 | [cmd/bd/graph_apply.go:32–103][B09], [atomic translation/cycle caveat:940–1013][B09a] | Graph input and one-request shared role application. |
| B10 | [cmd/bd/cook.go:38–93][B10] | Formula compilation versus proto persistence. |
| B11 | [cmd/bd/template.go:859–935][B11] | Transactional molecule issue/edge materialization. |
| B12 | [issueops/moleculestepper.go:14–42][B12], [explicit advance implementation:102–155][B12a] | One-step discovery/claim, not autonomous plan host. |
| B13 | [cmd/bd/gate.go:29–59][B13] | Existing async workflow waits. |
| B14 | [cmd/bd/events.go:19–64][B14] | Optional journal, polling, retention and branch/replica/raw-SQL limits. |
| B15 | [internal/storage/issueops/version_history.go:21–60][B15], [canonical snapshots:700–775][B15s] | Scoped history, local ordinal/single-writer caveat, canonical bytes. |
| B16 | [engdocs/PROJECT_CHARTER.md:11–57][B16], [schema boundary:86–101][B16s] | Issue scope, orchestration/storage/schema boundaries. |
| B17 | [internal/httpapi/routes.go:529–572][B17], [events routes:1112–1145][B17e] | Versioned context/ready and event capability surface. |
| B18 | [cmd/bd/serve.go:88–141][B18] | Server auth, no TLS, actor provenance and role posture. |
| B19 | [internal/httpapi/auth.go:16–27][B19] | Shared-secret full-surface/no-identity boundary. |
| B20 | [internal/storage/uow/tx.go:124–178][B20] | Retryable work callback and commit boundary. |
| B21 | [test/docsync/docsync_test.go:110–131][B21] | Hidden-directory documentation exclusion. |
| B22 | [cmd/bd/close.go:44–58][B22], [piggybacked --continue and --claim-next:335–394][B22p], [close batch policy:137–208][B22b], [ContinueResult and remedy hints:255–293][B22m] | Atomic close-plus-claim-next, molecule auto-close/advance, and shell-escaped `remedy` command hints. |
| B23 | [cmd/bd/last_touched.go:19–88][B23] | Workspace-local `.beads/last-touched` session cursor and `AllowLastTouchedFallback` non-interactive safety guard. |
| B24 | [cmd/bd/prime.go:1048–1144][B24], [wisp creation:338–355][B24w], [mol squash digest condensation:261–336][B24s] | Session context recovery, ephemeral vapor wisps, and destructive `mol squash` digest condensation. |
| B25 | [memoryops/doc.go:1–53][B25], [RememberRequest key derivation:5–45][B25m] | Sibling leaf package (`memoryops`), `beadserrors.ErrValidation` aliasing, and `"EMPTY MEANS DERIVE IT"` ergonomics. |
| B26 | [engdocs/ADDING_AN_ISSUEOPS_ROLE.md:14–75][B26], [13-step role cost and conformance gates:221–295][B26c] | Three-question role boundary test, entitlement separation, and per-role wiring/gate cost. |
| B27 | [engdocs/UI_PHILOSOPHY.md:201–211][B27] | Cognitive-load minimization, flags on existing commands, and `bd doctor --fix` routing. |
| B28 | [engdocs/ERROR_HANDLING.md:352–378][B28] | `HandleErrorWithHintRespectJSON` and structured JSON-aware error/remedy helpers. |
| B29 | [CHANGELOG.md:8–62][B29] | Unreleased `main` provenance: first-use endpoint credential pinning (`~/.config/beads/trusted-endpoints.json`) and wisp `arrival_seq`. |

[M00]: ../../../mandala/docs/README.md#L1-L72
[M01]: ../../../mandala/docs/01-purpose-and-governing-rules.md#L1-L22
[M02]: ../../../mandala/docs/02-logical-services-and-authority.md#L1-L22
[M03a]: ../../../mandala/docs/03-planning-model.md#L30-L53
[M03b]: ../../../mandala/docs/03-planning-model.md#L75-L106
[M03c]: ../../../mandala/docs/03-planning-model.md#L107-L136
[M04]: ../../../mandala/docs/04-planning-procedure.md#L29-L49
[M05a]: ../../../mandala/docs/05-contracts-and-bindings.md#L5-L22
[M05b]: ../../../mandala/docs/05-contracts-and-bindings.md#L23-L48
[M06]: ../../../mandala/docs/06-decisions-and-discovery.md#L1-L18
[M07a]: ../../../mandala/docs/07-dependencies-freezing-and-activation.md#L34-L65
[M07b]: ../../../mandala/docs/07-dependencies-freezing-and-activation.md#L66-L77
[M07c]: ../../../mandala/docs/07-dependencies-freezing-and-activation.md#L78-L102
[M08a]: ../../../mandala/docs/08-evidence-acceptance-and-closure.md#L28-L50
[M08b]: ../../../mandala/docs/08-evidence-acceptance-and-closure.md#L51-L76
[M09]: ../../../mandala/docs/09-rlm-context-and-code-mode.md#L29-L69
[M10]: ../../../mandala/docs/10-artifacts-tainting-and-sandbox-enforcement.md#L1-L19
[M11a]: ../../../mandala/docs/11-durable-execution-and-resource-accounting.md#L43-L67
[M11b]: ../../../mandala/docs/11-durable-execution-and-resource-accounting.md#L68-L93
[M12]: ../../../mandala/docs/12-state-cancellation-and-recovery.md#L1-L23
[M13]: ../../../mandala/docs/13-logical-sdk-contract.md#L1-L34
[M14]: ../../../mandala/docs/14-worked-lifecycle.md#L1-L17
[M15]: ../../../mandala/docs/15-conformance-and-implementation-readiness.md#L1-L29
[M16]: ../../../mandala/docs/16-design-basis-and-principal-refinements.md#L1-L15
[B00a]: ../../cmd/bd/version.go#L18-L21
[B00b]: ../../docs/cli-docs.pin#L1-L12
[B01]: ../../internal/types/types.go#L20-L140
[B01v]: ../../internal/types/types.go#L461-L483
[B02]: ../../internal/storage/sqlbuild/ready.go#L207-L285
[B03]: ../../issueops/readyclaimer.go#L1-L98
[B04]: ../../issueops/metadatacas.go#L19-L72
[B05]: ../../issueops/issueops.go#L1-L7
[B05l]: ../../issueops/issueops.go#L552-L599
[B06]: ../../internal/types/types.go#L1545-L1623
[B07]: ../../cmd/bd/batch.go#L25-L81
[B07r]: ../../cmd/bd/batch.go#L352-L374
[B08]: ../../issueops/batchapplier.go#L1-L48
[B08c]: ../../issueops/batchapplier.go#L168-L195
[B09]: ../../cmd/bd/graph_apply.go#L32-L103
[B09a]: ../../cmd/bd/graph_apply.go#L940-L1013
[B10]: ../../cmd/bd/cook.go#L38-L93
[B11]: ../../cmd/bd/template.go#L859-L935
[B12]: ../../issueops/moleculestepper.go#L14-L42
[B12a]: ../../issueops/moleculestepper.go#L102-L155
[B13]: ../../cmd/bd/gate.go#L29-L59
[B14]: ../../cmd/bd/events.go#L19-L64
[B15]: ../../internal/storage/issueops/version_history.go#L21-L60
[B15s]: ../../internal/storage/issueops/version_history.go#L700-L775
[B16]: ../../engdocs/PROJECT_CHARTER.md#L11-L57
[B16s]: ../../engdocs/PROJECT_CHARTER.md#L86-L101
[B17]: ../../internal/httpapi/routes.go#L529-L572
[B17e]: ../../internal/httpapi/routes.go#L1112-L1145
[B18]: ../../cmd/bd/serve.go#L88-L141
[B19]: ../../internal/httpapi/auth.go#L16-L27
[B20]: ../../internal/storage/uow/tx.go#L124-L178
[B21]: ../../test/docsync/docsync_test.go#L110-L131
[B22]: ../../cmd/bd/close.go#L44-L58
[B22p]: ../../cmd/bd/close.go#L335-L394
[B22b]: ../../internal/storage/issueops/close_batch.go#L137-L208
[B22m]: ../../cmd/bd/mol_current.go#L255-L293
[B23]: ../../cmd/bd/last_touched.go#L19-L88
[B24]: ../../cmd/bd/prime.go#L1048-L1144
[B24w]: ../../cmd/bd/wisp.go#L338-L355
[B24s]: ../../cmd/bd/mol_squash.go#L261-L336
[B25]: ../../memoryops/doc.go#L1-L53
[B25m]: ../../memoryops/memories.go#L5-L45
[B26]: ../../engdocs/ADDING_AN_ISSUEOPS_ROLE.md#L14-L75
[B26c]: ../../engdocs/ADDING_AN_ISSUEOPS_ROLE.md#L221-L295
[B27]: ../../engdocs/UI_PHILOSOPHY.md#L201-L211
[B28]: ../../engdocs/ERROR_HANDLING.md#L352-L378
[B29]: ../../CHANGELOG.md#L8-L62
