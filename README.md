# Linux Troubleshooting Practice

Hands-on Linux troubleshooting notes focused on **why a command was chosen, what its output revealed, and what to check next**.

The work comes from personal training and lab exercises, not professional Linux administration.

## What I have worked on

| Area | Hands-on work |
|---|---|
| [File and text investigation](01-file-and-text-investigation.md) | `find`, `grep`, `awk`, `sort`, `uniq`, log analysis, arithmetic, CSV work, and searching `/proc` |
| [Processes and system state](02-process-and-system-state.md) | process/file relationships, `ps`, `lsof`, `fuser`, CPU/load, memory/OOM history, storage, and basic IPC/cgroup exposure |
| [Services and automation](03-services-and-automation.md) | service inspection, systemd timers, scheduled backups, cleanup jobs, and following a scheduled task through to its result |
| [Networking and application paths](04-networking-and-application-paths.md) | listeners, bind addresses, ports, Nginx/backend tracing, TLS, FTP, and database troubleshooting exposure |
| [Whole-system health review](applied-investigations/system-health-review.md) | processes, listeners, CPU/load, memory history, storage, and service relationships on an unfamiliar server |

### Package management foundation

I also practiced the normal APT workflow with `apt update`, `apt upgrade`, `apt install`, `apt remove`, `apt search`, and `apt show`, along with the practical distinction between the user-oriented `apt` command and the older/script-friendly `apt-get` interface.

## Strongest investigation

The [whole-system health review](applied-investigations/system-health-review.md) is the most complete example in the repository. It combines process inspection, listener analysis, hardware context, CPU/load, current memory state, historical OOM evidence, and storage capacity to answer two separate questions:

1. What is this server doing?
2. Is the server healthy?

That investigation also reinforced that a high-utilization process or an open port is evidence to interpret, not automatically the root cause.

## How the notes are written

The useful pattern is kept simple:

```text
question or symptom
      ↓
command chosen for that question
      ↓
relevant output
      ↓
interpretation
      ↓
next check or verification
```

Detailed command trails are included only when the original commands and observations were retained accurately. Other completed areas are described at the level the evidence supports.