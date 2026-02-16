# watcher

Event router for Noctalia-driven sync actions.

Architecture decision is locked in `docs/ARCHITECTURE.md` (MVP = local IPC + `watchctl`, no OTel Collector yet).

## Files
- `bin/watchctl`: daemon/router
- `config/profiles.yaml`: routes + profiles (JSON content, valid YAML subset)
- `bin/noctalia-template-readd`: computes managed targets from Noctalia templates

## Core flow
`event -> route -> profile -> commands`

Default `theme_sync` profile runs:
1. `./bin/noctalia-template-readd --mode apply`
2. `chezmoi apply --force --include=files`

## Commands
Manual profile run:
```bash
./bin/watchctl run-profile theme_sync
```

Single event handling:
```bash
./bin/watchctl handle-event \
  --event '{"topic":"noctalia.theme.applied","source":"manual"}'
```

Daemon from stdin JSONL:
```bash
some_event_source | ./bin/watchctl daemon --stdin-jsonl
```

Daemon from a source command (must emit one JSON event per line):
```bash
./bin/watchctl daemon \
  --source-command '/path/to/noctalia-ipc-listener --jsonl'
```

Codex session refresh daemon (updates effective prompt after session JSONL changes):
```bash
./bin/watchctl daemon \
  --source-command "python ./bin/codex-event-source"
```

System package + security event daemon (pacman/journal high-signal topics):
```bash
./bin/watchctl daemon \
  --source-command "python ./bin/system-security-package-event-source"
```

Codex alert outputs:
- `~/.local/state/codex/watcher/watchctl-events.jsonl`
- `~/.local/state/codex/watcher/codex-alerts.jsonl`

Ops pane helpers:
- `bin/watchctl-control-dashboard`
- `bin/watchctl-tail-events`
- `bin/watchctl-tail-alerts`
- `bin/watchctl-tail-system-security`
- `bin/watchctl-systemd-status-pane`
- `bin/watchctl-tail-journal`
- `bin/watchctl-dome-oracle-status`
- `bin/watchctl-opsctrl-dashboard`

## Event shape
Minimal required field:
```json
{"topic": "noctalia.theme.applied"}
```

## Notes
- Debounce is configured in `config/profiles.yaml` (`defaults.debounce_seconds`).
- Logs are JSONL at `~/.local/state/codex/watcher/watchctl-events.jsonl` by default.
- `codex_on_fail` exists per-profile if you want automatic Codex diagnosis on command failures.
