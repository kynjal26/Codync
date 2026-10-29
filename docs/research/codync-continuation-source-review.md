# Codync continuation: source investigation

Reviewed 2026-10-29. Product decisions: [confirmed intent](../intent/codync-continuation.md).
Design boundary: [proposed capability map](../plans/codync-continuation-capability-map.md).

## Conclusion

Keep Codync's host, agent actors, ACP integrations, context management, revision sync, desktop
store/navigation, remote transport and platform packaging as the starting point. Adapt selected
Zyndi features at those boundaries. Do not make Codync a frontend for a second Zyndi host, copy
Zyndi's entire executor/runtime, or replace Codync's desktop app to obtain its visual design.

The selected work is substantial in three places: turning personal-device access into team
authorization; adding named provider accounts and direct API execution; and routing computer-use
requests to the correct member's desktop. Rich documents and page annotations are a separate
extension of Codync's existing artifacts. Much of voice and the large artifact view already
exists in Codync.

This report distinguishes observations from recommendations. **Source-backed** means the cited
path was traced, not that it passed a fresh test or a live device check. References Cxx and Zxx
at the end resolve to exact source revisions and line anchors.

## Baselines and method

| Source | Revision | Use |
| --- | --- | --- |
| Zyndi working repository | `b922a89dce1f9b20b4adecc9cccf99aef99fa5d0` | Feature implementation and tests |
| `kynjal26/Codync` GitHub HEAD | `2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0` | Destination baseline; desktop package declares 2.32.0 |
| Existing `../Codync` checkout | `e770dbd9ebaf486fe2e44aefa35ddcf0e278f174` | Older reference, preserved unchanged; not treated as current destination |

The remote HEAD was checked again for this investigation. Selected source was read from a
temporary bare clone and source snapshot outside both checkouts. Neither reference services nor
application implementations were changed. No runtime database, login material or repository
credential file was read. Existing untracked `.lavish/` material preceded this documentation step.

Read paths include host dispatch/authorization, event replay, Bot execution, fallback identities,
provider storage and sign-in, voice ownership/tools/calling, memory propagation, computer-use
drivers and desktop routing, artifacts/pages/editors/frame isolation, desktop tokens/store/views,
and migration/import code. Regression tests were inspected to establish intended behavior and
candidate verification cases. Target code and test suites were not executed in this source study;
no Mac/Windows runtime or rendered UI certification is claimed.

Earlier Zyndi comparisons are historical evidence only. In particular, claims that Zyndi is
stateless-only, lacks a complete Rust host, has no relay, or has no installers are not a reliable
current architecture description. The old plan to retire Codync has the opposite objective to
the confirmed continuation intent.

## 1. Team access: a genuine addition, not a device-scope rename

### What Codync already does

Codync's authenticated caller is local, an authorized device with scopes, or a pairing device.
Device scopes distinguish Control and Screen, while selected methods are local-only. Channels
recheck device authorization and close on revocation. This is a real existing boundary and
should remain. It is not a workspace member/role/resource model. [C01] [C02]

The SSE/channel event implementation catches up Bot rows, entries and artifacts. It filters some
local-only events and routes a call's live cues to its holding device. It does not implement
per-person audiences for ordinary history and catch-up. Adding member permissions just to RPC
writes or sidebar rows would therefore be incomplete. [C03]

### What Zyndi contributes

Zyndi resolves devices to people, rejects suspended/removed/unnamed people, classifies RPC
permissions by role and resource, stamps request authors, and limits member configuration changes.
Its privacy layer distinguishes shared entries from audienced entries; reads, events and model
input have to agree. Its session key separates a person's execution lane from the owner's and
separates private exchanges from shared conversations. [Z01] [Z02] [Z03]

Members are not all isolated private users in current Zyndi: ordinary shared chat remains shared,
and a Member generally uses every Bot while a Guest is assigned specific Bots. The desired
enterprise behavior must be specified explicitly rather than inferred from the word "member."

Zyndi also has restricted process launching, OS-user mapping, read-only/read-write shared folders,
account/model entitlements, and per-person desktop routing. Those are useful behavioral references,
but the relevant files depend on Zyndi's store, protocol, session registry and executor. [Z04]
[Z05] [Z06]

### Recommended adaptation

Extend Codync's existing authenticated caller with a verified workspace principal. Keep device
identity distinct from person identity. A supplied `personId` in an untrusted request must never
establish identity. Bind device-to-person membership at the host boundary, not just in the UI.

