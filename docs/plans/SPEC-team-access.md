# Specification: team-access onboarding and membership

Status: proposed for owner review, 2026-10-29. No implementation authorized by this document.
Module: `team-access` in [the capability map](codync-continuation-capability-map.md).
The owner's latest instruction authorizes designing this module using Clerk and Google;
other module boundaries remain proposed. This is the Specify phase, not an implementation task list.

## Objective and assumptions

An owner creates a workspace, connects its host and invites colleagues by email. A colleague
accepts an invitation, signs in and sees only the shared Bots and resources granted to them.
Ordinary members do not pair manually with the central host. Preserve all existing Codync
capabilities and its encrypted device channels.

Assumptions proposed for review:
- Initial deployment is the owner's team, one workspace with one central host. Model multiple
  workspaces and hosts without making them mandatory setup steps.
- Clerk remains identity/session and organization membership provider; Google is the primary
  login option. Preserve existing Apple support, particularly for the native iPhone client.
- Invite-only enrollment; no automatic admission by email domain. Non-Google login can use a
  verified-email method offered by the configured Clerk instance; exact enabled methods need
  a native-client compatibility check before release.
- Product authorization and Bot behavior live in the Rust host. Cloud code handles identity,
  invitation orchestration, host discovery and authorization delivery, not agent execution.
- Team access requires a reachable authorization service. Existing personal/local mode remains
  a distinct Codync mode; it is not a fallback that elevates a restricted remote member.

## Source baseline

Pinned Codync: `2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0`.
- `apps/desktop/src/main/account.ts`: existing Clerk account session and browser sign-in flow.
- `apps/desktop/src/renderer/store/account.ts`: account identity alone does not unlock a computer.
- `cloud/src/auth.ts`: Clerk verification plus signed host/device requests. Current `ClerkUser`
  only exposes user ID and optional email: organization and verified-email resolution are NEW.
- `cloud/src/routes/host.ts`: host-key registration and computer ownership, existing access routes.
- `cloud/src/routes/account.ts`, `cloud/src/routes/common.ts`, `cloud/src/routes/webhooks.ts`:
  account/access integration points; extend rather than fork the account service.
- `cloud/package.json`: `@clerk/backend ^3.20.1`, Svix, Cloudflare Workers/D1, Vitest.
- Desktop remains Electron/React; native iPhone remains SwiftUI. Rust host remains authoritative.

Zyndi references: `apps/desktop/src/renderer/components/PeopleSettings.tsx` offers email + role,
pending sign-ins, person details, suspension/removal and device revocation.
`rust/crates/zyndi-host/src/people_rpc.rs` supplies person-oriented host contracts. Adapt the
People experience and actor concept, not its whole sign-in/runtime implementation.

## Architecture and authority

1. Clerk owns external user identity, authentication sessions and organization membership.
2. The team's own Codync cloud service maps Clerk subjects/organizations to stable internal
   principals/workspaces, orchestrates invites and distributes signed, expiring membership
   assertions. It holds no Bot chat, prompt, memory or provider credentials for this feature.
3. Each Rust host owns its workspace binding, resource grants, effective authorization, device
   enrollment, audit entries and chat/model-context filtering. Host-local denials override cloud
   admission. A cloud organization admin is not automatically a local OS administrator.
4. Desktop/iPhone render authorized host state and submit intents. They never assert their own
   role, person ID or access scope as trusted authorization facts.

One external organization maps to one internal workspace. Use `(issuer, subject)` for identity,
not email as a permanent primary key. Email selects an invitation; after acceptance bind the
membership to the verified subject. Email changes do not silently transfer membership.
Do not auto-merge Apple private-relay and Google accounts by display name or guessed email.

An owned deployment needs its own Clerk application and Codync cloud/relay configuration.
Do not reuse upstream production credentials or assume its hosted service supports this team.

## Roles and grants

