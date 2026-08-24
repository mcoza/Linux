# 3. Services and Automation

This stage added systemd, cron, timers, service-to-script relationships, and introductory cgroup work. The biggest lesson was that the status of the launcher and the result of the work it launches are not the same thing.

## Learning to read systemd in context

The first useful distinction was between the manager, the command used to talk to it, and the process it launches:

```text
systemd    → long-running service manager, normally PID 1
systemctl  → command used to inspect/control systemd units
process    → the actual program systemd launched
```

The core commands became:

```bash
systemctl status <unit>
systemctl cat <unit>
journalctl -u <unit>
```

I use them for different questions:

- `systemctl status` — what state is the unit in now and which process is attached to it?
- `systemctl cat` — what is systemd configured to launch?
- `journalctl -u` — what happened over time?

Reading `systemctl status` also became less opaque once I learned to focus first on `Loaded`, `Active`, the main process/command, and the recent log lines instead of treating every field as equally important.

A useful distinction from this work was:

```text
start / stop      → runtime state now
enable / disable  → whether the unit starts automatically
```

`active (exited)` can also be normal for a successful oneshot unit; it does not necessarily mean a service crashed.

## Changing a service versus changing a running process

When I changed a `.service` file, I learned that editing the file does not automatically replace the process that is already running.

```bash
sudo systemctl daemon-reload
sudo systemctl restart <unit>
```

The relationship is:

```text
edit unit file
    ↓
daemon-reload makes systemd reread the definition
    ↓
restart launches a new process using the changed definition
```

If only a script referenced by `ExecStart=` changes, the unit definition itself has not changed, so the important step is usually rerunning or restarting the service rather than assuming `daemon-reload` is always required.

## Cron: scheduler state versus job state

A backup exercise introduced cron as a separate scheduler. On the system I was using, systemd managed the `cron.service`, while cron itself read crontab entries and launched the scheduled commands.

```bash
systemctl status cron
crontab -l
sudo crontab -l
```

A root crontab contained a scheduled command pointing to a nonexistent script:

```cron
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```

The intended script was `/opt/backup/backup.sh`.

The troubleshooting path became:

```text
backup missing
→ inspect the schedule
→ verify the scheduled path exists
→ correct the path
→ run the intended script manually
→ reproduce a stale-lock error
→ clear the stale lock
→ verify the backup succeeds
```

That exercise was important because fixing the scheduler exposed a second problem in the script path itself. A scheduled job can therefore fail at more than one layer.

## systemd timer plus a separate network problem

Another exercise involved a health script that used `curl http://localhost`. The script initially hung because an iptables OUTPUT rule blocked local traffic to `127.0.0.1:80`.

After that network problem was fixed, the script worked manually but still did not run every ten seconds. The second issue was a disabled systemd timer.

Useful commands included:

```bash
systemctl list-unit-files --type=timer
systemctl cat health.timer
systemctl enable --now health.timer
```

The mental model I took from it was:

```text
timer   → when something should run
service → what systemd launches
script  → the actual task logic
```

The timer could be wrong while the script was correct, or the script could fail while the timer was working.

## Service successfully runs the wrong script logic

A oneshot cleanup service launched `/opt/scripts/log-cleaner.sh` with `WorkingDirectory=/root`.

The script contained a command like:

```bash
find . -maxdepth 1 -name '*.log' -type f -mtime -7 -print -delete
```

Two pieces of evidence mattered:

- `.` meant the current working directory, which systemd had set to `/root`, not the intended log directory.
- `-mtime -7` selected files newer than seven days when the requirement was to remove files older than seven days.

The service could therefore exit successfully while still producing the wrong operational result.

This strengthened the rule:

```text
successful launcher status ≠ correct task result
```

## Service dependencies

A later API exercise made `Requires=` and `After=` concrete rather than abstract unit-file syntax. A synchronization service had to populate a catalog file before an API could start successfully.

Once the lower service was fixed, I still had to retry the API because fixing a dependency did not retroactively make the earlier failed start succeed.

That added another relationship to follow:

```text
failed service
→ inspect unit configuration
→ identify required lower service
→ fix and verify lower service
→ retry dependent service
```

## cgroup v2 memory control

A batch workload was supposed to have a 128 MiB hard memory cap under `/sys/fs/cgroup/sad-batch`.

The setup script wrote:

```bash
echo 134217728 > "$CGROUP/memory.high"
```

but the requirement was for `memory.max`.

The new concepts were:

```text
memory.high → pressure/throttling threshold
memory.max  → hard memory ceiling
```

The literal value `max` in `memory.max` meant there was no numeric hard ceiling. Changing the setup to write `134217728` to `memory.max`, rerunning the setup service, and verifying the control file corrected the configuration before the memory-heavy batch job was started.

This also reinforced that files under `/sys` can be kernel-backed control interfaces rather than ordinary disk files.

## What changed in my troubleshooting approach

At first, services, cron, timers, scripts, and cgroups felt like unrelated Linux topics. The repeated pattern became clearer after several exercises:

```text
who launches it?
      ↓
what is configured to run?
      ↓
did that command actually run?
      ↓
did the command do the intended work?
      ↓
what lower dependency or resource does it rely on?
      ↓
verify the final result
```

That is now more useful to me than memorizing a separate command sequence for every automation problem.
