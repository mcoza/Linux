# 2. Processes and System State

This stage connected visible symptoms to active processes and then expanded from individual processes to the health of the host.

## Tracing a changing file to a process

**Situation:** A log file was continuously receiving data, but the responsible process was unknown. The file needed to remain intact.

```bash
tail -f /var/log/example.log
fuser /var/log/example.log
lsof /var/log/example.log
ps -p <PID>
```

**Why these commands:**

- `tail -f` verifies that the file is actively changing.
- `fuser` starts with a known resource and identifies processes using it.
- `lsof` provides open-file relationships and additional process context.
- `ps` inspects the identified process before an action is taken.

**Reasoning:** The file was the symptom, not the cause. Removing it would not necessarily stop the writer and would violate the requirement to preserve the file. The safer sequence was to identify the owner, inspect it, stop only the responsible process, and verify that writes ceased.

Related tools such as `pgrep` and `kill` were used for process discovery and targeted termination.

## Searching live system state under `/proc`

`/proc` is a virtual filesystem that exposes kernel and process state. Searching it required distinguishing normal filesystem behavior from the characteristics of a dynamic interface.

**Reasoning applied:**

- permission errors do not prove that the search command is wrong
- entries may change while they are being inspected
- `find` answers where an object is
- `grep` answers what textual content it contains

## Understanding current utilization

**Situation:** I needed to determine whether CPU, load, memory, or storage pressure might explain system behavior.

```bash
lscpu
uptime
top
htop
ps auxf
free -m
vmstat
df -h
lsblk
```

**Why the commands must be combined:**

- `lscpu` supplies CPU context for interpreting load.
- `uptime` provides load averages, not the identity of the responsible process.
- `top`, `htop`, and `ps` connect utilization to processes.
- `free` and `vmstat` show current memory conditions from different views.
- `df -h` shows mounted filesystem usage.
- `lsblk` shows block devices and partition layout.

A high value is not automatically a fault. It must be interpreted against the host's hardware, workload, and purpose.

## Current state versus failure history

```bash
dmesg
journalctl
```

**Situation:** The current memory snapshot did not fully explain earlier system behavior.

**Why these commands:** Current-state commands answer what is happening now. Kernel and journal logs can show what happened earlier.

**Applied interpretation:** Historical evidence showed a prior out-of-memory event even though the machine could appear acceptable during the later inspection.

```text
acceptable current snapshot ≠ proof that no earlier failure occurred
```

## Additional applied work

A named-pipe troubleshooting exercise introduced FIFO/IPC behavior. A resource-control exercise introduced cgroups. These are recorded as applied exposure; the retained evidence does not support claims of deep IPC or cgroup administration.

## What changed in my troubleshooting approach

I learned to move from the affected resource to its process, and from a process snapshot to broader system context. I also learned not to treat present conditions as a complete account of system history.
