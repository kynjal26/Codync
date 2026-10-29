# Connected apps: Codync and OpenDuet, first investigation

2026-10-29. Research notes, not confirmed feature selections or implementation approval.
Codync remains the base; preserve its existing Extensions/connectors/skills/agent setup.

## Pinned sources and method

- Codync `2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0`, existing bare snapshot.
- OpenDuet `677e0fcd148d80b99dd788c6d6f9c1c9f40a72db`, read-only source clone in
  `/tmp/openduet-reference-20261029`. README plus connector docs/source inspected. No dependency
  install, scripts, services, provider calls or credentials accessed.
- Supplied image is a product overview: tools via MCP, agent endpoints via AG-UI and delivery
  channels are different interfaces. A logo is not proof of a production-ready adapter.

## What Codync already has

[Extensions documentation](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/marketplace.md)
describes Agents, Connectors and Skills per host; official MCP Registry discovery, custom command/
URL/JSON import, local and remote transports, sign-in, pinned package references and update logic.
Composio Connect is installed as an MCP connector using OAuth. The former separate Apps shelf and
legacy Platform-key setup were removed; existing connections remain supported. Therefore do not
restore an obsolete Apps page accidentally when preserving the user's Connect Apps experience.

Connectors are currently enabled for every Bot by default, with per-Bot opt-out. That is a
personal-host default to preserve in personal mode; the selected enterprise model needs explicit
effective grants rather than automatically sharing a newly connected personal account.

[Credential documentation](https://github.com/kynjal26/Codync/blob/2099c8a4c3ffa5d9ca31c0f18d05a63b2a03fef0/docs/features/connector-credentials.md)
covers host vault, OAuth, secret/login request cards and 1Password references. Keep them.

## OpenDuet contributions worth interviewing about

1. Personal vs Shared account mode per app; calls identify whose account is used. A shared
   account belongs to the deployment rather than pretending to be a member's personal login.
   Source: `openbot/server/src/plugins/account-mode.ts`, `shared-accounts.ts`,
   `openbot/docs/plugins/composio.md`.
2. Admin enables an app and grants individual tools to Bots; each person separately consents
   to their personal account. Installation, connection and tool permission are distinct.
   Source: `openbot/docs/plugins/google-workspace.md`, `plugins/access.ts`,
   `plugins/route-groups/grants.ts`.
3. A member can request missing app access; an admin resolves the request from the app's page.
   Source: `plugins/app-access.ts`, `app-access-tool.ts`, `route-groups/app-access.ts`.
4. Direct Google Workspace integration, not only a generic broker: Gmail reads/replies and
   calendar reads, per-person OAuth. It is a specific tool set, not the entire Gmail API.
   Source: `plugins/google-workspace-rest.ts`, `openbot/docs/plugins/google-workspace.md`.
5. Composio and Arcade broker adapters, alongside reviewed/direct connectors such as Drive
   and Notion. Different setup and credential custody from Codync's Composio Connect MCP route.
   Source: `openbot/docs/plugins/composio.md`, `arcade.md`, `google-drive.md`, `notion.md`.
6. Readiness checklist, reconnect guidance and visible tool-call audit outcomes. Potential UX
   additions; do not replace Codync's existing install/readiness checks.
   Source: README capabilities; `openbot/docs/plugins/google-workspace.md` troubleshooting.

Pinned source root for all relative paths above:
https://github.com/CopilotKit/OpenDuet/tree/677e0fcd148d80b99dd788c6d6f9c1c9f40a72db

## Separate decisions in the image

- MCP connects tools/apps. Codync already uses MCP.
- AG-UI connects external agent runtimes. Codync's ACP coding-agent support is a different
  strength; adopting an AG-UI adapter is separate work, not needed just to connect GitHub.
- Slack/Teams are additional conversation delivery channels. OpenDuet delegates integration to
  OpenBot/OpenTag; configuration requires channel identity linking and routing. SMS/phone are
  separate delivery features, not app credentials. Do not select all diagram branches implicitly.
  Source: README; `openbot/docs/configuration.md` Slack/Teams, identity and delivery sections.

## Constraints on adaptation

OpenDuet README labels early development and requires CopilotKit Intelligence for its threads/
memory. Keep Codync's Rust host, memory, ACP lifecycle and encrypted transport rather than adding
that runtime dependency. No full source transplant. Preserve MIT attribution for adapted code.
README review and inspected tests are not fresh real-provider certification.

AI auto-review and outbound sensitive-content checks are not automatically selected: they must
not override the owner's existing exclusion of AI approval review or smuggle in rejected Zyndi
safeguards. Tool permissions can be adapted without adopting that whole policy stack.

## Interview queue (one question at a time)

First resolve personal accounts vs company-shared accounts, then tool-level permissions, missing-
access request workflow, priority apps, and whether external messaging channels/AG-UI belong in
scope. No choices in this queue are confirmed yet. Existing provider/model account requirements
remain separate from connected-service accounts like GitHub, Sentry and Gmail.
