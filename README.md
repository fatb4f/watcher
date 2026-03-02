# watcher

Event router focused on Codex session/config signals.

Scope note:
- Session-memory refresh (`codex-refresh-context`) is out of scope here and belongs to `spawnd`.

Naming update:
- `meshctl` is the preferred operator entrypoint.
- `watchctl` remains as a compatibility alias.

Architecture decision is locked in `docs/ARCHITECTURE.md` (MVP = local IPC + `watchctl`, no OTel Collector yet).

## Files
- `bin/meshctl`: preferred daemon/router entrypoint
- `bin/watchctl`: daemon/router
- `config/profiles.yaml`: codex routes + profiles (JSON content, valid YAML subset)
- codex tooling moved to `spawn/bin/codex-*`

## Core flow
`event -> route -> profile -> commands`

## Commands
Manual profile run:
```bash
./bin/meshctl run-profile codex_config_validate
```

Single event handling:
```bash
./bin/meshctl handle-event \
  --event '{"topic":"noctalia.theme.applied","source":"manual"}'
```

Daemon from stdin JSONL:
```bash
some_event_source | ./bin/meshctl daemon --stdin-jsonl
```

Daemon from a source command (must emit one JSON event per line):
```bash
./bin/meshctl daemon \
  --source-command '/path/to/noctalia-ipc-listener --jsonl'
```

Codex session refresh daemon (updates effective prompt after session JSONL changes):
```bash
./bin/meshctl daemon \
  --source-command "python ./bin/codex-event-source"
```

Codex outputs:
- `~/.local/state/codex/watcher/watchctl-events.jsonl`
- `~/.local/state/codex/watcher/codex-alerts.jsonl`

## Event shape
Minimal required field:
```json
{"topic": "noctalia.theme.applied"}
```

## Notes
- Debounce is configured in `config/profiles.yaml` (`defaults.debounce_seconds`).
- Logs are JSONL at `~/.local/state/codex/watcher/watchctl-events.jsonl` by default.
- `codex_on_fail` exists per-profile if you want automatic Codex diagnosis on command failures.
- Optional runtime disable:
  - `MESH_DISABLED_PROFILES=codex_context_refresh,codex_stall_alert`
