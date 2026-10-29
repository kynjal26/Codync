# Confirmed intent: continue with Codync

Status: confirmed by the owner in this conversation; saved 2026-10-29.

## Outcome and reason

Continue with `kynjal26/Codync` as the team's enterprise workspace. Preserve Codync's
implementation quality, architecture and existing mechanisms. Select useful Zyndi capabilities
individually instead of bringing over its entire product or runtime.

**All existing Codync capabilities remain in scope.** The selected additions below are an
extension list, not a replacement feature list. An omitted Codync capability is retained by
default. Any removal or reduction requires a separate, explicit owner decision. See the
[README preservation baseline](../research/codync-readme-preservation-baseline.md).

## Desktop principle

**Codync's desktop implementation and UX are the base; Zyndi supplies the visual UI.**

Preserve Codync's navigation, workflows, controls and interaction behavior wherever the owner has
not explicitly selected an addition. Adapt Zyndi's colors, typography, spacing, icons and visual
treatment to those flows. This supersedes earlier discussion suggesting Zyndi's complete desktop
UX should replace Codync's. Retain Codync's mobile apps.

## Selected additions

- Company-managed connected apps: admins configure company connections, including assigned
  company-issued mailboxes, and grant staff/Bot access. Staff cannot independently add private
  accounts. Company policy governs work-data visibility and admin oversight. Preserve Codync's
  Extensions/connectors, OAuth, secure cards, vault and host execution. OpenDuet is a selective
  reference. This supersedes the earlier employee-managed/private-mailbox proposal.

- Enterprise team workspace: shared specialist Bots and shared conversations, roles and designated
  approvers. Colleagues use the same Bots with explicitly granted host folders and connected-app
  access; admin work retains Codync's normal host access.
- Team-aware knowledge: preserve Codync's memory implementation and automatic learning, extending
  ownership/access so ordinary company knowledge is shared and personal or sensitive knowledge
  can be restricted. Do not replace it with Zyndi's whole knowledge system by default.
- Provider accounts: multiple named accounts of the same provider, separate sign-ins, direct API
  providers and per-Bot account/model choice. Admins manage credentials and availability; members
  choose only approved accounts/models. Automatically fall back through an admin-configured
  order, visibly announce changes, and return to the preferred account when available. Extend
  Codync's existing fallback mechanism rather than replacing it wholesale.
- Voice: the voice speaks as the Bot, while actual work and reported results come from its agent.
  Bots may call the user or a colleague only when requested or through an explicitly configured
  routine; no independent decision to call. Call transcripts and reports are available to the
  requester, callee and workspace admins.
- Computer use: a central-host Bot can operate the requesting member's own computer through that
  member's desktop app. Mac and Windows are required at initial release. Admins set maximum
  access and members grant access to local apps/folders. Sharing a Bot alone grants no device
  access. Start with Codync's existing computer-use engine wherever practical.
- Creation workspace: richer interactive pages, HTML, dashboards, documents and memos; larger
  previews, document editing, targeted comments on elements or selected text, Bot-driven
  revisions, history and restore. Creators and designated editors may edit; admins manage access.
- Action policy: external messaging, publishing, payments and destructive actions require the
  responsible designated approver even under Full access. Explicit exceptions may authorize
  trusted Bots for specific jobs. Requesting work does not itself grant approval authority.
- Data continuity: preserve Bots, instructions, chats, memories, groups, routines, pages and
  shared-access settings. Archive historical records belonging to rejected features rather than
  deleting them.

## Explicit exclusions

Keep Codync's existing approach instead of porting these Zyndi additions:

- Exact-request binding for spoken approvals.
- Operation-receipt checks that hold retries of uncertain actions for a person.
- AI approval reviewer.
- Integrated media-production system and its separate asset/render pipeline.
- One-time Bot wake-up system.
- Central team handbook.
- In-app Bot evaluation suites.
- Resource/access inventory and probes.

## What this confirms

This confirms product intent and authorizes the source investigation and migration design. It
does not certify that selected controls are complete in either implementation. In particular,
the universal outbound/payment/destructive approval policy is new work, not a finished Zyndi
feature available to copy.

Preserve the reference checkout and running services. No implementation port, production data
conversion, service cutover or retirement is part of this documentation step. The old
`docs/plans/retire-codync.md` describes the opposite historical direction and is not the current
objective.
