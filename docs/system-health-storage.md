# System Health and Storage

## Purpose

System-health troubleshooting is not a single command. The main lesson from the Linux Server Review was to separate **what the server is doing** from **whether the server is healthy**.

A resource value only becomes meaningful after the system's role and workload are understood.

## Investigation order

```text
identify system purpose
        ↓
identify important services/processes
        ↓
check current CPU / memory / disk state
        ↓
check historical errors/events
        ↓
decide which observations are actually abnormal
```

## CPU information

```bash
lscpu
```

During the reviewed server exercise, the machine had **2 CPUs**.

That matters when interpreting load average because the same load value has a different meaning on a 2-CPU system than on a 16-CPU system.

## Load average

```bash
uptime
```

The reviewed system showed load above `2` on a 2-CPU host.

A useful simplified model is:

```text
load average
→ amount of runnable/uninterruptible work competing for system resources
```

It should be interpreted relative to CPU capacity and workload, not treated as a standalone pass/fail number.

## Process CPU usage

Tools practiced:

```bash
top
htop
ps
```

During the server review, notable process usage included roughly:

```text
Python web application   ~47–48% CPU
PostgreSQL               ~10–11% CPU
```

The correct next question was not immediately "kill the high-CPU process." It was whether those processes were expected for the server's purpose and whether the observed load represented a sustained problem.

## Memory

Current memory state was reviewed using:

```bash
free -m
vmstat
```

These tools provide a current snapshot, but the strongest lesson was that a current snapshot can miss a failure that already happened.

## Historical OOM investigation

System logs showed evidence of a prior **out-of-memory event**.

Sources used/reviewed included:

```bash
dmesg
journalctl
```

This created a reusable distinction:

```text
current system state
        ≠
complete failure history
```

A machine may look normal now even though the logs show that memory pressure caused a failure earlier.

## Disk capacity

```bash
df -h
```

The main filesystem on the reviewed server was only about **37% used**, which helped rule out simple disk-capacity exhaustion as the immediate issue.

That is an important troubleshooting habit: use evidence to eliminate plausible causes rather than assuming every resource problem is the bottleneck.

## Block-device layout

```bash
lsblk
```

`lsblk` helps answer a different question from `df`:

```text
df
→ mounted filesystem usage

lsblk
→ block devices, partitions, and layout
```

## Disk I/O tools introduced

The review also introduced:

```bash
iostat
sar -d
```

These can provide disk-performance data, but they are **not currently claimed as demonstrated proficiency** in this portfolio because they were introduced rather than practiced deeply.

## cgroups / resource control

The Genova scenario provided completed hands-on exposure to a cgroup-related troubleshooting problem.

That supports the concept that Linux can organize processes into control groups and apply/account for resource constraints. The repository does not reconstruct the exact Genova solution or claim deep cgroup administration; that area remains developing.

## Storage troubleshooting beyond capacity

A useful mental model is to distinguish several different storage questions:

```text
Is the filesystem full?
→ df

What block devices/partitions exist?
→ lsblk

Is storage slow or saturated?
→ iostat / sar (developing)

Is a process still holding a file/resource?
→ lsof / fuser
```

Different symptoms require different measurements.

## Reusable health-check model

```text
CPU / load
Memory
Disk capacity
Processes
Listeners/services
Historical logs
```

The order may change depending on the symptom, but the key is to correlate these areas instead of reading one number in isolation.

## Current level

**Practiced:** CPU/load interpretation, current memory review, historical OOM investigation, filesystem capacity, block-device layout.

**Practiced exposure:** cgroups/resource control.

**Developing:** disk I/O performance analysis with `iostat` and `sar`.
