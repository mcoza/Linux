# Linux System Troubleshooting

An ongoing hands-on Linux portfolio focused on system investigation, troubleshooting, and understanding how Linux components interact.

This repository is intentionally **cumulative**. Smaller technical exercises build reusable skills, while larger multi-component investigations are documented as case studies. The focus is on the troubleshooting process and what was learned, not on the training platform or individual challenge names.

## Current coverage

### Command line and data processing

- searching and filtering text with tools such as `grep`
- extracting and processing fields with `awk`
- counting and aggregating log data with `sort` and `uniq`
- shell-based arithmetic and output formatting
- CSV manipulation and merging
- filesystem and `/proc` investigation

### Processes, files, and IPC

- identifying processes interacting with files
- inspecting and terminating processes
- tracing file/process relationships with `ps`, `fuser`, and `lsof`
- following changing log files
- named-pipe troubleshooting

### System analysis

- CPU and load interpretation
- memory review and historical OOM investigation
- disk-capacity checks
- process and service discovery
- cgroup-related troubleshooting
- combining current system state with historical logs

### Networking and services

- TCP/UDP listener inspection
- interpreting `127.0.0.1`, `0.0.0.0`, and `:::` bind addresses
- mapping ports to services and processes
- port auditing with and without standard networking tools
- troubleshooting port conflicts
- tracing reverse-proxy traffic to backend services

### Services and automation

- scheduled task troubleshooting
- systemd timer investigation
- backup and maintenance automation failures
- service configuration and verification

### Security and application services

- SSL certificate renewal
- Nginx configuration and reverse proxying
- FTP synchronization troubleshooting
- database write troubleshooting

## Troubleshooting approach

The goal is not to memorize commands. The recurring workflow is:

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

A second pattern that became important during system review work is separating two questions:

```text
What is this system doing?

from

Is this system healthy?
```

That distinction helps avoid jumping directly from a high CPU, memory, or network value to a conclusion without understanding the role of the system first.

## Case studies

- [Linux Server Investigation](cases/linux-server-review.md) — system purpose, CPU/load, memory/OOM, disk, processes, and listening services
- [Nginx Port Conflict and Backend Routing](cases/nginx-port-conflict.md) — port conflict, service discovery, Nginx configuration, `proxy_pass`, and backend routing

## Skills progress

[skills-progress.md](skills-progress.md) tracks the Linux areas practiced so far without tying the portfolio to individual training scenarios.

It is a concise map of the technical areas that have been worked through and can be expanded as new Linux topics are practiced.

## How this repository grows

When new work is completed:

1. Update the relevant area in the skills-progress file.
2. Add reusable commands, concepts, or troubleshooting patterns to the appropriate section of the repository.
3. Create a case study only when the work connects multiple components or demonstrates a substantial troubleshooting process.

This keeps the repository maintainable as Linux skills expand without turning every short exercise into a separate project page.

## Scope

This is personal lab and training work. It is not production or professional work experience.
