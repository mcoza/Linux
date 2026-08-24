# Linux System Troubleshooting

An ongoing hands-on Linux portfolio focused on **system investigation, troubleshooting, and understanding how Linux components interact**.

This repository is intentionally cumulative. Short exercises build reusable skills; broader investigations become case studies only when multiple Linux components have to be understood together. The goal is not to collect challenge solutions or screenshots of terminal windows. The goal is to document the reasoning behind the commands, the system relationships they expose, and the troubleshooting patterns that transfer to unfamiliar systems.

This is personal lab and training work, not production or professional work experience.

## Current scope

The repository currently covers:

- package management with APT and the practical difference between `apt` and `apt-get`
- command-line text, log, and CSV processing
- process discovery and file/process relationships
- `/proc`, filesystem search, and basic IPC concepts
- CPU, load, memory, OOM, and disk-capacity investigation
- TCP/UDP listeners, bind addresses, ports, and service discovery
- systemd timers and scheduled-task troubleshooting
- cgroup/resource-control troubleshooting exposure
- TLS certificate maintenance
- Nginx site configuration and reverse-proxy/backend tracing
- FTP synchronization troubleshooting exposure
- database write troubleshooting exposure

The public practice record currently contains **20 confirmed completed SadServers scenarios: 19 Easy and 1 Medium**. Only exercises that reached the platform's successful completion state are counted. See [practice-log.md](practice-log.md).

## Troubleshooting model

The recurring workflow is:

```text
Symptom
  ↓
Observe current state
  ↓
Identify the responsible component
  ↓
Inspect process / service / configuration / logs
  ↓
Form a hypothesis
  ↓
Make a targeted change
  ↓
Verify the result
```

A second pattern that became important during server-review work is separating these two questions:

```text
What is this system doing?

from

Is this system healthy?
```

Understanding system purpose first makes resource values, listeners, and logs much easier to interpret correctly.

## Repository structure

```text
Linux/
├── README.md
├── practice-log.md
├── skills-progress.md
├── docs/
│   ├── package-management.md
│   ├── text-data-processing.md
│   ├── processes-files-proc-ipc.md
│   ├── system-health-storage.md
│   ├── networking-services.md
│   ├── services-automation.md
│   └── web-security-app-services.md
└── cases/
    ├── linux-server-review.md
    └── bergen-nginx-port-conflict.md
```

### `docs/`

These pages contain the **reusable concepts and command patterns** accumulated across multiple exercises. They are meant to grow as the same subsystem is encountered again.

| Area | Notes |
|---|---|
| [Package Management](docs/package-management.md) | APT workflow and `apt` vs `apt-get` |
| [Text & Data Processing](docs/text-data-processing.md) | `grep`, `awk`, `sort`, `uniq`, pipelines, arithmetic, CSV practice |
| [Processes, Files, `/proc` & IPC](docs/processes-files-proc-ipc.md) | PIDs, open files, process inspection, filesystem/content search, FIFOs |
| [System Health & Storage](docs/system-health-storage.md) | CPU/load, memory/OOM, disk capacity, current state vs historical evidence |
| [Networking & Services](docs/networking-services.md) | listeners, bind addresses, TCP/UDP states, ports, service relationships |
| [Services & Automation](docs/services-automation.md) | systemd, timers, scheduled tasks, backups, automated maintenance |
| [Web, Security & Application Services](docs/web-security-app-services.md) | TLS, Nginx, reverse proxies, FTP, database troubleshooting exposure |

### `cases/`

Case studies are reserved for investigations where several earlier skills connect together.

- [Linux Server Review](cases/linux-server-review.md) — infer server purpose, inspect CPU/load and memory, find historical OOM evidence, review disk capacity, processes, and listeners.
- [Nginx Port Conflict](cases/nginx-port-conflict.md) — trace an Nginx request path through active site configuration and `proxy_pass` while troubleshooting a port conflict.

### `practice-log.md`

A compact record of completed hands-on scenarios. It proves continued practice without turning every short exercise into a portfolio project.

### `skills-progress.md`

A current-state matrix showing what has been practiced, what is still developing, and what is intentionally **not yet claimed** as a strong skill.

## Evidence style

Most Linux evidence is clearer as text than as screenshots. The normal format is:

```text
problem
  ↓
command
  ↓
relevant output
  ↓
interpretation
  ↓
next action
```

Commands and trimmed output can be reviewed, searched, and copied directly. Screenshots will only be added when the visual layout itself provides useful evidence.

The repository also avoids reconstructing command sequences that are not retained accurately. If a scenario is known to have been completed but the exact troubleshooting path is no longer available, it is recorded in the practice log and referenced only at the concept level.

## How this repository grows

When new Linux work is completed:

1. Add the completed exercise or lab to `practice-log.md` when there is clear completion evidence.
2. Update the relevant `docs/` page with any reusable concept, command pattern, or troubleshooting lesson.
3. Update `skills-progress.md` if an area becomes stronger or a new area is introduced.
4. Add or expand a `cases/` page only when the work meaningfully connects several components or demonstrates a substantial investigation.

This keeps the repository useful as the learning path expands without turning it into a dump of isolated commands or challenge answers.

## Current direction

The strongest next areas for deeper repetition are:

- networking diagnosis beyond basic listener/port inspection
- Linux permissions and ownership
- systemd service/dependency troubleshooting
- DNS and firewall troubleshooting
- disk I/O performance investigation
- SSH troubleshooting

Those areas will move from **developing** to **practiced** only after additional hands-on work supports the claim.