Owner: workspace lifecycle, ownership transfer and all admin actions. Exactly one owner initially.
Admin: invitations, member management, provider availability and authorized host resource setup.
Member: shared Bots and explicitly granted resources; chooses approved accounts/models.
Guest: only explicitly selected Bots/conversations/resources; no default company-wide visibility.

Owner cannot be suspended, removed or demoted before an atomic ownership transfer. No self-
promotion. Admins cannot change the owner or grant a privilege they do not hold. Approver is a
separate resource/action designation, not a fifth global role. Membership, host access, resource
grants and personal desktop consent are separate records. Routine execution has its own principal
and must not inherit a removed member's authority.

## Owner onboarding

1. Welcome → Sign in (Google prominent; existing supported methods retained).
2. Choose Create workspace or Accept invitation. Invitees never create a workspace accidentally.
3. Name workspace → backend creates organization and internal mapping with an idempotency key.
   Partial provider failure leaves a resumable setup, not a second workspace on retry.
4. Connect central host: local desktop presents that host; server setup offers a one-time claim
   command/link. Host signs a challenge; authenticated owner confirms the displayed host
   fingerprint/name. Proof of account alone cannot claim an arbitrary computer. An already-owned
   host requires explicit transfer from its current authority.
5. Choose initial shared Bots/resources and approved accounts. Existing private Bots remain
   private until selected. Host binding becomes active only after both sides acknowledge.
6. Invite teammates, or finish and invite later. Checklist shows host connected, accounts ready,
   Bots shared, invitations sent. Host offline is a useful state, not a perpetual spinner.

## Invitation and member onboarding

Admin opens Settings → People → Invite. Enter one or several emails, default Member role,
and initial access selections. Bulk entry reports per-address results. Role/grants changes after
sending are explicit and visible. No provider secret is included in an invitation.

Invitation state: preparing → pending → accepted; pending can become expired or revoked.
Email dispatch failure remains pending with delivery failure metadata and a resend control;
it does not create membership. Proposed default expiry: seven days, configurable by owner.
Resend replaces the previous usable invitation token. Clerk's invitation is the canonical token;
application record stores external ID/status, never raw token in logs.

Recipient opens a browser invitation page: inviter, workspace, invited email and Join action.
Sign in/create account, verify the invited email, then accept. Existing accounts are supported.
Wrong account shows Switch account or verify/add the invited address; it never silently admits
another address. Reject expired/revoked/replayed tokens. An already accepted invitation for the
same person returns the existing membership, not a duplicate. Forwarding the link cannot change
which identity is eligible. Backend rechecks current invitation and membership with Clerk.

After acceptance, activate membership synchronously with the verified provider result. Do not
wait for a webhook to create the user before allowing onboarding to finish. Show Open desktop /
Open iPhone with browser fallback; OS deep links carry only a short-lived one-use exchange code
bound to the initiating device/state, never a reusable session token. Authenticate through the
system browser using existing Codync conventions; do not embed Google credentials in a webview.

Signing in directly without opening email shows pending invitations for verified addresses.
No invitation means No workspace access, with a request-access path that reveals no team roster.
Accepting one invitation does not accept invitations to other workspaces.

## Device enrollment and host admission

1. Client creates/loads its existing Codync device keys and proves key possession to cloud.
2. Authenticated member selects workspace/host. Cloud verifies active membership and issues a
   short-lived signed admission assertion bound to workspace, principal, membership generation,
   host ID, device public-key fingerprint, request nonce, audience and expiry.
3. Client sends assertion through the existing host enrollment/channel handshake and proves
   possession of that device key. Host verifies issuer/key/audience/nonce/time, its own workspace
   binding and current membership eligibility, then persists a person-bound device grant.
4. Host computes effective access from member role AND local grants AND device scopes. Never map
   every accepted member to Codync's unrestricted loopback caller or a broad Control grant.
5. Thereafter use Codync's encrypted channel, revision reconnect and device revocation. Membership
   renewal authorizes the channel; it does not replace encryption or send Clerk tokens per tool.