Define one access decision interface used by RPC, history, event catch-up/live delivery,
attachments, artifacts, search, export and model-context preparation. Partition persistent agent
context by the effective principal/privacy domain: a shared Bot must not resume the owner's
privileged process to answer a restricted colleague. Capability checks and OS restrictions serve
different purposes; approving a tool must not widen a restricted person's filesystem access.

Do not transplant Zyndi's header-based bridge, link daemon or 2,000-line access module wholesale.
Retain Codync's encrypted channel and revocation handling, extending its host-side authorization.

### Verification that must precede team cutover

- Two people use the same Bot with different host-folder and connector grants.
- The restricted person's process cannot read owner credentials or another person's files.
- History pagination, reconnect replay, search, previews, downloads and model context reveal only
  the permitted material; denied content is not merely hidden in rendering.
- Suspension/revocation applies to open connections and work waiting for approval.
- The owner/admin path retains Codync's normal host access; private call context never appears
  in a later shared answer.

Zyndi regression donors: `people.test.ts`, `access.test.ts`,
`native-private-entries.test.ts`, `native-auth-conformance.test.ts`. These are behavior donors,
not justification for importing the excluded operation-receipt subsystem.

## 2. Provider accounts: retain Codync selection, add stable account identities

### What Codync already does

Codync discovers installed harnesses and the ACP registry, selects models and model options,
drives agents through its actor/session machinery, and has failure classification, shared rests,
fallback carry-over and switch-back behavior. These are not Zyndi inventions. [C04] [C05] [C06]

Its current fallback identity is usually the backend, or a backend plus a hash of a custom
command. Validation allows at most three fallbacks and rejects duplicate backend identities.
That does not model two independently signed-in Claude accounts as first-class selectable
provider accounts. A custom launch command is an escape hatch, not equivalent account-management
UX, credential ownership or entitlement enforcement. [C06] [C07]

### What Zyndi contributes

Zyndi stores account UUIDs, provider kind, label, endpoint/binary, defaults and private sign-in-home
metadata. Client-facing account reads select metadata, not encrypted key material. API secrets
are sealed with an account-specific association. New owned account homes are private on Unix;
removal deletes the exact owned home rather than a foreign CLI login directory. [Z07] [Z08]

The desktop groups subscription accounts by agent, adds named sign-ins, displays status/usage,
selects defaults and cleans up accounts created for abandoned sign-in flows. Host-run sign-in
supports a remote Computer, with CLI-native authorization instead of asking the app to copy
login tokens. Named-account UI and flows are traced in the existing design-reference document,
including Monocode attribution. [Z09] [Z10] [Z11]

Zyndi also runs direct Anthropic and OpenAI-compatible API providers with its own tool loop,
in addition to subscription/ACP adapters. A preset list alone is not API-provider execution.
Zyndi's account-based fallback explicitly documents its Codync origins. [Z12] [Z13]

### Recommended adaptation

Add a stable account reference to Codync's Bot and fallback selection, while preserving the
existing backend/model configuration for compatibility. Scope rest, usage, sign-in state and
agent sessions by account identity. Permit multiple accounts of the same backend, and validate
fallback uniqueness by the effective account/model choice rather than just backend name.

Use Codync's existing actor to own all turns, stop requests, routine/group scheduling and
transcript emission. Direct API execution needs an adapter at that execution boundary, with
normalized streaming, tool calls, usage and errors. Do not run a second competing Zyndi queue or
copy its whole native tool loop into the actor. Whether a small driver abstraction or an ACP
adapter is the smaller fit remains a module-level design decision, not a settled source fact.

Preserve Codync's existing secure vault rather than replacing it with Zyndi's host-key format.
Keep each subscription's private config/login home separate; do not change the user's ordinary
CLI sign-in. Restrict account CRUD, credentials and global availability to admins. Apply a
member's entitlements to every fallback candidate, not only the initially selected account.
Zyndi's account/model plans are a useful reference; they are not monetary quotas. [C08] [Z06]

Do not copy its error handling blindly: the reviewed entitlement helper `choice_refusal` maps a
storage error through `.ok().flatten()`, making failure indistinguishable from no refusal to
its caller. The new account authorization contract must distinguish an unavailable policy store
from permission to use an account. This source observation is not a live exploit finding. [Z06]

### Acceptance examples

- Work and Personal Claude accounts coexist; signing out one does not sign out the other.
- A Bot chooses an account/model; a usage-limit fixture moves it to the next allowed account,
  announces the change, preserves continuity, then returns to the preferred account.
- A fallback cannot access an account a member is not entitled to use.
- API and harness execution both preserve Codync's cancellation, transcript and routine semantics.
- List/check/status replies and app logs never include credential values.
- Cancelling a new account's sign-in removes only the new account, not an existing login.

