# Codync README preservation baseline

Reviewed 2026-10-29 against `kynjal26/Codync` main at
`2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0`. Remote main was checked again and still matched.

Source: [pinned README](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/README.md).

## Preservation rule

Keep all existing Codync capabilities while adding the selected Zyndi capabilities. The
continuation capability map lists changes, not the complete retained product. Any removal,
reduction, or incompatible replacement needs an explicit owner decision. This document records
README claims for regression planning; it does not certify that each works on every platform.
Features found elsewhere in Codync remain covered by the same rule.

## README capability checklist

- Bring-your-own signed-in coding agents/subscriptions; ACP registry support, native ACP
  preference, automatic installed-agent detection and fetching missing adapters.
- Multiple concurrent persistent Bots with names, avatars, instructions, per-Bot agents,
  project folders and permission policies; persistent chat, session resume and New session.
- Group chats, mentions, ordered multi-round replies, quoted replies and Bot-to-Bot delegation
  through the team MCP server; coding-agent-native subagents remain available.
- Focused chat, Full conversation traces with tool calls/diffs/plans, activity status,
  approval cards and completion/attention notifications.
- Per-Bot memory and existing memory behavior, supplemented by the separately approved
  team/privacy changes rather than replaced.
- Rust host and SQLite history; shared host state across clients, revision-based reconnect
  and event synchronization; local data storage and existing computer identity.
- Native iPhone client, widgets, Live Activity, approvals, push, remote screen and voice;
  Electron desktop on Mac/Windows/Linux, menu bar/tray, usage display and host supervision;
  terminal UI and SSH access; headless Linux/server support.
- Existing agent usage/limit reporting, provider widgets and Claude status-line integration.
- Multi-computer discovery/access requests, per-device pairing and computer-side approval,
  device grants/revocation; optional account sign-in and QR pairing without an account.
- End-to-end encrypted cloud relay, direct local/Tailscale connections, offline mailbox
  and private-relay option; encrypted private push and per-device tickets.
- Existing remote screen/control and Bot computer tool on their supported platforms;
  existing hands-free iPhone Bot voice calls.
- Host service install/uninstall, pairing/status, token rotation, cloud/connected-assistant
  controls, computer-network management and device/access administration.
- Scriptable `codync-host send`, reply targeting, file/stdin input, wait/JSON output,
  approval-aware timeout behavior and documented exit codes.
- Existing packaging, installers, upgrades/reinstallation preserving data, client/host
  minimum-version compatibility and update coordination with iPhone releases.

## Important limits and extension boundaries

- README explicitly says Windows remote screen, on-device speech and Claude status-line hook
  are unavailable. Do not call those existing working Windows features. The selected Mac and
  Windows member-desktop computer-use requirement is additional acceptance work.
- README has conflicting iOS minimum versions (badge 18+, install text 26+). Resolve against
  application deployment configuration before documenting supported OS versions.
- Existing subscription authentication stays through the signed-in coding agents. Adding
  direct API accounts must not repurpose subscription login credentials as API keys.
- Existing remote access remains required. README says the Codync name/logo/cloud relay are
  outside the code license and hosted or multi-tenant services must use their own relay and
  accounts. Preserve transport capabilities while resolving the team's deployment arrangement;
  do not assume permission to use upstream hosted infrastructure for the enterprise service.
- Team entitlements and the explicitly selected action-policy changes may change who can use
  a capability or authorize an action; they do not silently remove the capability itself.

## Verification obligation

Before cutover, map each item to source, affected clients and appropriate regression checks.
Keep existing Codync tests and extend them for touched behavior. Exercise real Mac/Windows
computer use and native iPhone flows on available devices. A README review or a green host
build alone does not establish end-to-end feature preservation.