This automatic team admission is new protocol work, not existing Codync behavior. Capability-
negotiate `teamAccessV1`; old clients keep existing personal behavior but cannot enter team mode
until upgraded. Existing broad paired-device grants cannot be reused as member grants without
explicit classification. Account login alone remains insufficient to unlock a computer.

Personal desktop computer-use consent belongs to `desktop-device-bridge`: an admitted device is
not an execution target until its user enables that feature and chooses local apps/folders.

## Revocation, expiry and disconnected operation

Proposed measurable targets: host-local suspension takes effect before its success response;
cloud-origin suspension reaches connected hosts within five seconds normally. Admission leases
expire after at most 60 seconds. These are design targets to prove, not guarantees already met.

Every remote request and event delivery checks the current host membership generation, status,
lease expiry and resource grants. Active streams and model work do not bypass rechecks. When
removal is applied: invalidate grants/streams, remove queued person-owned work, cancel active
work where possible and stop further tool dispatch/result delivery. Already executed external
effects and previously downloaded data cannot be recalled. Handle shared routines via explicit
ownership reassignment; do not silently delete shared Bots/history.

People administration is submitted to the host, which applies a local deny immediately and
records a durable cloud-sync operation. UI distinguishes Locally blocked / Sync pending from
Fully removed. Cloud/Clerk dashboard changes arrive via verified webhook plus periodic provider
reconciliation. Neither a delayed webhook nor an unexpired JWT may erase a host-local denial.

Lease refresh must check provider session/membership liveness, not just reverify an old JWT.
If that freshness check or control-plane connectivity fails, do not mint a fresh lease. Existing
team operations stop when the lease expires; retain local drafts and explain Reconnecting.
No indefinite offline team authorization. Owner's deliberate local maintenance access remains
separate and audited; it must not be exposed to a remote member through loopback forwarding.

Webhook event IDs are deduplicated; use monotonic application membership generations and fresh
provider reads to resolve out-of-order events. Rejoining creates a new membership generation;
old devices, links and cached grants do not reactivate automatically.

## Data contracts (proposed, not existing endpoints)

Cloud data: workspace mapping, principal mapping, membership projection/status/generation,
invitation external ID/status/delivery, host binding, admission signing-key ID and durable sync
operations. Host data: workspace binding, principals, memberships, resource grants, device-person
bindings, consumed admission nonces, latest lease, audit events and pending admin sync operations.
Use additive D1/SQLite migrations; no credentials in these records or client-visible metadata.

Host operations, following existing JSON RPC conventions:
- `workspace.get` → current authorized workspace/setup state and capabilities.
- `people.list` → paginated permitted roster; guests receive no full directory by default.
- `people.invite` {emails, role, initialGrants, idempotencyKey} → per-email invitation results.
- `people.invitationResend` / `people.invitationRevoke` {invitationId, expectedRevision}.
- `people.update` {membershipId, expectedRevision, role?, grants?}.
- `people.suspend` / `people.remove` {membershipId, expectedRevision, idempotencyKey}.
- `workspace.transferOwner` {targetMembershipId, expectedRevision, recentAuthProof}.
- `people.me` → effective permissions, host availability and membership state.

Cloud operations: workspace create, host claim begin/confirm, invitation acceptance, device
admission and membership lease refresh. Use the existing cloud route/error/signature conventions;
exact wire schemas and SDK calls must be verified against installed versions before coding.
An authenticated browser acceptance endpoint validates session and CSRF/origin; app exchange
codes use PKCE/state and device binding. Keys live in secure platform storage.

All mutations validate actor authority on the server and use expected revisions for stale
changes. Idempotency keys are unique per intent with payload hash and atomic uniqueness;
duplicate in-flight operations return pending/conflict. This is control-plane retry safety,
not the excluded Zyndi operation-receipt system for uncertain agent actions.
Machine-readable errors: notAuthenticated, forbidden, invitationExpired, invitationRevoked,
invitationEmailMismatch, revisionConflict, hostOffline, membershipRevoked, authorizationUnavailable.
Adapt these to Codync's existing envelope; never leak whether unrelated emails are members.

