# Connected apps architecture: company-managed resources on Codync

Status: confirmed product direction; proposed technical design, 2026-10-29.
Supersedes the employee-managed accounts/private-mailbox proposal. No implementation or service changes.

## Confirmed product direction

Admins configure company app connections and assign them to staff and Bots. Staff use assigned
resources; they cannot independently add private accounts. A company-issued mailbox may be
assigned to a staff member's Bot. Company policy defines work-data visibility and admin oversight.
Assignment to one staff member does not imply access for every colleague. Existing explicit
voice-call/report audiences remain governed by that separately confirmed feature.

Codync remains the implementation base: Rust host, ACP lifecycle, MCP connectors, OAuth,
secure cards, vault, native clients and encrypted channels. Extend those boundaries; do not
introduce OpenDuet's runtime, memory service, broker platform or a parallel connector system.

## Source baseline

Codync 2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0:
- docs/features/marketplace.md: Extensions, registry/custom connectors, Composio Connect,
  per-Bot enablement, OAuth, proxy execution and pinned package updates.
- docs/features/connector-credentials.md: secure cards, vault, OAuth and local-process trust.
- host/src/market/{mod,install,requests,oauth,vault}.rs, market/composio.rs and host/src/mcp.rs
  are existing integration boundaries; retain desktop marketplace/settings patterns.
OpenDuet 677e0fcd148d80b99dd788c6d6f9c1c9f40a72db supplies account/grant ideas only.
See [source investigation](../research/codync-openduet-connected-apps.md).

## Separate concepts, one host-owned policy

Connector installation is code/endpoint and supported tools on a host. Connection identifies
one verified company account/integration, scopes, credentials reference and status. Assignment
identifies staff/groups/Bots allowed to use it. Result audience follows company permissions.
Action approval remains separate from permission to access the account.

Company ownership is the default. A mailbox's external identity says which mailbox it reaches;
it does not create employee-private application ownership. Model-provider accounts remain a
separate domain. Admin oversight follows authorized company roles and is auditable. No claim
of privacy from the host operator: Codync documents its same-user process trust boundary.

## End-to-end flow

1. Admin opens existing Extensions → Connectors and installs/selects a connector.
2. Admin completes existing secure setup/OAuth and names the company connection: Engineering
   GitHub, Production Sentry, or Maya — company mailbox. Verify the actual external identity.
3. Admin assigns staff/groups and Bots. Start with Codync's connector-level enablement;
   promise per-tool/resource restrictions only where host/provider enforcement supports them.
4. Staff sees assigned connections and which account will be used. Missing access offers
   Request access, not independent personal-account setup.
5. Host resolves a permitted connection from authenticated actor and Bot context, binds it to
   the run and checks permissions at execution. Model arguments cannot assert an account owner.
6. Results/traces use company visibility. Restricted departmental data must not enter a group
   whose members lack access. Company admin oversight is not an employee-controlled toggle.
7. Sends/publication/payments/destructive actions follow the separately selected action policy.

### Company mailbox example

Admin enables mail integration and assigns Maya's company-issued mailbox to Maya and Sales
Assistant, with company-defined oversight. Maya asks for a summary; the host selects that mailbox
and records requester, Bot, connection and audience. Leo needs a company grant to access it.

Admin configuration cannot bypass a provider's authorization requirements. The chosen connector
may need user OAuth consent, an organization integration installation or supported delegation.
If a staff member must complete consent, it finishes an admin-created company assignment and
must return the expected approved account. It does not allow arbitrary account addition.
Do not assume Google domain-wide delegation or mailbox impersonation is available through an
existing MCP connector. Verify the supported route before specifying provider implementation.

## Fit to Codync and code-quality constraints

- Preserve the registry/catalog/install/update machinery and existing connector IDs/sign-ins.
- Extend existing records with assignment metadata. Introduce separate connection identity only
  where multiple authenticated instances require it; do not duplicate the connector platform.
- Keep per-Bot enablement. Enterprise effective access also requires staff assignment;
  preserve existing personal-host defaults instead of changing every Codync installation.
- Enforce in host admission, MCP configuration/proxy and invocation paths. Renderer filtering
  is insufficient. Pin account identity at launch; rebuild affected harness sessions when
  account/grants change. Never silently switch to an admin or colleague account.
- If an agent launches a connector directly, prove claimed tool/resource limits at an actual
  mediated call boundary. Otherwise offer connector-level access plus provider scopes honestly.
- Reuse secure cards/vault/OAuth. Reject unauthorized setup through direct APIs, stale cards or
  old broad device grants. No generic credential getter and no raw credentials in chat.
- Keep existing settings/navigation/store. Do not restore Codync's removed legacy Apps shelf.
- Add protocol fields and migrations; capability-negotiate team mode and preserve old personal
  clients. Do not enroll an incompatible client as an unrestricted team caller.
- Restricted member work must not inherit broad local/admin execution authority. State actual
  host/process isolation limits; prompts alone cannot enforce them.

## Routines, delegation and revocation

Runs carry initiating staff/service identity, workspace, Bot, connection and company audience.
Delegation preserves authority. Third-party mail/webhook content cannot grant authority.
Company routines use authorized company connections and named maintainers. Departure revokes
staff access and pauses/reassigns dependent work without deleting company resources or silently
transferring person-bound OAuth credentials. Historical authorship remains intact.

Disable/disconnect immediately blocks local use and invalidates handles/sessions; attempt vendor
revocation and show Provider revocation pending separately during outages. Apply team-access
membership generations/leases. Authorize traces, replay, files, notifications and model context
consistently with company grants. Retention rules cover caches/messages/memory as well as tokens.

## Architectural order and verification

Review current schema/launch/proxy boundaries → minimal assignment contracts on team-access →
existing secure setup plus assigned-resource UI → execution/session invalidation → routines and
offboarding → migration rehearsal. This is design sequencing, not an approved coding task list.
Review any new module boundary before writing a standalone connected-apps module specification.

Classify imported connections explicitly as company resources; preserve IDs, vault references,
sign-ins and Bot toggles. No automatic grant to every staff member. Additive migration and rollback.

Required acceptance:
- Admin assigns company connection; staff uses it without secret disclosure.
- Staff cannot add personal accounts through UI/API or activate mismatched OAuth identities.
- Wrong staff/Bot/resource/account and stale grants/sessions are refused.
- Admin oversight follows company permissions; colleagues cannot retrieve unassigned content
  through traces, SSE replay, search, files, notifications, memory or delegated Bot sessions.
- Every promised restriction holds at the real connector/proxy/provider boundary.
- Existing discovery/install/OAuth/update, secure cards, native clients and personal mode pass
  regression checks. No second vault, scheduler or connector runtime.
- Real-provider consent is verified in an isolated profile before cutover; mocked tests alone
  cannot certify it. Use existing host/cloud/desktop tests through Portly per SPEC-team-access.md.

## Remaining technical decisions

Exact company role/resource visibility matrix, first provider/tool set, multiple-connection needs
and each provider's supported organization authorization path. Document the exact Codync source
extensions and why an existing abstraction is insufficient before introducing another one.
Reject duplicated policy, parallel state and renderer-only enforcement in architecture review.
