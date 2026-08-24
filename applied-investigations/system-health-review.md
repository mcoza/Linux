# Applied Investigation: Whole-System Health Review

This was the point where several earlier Linux topics stopped feeling like separate command categories and started becoming one investigation.

The task was to review an unfamiliar server and answer two questions:

1. What is this server doing?
2. Is anything I am seeing actually unhealthy?

I initially tended to mix those together. This exercise forced me to establish the system's apparent purpose before deciding whether a process, port, or utilization value was abnormal.

## 1. Start with processes and listeners

```bash
ps auxf
ss -tlpn
netstat -tlpn
```

Observed listeners included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Other observed services included `chronyd` and `dhclient`.

At first, the temptation was to treat unfamiliar listeners as possible problems. The more useful question was what role each listener appeared to have.

The bind addresses helped:

```text
0.0.0.0 / :::  → broadly listening
127.0.0.1      → local-only dependency
```

Combined with the process names, the layout suggested something like:

```text
network-facing service
        ↓
proxy/front-end layer
        ↓
local Python application
        ↓
local PostgreSQL database
```

That was an inference from the evidence, not proof of the complete architecture. The important correction was:

```text
unfamiliar ≠ bad
```

## 2. Put CPU/load in hardware context

```bash
lscpu
uptime
top
htop
ps
```

Observed during the review:

- 2 CPUs
- load average above 2
- Python application around 47–48% CPU
- PostgreSQL around 10–11% CPU

`uptime` told me the load value, but not whether that value was reasonable or which process was contributing to it. `lscpu` supplied the CPU count, while `top`/`ps` connected the utilization back to running processes.

The high-CPU Python process also appeared to be part of the application path, so "it uses the most CPU" was not enough evidence to justify killing it.

The question changed from:

```text
Which process is high?
```

to:

```text
Is the load persistent, and does the workload explain it?
```

## 3. Compare current memory with failure history

```bash
free -m
vmstat
dmesg
journalctl
```

The current memory snapshot looked much less dramatic than the earlier system behavior suggested. Historical logs showed a previous out-of-memory event.

That connected two different kinds of evidence:

```text
free / vmstat  → what memory looks like now
dmesg / journal → what happened earlier
```

The main lesson was:

```text
normal-looking state now ≠ proof that no failure happened earlier
```

A process may already have been killed, or the workload may have changed before the investigation began.

## 4. Check whether storage capacity explains it

```bash
df -h
lsblk
```

The main filesystem was about 37% used.

That made basic capacity exhaustion unlikely as the immediate explanation. It did not rule out every storage problem—permissions, open/deleted files, inode exhaustion, or I/O latency would need different evidence—but it let me stop treating "disk might be full" as an equally likely explanation.

## How the investigation came together

| Question | Evidence used | What it changed |
|---|---|---|
| What services appear to make up the host? | `ps`, `ss`, bind addresses | Built a tentative application map |
| Is the load meaningful? | CPU count, load average, process CPU | Added hardware and workload context |
| Was there a memory problem? | current memory plus logs | Revealed an earlier OOM event |
| Is simple disk exhaustion likely? | `df`, `lsblk` | Narrowed that explanation |

The important part was not any single command. It was the order in which each answer changed the next question.

## What this added to my troubleshooting model

Earlier exercises often had a fairly direct path:

```text
find the file
count the value
identify the PID
```

This review was different because several observations could all be true without any one of them being the root cause.

The model became:

```text
inventory what exists
→ infer the host's purpose carefully
→ measure current state
→ check historical evidence
→ narrow explanations
→ avoid changing a component until its role is understood
```

That same reasoning later helped with service dependencies, port conflicts, Nginx backends, and other multi-component exercises.
