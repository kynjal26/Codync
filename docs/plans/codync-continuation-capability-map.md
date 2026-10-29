# Proposed capability map: Codync continuation

Status: proposed for review, 2026-10-29. Product intent is confirmed; these module boundaries
and dependency directions are not yet approved.

Read alongside the [intent](../intent/codync-continuation.md) and
[source investigation](../research/codync-continuation-source-review.md).

## Governing decision

Codync is the destination implementation and UX. Zyndi supplies selected capabilities and desktop
visual design. Every module extends a Codync boundary; none introduces a second host, duplicate
turn scheduler, replacement encrypted transport or wholesale desktop renderer migration.

The nine modules describe additions and changes only. They do not define the complete product.
All existing Codync capabilities remain required, including areas outside these modules; use
the [README preservation baseline](../research/codync-readme-preservation-baseline.md) during
specification and regression verification. No capability is removed by omission from this map.

## Capability map

| Module ID | Responsibility | Depends on |
| --- | --- | --- |
| `team-access` | Workspace principals, roles, resource grants, actor propagation, shared/private history and agent-context isolation | — |
| `provider-accounts` | Named subscription/API accounts, private sign-ins, admin availability, member entitlements, account-aware model choice and Codync fallback | `team-access` |
| `team-knowledge` | Workspace/person/restricted memory ownership using Codync's files, evidence checks, dreaming and forgetting | `team-access` |
| `action-policy` | Deterministic action classes, designated approvers and explicit exceptions above ordinary Ask/Auto | `team-access`, `provider-accounts` |
| `desktop-device-bridge` | Person-bound central-host → desktop execution, admin ceiling plus local consent, Codync driver integration; Mac and Windows | `team-access`, `action-policy` |
| `team-voice` | First-person Bot voice, requested/routine-triggered outbound calls, native client ringing and private reports | `team-access`, `provider-accounts`, `action-policy` |
| `creation-workspace` | Extend Codync artifacts with rich documents/canvases, revisions, restore, designated editors and anchored feedback | `team-access`, `action-policy` |
| `desktop-ui` | Apply Zyndi's visual language to Codync's existing desktop components, preserving UX and state semantics | — |
| `data-migration` | Audienced history/content/account mapping, preserved archive, dry-run reconciliation, routine ownership and rollback rehearsal | `team-access`, `provider-accounts`, `team-knowledge`, `action-policy`, `desktop-device-bridge`, `team-voice`, `creation-workspace` |

Dependency means a provider contract must be available before the dependent capability is enabled.
It does not prohibit drafting interfaces or inspecting migration schemas earlier. There are no
cycles: presentation consumes domain results, and domain modules never depend on the styling pass.

## Proposed order

1. Review the source evidence and this map. Capture existing Codync behavior as the compatibility
   baseline; no operational cutover follows from approving module boundaries.
2. Establish `team-access`. The desktop visual pass can be specified independently using existing
   Codync fixtures; it must not pull in unselected Zyndi features.
3. Extend accounts and knowledge at their existing Codync boundaries.
4. Resolve the action-policy enforcement boundary and implement that contract before enabling
   mandatory-approval claims on team actions.
5. Specify and verify desktop routing, voice and creation-workspace additions against those
   contracts. Initial computer-use acceptance requires both Mac and Windows.
6. Rehearse data conversion and the whole team experience on a separate profile, then prepare
   the consumer-by-consumer cutover and rollback for review.

This is dependency ordering, not an estimate or a task-by-task implementation plan. Team sharing
is required at initial cutover; intermediate development slices do not justify releasing a
single-user-only replacement and calling the migration done.

## Review boundaries per module

### `team-access`

Keep Codync device cryptography/revocation. Add verified person/role/resource context to host
dispatch, catch-up/live events, file/artifact delivery and the agent's effective context. Shared
Bots remain shared, but owner execution context is not reused for restricted people. Define
ordinary shared conversation visibility separately from private calls and restricted knowledge.

Must prove: wrong-role and wrong-resource refusals, revoked membership, no cross-person model
context leak, consistent paging/replay/download filtering. It owns authorization facts, not
department payment-policy decisions or provider credentials.

### `provider-accounts`

Keep Codync harness discovery, model picker and actor/fallback lifecycle. Introduce named account
identity into launches, session lookup, usage and rests, and add direct API execution at the
same execution boundary. Keep credentials in Codync's secure storage. Do not adopt Zyndi's
global usage-threshold/backoff choices merely because its account UI is reused.

Must prove: independent same-provider logins, permitted model selection, member-safe fallback,
announced switch and preferred-account return, cancellation continuity and credential-free
client metadata. Provider quotas/billing and media-generation routing are excluded.

### `team-knowledge`

