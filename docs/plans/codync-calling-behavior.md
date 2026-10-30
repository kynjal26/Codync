# Codync calling behavior

Status: core rule confirmed by the owner on 2026-10-30; the open decisions below remain proposed.
This page is the single source of truth for when a Bot may call. It supersedes the narrower
"requested/routine-only" wording in the continuation intent, capability map and source review.

## Decision

A Bot may call only inside a turn. A turn can start because:

- a person writes to the Bot;
- a routine fires (schedule or real-time event);
- another Bot writes;
- a task opens or a helper finishes;
- the Bot's own background work finishes.

Inside that turn, the Bot decides whether to call. There is no separate background caller outside
a turn, and no automatic urgency rule. The Bot can call on its own when its own work starts a turn.

The requester is the person the turn is for. The Bot is the actor. The person called is the
callee. The requester, callee and workspace admins can see the transcript and report; other
members see only that a call happened.

The host must reject a call without a valid turn context. This rule governs Bot-initiated calls.
Codync's existing user-started live call is a separate path; if retained, the host must mint a
`person_request` turn when the user taps call, or the path must be explicitly exempt.

## What this means

- The host does not require a fresh "call me" message. The turn is the authorization.
- The Bot may start its own turn and decide to call. This is the "the Bot called me on its own"
  case.
- A call report may start a turn; the recommended rule is that it must not initiate another call
  (open decision 3). That prevents loops.
- A scheduled call carries the turn context from creation. It cannot be created from a private
  turn.
- A shared Bot's self-turn should be bound to a person; if no person is bound, the owner is the
  fallback (open decision 2).

## Current Codync status (2026-10-30 audit)

Codync today has host-owned live voice, on-device/cloud speech, realtime providers, call tools,
transcript lines, reconnect and call views. It does not have:

- a Bot-facing `call_user` tool;
- outbound invitations, ring/answer/decline/cancel or one-winner answering;
- call scheduling or `end_user_call`;
- per-person active-call or spacing limits;
- requester/callee/audience provenance;
- native incoming-call ringing (iOS PushKit/CallKit, Android FCM/Telecom);
- a private report path; the current end path copies the transcript into the shared chat.

The rule on this page is the target. The capability map and source review record the gap. Do not
describe Codync as already having closed-app ringing.

## Trigger and provenance

| Trigger | Turn started by | Requester | Example |
| --- | --- | --- | --- |
| `person_request` | a person's message | that person | Kofi writes "Call Caroline." |
| `routine` | a routine | the routine owner | The 8:30 routine calls the owner. |
| `bot_message` | another Bot | the person the other Bot acts for | Bot A tells Bot B that Kofi needs a call. |
| `task` | a task | the person the task serves | A task for Kofi finishes and the Bot calls Kofi. |
| `helper` | a helper result | the person the helper serves | A helper returns a result and the Bot calls the owner. |
| `self_turn` | the Bot's own work | the owner or the person the Bot acts for | The Bot finishes a background task and calls the owner. |

Requester is provenance and reporting. It is not the permission gate.

## Authorization and limits

The generic tool policy (`action-policy`) decides whether `call_user` is allowed:

- default: always allow;
- Admin can require approval, require review, or block;
- hidden Bots cannot call.

The call domain (`team-voice`) enforces:

- the target is the owner or a person shared with the Bot (`team-access`);
- a voice-provider account is available (`provider-accounts`);
- the callee has at least one enabled device that supports the provider;
- one active call per person;
- one-minute gap between call starts, Computer-wide in the Zyndi reference implementation and
  waived when the callee wrote to that Bot in the last five minutes (open decision 1);
- no daily cap;
- a scheduled call cannot be requested from a private turn;
- a late scheduled call is dropped after 10 minutes.

The host enforces these rules. Clients and prompts do not.

## Ring, answer and failure

- Ring for 45 seconds.
- No audio before answer.
- One device answers; the others stop ringing.
- If nobody answers, the callee declines, or the call fails: one `call-unanswered` entry in the
  chat, and the Bot is told to write instead.
- If a person asked for the call, the private report goes to the requester and admins.

## Scheduling

- `inMinutes` from 1 to 1440.
- `end_user_call` cancels or ends.
- A scheduled call carries the turn context at creation.
- A scheduled call cannot be requested from a private turn.
- A late scheduled call is dropped after 10 minutes.

## Data and audience

- Store the call's provenance: turn, requester, actor, trigger, callee, reason.
- Store the report/transcript with its audience facts; `team-access` resolves current admins.
- Do not copy the transcript into the shared chat. The shared chat sees only that a call happened.
- Do not store provider credentials in call records; reference the provider account.
- Add migrations at the end; never rewrite an applied migration.

## Target platform behavior

- Phones ring through push where the OS delivers: iOS PushKit/CallKit, Android FCM/Telecom.
- Desktop rings through the existing host event stream only while the app is running.
- Codync today has no native incoming-call path. iOS PushKit/CallKit, Android FCM/Telecom and
  desktop incoming ring are part of the build. The Zyndi reference has a push path, but
  background/locked-screen delivery is not verified. Do not claim closed-app ringing is
  implemented in Codync.
- `desktop-device-bridge` is for computer use, not for call ringing.

## Architecture placement

- `team-voice` owns the call lifecycle.
- `team-voice` also adds the missing turn sources for task, helper, self-turn and call report;
  Codync's current `TurnSource` has only User, Routine, Group, Bot and Host.
- `team-access` owns person/role/resource facts and report visibility.
- `provider-accounts` owns the voice account reference.
- `action-policy` owns the generic approval/block decision.
- `data-migration` maps existing calls, invitations, devices and transcripts with audience.
- No second scheduler, host, call system or client-side policy.

## Open decisions

1. Keep the one-minute gap Computer-wide (recommended) or make it per Bot?
2. Allow `self_turn` only when the turn is person-bound (recommended) or always?
3. Keep call-report turns unable to start a new call (recommended) or allow them?
4. Closed-app ringing requires native implementation plus FCM/APNs credentials and physical
   devices.

## Implementation references

- Zyndi `turn_author`: `rust/crates/zyndi-store/src/people.rs:575`.
- Zyndi `call_user`: `rust/crates/zyndi-host/src/tools/calls.rs:168`.
- Zyndi ring/spacing/scheduling: `rust/crates/zyndi-host/src/calls.rs:19`, `:460`, `:505-512`,
  `:594`.
- ADRs 0005, 0023, 0026, 0028, 0033.
- `docs/calling/architecture.md`, `docs/calling/deployment.md:127`,
  `docs/calling/verification.md:89`.
