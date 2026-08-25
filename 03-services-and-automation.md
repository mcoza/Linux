# 3. Services and Automation

This section documents how I work with system-managed services, scheduled jobs, service dependencies, and introductory cgroup resource controls.

## What I know

I understand these as different parts of the system:

```text
systemd   → service manager, normally PID 1
systemctl → client used to inspect/control systemd
.service  → unit definition containing instructions
process   → running instance of the launched program/script
journal   → recorded service/system history
cron      → scheduler for time-based commands
.timer    → systemd unit that schedules another unit
cgroup    → kernel mechanism for grouping/controlling processes
```

The useful relationship is:

```text
systemd
  ↓ reads
.service unit
  ↓ ExecStart=
program or script
  ↓ becomes
process
```

## What I can do with systemd

### Inspect service state without confusing it with the process itself

```bash
systemctl status <unit>
systemctl cat <unit>
journalctl -u <unit>
```

They answer different questions:

```text
status       → what is the unit doing now, and what happened recently?
cat          → what instructions did systemd load?
journalctl   → what did the unit report over time?
```

When reading `systemctl status`, I focus first on:

```text
Loaded      → was the unit found, and where?
Active      → running, exited, inactive, or failed?
Process/PID → what actually ran?
recent logs → what did that run report?
```

Common states:

```text
active (running) → a long-running process is alive
active (exited)  → the job completed successfully; normal for some oneshot units
inactive (dead)  → not currently running
failed           → the attempted run failed
```

I also separate runtime state from startup policy:

```text
start / stop       → affect the current runtime state
enable / disable   → affect automatic startup
```

A unit can therefore be enabled but stopped, or disabled but currently running.

### Follow a unit into the real work

If `ExecStart=` points to a script, systemd is only the launcher. The script becomes the next object to inspect.

```text
service failed
→ systemctl status
→ systemctl cat
→ ExecStart points to script/program
→ run/read that component
→ follow its own error
```

That prevents me from treating every service failure as a systemd configuration problem.

### Reapply configuration correctly

If I edit the unit file itself:

```bash
sudo systemctl daemon-reload
sudo systemctl restart <unit>
```

`daemon-reload` makes systemd reread the changed unit definition. Restarting launches a new process from that definition.

If I only edit a script referenced by `ExecStart=`, I do not need `daemon-reload` for the script change, but the service still has to run again before the new script contents are used.

## What I can do with scheduled work

### Diagnose a cron job that is not producing its result

The layers are:

```text
cron daemon
→ crontab schedule
→ command/script path
→ script behavior
→ expected output
```

Useful inspection commands include:

```bash
crontab -l
sudo crontab -l
```

A concrete example from lab work had a job scheduled every five minutes but pointing to the wrong script:

```cron
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```

The intended script was:

```text
/opt/backup/backup.sh
```

After the schedule was corrected, manually running the real script exposed a second problem: a stale lock file made the script believe another backup was already active.

That produced a reusable troubleshooting chain:

```text
expected output missing
→ inspect schedule
→ verify command/path
→ run the task manually
→ fix task-level error
→ verify the output
```

The scheduler and the task can fail independently.

### Diagnose work that succeeds manually but not automatically

A systemd timer gives another scheduling relationship:

```text
timer   → when
service → what systemd launches
script  → task logic
```

Useful commands:

```bash
systemctl list-unit-files --type=timer
systemctl cat <timer>
systemctl enable --now <timer>
```

If a script succeeds when I run it manually but does not execute on schedule, that is evidence to inspect the scheduler/launcher rather than keep changing the script blindly.

## What I can do with service dependencies

I understand `Requires=` and `After=` as explicit systemd relationships rather than vague statements that two processes are “dependent.”

A typical path is:

```text
API service fails with dependency error
→ inspect unit
→ identify required synchronization/service unit
→ fix and verify the lower unit
→ retry the API service
```

Fixing the required unit does not automatically replay a dependent service's earlier failed start attempt.

## What I can do with service-launched scripts

A service can report a successful exit even when the script did the wrong thing.

For example, a cleanup script contained:

```bash
find . -maxdepth 1 -name '*.log' -type f -mtime -7 -print -delete
```

Two details made the logic wrong for a requirement to remove old logs from a specific log directory:

```text
.         → uses the service's current WorkingDirectory
-mtime -7 → selects files newer than 7 days
```

The corrected form used the intended directory and older-than comparison:

```bash
find "$LOG_DIR" -maxdepth 1 -name '*.log' -type f -mtime +"$DAYS" -print -delete
```

That reinforces:

```text
unit exited successfully
≠ intended operational result is correct
```

## What I know about cgroup v2 controls

I have introductory hands-on experience inspecting and correcting a cgroup memory limit.

A setup script used:

```bash
CGROUP=/sys/fs/cgroup/sad-batch
mkdir -p "$CGROUP"
echo 134217728 > "$CGROUP/memory.high"
```

The required behavior was a hard 128 MiB ceiling, but the script wrote to the wrong control.

```text
memory.high → pressure/throttling threshold
memory.max  → hard memory ceiling
```

Inspecting:

```bash
cat /sys/fs/cgroup/sad-batch/memory.max
```

returned:

```text
max
```

which meant there was no numeric hard ceiling set.

The troubleshooting path was:

```text
required state: memory.max = 134217728
→ inspect live memory.max
→ value is max
→ inspect setup script
→ script writes to memory.high
→ change target to memory.max
→ rerun the setup service
→ inspect memory.max again
→ verify 134217728
```

This also connected familiar shell redirection to a new mechanism: files under `/sys/fs/cgroup` are kernel-backed control interfaces, not ordinary configuration files stored on disk.

## How symptoms narrow my first inspection

```text
"service failed to start"             → systemctl status
"what does this service launch?"      → systemctl cat / ExecStart
"what happened earlier?"              → journalctl -u
"works manually, not automatically"   → scheduler/launcher
"runs every N minutes"                → cron or systemd timer
"dependency failed"                   → inspect Requires=/After= relationships
"configured limit is not taking effect" → inspect live cgroup control + setup logic
```

## What this lets me do

I can trace a managed or scheduled workload through its layers instead of stopping at the launcher:

```text
manager/scheduler
→ unit or schedule
→ program/script
→ process/task behavior
→ dependency/resource
→ expected result
```
