# 3. Services and Automation

This stage added systemd, cron, timers, service-to-script relationships, and introductory cgroup work. The main lesson was that a launcher reporting success does not prove the intended task succeeded.

## systemd: manager, unit, process

At first, `systemd`, `systemctl`, and the process being managed blurred together. The distinction that made later exercises easier was:

```text
systemd   → service manager, normally PID 1
systemctl → command used to inspect/control systemd
process   → program launched by the unit
```

The commands I used most were:

```bash
systemctl status <unit>
systemctl cat <unit>
journalctl -u <unit>
```

They answer different questions:

- `status` — what is happening now?
- `cat` — what is systemd configured to launch?
- `journalctl -u` — what happened over time?

I also learned to read `systemctl status` by focusing first on `Loaded`, `Active`, the main process/command, and the recent log lines. `start/stop` affect runtime state, while `enable/disable` control automatic startup.

When a unit file changes, editing the file is only the first step:

```bash
sudo systemctl daemon-reload
sudo systemctl restart <unit>
```

`daemon-reload` makes systemd reread the unit definition; the restart creates a new process using it.

## Cron: fix the schedule, then test the job

A backup job was scheduled every five minutes but pointed to a nonexistent script:

```cron
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```

The intended script was `/opt/backup/backup.sh`.

The useful sequence was:

```text
backup missing
→ inspect crontab
→ verify the scheduled path
→ correct it
→ run the real script manually
→ reproduce a stale-lock error
→ clear the lock
→ verify the backup
```

This was the first time it became obvious that a scheduler problem and a script problem can exist at the same time.

## Timer: when versus what

Another exercise used a systemd timer to run a health script every ten seconds. The script first failed because an iptables OUTPUT rule blocked `curl http://localhost`. After that was fixed, the script worked manually but still did not run on schedule because the timer was disabled.

Useful commands included:

```bash
systemctl list-unit-files --type=timer
systemctl cat health.timer
systemctl enable --now health.timer
```

The mental model became:

```text
timer   → when
service → what systemd launches
script  → task logic
```

## A service can successfully run the wrong logic

A oneshot cleanup service launched a script with `WorkingDirectory=/root`. The script used:

```bash
find . -maxdepth 1 -name '*.log' -type f -mtime -7 -print -delete
```

Two details mattered:

- `.` meant `/root`, not the intended log directory.
- `-mtime -7` selected newer files, while the requirement was to remove older files.

The unit could therefore report success even though the operational result was wrong.

## Following a real dependency

A later API exercise made systemd dependencies concrete. A synchronization service had to populate a catalog before the API could start. Its unit used `Requires=` and `After=`.

Once the synchronization service was fixed, the API still had to be retried because repairing the lower dependency did not retroactively repair the earlier failed start.

```text
failed service
→ inspect unit
→ identify required lower service
→ fix and verify lower service
→ retry dependent service
```

## cgroup v2: configured resource limit

A batch workload was supposed to have a 128 MiB hard memory limit. Its setup script wrote the value to:

```text
memory.high
```

but the required hard ceiling was:

```text
memory.max
```

The new concepts were:

```text
memory.high → pressure/throttling threshold
memory.max  → hard limit
```

The literal value `max` in `memory.max` meant no numeric hard ceiling was set. Correcting the setup script, rerunning the setup service, and verifying the control file fixed the configuration before the batch job was started.

## What changed in my troubleshooting approach

These topics initially felt separate. Repetition made the common path clearer:

```text
who launches it?
→ what is configured to run?
→ did it run?
→ did it produce the intended result?
→ what dependency or resource sits underneath it?
→ verify the result
```

That pattern is now more useful to me than memorizing one command sequence for cron, another for systemd, and another for timers.
