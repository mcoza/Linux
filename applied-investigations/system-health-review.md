# Applied Investigation: Unfamiliar Linux Server Baseline

This example combines several of the repository's skills into one practical question:

```text
What is this Linux host doing,
and is any observed state actually unhealthy?
```

The goal is not to assume that an unfamiliar process, listener, or utilization value is bad. The goal is to inventory the host, build a tentative picture from evidence, and then decide what deserves deeper investigation.

## Capability demonstrated

I can use process, socket, CPU/load, memory, log-history, and storage evidence together to form an initial baseline of an unfamiliar Linux system.

## 1. Inventory processes and listeners

Useful commands:

```bash
ps auxf
ss -tlpn
netstat -tlpn
```

Observed listeners in this lab system included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Other running services included `chronyd` and `dhclient`.

The bind addresses give useful context:

```text
0.0.0.0 / ::: → broadly listening on available interfaces
127.0.0.1     → local-only listener
```

Combined with process names, the system suggested a possible application path such as:

```text
network-facing service
        ↓
proxy/front-end layer
        ↓
local Python application
        ↓
local PostgreSQL database
```

That is an inference, not proof of the complete architecture. The process names and bind addresses justify investigating that relationship; they do not prove every hop.

The rule I carry forward is:

```text
unfamiliar ≠ unhealthy
```

## 2. Put load in hardware and process context

Useful commands:

```bash
lscpu
uptime
top
htop
ps
```

Observed during the review:

```text
CPU count          → 2
load average       → above 2
Python process CPU → about 47–48%
PostgreSQL CPU     → about 10–11%
```

Each command answers a different part of the question:

```text
lscpu      → how much CPU capacity exists?
uptime     → what is the system load?
top/htop   → which processes are consuming resources now?
ps         → what are those processes and how are they related?
```

A process using the most CPU is not automatically the process to kill. If that process appears to be part of the host's application path, its utilization has to be evaluated against workload and persistence rather than treated as suspicious by itself.

## 3. Compare present memory state with failure history

Useful commands:

```bash
free -m
vmstat
dmesg
journalctl
```

The current memory snapshot did not look severe, but historical evidence showed that an out-of-memory event had occurred earlier.

That separates two questions:

```text
free / vmstat      → what does memory look like now?
dmesg / journalctl → what happened earlier?
```

This matters because the process involved in an earlier failure may already have exited or been killed by the time the host is inspected.

```text
healthy-looking state now
≠ proof that no earlier failure occurred
```

## 4. Check whether basic storage capacity explains the problem

Useful commands:

```bash
df -h
lsblk
```

The main filesystem was about 37% used.

That made simple filesystem-capacity exhaustion unlikely as the immediate explanation. It did not rule out other storage-related problems such as permissions, inode exhaustion, open/deleted files, or I/O latency; those would require different evidence.

The point of the check was to narrow possibilities rather than to declare the entire storage subsystem healthy.

## How the evidence changes the next question

| Question | Evidence | What I can conclude |
|---|---|---|
| What services are present? | `ps`, `ss`, process names, bind addresses | Build a tentative service/application map |
| Is load meaningful? | CPU count, load average, process CPU | Interpret load in host/workload context |
| Could memory have failed earlier? | current memory + kernel/journal history | Detect an earlier OOM despite a later normal snapshot |
| Is simple disk-full exhaustion likely? | `df`, `lsblk` | Basic filesystem capacity was not the immediate issue |

## What I can do from this baseline

After this first pass, I can choose a more focused investigation instead of changing components randomly.

```text
inventory what exists
→ infer relationships carefully
→ measure current state
→ check historical evidence
→ eliminate unsupported explanations
→ choose the next subsystem based on evidence
```

This is the same troubleshooting model used elsewhere in the repository, but applied across several Linux subsystems at once.
