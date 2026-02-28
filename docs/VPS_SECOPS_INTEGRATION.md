# VPS SecOps Integration

This integration wires `watchctl` to `vps_secops` health events and guardrail actions.

## Components
- watcher profile config: `config/profiles_vps_secops.yaml`
- daemon wrapper: `bin/vps-secops-watcher-daemon`
- event source (from vps_secops): `scripts/vps_watcher_event_source.py`

## Run
```bash
VPS_SECOPS_ROOT=$HOME/src/vps_secops \
  $HOME/src/watcher/bin/vps-secops-watcher-daemon
```

## Behavior
- `vps_secops.pipeline.coverage.low` -> enrichment sweep
- `vps_secops.pipeline.pack.stale` -> full pack + sweep + incident promotion stub
- `vps_secops.pipeline.report.missing` -> enrichment sweep

## Notes
- This profile is separate from default watcher profiles to avoid impacting existing routes.
- Commands are executed by watchctl with `WATCH_EVENT_JSON` exported.
