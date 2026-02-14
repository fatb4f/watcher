# Watcher Architecture (MVP)

Date: 2026-02-14
Status: Locked

## Decision

Use a lightweight local event pipeline:

- Event sources: Hyprland IPC + Quickshell actions (JSONL adapters)
- Router: `watchctl`
- Actions: profile commands from `profiles.yaml`
- Evidence log: local JSONL (`~/.local/state/codex/watcher/watchctl-events.jsonl`)

Do **not** run OpenTelemetry Collector for MVP.

## Why

- Current scope is local IPC monitoring + deterministic local actions.
- Collector adds operational overhead without immediate value.
- JSONL logs are enough for replay/debug at this stage.

## Promotion Criteria (when to add OTel Collector)

Add Collector only when one or more are true:

1. Need cross-process/cross-host trace correlation.
2. Need backend dashboards/alerting from unified telemetry.
3. Need long-term indexed retention beyond local JSONL.

## Implementation Contract

1. Adapters emit normalized JSON events (`topic`, `ts`, `source`, `payload`, optional ids).
2. `watchctl` routes events to profiles by topic.
3. Profiles run deterministic shell commands.
4. Failures are logged; optional Codex-on-fail remains opt-in per profile.
