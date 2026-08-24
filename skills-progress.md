# Linux Skills Progress

This page tracks the current state of Linux skills practiced through hands-on labs and troubleshooting exercises. It is deliberately conservative: **completed exposure is not the same thing as strong proficiency**.

## Current matrix

| Area | Current level | What is supported | Related notes |
|---|---|---|---|
| Package management | Practiced | APT workflow, `apt` vs `apt-get`, install/search/show/remove/update concepts | [Package Management](docs/package-management.md) |
| Text / log processing | Practiced | `grep`, `awk`, `sort`, `uniq`, pipelines, field extraction, aggregation, arithmetic | [Text & Data Processing](docs/text-data-processing.md) |
| CSV / structured text | Practiced | Completed CSV transformation and merge exercises | [Text & Data Processing](docs/text-data-processing.md) |
| Processes / open files | Practiced | `ps`, `pgrep`, `kill`, `fuser`, `lsof`, `tail -f`, file-to-process investigation | [Processes, Files, `/proc` & IPC](docs/processes-files-proc-ipc.md) |
| `/proc` / runtime state | Practiced | Filesystem/content investigation under `/proc`, `find`, `grep`, regex anchors | [Processes, Files, `/proc` & IPC](docs/processes-files-proc-ipc.md) |
| IPC / named pipes | Practiced exposure | Completed named-pipe troubleshooting exercise; deeper FIFO behavior still developing | [Processes, Files, `/proc` & IPC](docs/processes-files-proc-ipc.md) |
| CPU / load | Practiced | `lscpu`, `uptime`, `top`, `htop`, process CPU review, load relative to CPU count | [System Health & Storage](docs/system-health-storage.md) |
| Memory / OOM | Practiced | `free`, `vmstat`, `dmesg`, `journalctl`, historical OOM investigation | [System Health & Storage](docs/system-health-storage.md) |
| Disk capacity / devices | Practiced | `df -h`, `lsblk`, ruling out simple capacity exhaustion | [System Health & Storage](docs/system-health-storage.md) |
| Disk I/O performance | Developing | `iostat` and `sar -d` introduced, not treated as demonstrated proficiency | [System Health & Storage](docs/system-health-storage.md) |
| cgroups / resource control | Practiced exposure | Completed cgroup troubleshooting scenario; deeper administration still developing | [System Health & Storage](docs/system-health-storage.md) |
| Ports / listeners | Practiced | `ss`, `netstat`, bind addresses, TCP `LISTEN`, UDP `UNCONN`, service-to-port mapping | [Networking & Services](docs/networking-services.md) |
| Network diagnosis | Developing | Multiple port/service exercises completed; deeper packet/routing diagnosis needs repetition | [Networking & Services](docs/networking-services.md) |
| systemd / timers | Practiced exposure | Completed timer/service troubleshooting; deeper dependency and unit-file work still developing | [Services & Automation](docs/services-automation.md) |
| Scheduled automation | Practiced exposure | Backup and cleanup/maintenance failure scenarios completed | [Services & Automation](docs/services-automation.md) |
| TLS certificates | Practiced exposure | Completed certificate-renewal exercise; deeper PKI operations not claimed | [Web, Security & Application Services](docs/web-security-app-services.md) |
| Nginx / reverse proxy | Practiced | Site config inspection, `sites-available` / `sites-enabled`, `proxy_pass`, backend tracing | [Web, Security & Application Services](docs/web-security-app-services.md) |
| FTP synchronization | Practiced exposure | Completed FTP catalog-sync troubleshooting scenario | [Web, Security & Application Services](docs/web-security-app-services.md) |
| Database troubleshooting | Practiced exposure | Completed database-write troubleshooting scenario; database administration not claimed | [Web, Security & Application Services](docs/web-security-app-services.md) |

## Strongest current troubleshooting patterns

### Resource → process

```text
unexpected CPU / memory / load
        ↓
identify top consumers
        ↓
understand system role
        ↓
check current state
        ↓
check historical events
```

### File → process

```text
file is changing / locked / in use
        ↓
identify process using it
        ↓
inspect PID/process state
        ↓
make targeted change
        ↓
verify file behavior
```

### Port → service → configuration

```text
port or connectivity symptom
        ↓
inspect listeners
        ↓
map listener to process/service
        ↓
inspect bind address
        ↓
trace service configuration/dependency
        ↓
verify request path
```

### Automation failure

```text
expected task did not happen
        ↓
identify scheduler/service
        ↓
inspect configuration
        ↓
inspect execution evidence/logs
        ↓
correct cause
        ↓
verify future execution path
```

## Developing areas

These remain intentionally marked as developing until more hands-on work supports stronger claims:

- deeper Linux permissions and ownership administration
- DNS troubleshooting
- firewalling
- SSH troubleshooting
- systemd dependency/unit-file troubleshooting
- disk I/O performance analysis
- deeper cgroup administration
- database administration
- deeper TLS/PKI operations

## Not currently claimed

The repository does **not** currently claim strong hands-on proficiency in:

- Kubernetes administration
- advanced Git debugging/bisect workflows
- AWS storage administration
- advanced SSH recovery
- production database administration
- production web-server administration

Some of those areas have been encountered or attempted in training, but they are not promoted here until the hands-on record supports the claim.

## Update rule

A topic moves upward only when new work adds evidence:

```text
introduced
   ↓
practiced exposure
   ↓
practiced repeatedly
   ↓
stronger portfolio case
```

The labels describe the current evidence, not a permanent ceiling.
