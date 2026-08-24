# Linux System Troubleshooting

Hands-on Linux troubleshooting practice focused on understanding **processes, services, logs, networking, and system state** rather than memorizing commands.

This is personal lab/training work, not professional Linux administration.

## Snapshot

- **20 confirmed completed SadServers scenarios** — 19 Easy, 1 Medium
- process and open-file investigation
- command-line log/text/CSV processing
- APT package-management basics
- CPU, load, memory, OOM, and disk investigation
- TCP/UDP listeners, ports, and bind addresses
- systemd timer and scheduled-task troubleshooting
- Nginx reverse-proxy/backend tracing
- introductory cgroup, TLS, FTP, and database troubleshooting exposure

See the [practice log](practice-log.md) for the completed scenario list.

## Strongest troubleshooting examples

### [Whole-System Health Review](cases/system-health-review.md)
Reviewed an unfamiliar server by mapping processes and listeners, checking CPU/load and memory, finding historical OOM evidence, reviewing disk capacity, and using those observations to infer the system's role and health.

### [Nginx Port Conflict](cases/nginx-port-conflict.md)
Traced a port/service problem through Nginx site configuration and a local `proxy_pass` backend. The case documents the verified investigation path without reconstructing the final remediation commands from memory.

## Technical notes

| Area | Coverage |
|---|---|
| [Command Line & Data](notes/command-line-and-data.md) | APT, `grep`, `awk`, pipelines, counting, arithmetic, CSV/text work |
| [System Troubleshooting](notes/system-troubleshooting.md) | processes, open files, `/proc`, CPU/load, memory/OOM, storage, timers |
| [Networking & Services](notes/networking-and-services.md) | listeners, bind addresses, ports, service relationships, Nginx, TLS/FTP/database exposure |

## Troubleshooting approach

```text
observe the symptom
      ↓
identify the responsible process/service/resource
      ↓
inspect current state and relevant logs/configuration
      ↓
form a hypothesis
      ↓
make a targeted change
      ↓
verify the result
```

One lesson from the server-review work was to separate **what the system is doing** from **whether it is healthy**. A high resource value or open port only becomes meaningful in the context of the server's role.

## Evidence

Commands and selected output are shown as text when the original details are retained accurately. Completed exercises with incomplete command trails are listed in the practice log rather than reconstructed from memory.

```text
Linux/
├── README.md
├── practice-log.md
├── notes/
│   ├── command-line-and-data.md
│   ├── system-troubleshooting.md
│   └── networking-and-services.md
└── cases/
    ├── system-health-review.md
    └── nginx-port-conflict.md
```