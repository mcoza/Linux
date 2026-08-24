# Linux Server Review

## Goal

Review an unfamiliar Linux server and answer two separate questions:

1. **What is this server doing?**
2. **Is the server healthy?**

The exercise was useful because it forced service discovery, resource review, log inspection, and network interpretation into one troubleshooting workflow instead of treating each command as an isolated task.

## System purpose

The server's role was inferred by inspecting running processes and listening sockets rather than assuming its purpose from a hostname or one service.

Relevant listeners included:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Other running services noticed during the review included `chronyd` and `dhclient`.

This exposed an important service-layout pattern:

```text
Externally reachable service
          ↓
proxy / front-end layer
          ↓
local application
          ↓
local database
```

The loopback-bound Python and PostgreSQL services were not directly exposed through every network interface, while SSH and HAProxy were listening more broadly.

## Network interpretation

Listener inspection reinforced the meaning of common bind addresses:

```text
127.0.0.1
→ loopback; reachable only from the local host

0.0.0.0
→ all IPv4 interfaces

:::
→ IPv6 unspecified/all IPv6 interfaces
```

Tools used or reviewed for this part included:

```bash
ss -tlpn
netstat -tlpn
ps auxf
```

The exercise also covered the difference between TCP `LISTEN` state and UDP output such as `UNCONN`.

## CPU and load

The server had **2 CPUs**.

During the review, the Python web application was using roughly **47–48% CPU**, while PostgreSQL was using roughly **10–11%**. The system load average was above `2`.

Commands used to inspect CPU/process state included:

```bash
lscpu
uptime
top
htop
ps
```

The important lesson was that load average is not meaningful in isolation. A load near or above the number of available CPUs deserves more attention than the same load on a much larger system.

## Memory and historical failures

Current memory state was reviewed with tools such as:

```bash
free -m
vmstat
```

The investigation then moved beyond the current snapshot into system logs. Historical evidence showed that the machine had experienced an **out-of-memory (OOM) event**.

Log sources discussed and inspected included:

```bash
dmesg
journalctl
```

This created another reusable troubleshooting distinction:

```text
Current resource usage
        ≠
complete system history
```

A system can look acceptable at the moment of inspection while logs show that it failed earlier.

## Disk

Filesystem capacity was checked with:

```bash
df -h
lsblk
```

The primary filesystem was only about **37% used**, so simple disk-capacity exhaustion was not the immediate problem.

Additional disk-performance tools such as `iostat` and `sar -d` were introduced during the review, but they are not treated here as demonstrated proficiency.

## Troubleshooting model learned

The strongest lesson from the exercise was to separate **purpose** from **utilization**.

A better investigation order is:

```text
1. Identify the system's role
2. Identify important services and dependencies
3. Check current resource state
4. Check historical errors/events
5. Determine which observations are actually abnormal for that role
```

That prevents jumping straight from a high CPU percentage, a memory value, or an open port to an unsupported conclusion.

## Skills demonstrated

- Linux process inspection
- listener and port interpretation
- basic service-architecture inference
- CPU/load investigation
- memory review
- historical OOM investigation
- disk-capacity review
- use of current state and logs together during troubleshooting
