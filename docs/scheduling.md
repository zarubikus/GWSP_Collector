# Scheduling

Run the collector once per schedule. The state file prevents duplicate output, so the scheduler can run every 5 minutes or every hour.

## Linux systemd timer

`/etc/systemd/system/gwsp-collector.service`:

```ini
[Unit]
Description=Google Workspace to Wazuh collector

[Service]
Type=oneshot
ExecStart=/opt/gwsp-collector/venv/bin/gwsp-collector run --config /etc/gwsp-collector/config.yaml
User=gwspcollector
Group=gwspcollector
```

`/etc/systemd/system/gwsp-collector.timer`:

```ini
[Unit]
Description=Run Google Workspace collector

[Timer]
OnBootSec=2m
OnUnitActiveSec=5m
Persistent=true

[Install]
WantedBy=timers.target
```

Enable it:

```bash
systemctl daemon-reload
systemctl enable --now gwsp-collector.timer
```

## Linux cron

```cron
*/5 * * * * /opt/gwsp-collector/venv/bin/gwsp-collector run --config /etc/gwsp-collector/config.yaml
```

For hourly collection:

```cron
0 * * * * /opt/gwsp-collector/venv/bin/gwsp-collector run --config /etc/gwsp-collector/config.yaml
```

## Windows Task Scheduler

Create a task that runs:

```powershell
C:\gwsp-collector\.venv\Scripts\gwsp-collector.exe run --config C:\ProgramData\gwsp-collector\config.yaml
```

Use a 5-minute or 1-hour repeating trigger. Configure the task account so it can read the service account key and write the JSONL/state files.