Donor tests: `accounts.test.ts`, `sign-in.test.ts`, `subscription-usage.test.ts`,
`fallbacks.test.ts`, desktop `accounts.spec.ts` and `api-providers.spec.ts`.
Preserve Codync's existing `fallback_e2e.rs` and sign-in/agent tests as the baseline.

## 3. Desktop: visual adaptation, not a frontend replacement

### What is already in Codync

Codync has its own React/Electron desktop store, local/SSH/remote host connections, agent/model
picker, grouped settings, resizable side panels, working steps, approval cards, library,
full-size artifacts, voice controls, platform helper setup and updater. These remain the UX
and implementation base. [C09] [C10] [C11]

The declared dependency ranges differ: Codync desktop uses Electron `^44.5.1`, React `^19.3.0`,
Vite `^7.3.6`; Zyndi declares Electron `^45.0.0`, React `^19.3.1`, Vite `^8.3.5`, React Query
and Tiptap. These are manifest ranges, not a verified inventory of installed versions.
Visual styling does not justify upgrading Codync's runtime or adopting Zyndi's client state stack.

### What Zyndi contributes

Zyndi has a distinct token vocabulary: blue action/user accents, softer sidebar/surface ramps,
hairlines, switches and a different base type scale. Codync has its own monochrome theme,
semantic colors, radii, text scaling and motion. Map roles between these systems rather than
paste Zyndi's global stylesheet over Codync. [C12] [Z14]

Some Zyndi interaction structures were explicitly ported from Codync: its details view names
Codync's working indicator/trace as sources, and its settings mapping identifies Codync's
computer, pairing and update screens. Preserve attribution and avoid re-porting these back as
if they were new UX. [Z11] [Z15]

### Concrete visual mapping to review

| Codync surface retained | Zyndi visual donor | Preserve from Codync |
| --- | --- | --- |
| Desktop theme and controls | `styles.css`, `common.tsx`, `icons.tsx` | Semantic states, text-size preference, keyboard/focus behavior, reduced motion |
| Roster/sidebar | `Sidebar.tsx`, character drawing | Computer selection, sorting/filtering, unread behavior and existing shortcuts |
| Conversation/composer | Bot/chat components and composer styles | Draft ownership, sending/reply semantics, attachments, scrolling and interruption behavior |
| Settings and agent picker | `SettingsDialog.tsx`, `ProviderAccounts.tsx` | Settings grouping/navigation and platform setup flows; add account controls where required |
| Working details/approvals | `DetailsView.tsx`, permission styling | Codync's trace/final-answer distinction and approval behavior, subject to selected team policy |
| Artifact surface | `Pages.tsx`, document styles | Codync's existing open/close/rename/share flow, extended for explicitly selected editing tools |
| Call controls | Zyndi call styling | Codync's audio ownership, engine lifecycle and accessible controls where already supported |

A renderer rewrite to Zyndi's `App.tsx`, global RPC contract, React Query or `packages/link`
would import its state semantics and violate the selected Codync-UX principle. A styling pass
must pass the same interaction tests before new functional extensions are enabled.

No pixel-fidelity claim is made yet. A follow-up visual baseline needs real Codync and Zyndi
screens from isolated fixtures on Mac and Windows, light/dark, normal/large text, working/error,
and constrained window sizes. Generated mockups cannot substitute for these screenshots.

## 4. Rich creation workspace: extend existing artifacts

### Codync already has the larger layout

Its artifact is a self-contained HTML document with a Bot/chat card and Library entry. The
desktop opens it full-size over the chat, provides rename/share, reloads updates and keeps a
composer below it that replies to the artifact card. A basic HTML dashboard and a large preview
are already supported. [C13] [C14]

### Actual selected Zyndi additions

- Structured interactive canvas source/data, compile diagnostics and a page-kit runtime.
- Markdown documents with an in-place rich editor and autosave state.
- Revisions of source/data/build output; compare-and-set writes and restore.
- Page-element/text selection and anchored comments sent to the Bot for targeted changes.
- Find, export and reusable document data blocks where part of the selected creation flow.

The host writes a new revision rather than silently overwriting concurrent work, and a failed
build preserves the last good revision. The editor and its pending-save behavior are substantive
UX additions, not just colors. [Z16] [Z17] [Z18]

Creator/designated-editor permissions are **a target requirement**, not an existing Zyndi
per-page editor ACL proven by this source. The current Zyndi member-rule surface allows page
operations broadly. Build a real artifact/document permission model in Codync's team layer. [Z01] [Z32]

### Keep artifact compatibility and separate runtimes

