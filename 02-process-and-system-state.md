# 2. Processes and System State

This section documents how I inspect running processes and use system-state evidence to decide whether a problem belongs to one process, a resource, or the wider host.

## What I know

I have hands-on practice with:

```text
ps / ps auxf → inspect processes and process trees
pgrep        → find PIDs by process name
fuser        → identify processes using a known file or port
lsof         → inspect open-file relationships
top / htop   → inspect live process/resource usage
lscpu        → inspect CPU context
uptime       → read load averages
free / vmstat→ inspect current memory state
dmesg        → inspect kernel messages
journalctl   → inspect historical system/service logs
df           → inspect filesystem capacity
du           → measure visible file/directory usage
lsblk        → inspect disks and partitions
```

I also understand the basic relationship:

```text
program/script
      ↓ running instance
process
      ↓ identified by
PID
      ↓ may have
PPID / parent process
```

A PID tells me which running process I found. A PPID gives me another relationship to follow when I need to know what launched it.

## What I can do with those tools

### Trace a known file to the process using it

If a file is continuously changing and I need to know what is writing to it, I can start from the file rather than guess at process names.

```bash
tail -f /var/log/example.log
fuser /var/log/example.log
lsof /var/log/example.log
ps -fp <PID>
```

The reasoning is:

```text
file is changing
→ confirm it is still changing
→ identify process using the file
→ inspect that PID
→ decide whether stopping that process is justified
```

This is safer than deleting the file when the requirement is to stop the writer while preserving the file.

### Trace a port to its process

If a program reports that a specific TCP port is already in use, I can ask which process owns that resource.

```bash
sudo fuser -v 8000/tcp
ps -fp <PID>
```

or use a socket view:

```bash
sudo ss -ltnp
```

The relationship becomes:

```text
known port
→ owning PID
→ process command
→ PPID
→ parent / service / configuration if needed
```

This is important because identifying the PID does not automatically mean the correct action is to kill it. The parent or service manager may be responsible for starting it again.

### Put CPU and load in context

I can combine host capacity with process utilization instead of treating a load number by itself as a diagnosis.

```bash
lscpu
uptime
top
htop
ps auxf
```

These answer different questions:

```text
lscpu       → how much CPU capacity is available?
uptime      → what are the recent load averages?
top / htop  → which processes are active now?
ps auxf     → what processes exist and how are they related?
```

For example, a load average above 2 has different meaning on a two-CPU VM than on a host with many more CPUs. The number needs hardware and workload context.

### Separate current memory state from earlier failure evidence

A current snapshot can look normal after a failure has already happened.

```bash
free -m
vmstat
dmesg
journalctl
```

I use them as two different evidence types:

```text
free / vmstat      → what memory looks like now
dmesg / journalctl → whether an earlier OOM or related failure was recorded
```

That prevents me from treating a healthy-looking current snapshot as proof that memory was never a problem.

### Diagnose filesystem-capacity problems

`df` and `du` answer related but different questions:

```text
df → which filesystem is full?
du → what visible files/directories are consuming space?
```

A useful sequence is:

```bash
df -h
du -h <path>
```

If `df` reports substantially more used space than `du` can account for, I know to consider resources such as deleted files that are still held open by a process and inspect with `lsof` rather than assuming one of the commands is wrong.

### Understand simple process-to-process blocking

A named pipe/FIFO introduced a different resource relationship:

```text
producer process
      ↓ writes
FIFO buffer
      ↓ read by
consumer process
```

Because the pipe has limited buffering, a producer can block when the consumer is not reading quickly enough. That is a process interaction problem, not ordinary disk-file latency.

## How symptoms narrow my first inspection

```text
"what is using/writing this file?" → fuser / lsof → PID → ps
"port already in use"              → ss / fuser → PID → ps
"which process is using CPU?"      → top / htop / ps
"possible memory pressure"         → free / vmstat + process view
"did OOM happen earlier?"          → dmesg / journalctl
"no space left"                    → df → du
"df and du disagree"               → inspect open/deleted files with lsof
```

These are starting points, not conclusions. The command tells me what state exists; the output still has to be connected back to the original symptom.

## What this lets me do

I can move from a resource symptom to the process behind it, then decide whether I need to continue into its parent, service manager, logs, or broader host state.

```text
symptom/resource
→ owner
→ process
→ parent/context
→ current + historical evidence
→ targeted next step
```
