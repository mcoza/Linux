# Linux Skills Progress

This page tracks Linux topics that have been practiced hands-on. It is organized by technical area rather than by individual training exercises.

## Command line and text processing

Practiced:

- `grep` for searching and filtering text
- `awk` for field extraction, summation, and structured processing
- `sort` and `uniq -c` for aggregation and frequency counting
- `head` and `tail` for narrowing command output
- `bc` for decimal arithmetic and output precision
- CSV transformation and merging
- command pipelines that combine several small tools to answer one question

Reusable pattern:

```text
raw text or log
      ↓
extract relevant fields
      ↓
filter / normalize
      ↓
aggregate / calculate
      ↓
inspect final result
```

## Processes, files, and `/proc`

Practiced:

- `ps` and `pgrep` for process discovery
- `kill` for targeted process termination
- `fuser` and `lsof` for relating files and ports to processes
- `tail -f` for observing actively changing files
- `find` for locating filesystem objects
- `grep` with `find` for searching file contents
- `/proc` as a source of process and kernel-related runtime information
- regular-expression anchors such as `^` for matching the start of a line
- named-pipe / FIFO troubleshooting

Reusable pattern:

```text
file or resource symptom
        ↓
identify the process using it
        ↓
inspect process state
        ↓
make a targeted change
        ↓
verify the symptom stopped
```

## System health and resource investigation

Practiced:

- `lscpu` for CPU information
- `uptime` for load averages
- `top` / `htop` for live process and utilization review
- `free -m` and `vmstat` for memory state
- `dmesg` and `journalctl` for historical system events
- identifying evidence of an out-of-memory event
- `df -h` for filesystem utilization
- `lsblk` for block-device layout
- cgroup-related troubleshooting

Important distinction:

```text
current system state
        ≠
complete failure history
```

Logs can show a failure that is no longer visible in the current resource snapshot.

## Networking and service discovery

Practiced:

- `ss -tlpn` and `netstat -tlpn` for listener inspection
- associating ports with services and processes
- interpreting common bind addresses
- distinguishing TCP `LISTEN` from UDP output such as `UNCONN`
- port investigation when standard networking utilities are unavailable
- troubleshooting port conflicts
- tracing traffic from a front-end service to a local backend

Bind-address model:

```text
127.0.0.1
→ loopback / local host only

0.0.0.0
→ all IPv4 interfaces

:::
→ IPv6 unspecified / all IPv6 interfaces
```

## Services, scheduling, and automation

Practiced:

- systemd service investigation
- systemd timer troubleshooting
- scheduled backup troubleshooting
- automated cleanup / maintenance troubleshooting
- verifying whether a service or scheduled task actually performed the intended work

Reusable pattern:

```text
expected automated action did not occur
        ↓
identify scheduler / service
        ↓
inspect configuration
        ↓
inspect logs / execution evidence
        ↓
correct the failure
        ↓
verify the next execution path
```

## Web, security, and application services

Practiced:

- SSL certificate renewal
- Nginx site configuration
- Debian/Ubuntu `sites-available` and `sites-enabled` relationship
- Nginx `proxy_pass`
- reverse-proxy/backend relationships
- FTP synchronization troubleshooting
- database write troubleshooting

Current web-service model:

```text
client
  ↓
front-end / reverse proxy
  ↓
local backend service
  ↓
application dependencies
```

## Developing areas

These are areas that have been introduced or practiced but still need more repetition before treating them as strong skills:

- deeper network diagnosis
- disk I/O performance analysis with tools such as `iostat` and `sar`
- systemd internals and service dependency troubleshooting
- Linux permissions and ownership beyond basic use
- DNS troubleshooting
- firewalling
- SSH troubleshooting

This page will be updated as those areas become more established through additional hands-on work.
