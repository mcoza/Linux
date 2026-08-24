# Case Study: Linux Server Review

## Goal

Review an unfamiliar Linux server and answer two questions:

1. **What is this server doing?**
2. **Is it healthy?**

The exercise required process discovery, resource review, log inspection, and listener/service mapping in one investigation.

## 1. Infer the server's role

Commands used/reviewed included:

```bash
ps auxf
ss -tlpn
netstat -tlpn
```

Relevant listeners observed:

```text
sshd        0.0.0.0:22
HAProxy     0.0.0.0:8000
gotty       :::8080
python3     127.0.0.1:9000
PostgreSQL  127.0.0.1:5432
```

Other running services noticed included `chronyd` and `dhclient`.

The layout suggested a service chain rather than unrelated processes:

```text
network-facing service
        ↓
proxy/front-end layer
        ↓
local Python application
        ↓
local PostgreSQL database
```

The bind addresses also mattered:

```text
127.0.0.1 -> local host only
0.0.0.0   -> all IPv4 interfaces
:::       -> IPv6 unspecified / all IPv6 interfaces
```

## 2. Check CPU and load

```bash
lscpu
uptime
top
htop
ps
```

Observed during the review:

- **2 CPUs**
- load above **2**
- Python web application around **47–48% CPU**
- PostgreSQL around **10–11% CPU**

The important step was correlating utilization with the server's apparent purpose instead of assuming that the highest-CPU process should be killed.

## 3. Compare current memory with historical evidence

Current memory was reviewed with:

```bash
free -m
vmstat
```

Historical evidence was checked with tools including:

```bash
dmesg
journalctl
```

The logs showed a prior **out-of-memory (OOM) event**.

That produced one of the strongest lessons from the exercise:

```text
current system state
        ≠
complete failure history
```

A system may look acceptable during inspection even though logs show an earlier failure.

## 4. Check storage

```bash
df -h
lsblk
```

The main filesystem was about **37% used**, which helped rule out simple disk-capacity exhaustion as the immediate problem.

`df -h` showed filesystem usage; `lsblk` showed the underlying block-device/partition layout.

## Summary

| Area | Observation | Interpretation |
|---|---|---|
| CPU | 2 CPUs | Context for interpreting load |
| Load | Above 2 | Worth investigation on this host |
| Processes | Python app + PostgreSQL notable CPU users | Likely workload components, not automatically faults |
| Memory | Current state reviewed | Snapshot alone did not show the full history |
| Logs | Prior OOM event | Historical memory failure existed |
| Disk | ~37% used | Capacity exhaustion not the immediate issue |
| Network | Broad listeners + loopback app/database | Helped infer service architecture |

## Reusable troubleshooting pattern

```text
determine system purpose
      ↓
map important processes/services
      ↓
inspect current resources
      ↓
inspect historical errors
      ↓
correlate the evidence
```

## What this demonstrates

- Linux process inspection
- service/listener mapping
- bind-address interpretation
- basic service-architecture inference
- CPU/load investigation
- memory review and historical OOM investigation
- disk-capacity review
- using current state and logs together

This case does not claim advanced performance engineering, packet-level networking, or deep systemd administration.