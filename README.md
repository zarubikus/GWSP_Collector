# GWSP Collector

GWSP Collector pulls Google Workspace Admin SDK Reports activity and Alert Center alerts, normalizes them to single-line JSON events, and writes them to a JSONL file for a local Wazuh agent to forward to a remote Wazuh manager.

The collector is incremental. It keeps a JSON state file with per-source checkpoints and only writes events that were not already committed.

## Features

- Google Workspace service account authentication with domain-wide delegation.
- Admin SDK Reports collection across configured applications.
- Alert Center collection.
- Pagination support for Reports and Alert Center APIs.
- Timestamp overlap and same-timestamp deduplication for delayed events.
- Gmail Reports window splitting to respect the 30-day API window.
- Cross-platform paths and scheduling examples.
- Wazuh `localfile` examples and starter rules.

## Install

Create a virtual environment and install the package:

```powershell
python -m venv .venv
.\.venv\Scripts\python -m pip install -e .
```

On Linux:

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -e .
```

## Google Workspace Setup

Follow [docs/google-workspace-setup.md](docs/google-workspace-setup.md).

Required OAuth scopes:

```text
https://www.googleapis.com/auth/admin.reports.audit.readonly,https://www.googleapis.com/auth/apps.alerts
```

## Configure

Copy [config.example.yaml](config.example.yaml) to a secure location and edit:

- `google.service_account_file` or `google.service_account_file_env`
- `google.delegated_admin` or `google.delegated_admin_env`
- `collector.output_dir`
- `collector.output_file_template`
- `collector.state_path`
- `reports.applications`

Validate:

```bash
gwsp-collector validate-config --config /etc/gwsp-collector/config.yaml
```

Run once:

```bash
gwsp-collector run --config /etc/gwsp-collector/config.yaml
```

Show the planned Google API requests without contacting Google Workspace:

```bash
gwsp-collector run --config /etc/gwsp-collector/config.yaml --dry-run
```

The Windows-compatible single-dash form also works:

```bat
gwsp-collector run --config C:\ProgramData\gwsp-collector\config.yaml -dry-run
```

On Windows, edit [scripts/run-windows.bat](scripts/run-windows.bat) with your config, service account key, and delegated admin values, then run it from Task Scheduler or a terminal.

On Ubuntu/Linux, use [scripts/run-ubuntu.sh](scripts/run-ubuntu.sh). It defaults to `/opt/gwsp-collector` and expects config/credentials under `/opt/gwsp-collector/local/`.

Daily log files older than `collector.compress_after_days` are compressed and moved under `logs/YYYY/YYYY-MM/`. If `collector.output_dir` is already named `logs`, that directory is used; otherwise a `logs` subfolder is created. Existing archive names are never overwritten; a numeric suffix is added when needed. Each `.gz` archive gets a neighboring `.gz.sha256` file with its SHA256 digest.

Print state:

```bash
gwsp-collector state --config /etc/gwsp-collector/config.yaml
```

For a long-running process instead of cron/systemd:

```bash
gwsp-collector run --config /etc/gwsp-collector/config.yaml --loop
```

Backfill and archive historical daily data without touching normal incremental state:

```bash
gwsp-collector archive --dry-run
gwsp-collector archive
gwsp-collector archive --start-time 2026-01-01 --end-lag-days 5 --dry-run
```

Archive mode defaults to `/opt/gwsp-collector/local/config.yaml` when `--config` is not provided. If `--start-time` is not provided, it starts at the current UTC date minus 180 days, then collects one day at a time through today minus `--end-lag-days` days. If a compressed archive already exists under `collector.output_dir/logs/YYYY/YYYY-MM/`, that day is skipped.

## Wazuh

Configure the Wazuh agent on the collector host to tail the JSONL file:

- Linux example: [wazuh/ossec-linux.xml](wazuh/ossec-linux.xml)
- Windows example: [wazuh/ossec-windows.xml](wazuh/ossec-windows.xml)

Copy starter rules to the Wazuh manager:

- [wazuh/rules/google_workspace_rules.xml](wazuh/rules/google_workspace_rules.xml)

Restart the Wazuh agent after changing `ossec.conf`. Restart the Wazuh manager after adding rules.

## Scheduling

See [docs/scheduling.md](docs/scheduling.md) for Linux systemd timer, cron, and Windows Task Scheduler examples.

The default `collector.poll_interval` is `300s`. For scheduled runs, the scheduler controls cadence; the state file prevents duplicate output.

## State

The state file tracks:

- `reports.<application>.last_committed_time`
- `reports.<application>.seen_ids`
- `alerts.last_committed_create_time`
- `alerts.seen_ids`
- `last_success`
- `last_error`

State is committed only after JSONL writes complete. If a run fails, the next run retries from the previous committed checkpoint with the configured overlap.

## Tests

The included tests use only stdlib and mocked clients:

```powershell
$env:PYTHONPATH = "src"
python -m unittest discover -s tests
```

On Linux:

```bash
PYTHONPATH=src python -m unittest discover -s tests
```