## UX states and administration

### Preserve Codync secure credential cards

Codync already implements `request_secret`, `request_connection` and `request_login` in
`host/src/market/requests.rs`, with dedicated desktop/iPhone forms and host-vault storage.
Reference: [connector credentials](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/connector-credentials.md).
Keep this mechanism: secret values go through the authenticated encrypted channel to their
intended host/connector, never into ordinary messages or non-secret completion notices.
Prefer OAuth where the connector supports it; retain optional 1Password references.

Team extension: existing broad Control-device permission to finish a credential request is
insufficient. Require an active authorized actor for that specific account/connector/request;
admin-managed shared credentials are completed by an admin or explicitly delegated credential
manager. Members use assigned company connectors without reading stored secrets and cannot
independently add private accounts. Provider-required user consent completes an admin-created
company assignment. Company policy governs work-data visibility and admin oversight. Do not
broadcast credential setup forms or sensitive login metadata to the whole Bot conversation.
Recheck authority when
submitting, even if the form was opened before suspension. Audit who configured which destination
and outcome, never the value. Reject changed destinations, expired requests and wrong actors.

Website `type_login` checks the site/app and password field and does not return the password to
the Bot. It is not implemented on Windows at this source revision. Preserve supported behavior;
Windows support requires separate acceptance work or an explicit manual-sign-in flow. Do not
claim the vault isolates secrets from arbitrary same-OS-user code or unrestricted agent shells;
Codync documents that boundary. New team isolation must respect that limitation.

Acceptance: raw credentials absent from chat, model context, events and logs; unauthorized
members cannot submit/read admin-managed credentials or see unauthorized setup requests; successful setup
resumes the intended Bot with metadata only. Verify both direct API calls and replay streams.

People tabs: Members, Invitations, Requests (explicit admin approval only). Each member detail:
role, granted Bots/resources/accounts/models, signed-in devices, suspend/remove controls.
Show effective access plainly; credential values never appear for members. Audit invitation,
acceptance, permission changes, device enroll/revoke, suspension and ownership transfer.

Cover interrupted setup, host offline, expired invitation, wrong account, removed membership,
unsupported client, revoked device, provider outage and partial email delivery. Persist setup
progress. Keep Codync navigation/settings UX and apply Zyndi visual tokens; do not replace the
desktop store or introduce a second onboarding state machine inside each client.

## Project structure, style and commands

This document was originally drafted in Zyndi while the existing Codync checkout was read-only.
The owner has now authorized this separate Codync continuation checkout as the writable
destination. The original sibling checkout and its services remain read-only references.
This documentation transfer does not itself approve application implementation or live cutover.
Extend `cloud/src/routes/`, `cloud/src/auth.ts`, cloud D1 migrations/tests; host API/access/storage
and migrations; desktop account/store/settings; native iPhone account/host-client/screens.
Confirm exact host files when implementation checkout is prepared. Keep shared protocol fields
camelCase, Rust internals snake_case, existing structured errors and host-owned policy.

Contract example (illustrative; actor comes from authenticated transport, not payload):
```rust
let actor = caller.require_team_member()?;
let membership = store.active_membership(actor.workspace_id(), actor.person_id())?;
policy.require(&membership, Permission::ManagePeople)?;
store.update_member_if_revision(target_id, expected_revision, change)?;
```

Future bounded checks, in the writable checkout, through Portly:
```bash
portly status
job_id="$(portly temp 'cargo test' --path <writable-codync>/host --timeout 30m)"
portly wait "$job_id"
job_id="$(portly temp 'npm test && npm run typecheck' --path <writable-codync>/cloud --timeout 30m)"
portly wait "$job_id"
job_id="$(portly temp 'npm test && npm run typecheck' --path <writable-codync>/apps/desktop --timeout 30m)"
portly wait "$job_id"
```
Native iPhone checks require an available Mac/iOS device environment. Do not claim a Linux run
validates SwiftUI flows. No deploy commands, email sends or account configuration changes here.

