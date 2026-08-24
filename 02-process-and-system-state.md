# 2. Processes and System State

This stage connected visible symptoms to active processes and then expanded from individual processes to the health of the host. The main improvement was learning to follow relationships instead of treating `ps`, `fuser`, `lsof`, and utilization commands as separate tools.

## Tracing a changing file to a process

**Situation:** A log file was continuously receiving data, but the responsible process was unknown. The file needed to remain intact.

```bash
tail -f /var/log/example.log
fuser /var/log/example.log
lsof /var/log/example.log
ps -fp <PID>
```

**Why these commands:**

- `tail -f` verifies that the file is actively changing.
- `fuser` starts with a known resource and identifies processes using it.
- `lsof` provides open-file relationships and additional process context.
- `ps` inspects the identified process before an action is taken.

**Reasoning:** The file was the symptom, not the cause. Removing it would not necessarily stop the writer and would violate the requirement to preserve the file. The safer sequence was to identify the owner, inspect it, stop only the responsible process, and verify that writes ceased.

Related tools such as `pgrep` and `kill` were used for process discovery and targeted termination.

## Following a resource into its process tree

Later work made the same pattern more concrete with network ports. If an application reported that a port was already in use, I could start with the known resource rather than scan the entire machine blindly:

```bash
fuser -v 8000/tcp
ps -fp <PID>
```

From `ps`, the `PPID` gave me another relationship to follow. In one exercise, that led from a Python child process to its parent and then to PID 1/systemd, which explained why killing only the child process would not have addressed the underlying service configuration.

The pattern became:

```text
known file or port
      ↓
fuser / lsof / ss
      ↓
PID
      ↓
ps
      ↓
process and PPID
      ↓
parent, service, or configuration
```

This was a useful change in my troubleshooting because a PID became a waypoint rather than the final answer.

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

A high value is not automatically a fault. It has to be interpreted against the host's hardware, workload, and purpose.

## Current state versus failure history

```bash
dmesg
journalctl
```

**Situation:** The current memory snapshot did not fully explain earlier system behavior.

Current-state commands answer what is happening now. Kernel and journal logs can show what happened earlier.

Historical evidence showed a prior out-of-memory event even though the machine could appear acceptable during the later inspection.

```text
acceptable current snapshot ≠ proof that no earlier failure occurred
```

## Filesystem usage versus visible files

Another exercise reinforced that `df` and `du` answer different questions:

```text
df → filesystem-level space usage
du → space attributed to visible files/directories
```

When those views disagree, an open file can be part of the explanation because a process may still hold a deleted file open. That connects storage troubleshooting back to `lsof` and process ownership instead of treating disk usage as a completely separate topic.

## Additional applied work

A named-pipe exercise introduced FIFO/IPC behavior. Resource-control exercises later introduced cgroups. These are recorded as applied exposure rather than deep IPC or cgroup administration.

## What changed in my troubleshooting approach

Earlier, I tended to see a process name, open port, or high utilization value and ask whether it was "the problem." I now try to first establish what owns the affected resource, what role that process has, what launched it, and whether the evidence actually connects it to the reported symptom.