Keep Codync's memory engine. Define memory layers/access from workspace principal and restricted
membership. Reuse its provenance and forgetting; do not add Zyndi's second capture ledger,
success-only memory staging or a handbook as an incidental dependency.

Must prove: team facts propagate, restricted/private material stays out of shared memory and
model context, correction rights match the selected authority, and forgetting still holds.

### `action-policy`

Keep ordinary Codync Ask/Auto. Add mandatory classes for external messaging, publishing,
payments and destructive actions, eligible approvers and narrow configured exceptions. Ordinary
Auto/Full access cannot bypass those classes. AI review and uncertain-operation retry holds
remain excluded.

Must first settle: managed-tool enforcement versus a universal promise covering arbitrary shell,
browser and generated-page traffic. This affects credential/egress mediation and admin freedom;
it cannot be solved by wording the Bot's prompt. Define previews and policy change behavior in
the module specification. No selected policy can be presented as a finished Zyndi transplant.

### `desktop-device-bridge`

Keep Codync computer-use tools and platform drivers. Add central dispatch to the correct
member's selected desktop with verified actor/device identity. Combine admin limits with local
grants; preserve Codync setup, platform permission ownership and takeover semantics. Do not
import Zyndi's entire desktop executor or broadcast owner requests to every admin desktop.

Must prove: central host to Mac and Windows, refusal of wrong-device responses, disconnected
target handling, consent/revocation and background interaction without unexpected cursor takeover.

### `team-voice`

Keep Codync host-owned calls and audio engines. Adapt Bot identity, explicit call authorization,
callee invitations and person/private-report ownership. Retain Codync's spoken approval flow,
subject to the team approver's authority; no Zyndi exact-request voice-ID safeguard is selected.

Must prove: requested/routine-only call initiation, one answering device, private reports and
history, native-client ringing/cancel/decline and no disclosure through later shared turns.
Experimental native Codex voice is not a prerequisite for first-person Bot identity.

### `creation-workspace`

Keep Codync artifact publishing, Library and full-size artifact/composer UX. Extend content kinds,
revisions and document editing; add anchored feedback and actor-aware editing authority. Preserve
old HTML artifacts and define an isolated compiled-canvas runtime rather than silently changing
the existing artifact's storage/network capabilities.

Must prove: creator/editor/viewer rules on direct and Bot edits, stale revision refusal, last-good
render after a failed compile, comment anchoring/revisions, autosave recovery and export. Ordinary
mobile viewing remains compatible; equivalent native rich editing is not assumed without design.

### `desktop-ui`

Use Zyndi's visual palette, typography, spacing, icons and surface styling as the donor. Codync
keeps navigation, state store, settings organization, keyboard behavior, scrolling, drafts,
accessibility and platform integration. Functional changes are owned by the relevant capability,
not smuggled into CSS or a renderer replacement.

Must prove: before/after interaction parity on unchanged flows plus rendered Mac/Windows checks
in light/dark, text scaling, narrow layouts and loading/error/working states. Prefer a semantic
token/component adaptation over copying Zyndi's global stylesheet or upgrading the framework.

### `data-migration`

Map data into Codync's evolved schema with explicit source IDs and audience preservation. Keep
files and revisions with their references, reauthorize/transfer credentials without exposing
values, and leave rejected-feature history archived. The old Codync→Zyndi mapper is evidence,
not a reversible converter.

Must prove: repeatable dry run, identity/content/permission reconciliation, recoverable snapshots,
fresh harness handoff, one owner per external automation and a rehearsed rollback that accounts
for writes after cutover. No live conversion or service retirement is authorized by this map.

## Constraints shared by every module

- Keep host business behavior authoritative; desktop/mobile render the same authorized state.
- Extend Codync source conventions and tests; preserve native clients and existing wire behavior
  through additive capability negotiation where required.
- Add migrations; never rewrite an already applied migration or repoint both hosts at one profile.
- Preserve Codync's driver, secure vault, updater and encrypted channels where they meet the need.
- Keep Apache/MIT and other applicable attribution when adapting Zyndi/reference work.
- Do not port the excluded features through incidental dependencies; extract narrow interfaces.
- Do not represent source review/fake-provider tests as real platform, push or provider certification.

## Next artifact after map approval

The owner's subsequent instruction authorized architecture of team onboarding. Its scoped
proposal is [SPEC-team-access.md](SPEC-team-access.md), awaiting specification review. This
does not mark every module boundary approved or authorize implementation.

Write module specifications in the repository's existing documentation convention, indexed by
these stable IDs. Each specification will define its interface changes, schema compatibility,
commands, test cases, boundaries and remaining decisions. Implementation plans and tasks follow
approved specifications; this file does not authorize a bulk port.
