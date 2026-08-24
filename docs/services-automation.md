# Services, Scheduling, and Automation

## Purpose

Several completed Linux exercises involved a task that was supposed to happen automatically but did not.

That creates a different troubleshooting question from an interactive command failure:

> **Which component was responsible for running the task, and where in that execution chain did it fail?**

## Automation troubleshooting model

```text
expected result missing
        ↓
identify scheduler / timer / service
        ↓
inspect configuration
        ↓
inspect execution state and logs
        ↓
inspect the command/script being invoked
        ↓
correct the failure
        ↓
verify the intended execution path
```

The useful habit is to check the whole chain instead of assuming the scheduler itself is always the problem.

## systemd service investigation

Systemd-related work introduced the idea that a running Linux application may be managed through a unit rather than launched manually.

Useful questions include:

```text
Does the unit exist?
Is it active?
Did it fail?
What command does it execute?
What dependency or timer triggers it?
What do its logs show?
```

`systemctl` and `journalctl` are central tools for this style of investigation, although deeper systemd dependency and unit-file work is still developing.

## systemd timers

The completed **Cairo — Time for a Timer** exercise provided hands-on timer troubleshooting exposure.

The reusable model is:

```text
timer
  ↓
service/unit
  ↓
script or command
  ↓
application/service behavior
  ↓
expected output/log/result
```

If the final result is missing, each link can be inspected independently.

The exact Cairo command sequence is not recreated here because it is not retained with enough detail for a faithful write-up.

## Scheduled backups

The completed **Alexandria — The Vanishing Backups** exercise involved a scheduled backup failure.

This supports a general distinction:

```text
script works manually
        ≠
script works when scheduled
```

Scheduled execution may differ because of:

- environment variables
- working directory
- permissions
- PATH
- shell/interpreter assumptions
- timing
- missing files or directories

The repository records the troubleshooting category without inventing the exact failure mechanism from memory.

## Automated maintenance

The completed **Valladolid — Cleaner not cleaning** exercise added another automation-failure example.

The transferable lesson is to verify both:

```text
Did the scheduler run?
```

and:

```text
Did the scheduled command actually produce the intended result?
```

A successful scheduler trigger does not guarantee that the script or maintenance action succeeded.

## Logs as execution evidence

Automation problems are especially dependent on historical evidence because the task often runs when nobody is watching.

That makes logs important for answering:

```text
Did it run?
When did it run?
What exited/faulted?
What component emitted the error?
```

This connects automation troubleshooting to the broader lesson from system health work:

```text
current state
≠
historical execution evidence
```

## Verification

A fix should be verified at the level the user or system actually cares about.

For example:

```text
timer says active
```

is weaker evidence than:

```text
timer triggered
→ service ran
→ command completed
→ expected file/log/output changed
```

## Current level

**Practiced exposure:** systemd timers, scheduled backup failures, automated maintenance failures.

**Developing:** deeper systemd unit/dependency troubleshooting, cron environment behavior, service ordering, and production scheduling practices.
