# 3. Services and Automation

This stage focused on an important distinction: a scheduler can run successfully while the command it launches still fails.

## Service investigation

```bash
systemctl status <unit>
journalctl -u <unit>
```

**When I use them:** A service is unavailable, a timer does not produce its expected result, or a scheduled operation appears to have failed.

**Why both matter:** `systemctl` exposes unit state and recent status information. `journalctl -u` provides the service-specific event history needed to understand why the unit started, stopped, or failed.

A service reported as active does not prove that every operation performed by the application is succeeding.

## Timers and scheduled work

I applied this reasoning while troubleshooting a systemd timer, missing scheduled backups, and failed automated cleanup.

```text
expected output is missing
      ↓
identify the scheduler, timer, or service
      ↓
verify that it fired
      ↓
inspect its logs
      ↓
inspect the command or script it launched
      ↓
check permissions, paths, environment, and destination
      ↓
verify the expected output
```

**Reasoning:** The missing artifact is only the visible symptom. Possible failure points include the schedule, unit configuration, executable path, command logic, permissions, runtime environment, and output destination.

## Resource controls

A completed cgroup exercise provided introductory exposure to Linux resource controls.

**Reasoning learned:** A process problem may be caused by a configured resource boundary rather than a lack of physical resources across the entire machine. The relevant question becomes whether the process is constrained by its group and whether the configuration matches the intended limit.

The precise command trail was not retained, so this is not presented as demonstrated cgroup administration.

## What changed in my troubleshooting approach

I stopped treating “the timer ran” and “the task succeeded” as equivalent statements. Troubleshooting automation requires following the entire execution chain through to the intended result.
