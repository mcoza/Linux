# System Troubleshooting

This note combines the process, filesystem, system-health, and automation work practiced across the Linux exercises.

## Process and file relationships

A recurring troubleshooting pattern was moving from a visible resource back to the process responsible for it:

```text
file / socket symptom
      ↓
identify the process
      ↓
inspect process state
      ↓
make a targeted change
      ↓
verify the symptom changed
```

Tools practiced included:

```bash
tail -f
fuser
lsof
ps
ps auxf
pgrep
kill
```

One process investigation used this pattern to identify and stop the process continuously writing to a log while preserving the log file.

## `find`, `grep`, and `/proc`

`find` locates filesystem objects; `grep` searches text content. They can be combined when the question is which file in a tree contains a particular value.

Example pattern:

```bash
find /path -type f -exec grep -l 'pattern' {} +
```

One filesystem-search exercise applied this idea under `/proc/sys`. `/proc` is a virtual filesystem exposing live process/kernel state, so permission errors and dynamically changing entries are normal parts of working there.

Another completed exercise provided exposure to named pipes/FIFOs. The exact command sequence is not retained, so the portfolio does not reconstruct it.

## CPU, load, memory, and storage

The whole-system review used:

```bash
lscpu
uptime
top
htop
free -m
vmstat
dmesg
journalctl
df -h
lsblk
```

Key observations from that exercise included:

- 2 CPUs
- load above 2 during the review
- Python web application around 47–48% CPU
- PostgreSQL around 10–11% CPU
- a prior out-of-memory event found in system logs
- the main filesystem around 37% used

The main lesson was to separate **current state** from **historical evidence**. A machine can look acceptable at the moment of inspection while logs show an earlier failure.

`df -h` and `lsblk` also answer different questions:

```text
df -h -> mounted filesystem usage
lsblk -> block-device and partition layout
```

Disk-I/O tools such as `iostat` and `sar -d` were introduced, but are not presented as demonstrated proficiency.

## Services and scheduled work

Completed exercises included systemd timer, scheduled backup, and automated-cleanup failures.

The reusable troubleshooting chain is:

```text
expected result missing
      ↓
identify timer / scheduler / service
      ↓
inspect unit or configuration
      ↓
inspect logs
      ↓
inspect the command or script being run
      ↓
verify the final result
```

This work provided hands-on exposure to `systemctl`, `journalctl`, systemd timers, scheduled tasks, and the difference between a scheduler firing and the underlying command actually succeeding.

A completed resource-control exercise also provided exposure to cgroup troubleshooting. Deep cgroup administration is not claimed.

## What this demonstrates

- process discovery and targeted termination
- open-file/process relationships
- filesystem and content searching
- `/proc` familiarity
- CPU/load and memory review
- historical OOM investigation
- disk-capacity and block-device inspection
- basic systemd/timer troubleshooting
- scheduled-task failure investigation
- introductory FIFO and cgroup troubleshooting exposure

For the strongest multi-component example, see [Whole-System Health Review](../cases/system-health-review.md).