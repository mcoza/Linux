# Applied Investigation: Whole-System Health Review

## Objective

Review an unfamiliar Linux server and answer two different questions:

1. What is this server doing?
2. Is it healthy?

The investigation required correlating processes, listeners, current utilization, historical errors, and storage rather than allowing one command to determine the conclusion.

## 1. Establish system purpose

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

**Interpretation:** The listener layout suggested connected components rather than unrelated processes:

```text
network-facing service
        ↓
proxy or front-end layer
        ↓
local Python application
        ↓
local PostgreSQL database
```

The broadly bound services appeared intended to receive network traffic, while the loopback-bound application and database appeared to be internal dependencies. This was an inference from the evidence, not proof of the complete architecture.

## 2. Interpret CPU and load with hardware context

```bash
lscpu
uptime
top
htop
ps
```

Observed during the review:

- 2 CPUs
- load above 2
- Python application around 47–48% CPU
- PostgreSQL around 10–11% CPU

**Reasoning:** A load value becomes meaningful only in relation to CPU availability and workload. The highest-CPU process was also part of the apparent application path, so terminating it simply because it ranked first would not be justified.

**Next step:** Determine whether the load persisted, identify runnable or blocked work, and correlate utilization with requests and application logs.

## 3. Compare current memory with historical evidence

```bash
free -m
vmstat
dmesg
journalctl
```

The current commands provided a snapshot. The logs showed a previous out-of-memory event.

**Reasoning:** A later snapshot can look acceptable after the kernel has already terminated a process or workload conditions have changed. Historical evidence was necessary before ruling out memory pressure.

**Next step:** Identify the affected process and timestamp, then correlate the event with application activity and memory trends.

## 4. Check whether storage capacity explains the problem

```bash
df -h
lsblk
```

The main filesystem was approximately 37% used.

**Interpretation:** Simple filesystem-capacity exhaustion was not the immediate explanation. `df -h` answered how much mounted filesystem space was used; `lsblk` answered how devices and partitions were arranged.

This did not rule out permissions, inode exhaustion, or I/O latency because those require different evidence.

## Evidence summary

| Area | Evidence | Interpretation |
|---|---|---|
| Purpose | Proxy, local application, and local database listeners | Suggested a multi-component application host |
| CPU context | 2 CPUs and load above 2 | Worth continued investigation, not a diagnosis by itself |
| Processes | Application and database consumed notable CPU | Likely workload components |
| Memory | Current snapshot plus historical OOM | Earlier memory failure existed |
| Storage | Main filesystem about 37% used | Basic capacity exhaustion was unlikely |
| Exposure | Broad proxy/front-end bindings and local dependencies | Helped distinguish external and internal service roles |

## Reasoning demonstrated

This investigation joined the earlier learning stages:

- text and command output were reduced to relevant evidence
- processes were connected to resource use
- current measurements were compared with historical logs
- listeners and bind addresses were used to infer service relationships
- alternative explanations were narrowed without overstating certainty

The main lesson was that system purpose and system health are related but separate questions. Understanding the workload is necessary before deciding whether an observed process, listener, or resource value is abnormal.
