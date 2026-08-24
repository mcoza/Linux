# Case Study: Linux Server Review

## Goal

Review an unfamiliar Linux server and answer two separate questions:

1. **What is this server doing?**
2. **Is the server healthy?**

This became one of the strongest exercises in the repository because it required process discovery, resource review, log inspection, service mapping, and network interpretation in one investigation.

## Why those questions are separate

A common troubleshooting mistake is to see a high number and immediately label it a fault.

Examples:

```text
high CPU
high load
open port
large memory use
```

None of those observations is meaningful enough by itself.

The investigation first tried to understand the role of the machine, then interpreted resource usage in that context.

```text
system purpose
    ↓
important processes/services
    ↓
current utilization
    ↓
historical errors
    ↓
actual health assessment
```

## 1. Identify the system's role

Processes and listening sockets were used to infer what the host was doing rather than relying on a hostname or assumption.

Relevant listeners observed during the review included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Other running services noticed included:

```text
chronyd
dhclient
```

Commands used/reviewed for this stage included:

```bash
ps auxf
ss -tlpn
netstat -tlpn
```

### Interpretation

The listener layout suggested a service chain rather than five unrelated processes:

```text
network-facing service
        ↓
proxy/front-end layer
        ↓
local Python application
        ↓
local PostgreSQL database
```

The Python application and PostgreSQL were both bound to loopback addresses, while SSH and HAProxy listened more broadly.

That distinction helped infer which components were intended to be directly reachable and which were internal dependencies.

## 2. Interpret bind addresses

The exercise reinforced the meaning of common listener addresses.

```text
127.0.0.1
→ loopback; local host only

0.0.0.0
→ all IPv4 interfaces

:::
→ IPv6 unspecified / all IPv6 interfaces
```

This made a service listing useful architecturally rather than just a list of port numbers.

The review also covered the difference between:

```text
TCP LISTEN
```

and UDP output such as:

```text
UNCONN
```

`UNCONN` is not automatically an error; it reflects UDP's connectionless socket model.

## 3. CPU and load

CPU information was checked with:

```bash
lscpu
```

The server had:

```text
2 CPUs
```

Load was reviewed with:

```bash
uptime
```

and was above `2` during the investigation.

Live process utilization was reviewed with:

```bash
top
htop
ps
```

Notable observations included approximately:

```text
Python web application   47–48% CPU
PostgreSQL               10–11% CPU
```

### Interpretation

The important lesson was to compare load with available CPU capacity and then identify which processes were responsible for the work.

The observation alone did not justify killing the Python process because the application appeared to be part of the server's intended function.

## 4. Memory

Current memory state was checked with:

```bash
free -m
vmstat
```

That answered:

> What does memory look like right now?

It did not answer:

> Has memory already caused a failure?

So the investigation moved into historical evidence.

## 5. Historical OOM evidence

System logs were inspected with tools including:

```bash
dmesg
journalctl
```

The logs showed that the machine had experienced an **out-of-memory (OOM) event**.

This was one of the most important lessons from the exercise:

```text
current resource snapshot
        ≠
complete failure history
```

A system can appear acceptable during inspection while historical logs show that it previously exhausted memory.

## 6. Disk capacity

Filesystem utilization was checked with:

```bash
df -h
```

The primary filesystem was only about:

```text
37% used
```

So simple disk-capacity exhaustion was ruled out as the immediate cause.

Block-device layout was also reviewed with:

```bash
lsblk
```

### Interpretation

This separated two questions:

```text
df -h
→ how full are mounted filesystems?

lsblk
→ what block devices/partitions exist?
```

Additional disk-performance tools such as `iostat` and `sar -d` were introduced during the review, but they are not presented as demonstrated proficiency because they were not practiced deeply enough.

## Investigation summary

The review produced a system-level picture:

| Area | Observation | Interpretation |
|---|---|---|
| CPU | 2 CPUs | Important context for interpreting load |
| Load | Above 2 | Worth investigating on a 2-CPU host |
| Processes | Python app and PostgreSQL notable CPU users | Likely expected workload components, not automatically faults |
| Memory | Current state reviewed | Snapshot alone did not show full history |
| Logs | Prior OOM event found | Historical memory failure existed |
| Disk | ~37% used | Capacity exhaustion not the immediate issue |
| Network | Front-end listeners + loopback app/database | Helped infer service architecture |

## Reusable troubleshooting model

The strongest pattern from the exercise is:

```text
1. Determine system purpose
2. Map important processes/services
3. Inspect current resource state
4. Inspect historical errors/events
5. Correlate the evidence
6. Avoid changing components that are behaving normally for the workload
```

## What this case demonstrates

- Linux process inspection
- service and listener mapping
- bind-address interpretation
- basic service-architecture inference
- CPU/load investigation
- current memory review
- historical OOM investigation
- disk-capacity review
- using current state and logs together
- ruling out unsupported causes rather than jumping to a fix

## Current limits

This case does not claim advanced performance engineering. Disk I/O analysis, packet-level networking, and deeper systemd dependency analysis remain developing areas documented in [skills-progress.md](../skills-progress.md).