Preserve Codync HTML artifacts and IDs. Add document/revision/comment types at the existing
artifact boundary instead of creating a second unrelated Library. Existing `publish_artifact`,
`readArtifact` and old clients must keep functioning; unsupported rich edits need an explicit
capability/read-only fallback rather than malformed entries or silent data loss.

Zyndi's compiled-page frame is much more restrictive: an opaque sandbox with a CSP disabling
network, forms, nested frames and workers. Codync's artifact viewer has a separate origin,
allows scripts/storage and broader form/download behavior, with controlled external navigation.
These policies are not interchangeable. Retain existing HTML behavior unless explicitly changed;
specify an isolated runtime for new compiled canvases. Do not weaken Zyndi's compiled-page
isolation just to make it behave like arbitrary HTML. [C15] [C22] [Z19]

Acceptance includes concurrent Bot/user writes, failed compilation, stale anchors after a
revision, restore, autosave failure/retry and non-editor refusals. Test exports as well as the
in-app view. Zyndi donors: host `native-pages.test.ts`, desktop `pages.spec.ts`;
Codync baseline: `artifacts_e2e.rs` and desktop artifact store/frame tests.

## 5. Computer use: preserve Codync's driver, add person-bound execution routing

### Existing Codync capability

Codync already offers background app control through its built-in `computer` MCP server and
cua-driver/platform helpers, including Mac and Windows. Its host enforces one active Bot at a
time for computer use, and phone takeover restricts Bots to looking. It has local-only setup,
platform permissions, secure credential typing and driver lifecycle rules. Keep these. [C16] [C23]

### Zyndi's relevant addition

Zyndi separates the central host from the user's desktop executor. Host requests carry the
person they act for; the event stream and bridge select the right desktop. Local grants are
per Bot/action/subject; host capability grants, member consent and the executor's ceiling are
separate controls. Windows uses its packaged ZComputerUse implementation, while Mac uses the
Mac implementation. That is useful routing/consent work, not a reason to replace Codync's
existing driver with a second engine. [Z20] [Z21] [Z22]

### Recommended boundary

Extend Codync's computer-use dispatch with an execution target. Local host execution continues
through its current driver; a central-host request for a member goes over an authenticated
desktop channel and executes through that desktop's existing Codync helper. The request's actor
and target device must come from verified membership/device state, not model-written fields.

Intersect admin ceiling, Bot capability and local consent. Reject another person's response;
revoking local access stops future dispatch, and cancellation does not imply an already performed
click was undone. Keep the selected ordinary recovery behavior, without porting the excluded
uncertain-operation retry subsystem.

Zyndi's current owner/admin requests can be broadcast to multiple admin desktops; its own ADR
notes that one reconnect can affect another's request. Do not copy that ambiguity. Bind an
execution to one selected desktop with an explicit disconnected/unavailable outcome. [Z20]

Mac and Windows are both initial-release gates. Test central Linux host → member Mac and central
Linux host → member Windows, wrong-device responses, permission refusal/revocation, reconnect,
background control and phone takeover. An Android emulator cannot certify these desktop routes.
Preserve existing Linux behavior; no new Linux desktop-routing parity is implied by the interview.

## 6. Voice: retain the host-owned call system; add selected identity and calling behavior

### Already in Codync

Host-owned calls, on-device/cloud speech, OpenAI/Gemini realtime, call tools, transcript lines,
reconnect catch-up, speech cues and full-size call conversation views already exist. Codync's
voice describes itself as a relay for the Bot. This does not need a whole Zyndi voice transplant.
[C17] [C18]

### Selected Zyndi contributions

The first-person voice uses the Bot's name/personality while real work goes through the agent,
and it is instructed to report project results only after receiving them. Outbound calling adds
callee device registration, invitations, expiry/decline/cancel, one-winner answering and private
call reporting. Call content has an audience and private agent lanes; live cue ownership alone
is not equivalent privacy. [Z23] [Z24] [Z25] [Z02] [Z03]

Use Codync's call storage/announcement/reconnect path as the foundation, extend it with person,
requester, authorized trigger, callee and report audience, and adapt first-person instructions.
Add ringing to existing native mobile clients: do not embed Zyndi's Expo app to obtain it.

### Important separations

