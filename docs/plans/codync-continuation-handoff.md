# Codync enterprise continuation: handoff

Transferred at the owner's request on 2026-10-29 from
[Zyndi commit f78727a4](https://github.com/kynjal26/zyndi/commit/f78727a43e42d7a21bc00271a1fdbe30cb2cd70b).
Codync destination baseline: `2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0`.
This branch transfers design/research only. No source code, runtime data or credentials were ported.

## Start here

1. [Confirmed intent](../intent/codync-continuation.md).
2. [Existing Codync feature preservation](../research/codync-readme-preservation-baseline.md).
3. [Source investigation](../research/codync-continuation-source-review.md) and
   [55 pinned evidence records](../research/codync-continuation-evidence.json).
4. [Proposed capability map](codync-continuation-capability-map.md).
5. [Proposed team-access specification](SPEC-team-access.md).
6. [Current company-managed connections architecture](codync-connected-apps-architecture.md).
7. [OpenDuet connected-apps investigation](../research/codync-openduet-connected-apps.md).

## Decision precedence

Confirmed intent and later explicit owner corrections take priority over earlier research
recommendations. Company connections are admin-managed, assigned to staff/Bots, and governed by
company visibility policy; staff cannot independently add private accounts. The earlier employee-
managed/private-mailbox proposal is superseded. OpenDuet is a selective reference only.

Codync's architecture, UX, host ownership, connectors, vault, ACP lifecycle and existing features
remain the base. Zyndi supplies selected capabilities and desktop visual UI. Team sharing is
required at initial cutover. The destination is current `kynjal26/Codync` main, not a wholesale
Zyndi runtime transplant.

Clerk with Google-primary onboarding is the selected design direction. Detailed module
boundaries, admission leases, outage policy, role matrices and technical specifications retain
their documented proposed status. Copying them does not mark every proposal approved.

Research files describe pinned historical source. Their source-relative Zyndi paths and local
snapshot locations are provenance, not instructions to run Zyndi code in Codync. The evidence
manifest deliberately retains original repository IDs, hashes and commit anchors.
Zyndi's historical retire-Codync plan mentioned in the intent stays in Zyndi and is superseded.

## Workspace and development

Writable working checkout: `/home/dev/workspace/repos/Codync-continuation`.
Transfer branch: `docs/enterprise-continuation`, based on this fork's main.
Preserved reference: `/home/dev/workspace/repos/Codync`; preserve its checkout and services.
Zyndi and OpenDuet remain read-only implementation/design references for future work.

Follow this repository's AGENTS.md and existing architecture/clean-code guidance. Implementation
features use their own task branches/worktrees and isolated development profiles per Codync's
worktree sandbox conventions. No development profile or service was started by this transfer.
Never reuse a running reference/production profile or migrate real data during routine testing.

Use Portly for development servers and bounded builds/tests/code generation in this environment:
`portly status`, then a managed server or `portly temp` followed by `portly wait`. Preserve source
compatibility, add migrations at the end and avoid duplicate host/client business logic.

Next step is specification review and scoped implementation planning in this Codync checkout,
with actual connector/provider authorization verified before claiming support. This handoff
does not authorize deployment, live invitations, runtime migration or retirement of Zyndi.

## Transfer validation

Verify all eight transferred files exist, the evidence JSON parses, local Markdown links resolve,
and the staged diff contains documentation only. Compare copied bytes to the source except for
the explicit checkout-status correction in SPEC-team-access.md. No app E2E applies because no
application behavior changed. Reference checkout HEAD/status must remain unchanged.