## Success criteria and verification

- Owner completes workspace/host setup; retry at every boundary produces one workspace/binding.
- New and existing users accept invites with the intended verified identity; forwarded, expired,
  revoked and wrong-account links refuse; duplicate acceptance remains one membership.
- Member signs in on desktop/iPhone without central-host QR and sees only granted shared Bots.
- User from workspace A cannot enumerate/download/subscribe to workspace B, even using known IDs.
- Privacy holds for full traces, attachments, artifact routes, revision replay, notifications,
  group delegation, memory and agent session context—not only roster filtering.
- Revoked membership/device loses streams and tool authority within the measured lease bound;
  delayed/duplicated webhook, stale JWT, host restart and network partition cannot revive it.
- Invitation acceptance works before webhook delivery. Provider errors do not become approvals.
- Owner transfer preserves exactly one owner; stale role edits and concurrent remove/rejoin fail
  safely. Account merge/email change requires a deliberate identity migration path.
- Existing personal Codync clients/CLI/SSH behavior remains intact on personal hosts. Team host
  remote identities never gain local-owner rights through legacy caller handling.
- Run host/cloud unit and integration tests, two-host/two-workspace negative tests, desktop
  interaction tests, real Clerk test-instance invite flow with consenting test addresses, and
  native iPhone sign-in/enrollment. Mock tests alone do not certify email or native sign-in.

## Migration and boundaries

Existing Codync personal hosts remain personal until owner opts in. Snapshot DB + referenced
files/keys; retain computer identity and existing clients. Classify paired devices as owner or
member explicitly. Zyndi emails create proposed invitations, not active memberships; map verified
subjects deliberately and preserve authorship/privacy. Never activate an imported admin by email
alone. Team enrollment requires client upgrade; negotiate capabilities without corrupting older
personal profiles. Rollback revokes new admissions and restores profile snapshots, accounting for
writes since conversion; no simultaneous writers to the same profile.

Always: preserve Codync transport/actor/client behavior, enforce on host, add migrations,
write meaningful negative authorization tests, preserve licenses and keep secrets out of logs.
Review separately: live cutover, production account/relay setup, real invitation delivery and
changed product requirements. Never: mutate reference checkout, trust client role/email alone,
silently drop Codync features or port excluded Zyndi safeguards as incidental dependencies.

## Decisions for review and remaining implementation checks

Proposed defaults to approve together: Clerk + Google-primary, invite-only, Owner/Admin/Member/
Guest, one initial central host, seven-day invitation expiry, 60-second maximum team admission
lease with fail-closed team operation, explicit host claim, no central QR for ordinary members.

Before coding, verify Clerk plan/organization limits and enabled authentication methods, native
SDK support for invitation handoff, admission signing-key rotation and lease-refresh load against
actual team size. No plan pricing or SDK compatibility is assumed. Fresh online authorization
adds an availability dependency; relaxing it would require explicitly accepting a longer
revocation window. Final task ordering follows approval of this specification.

## Official documentation reviewed

- https://clerk.com/docs/guides/organizations/overview
- https://clerk.com/docs/guides/organizations/add-members/verified-domains
- https://clerk.com/docs/guides/sessions/session-tokens
- https://clerk.com/docs/guides/sessions/customize-session-tokens
- https://clerk.com/docs/guides/development/webhooks/syncing
- https://clerk.com/docs/guides/development/webhooks/overview

Clerk docs establish organization and identity facilities, token freshness and asynchronous
webhook limitations. The device assertions, host lease rules and resource model above are our
proposed Codync extension, not a Clerk-provided device authorization guarantee.