| Item | Source finding | Disposition |
| --- | --- | --- |
| Voice speaks as Bot | First-person prompt; agent remains executor | Selected |
| Bot calls a colleague | Zyndi invitation/push/answer path | Selected; preserve explicit requester/routine authorization |
| Private transcript/report | Zyndi audience filtering and private session lane | Selected; requires team-access foundation |
| Exact-request spoken approval ID | Zyndi requires `approvalId`; Codync looks up pending approval | Explicitly excluded; preserve Codync voice-answer behavior, within team approver permissions |
| Native Codex-thread realtime | Zyndi uses experimental `thread/realtime/*`; rejects calls with a non-null person ID | Not implied by first-person voice; no need to replace Codync's agent transport just for it |
| OpenAI engine | Zyndi calls `/live/sessions`; Codync mints `/realtime/client_secrets` | Different transports, not a drop-in swap; preserve Codync engine initially |
| Timing/prewarm refinements | Zyndi has latency metrics and early setup | Useful evidence, not a proven latency guarantee or separately approved engine rewrite |

Zyndi's source explicitly permits provider prewarming while an eligible device is ringing.
Do not describe the current implementation as having no provider activity until answer. Any
prewarm design must distinguish permission to ring, permission to prepare, and permission to
connect audio. [Z24]

Donor fixtures check single-winner claims, wrong provider/device rejection, duplicate end,
push-token invalidation without killing an active call, and private reports. They do not prove
APNs/FCM delivery, OS background behavior, Bluetooth routing or real voice quality. Native
Codex voice fixtures use a fake agent, so they do not certify experimental API compatibility.

## 7. Team knowledge: keep Codync memory rather than import a second learner

Codync already has shared and per-Bot Markdown layers, source-linked facts, revision history,
automatic dreaming with evidence checks, guarded edits, propagation to active sessions and
forgetting tombstones. Calling all these Zyndi additions would be incorrect. Its shared layer
is enabled by default for managed workspaces and disabled by default for project-folder Bots.
[C19] [C20]

Retain that implementation. Introduce workspace/person/restricted ownership at its layer/access
boundary, with source identity from team membership. Automatic learning must not turn a private
call or a sensitive personal fact into company memory. Knowledge a restricted person must not
see must be withheld from the agent context itself, not only from a memory screen.

Zyndi's scoped memory and source-filtering tests are useful examples, but its entire capture
process and separate knowledge ledger are not automatically selected. Its success-only held
memory edits also change Codync's immediate-write semantics; they were not separately selected
and must not enter through a broad memory-module copy. [Z26] [Z27]

Department restriction is more than the existing owner/member/guest roles. The module spec must
define who may publish/correct which restricted layer. No new vector database, extra handbook
or rewritten compaction mechanism follows from this requirement.

## 8. Designated action approval: much of this is new work

Codync has Ask/Auto per Bot and some built-in management approvals. Zyndi adds per-tool modes,
command-family rules, Full access, optional AI review and a unified executor. However, its
ordinary Full access path bypasses non-blocked approval modes. The proposed outbound guard
explicitly describes the missing universal policy. Neither codebase provides the selected
department/designated-approver enterprise policy as a complete implementation. [C21] [Z28] [Z29]

Implement a deterministic action policy at Codync's host dispatch/agent permission boundaries:
actor, action class, target, approved scope, configured exception, and eligible approver. Ask/Auto
can continue for ordinary actions; it cannot silently override a mandatory action-class approval.
Do not import the excluded AI reviewer or operation-receipt retry holds to accomplish this.

Keeping Codync's recovery is an intentional product choice. Mandatory action approval does not
by itself provide exactly-once delivery or establish whether an external action completed before
a crash; the design must not claim those excluded receipt guarantees.

An approval must authorize the actual pending action rather than rewrite the action's inputs;
normal authorization and request routing are required even though the extra spoken-ID safeguard
is excluded. Revocation and role checks remain required team behavior.

### Unresolved enforcement boundary

An agent with arbitrary shell/network access can send a message through `curl`, a CLI or a
script instead of a classified connector. A generated HTML artifact can also perform network
work outside the host's tool dispatcher. A toolbar checkbox or prompt rule cannot enforce
"nothing external without approval" across these paths. Zyndi's own outbound-guard spec notes
the shell limitation. Command-family heuristics are not a general effect detector. [Z29] [C15]

The module spec must choose explicitly between (a) a guarantee for managed tools, with arbitrary
shell/browser paths disclosed as outside it, and (b) broader enforcement through credential/egress
mediation plus approval for escape paths. Option (b) changes the practical meaning of unrestricted
admin execution. This tradeoff is not resolved by the product interview and is a release blocker
for any claim of universal enforcement. Do not silently choose the weaker promise or the more
restrictive UX.

Zyndi's command classifier allows some program names as read-only without interpreting every
behavioral flag. Treat it as convenience logic requiring adversarial tests, not an OS boundary
or proof that a command is harmless. This is a source limitation, not a reproduced exploit
finding in this review. [Z30]

## 9. Migration: preserve meaning, not just row counts

The existing importer goes from Codync to Zyndi. Its history mapper normalizes entry kinds,
caps some long outputs and omits some reasoning/pass entries. Reversing that function cannot
reconstruct information that was not imported. A new exporter/importer with a mapping report is
needed for the selected direction. [Z31]

Use additive target migrations and a separate profile seeded from consistent snapshots. Preserve
stable IDs or keep an explicit source→target map; retain timestamps, authors, attachments,
audiences, reply links, routines and page revisions. Do not reinterpret unknown legacy audiences
as public. Preserve deleted/hidden status and manual skill selections. Archive rejected-feature
records without activating their jobs or UI.

Credential ciphertext cannot simply be copied between Zyndi's sealed format and Codync's vault.
Plan controlled reauthorization or an owner-run transfer with no credential values in logs or
repository files. Database snapshots alone do not preserve page/workspace/attachment files or
the decryption material; the backup/restore rehearsal must cover referenced content. [Z07] [C08]

Agents' internal harness sessions are not portable transcripts. Import user-visible history and
provide a bounded, audienced handoff into a fresh Codync session rather than claiming that a
Zyndi session ID can resume unchanged under Codync's adapter.

Earlier retirement records show that Bot workspaces were moved and old paths left as symlinks.
Do not assume the old Codync profile contains the latest work or reverse those moves blindly.
Produce a proposed path mapping without inspecting live personal content in this study.

Only one installation may execute each external-facing routine, webhook or inbox job during
cutover. Stage with those jobs disabled, verify each consumer and explicitly transfer ownership.
Rollback needs preserved profile/files plus an agreed handling of new writes after cutover;
restarting the old binary alone is not a complete rollback plan.

## 10. Regression and release evidence

This stage adds documents only. No functional tests were rerun, no paid model calls were made,
and no application was started. Static verification checks the source-reference paths, line
anchors, pinned revisions, required module IDs and acyclic dependencies.

Implementation verification should extend Codync's existing fixtures rather than replacing its
test stack with Zyndi's. The examined suites include host `e2e.rs`, `fallback_e2e.rs`,
`calls_e2e.rs`, `artifacts_e2e.rs`, `memory_files_e2e.rs`, `team_e2e.rs`, and
`routine_retry_e2e.rs`; desktop uses its declared Node test script and TypeScript check. New
cross-role cases must drive the real Codync executable with fake agents and dummy accounts.

Run bounded build/test jobs through Portly. Example future commands, after the implementation
checkout and scope are approved (not executed here):

```sh
job_id="$(portly temp 'cargo test --locked' --path <implementation-checkout>/host --timeout 30m)"
portly wait "$job_id"
job_id="$(portly temp 'npm run typecheck && npm test' --path <implementation-checkout>/apps/desktop --timeout 30m)"
portly wait "$job_id"
```

Mac/Windows packaging and real device sessions are required for desktop execution; physical
phones and provisioned push are required for incoming-call background claims. Record separately:
source review, fake-agent tests, rendered UX, real provider behavior, and operational cutover.
Passing one level does not certify the others. Preserve existing encryption, update verification,
local-only permission setup, secure credential typing and attribution.

## Open decisions for module specifications

1. **Action enforcement:** managed-tool guarantee or broader egress/credential mediation; reconcile
   that choice with admin host freedom and artifact network behavior.
2. **Direct API integration:** choose the smallest adapter preserving Codync actor/session and
   permission semantics; identify which providers beyond Anthropic/OpenAI-compatible are needed.
3. **Editor identity:** define page creator, designated editors, admin overrides, restricted viewers
   and how Bot edits inherit their requester's authority.
4. **Device selection:** define selection when a person has several desktops, including owner/admin
   devices; no implicit broadcast to all admin computers.
5. **Mobile capability evolution:** retain the native clients, but define read-only compatibility,
   team identity rollout and ringing/report support for each released client version.
6. **Knowledge authority:** department/restricted-layer membership and correction rights on top of
   Codync's existing automatic learning.
7. **Cutover operation:** identify the active deployment and routine consumers through a separately
   authorized operational inventory; rehearse a recoverable export on a copy before live changes.

These do not reopen the confirmed product direction. They bound the technical decisions that
must be made without importing rejected Zyndi behavior.

## Source references

- [C01](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/api/devices.rs#L12): `host/src/api/devices.rs`, line 12.
- [C02](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/remote/channel.rs#L382): `host/src/remote/channel.rs`, line 382.
- [C03](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/api/events.rs#L55): `host/src/api/events.rs`, line 55.
- [C04](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/backends.rs#L80): `host/src/agent/backends.rs`, line 80.
- [C05](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/bot/mod.rs#L1): `host/src/agent/bot/mod.rs`, line 1.
- [C06](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/fallback/mod.rs#L106): `host/src/agent/fallback/mod.rs`, line 106.
- [C07](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/fallback/rest.rs#L45): `host/src/agent/fallback/rest.rs`, line 45.
- [C08](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/market/vault.rs#L19): `host/src/market/vault.rs`, line 19.
- [C09](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/client/host-client.ts#L143): `apps/desktop/src/renderer/client/host-client.ts`, line 143.
- [C10](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/store/bot-store.ts#L51): `apps/desktop/src/renderer/store/bot-store.ts`, line 51.
- [C11](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/thread/ChatSidePanel.tsx#L11): `apps/desktop/src/renderer/views/thread/ChatSidePanel.tsx`, line 11.
- [C12](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/styles/theme.css#L3): `apps/desktop/src/renderer/styles/theme.css`, line 3.
- [C13](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/artifacts.rs#L30): `host/src/chat/artifacts.rs`, line 30.
- [C14](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/library/ArtifactView.tsx#L18): `apps/desktop/src/renderer/views/library/ArtifactView.tsx`, line 18.
- [C15](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/main/artifacts.ts#L19): `apps/desktop/src/main/artifacts.ts`, line 19.
- [C16](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/computer-use.md#L1): `docs/features/computer-use.md`, line 1.
- [C17](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/voice-call.md#L1): `docs/features/voice-call.md`, line 1.
- [C18](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/calls/tools.rs#L111): `host/src/chat/calls/tools.rs`, line 111.
- [C19](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/context-and-memory.md#L42): `docs/features/context-and-memory.md`, line 42.
- [C20](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/memory/mod.rs#L27): `host/src/chat/memory/mod.rs`, line 27.
- [C21](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/bot/updates.rs#L154): `host/src/agent/bot/updates.rs`, line 154.
- [C22](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/library/ArtifactParts.tsx#L29): `apps/desktop/src/renderer/views/library/ArtifactParts.tsx`, line 29.
- [C23](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/screen/cua.rs#L35): `host/src/screen/cua.rs`, line 35.
- [Z01](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/access.rs#L313): `rust/crates/zyndi-host/src/access.rs`, line 313.
- [Z02](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-store/src/privacy.rs#L46): `rust/crates/zyndi-store/src/privacy.rs`, line 46.
- [Z03](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/sessions.rs#L109): `rust/crates/zyndi-host/src/sessions.rs`, line 109.
- [Z04](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/sandbox.rs#L38): `rust/crates/zyndi-host/src/tools/sandbox.rs`, line 38.
- [Z05](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/run_as.rs#L24): `rust/crates/zyndi-host/src/run_as.rs`, line 24.
- [Z06](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/governance.rs#L247): `rust/crates/zyndi-host/src/governance.rs`, line 247.
- [Z07](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-store/src/providers.rs#L9): `rust/crates/zyndi-store/src/providers.rs`, line 9.
- [Z08](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/provider_catalog_rpc.rs#L7): `rust/crates/zyndi-host/src/provider_catalog_rpc.rs`, line 7.
- [Z09](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/ProviderAccounts.tsx#L73): `apps/desktop/src/renderer/components/ProviderAccounts.tsx`, line 73.
- [Z10](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/sign_in_rpc.rs#L20): `rust/crates/zyndi-host/src/sign_in_rpc.rs`, line 20.
- [Z11](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/desktop-codync-reference.md#L1): `docs/desktop-codync-reference.md`, line 1.
- [Z12](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-providers/src/lib.rs#L399): `rust/crates/zyndi-providers/src/lib.rs`, line 399.
- [Z13](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/fallback.rs#L1): `rust/crates/zyndi-host/src/fallback.rs`, line 1.
- [Z14](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/styles.css#L1): `apps/desktop/src/renderer/styles.css`, line 1.
- [Z15](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/DetailsView.tsx#L9): `apps/desktop/src/renderer/components/DetailsView.tsx`, line 9.
- [Z16](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/pages/mod.rs#L26): `rust/crates/zyndi-host/src/pages/mod.rs`, line 26.
- [Z17](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/DocumentEditor.tsx#L112): `apps/desktop/src/renderer/components/DocumentEditor.tsx`, line 112.
- [Z18](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/Pages.tsx#L796): `apps/desktop/src/renderer/components/Pages.tsx`, line 796.
- [Z19](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/main/page-frame.ts#L31): `apps/desktop/src/main/page-frame.ts`, line 31.
- [Z20](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/rust-integration/adr/0006-computer-use-on-the-mac.md#L79): `docs/rust-integration/adr/0006-computer-use-on-the-mac.md`, line 79.
- [Z21](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/mac_bridge.rs#L144): `rust/crates/zyndi-host/src/mac_bridge.rs`, line 144.
- [Z22](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/executor/executor.ts#L24): `apps/desktop/src/executor/executor.ts`, line 24.
- [Z23](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/voice_context.rs#L220): `rust/crates/zyndi-host/src/voice_context.rs`, line 220.
- [Z24](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/calls.rs#L833): `rust/crates/zyndi-host/src/calls.rs`, line 833.
- [Z25](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/voice_calls.rs#L608): `rust/crates/zyndi-host/src/voice_calls.rs`, line 608.
- [Z26](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/knowledge.rs#L1): `rust/crates/zyndi-host/src/knowledge.rs`, line 1.
- [Z27](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/memory_ops.rs#L558): `rust/crates/zyndi-host/src/memory_ops.rs`, line 558.
- [Z28](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/executor.rs#L1207): `rust/crates/zyndi-host/src/tools/executor.rs`, line 1207.
- [Z29](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/specs/outbound-guard.md#L15): `docs/specs/outbound-guard.md`, line 15.
- [Z30](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/command_policy.rs#L20): `rust/crates/zyndi-host/src/tools/command_policy.rs`, line 20.
- [Z31](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/codync_import/history.rs#L125): `rust/crates/zyndi-host/src/codync_import/history.rs`, line 125.
- [Z32](https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/packages/protocol/src/domain.ts#L2060): `packages/protocol/src/domain.ts`, line 2060.

[C01]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/api/devices.rs#L12
[C02]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/remote/channel.rs#L382
[C03]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/api/events.rs#L55
[C04]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/backends.rs#L80
[C05]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/bot/mod.rs#L1
[C06]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/fallback/mod.rs#L106
[C07]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/fallback/rest.rs#L45
[C08]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/market/vault.rs#L19
[C09]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/client/host-client.ts#L143
[C10]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/store/bot-store.ts#L51
[C11]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/thread/ChatSidePanel.tsx#L11
[C12]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/styles/theme.css#L3
[C13]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/artifacts.rs#L30
[C14]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/library/ArtifactView.tsx#L18
[C15]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/main/artifacts.ts#L19
[C16]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/computer-use.md#L1
[C17]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/voice-call.md#L1
[C18]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/calls/tools.rs#L111
[C19]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/context-and-memory.md#L42
[C20]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/chat/memory/mod.rs#L27
[C21]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/agent/bot/updates.rs#L154
[C22]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/apps/desktop/src/renderer/views/library/ArtifactParts.tsx#L29
[C23]: https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/host/src/screen/cua.rs#L35
[Z01]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/access.rs#L313
[Z02]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-store/src/privacy.rs#L46
[Z03]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/sessions.rs#L109
[Z04]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/sandbox.rs#L38
[Z05]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/run_as.rs#L24
[Z06]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/governance.rs#L247
[Z07]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-store/src/providers.rs#L9
[Z08]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/provider_catalog_rpc.rs#L7
[Z09]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/ProviderAccounts.tsx#L73
[Z10]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/sign_in_rpc.rs#L20
[Z11]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/desktop-codync-reference.md#L1
[Z12]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-providers/src/lib.rs#L399
[Z13]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/fallback.rs#L1
[Z14]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/styles.css#L1
[Z15]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/DetailsView.tsx#L9
[Z16]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/pages/mod.rs#L26
[Z17]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/DocumentEditor.tsx#L112
[Z18]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/renderer/components/Pages.tsx#L796
[Z19]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/main/page-frame.ts#L31
[Z20]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/rust-integration/adr/0006-computer-use-on-the-mac.md#L79
[Z21]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/mac_bridge.rs#L144
[Z22]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/apps/desktop/src/executor/executor.ts#L24
[Z23]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/voice_context.rs#L220
[Z24]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/calls.rs#L833
[Z25]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/voice_calls.rs#L608
[Z26]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/knowledge.rs#L1
[Z27]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/memory_ops.rs#L558
[Z28]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/executor.rs#L1207
[Z29]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/docs/specs/outbound-guard.md#L15
[Z30]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/tools/command_policy.rs#L20
[Z31]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/rust/crates/zyndi-host/src/codync_import/history.rs#L125
[Z32]: https://github.com/kynjal26/zyndi/blob/b922a89dce1f9b20b4adecc9cccf99aef99fa5d0/packages/protocol/src/domain.ts#L2060
